---
layout: post
title:  "D[a]emonic threads"
date:   2019-11-21
author: Phil Connell
---

Lacking a solid plan-in-advance for the day, I decided on the (seemingly
masochistic!) plan to look into the odd post-finalization crash I started on
[last time](../21/PhilConnell.html).

Recapping my previous summary:

- The [subinterpreter patch](https://bugs.python.org/issue33608) caused crashes
  during shutdown of multithreaded processes with long-lived daemon threads.

- The crashes were seen intermittently on a handful of the Python
  [Buildbot](https://buildbot.net/) fleet, mostly on FreeBSD and never on
  Linux!


## Victor Rides To The Rescue

So the first obvious question was: had anyone provided a core+symbols, or
decoded backtraces?

Luckily, on one of the linked issues [Victor Stinner had
posted exactly that](https://bugs.python.org/issue36114#msg337090):

```
Thread 2 (LWP 101669):
#0  0x0000000801e0495d in CRYPTO_free () from /usr/local/lib/libcrypto.so.11
#1  0x0000000801de09bc in ?? () from /usr/local/lib/libcrypto.so.11
#2  0x0000000801e005b7 in OPENSSL_cleanup () from /usr/local/lib/libcrypto.so.11
#3  0x0000000800825ab1 in __cxa_finalize () from /lib/libc.so.7
#4  0x00000008007b2791 in exit () from /lib/libc.so.7
#5  0x00000008004ca84e in Py_Exit (sts=0) at Python/pylifecycle.c:2166
#6  0x00000008004d6ffb in handle_system_exit () at Python/pythonrun.c:641
#7  0x00000008004d6b07 in PyErr_PrintEx (set_sys_last_vars=1) at Python/pythonrun.c:651
#8  0x00000008004d698e in PyErr_Print () at Python/pythonrun.c:547
#9  PyRun_SimpleStringFlags (command=0x80150de38 "from multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=4, pipe_handle=9)\n", flags=0x7fffffffe860) at Python/pythonrun.c:462
#10 0x00000008004f7a39 in pymain_run_command (command=<optimized out>, cf=0x0) at Modules/main.c:527
#11 pymain_run_python (interp=<optimized out>, exitcode=<optimized out>) at Modules/main.c:804
#12 pymain_main (args=<optimized out>) at Modules/main.c:896
#13 0x00000008004f84f7 in _Py_UnixMain (argc=<optimized out>, argv=0x801bdb848) at Modules/main.c:937
#14 0x0000000000201120 in _start (ap=<optimized out>, cleanup=<optimized out>) at /usr/src/lib/csu/amd64/crt1.c:76

Thread 1 (LWP 100922):
#0  0x000000080047ca84 in take_gil (tstate=0x80262ba10) at Python/ceval_gil.h:216
#1  0x000000080047d074 in PyEval_RestoreThread (tstate=0x80262ba10) at Python/ceval.c:281
#2  0x00000008004f3325 in _Py_write_impl (fd=5, buf=0x8025e3e00, count=29, gil_held=1) at Python/fileutils.c:1558
#3  0x00000008005051aa in os_write_impl (module=<optimized out>, fd=<optimized out>, data=<optimized out>) at ./Modules/posixmodule.c:8798
#4  os_write (module=<optimized out>, args=0x8015b3ba8, nargs=<optimized out>) at ./Modules/clinic/posixmodule.c.h:4182
#5  0x0000000800385a5d in _PyMethodDef_RawFastCallKeywords (method=<optimized out>, self=0x80127a5f0, args=<optimized out>, nargs=2, kwnames=<optimized out>) at Objects/call.c:653
#6  0x00000008003847de in _PyCFunction_FastCallKeywords (func=0x8012820c0, args=0x0, nargs=0, kwnames=0xdbdbdbdbdbdbdbdb) at Objects/call.c:732
#7  0x000000080048e1f5 in call_function (pp_stack=0x7fffdf7faf98, oparg=<optimized out>, kwnames=0x0) at Python/ceval.c:4673
#8  0x00000008004892fa in _PyEval_EvalFrameDefault (f=0x8015b3a00, throwflag=<optimized out>) at Python/ceval.c:3294
#9  0x000000080048f03a in PyEval_EvalFrameEx (f=<optimized out>, throwflag=<error reading variable: Cannot access memory at address 0x0>) at Python/ceval.c:624
#10 _PyEval_EvalCodeWithName (_co=<optimized out>, globals=<optimized out>, locals=<optimized out>, args=<optimized out>, argcount=2, kwnames=0x0, kwargs=0x8025e25f8, kwcount=0, kwstep=1, defs=0x802567798,
    defcount=1, kwdefs=0x0, closure=0x0, name=0x8017e2930, qualname=0x80255b7c0) at Python/ceval.c:4035
#11 0x0000000800384690 in _PyFunction_FastCallKeywords (func=<optimized out>, stack=<optimized out>, nargs=0, kwnames=<optimized out>) at Objects/call.c:435
#12 0x000000080048e35f in call_function (pp_stack=0x7fffdf7fb2f0, oparg=<optimized out>, kwnames=0x0) at Python/ceval.c:4721
```

Thread 2 is the main thread, and has just finalized the interpreter and called
`exit()`:

```c
void _Py_NO_RETURN
Py_Exit(int sts)
{
    if (Py_FinalizeEx() < 0) {
        sts = 120;
    }

    exit(sts);  /* <-- frame 5 of thread 2 is here */
}
```

Thread 1 is the villain of our story -- it has just finished a blocking
`write()` call and so decided to reacquire the GIL:

```c
_Py_write_impl(int fd, const void *buf, size_t count, int gil_held)
{
    ...
            Py_BEGIN_ALLOW_THREADS
            errno = 0;
#ifdef MS_WINDOWS
            n = write(fd, buf, (int)count);
#else
            n = write(fd, buf, count);
#endif
            /* save/restore errno because PyErr_CheckSignals()
             * and PyErr_SetFromErrno() can modify it */
            err = errno;
            Py_END_ALLOW_THREADS    /* <-- frame 2 of thread 1 is here */
```

However, we've just finalized the interpreter in Thread 2, cleaning up all of
Python's runtime state. So, unsurprisingly, Thread 1's attempt to access its
threadstate object doesn't end well:

```
(gdb) p tstate
$2 = (PyThreadState *) 0x8027e2050
(gdb) p *tstate
$3 = {
  prev = 0xdbdbdbdbdbdbdbdb,
  next = 0xdbdbdbdbdbdbdbdb,
  interp = 0xdbdbdbdbdbdbdbdb,
  ...
}

(gdb) p _PyRuntime.gilstate.tstate_current
$4 = {
  _value = 0
}
```


## Reproducing The Crash

Interestingly, there's nothing about the sequence of events above that has
anything do to with Eric's subinterpreter changes. So this looks a lot like a
baseline race condition that's simply made (more) likely. Can we reproduce it?

Some hacking about later (and after **much** relearning of how to embed
Python...) I eventually came up with some crashing code (hurrah :)):

```c
#include <assert.h>
#include <pthread.h>
#include <time.h>

#include <Python.h>


void *
evil_main (void *_)
{
    PyGILState_STATE gilstate;

    gilstate = PyGILState_Ensure();
    printf("OTHER: acquired GIL\n");

    Py_BEGIN_ALLOW_THREADS
    printf("OTHER: released GIL\n");
    sleep(2);
    printf("OTHER: attempt to acquire GIL...crash!\n");
    Py_END_ALLOW_THREADS

    /* Dead code */
    assert(0);
    PyGILState_Release(gilstate);
    return (NULL);
}


int
main (int argc, char **argv)
{
    int                 rc;
    pthread_t           evil;

    /* Suppress warnings from get_path.c */
    Py_FrozenFlag = 1;

    /* Don't try to import site */
    Py_NoSiteFlag = 1;

    Py_SetPath(L"./Lib");
    Py_Initialize();
    PyEval_InitThreads();

    rc = pthread_create(&evil, NULL, evil_main, NULL);
    assert(rc == 0);

    Py_BEGIN_ALLOW_THREADS
    printf("MAIN: allow other thread to execute\n");
    sleep(1);
    Py_END_ALLOW_THREADS

    Py_Finalize();
    printf("MAIN: interpreter finalized\n");

    sleep(10);
    /* Dead code */
    assert(0);
    return (0);
}
```

The intention is to force the same kind of race condition:

- Spawn a thread and allow it to acquire the GIL.
- Drop the GIL in the thread and execute a blocking call.
- Meanwhile, finalize Python in the main thread.
- When the other thread wakes and tries to reacquire the GIL, it crashes!

This is nicely illustrated by valgrind:

```
$ valgrind --suppressions=Misc/valgrind-python.supp fini_crash
==266836== Memcheck, a memory error detector
==266836== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==266836== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==266836== Command: fini_crash
==266836==
MAIN: allow other thread to execute
OTHER: acquired GIL
OTHER: released GIL
MAIN: interpreter finalized
OTHER: attempt to acquire GIL...crash!
==266836== Thread 2:
==266836== Invalid read of size 8
==266836==    at 0x15607D: PyEval_RestoreThread (ceval.c:389)
==266836==    by 0x15479F: evil_main (in /home/phconnel/dev/cpython/fini_crash)
==266836==    by 0x48B94CE: start_thread (in /usr/lib/libpthread-2.30.so)
==266836==    by 0x4B232D2: clone (in /usr/lib/libc-2.30.so)
==266836==  Address 0x4d17270 is 16 bytes inside a block of size 264 free'd
==266836==    at 0x48399AB: free (vg_replace_malloc.c:540)
==266836==    by 0x1773FF: tstate_delete_common (pystate.c:829)
==266836==    by 0x1773FF: _PyThreadState_Delete (pystate.c:848)
==266836==    by 0x1773FF: zapthreads (pystate.c:311)
==266836==    by 0x1773FF: PyInterpreterState_Delete (pystate.c:321)
==266836==    by 0x174920: finalize_interp_delete (pylifecycle.c:1242)
==266836==    by 0x174920: Py_FinalizeEx.part.0 (pylifecycle.c:1400)
==266836==    by 0x15487B: main (in /home/phconnel/dev/cpython/fini_crash)
==266836==  Block was alloc'd at
==266836==    at 0x483877F: malloc (vg_replace_malloc.c:309)
==266836==    by 0x178D7C: new_threadstate (pystate.c:557)
==266836==    by 0x178D7C: PyThreadState_New (pystate.c:629)
==266836==    by 0x178D7C: PyGILState_Ensure (pystate.c:1288)
==266836==    by 0x154759: evil_main (in /home/phconnel/dev/cpython/fini_crash)
==266836==    by 0x48B94CE: start_thread (in /usr/lib/libpthread-2.30.so)
==266836==    by 0x4B232D2: clone (in /usr/lib/libc-2.30.so)
```

This looks exactly like the crash reported by Victor but reproduced on an
unmodified `master` checkout of CPython!

## Can We Fix It?

In the case where the non-main thread is using `Py_BEGIN_ALLOW_THREADS` and
`Py_END_ALLOW_THREADS` the problem is that the threadstate object is stored in
a pointer by the `BEGIN` macro but then quietly invalidated before the `END`
macro attempts to use it.

So realistically the only way to fix this (without somehow banning this
behaviour in general) is by adding a layer of indirection. My patch on [issue
33608](https://bugs.python.org/issue33608) does this, wrapping the threadstate
pointer in a structure with references in both directions between the wrapper
and threadstate:

```c
// The PyThreadState typedef is in Include/pystate.h.
struct _ts {
    ...
    /* (Optional) wrapper that will survive this thread state being freed, used
     * by daemon threads that wake up after interpreter finalization.
     */
    struct _ts_wrapper *wrapper;
    ...
};

// The PyThreadStateWrapper typedef is in Include/pystate.h.
struct _ts_wrapper {
    PyThreadState *ts;
};
```

This gives a way to invalidate the state held by the non-main thread: set the
`ts` pointer to `NULL` in the wrapper if the Python runtime is being finalized.

## Open Questions

There are a few of these! I'm particularly interested in what the others on
[the issue](https://bugs.python.org/issue33608) think of this direction:

1. Is this an acceptable change in general? (For what it's worth, I don't see
   an obvious alternative!)
2. What's needed to productize the patch? There are a couple of standout things
   for me:
    - There's one race condition that still needs to be fixed (what happens if
      finalization is triggered while the non-main thread is attempting to
      acquire the GIL?)
    - What's the best approach for adding tests for this case that can be
      maintained long-term?
3. To what extent does this actually fix the problem exposed by Eric's pending
   calls changes? It certainly matches Victor's particular backtrace (and I'm
   fairly confident entirely addresses that instance) but are there other
   cases?

