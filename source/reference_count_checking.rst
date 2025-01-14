Reference Count Checking
========================

Reference count checking is enabled by build the perf tool like::

   make -C tools/perf DEBUG=1 EXTRA_CFLAGS="-fsanitize=address -DREFCNT_CHECKING=1"

Breaking that apart:

- ``DEBUG=1`` enables debug compiler flags like ``-g``
- ``-fsanitize=address`` enables the address sanitizer that looks for runtime memory problems and the leak sanitizer to look for memory leaks
- ``-DREFCNT_CHECKING=1`` enables the reference count checker in the source code (automatically set if address or leak sanitizer defines are present)

The way the reference count checker works is to take a struct like:

.. code-block:: cpp

   struct nsinfo {
           pid_t                   pid;
           pid_t                   tgid;
           pid_t                   nstgid;
           bool                    need_setns;
           bool                    in_pidns;
           char                    *mntns_path;
           refcount_t              refcnt;
   };

and to convert it to two structs:

.. code-block:: cpp

   struct original_nsinfo {
           pid_t                   pid;
           pid_t                   tgid;
           pid_t                   nstgid;
           bool                    need_setns;
           bool                    in_pidns;
           char                    *mntns_path;
           refcount_t              refcnt;
   };

   struct nsinfo {
           struct original_nsinfo *orig;
   };

To make the code more readable, this is done within a macro called
**DECLARE_RC_STRUCT** and if REFCNT_CHECKING isn't enable then the two
struct version isn't used.

Once we have two structs the reference count checking works by every
time a **get** is done a malloc is done of the, in this case, **struct
nsinfo** with the original pointer copied. Every time a **put** is done
the pointer is cleared and the memory freed. To access the variables in
the struct then the **orig** pointer must be used.

What does this give us? Well if we do a **get** without a **put** then
leak sanitizer will report the memory leak at the point the get was
done. If we do a double **put** then there is a use after free,
similarly for a use of a "put" struct's variable after it was put.
Basically the intermediate pointer whose lifetime matches a get and put,
gives us something approximating Resource Acquisition Is Initialization
(RAII) in C. This is done without requiring additional tokens be passed
or APIs change, but it does require changes around the reference counted
struct and accessing its variables.

Here is a real example from a failing test::

   $ perf test 27 -vv
    27: Share thread maps                                               :
   --- start ---
   test child forked, pid 465481
   =================================================================
   ==465481==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000050d0 at pc 0x5561bf855050 bp 0x7ffd2e049b50 sp 0x7ffd2e049b48
   READ of size 8 at 0x6020000050d0 thread T0
       #0 0x5561bf85504f in maps__refcnt tools/perf/util/maps.h:94
       #1 0x5561bf855a08 in test__thread_maps_share tests/thread-maps-share.c:80
       #2 0x5561bf80615f in run_test tests/builtin-test.c:238
       #3 0x5561bf806404 in test_and_print tests/builtin-test.c:267
       #4 0x5561bf807233 in __cmd_test tests/builtin-test.c:404
       #5 0x5561bf808702 in cmd_test tests/builtin-test.c:561
       #6 0x5561bf891a6f in run_builtin tools/perf/perf.c:323
       #7 0x5561bf891fe0 in handle_internal_command tools/perf/perf.c:377
       #8 0x5561bf8923a8 in run_argv tools/perf/perf.c:421
       #9 0x5561bf892910 in main tools/perf/perf.c:537

   0x6020000050d0 is located 0 bytes inside of 8-byte region [0x6020000050d0,0x6020000050d8)
   freed by thread T0 here:
       #0 0x7fe6de4b76a8 in __interceptor_free libsanitizer/asan/asan_malloc_linux.cpp:52
       #1 0x5561bf992736 in maps__put util/maps.c:196
       #2 0x5561bf9b4d9e in thread__delete util/thread.c:92
       #3 0x5561bf9b50ff in thread__put util/thread.c:151
       #4 0x5561bf8559f9 in test__thread_maps_share tests/thread-maps-share.c:79
       #5 0x5561bf80615f in run_test tests/builtin-test.c:238
       #6 0x5561bf806404 in test_and_print tests/builtin-test.c:267
       #7 0x5561bf807233 in __cmd_test tests/builtin-test.c:404
       #8 0x5561bf808702 in cmd_test tests/builtin-test.c:561
       #9 0x5561bf891a6f in run_builtin tools/perf/perf.c:323
       #10 0x5561bf891fe0 in handle_internal_command tools/perf/perf.c:377
       #11 0x5561bf8923a8 in run_argv tools/perf/perf.c:421
       #12 0x5561bf892910 in main tools/perf/perf.c:537

   previously allocated by thread T0 here:
       #0 0x7fe6de4b89cf in __interceptor_malloc libsanitizer/asan/asan_malloc_linux.cpp:69
       #1 0x5561bf9924c2 in maps__new util/maps.c:168
       #2 0x5561bf9b4878 in thread__init_maps util/thread.c:27
       #3 0x5561bf97ccdf in ____machine__findnew_thread util/machine.c:658
       #4 0x5561bf97ce02 in __machine__findnew_thread util/machine.c:677
       #5 0x5561bf97ce72 in machine__findnew_thread util/machine.c:687
       #6 0x5561bf85518f in test__thread_maps_share tests/thread-maps-share.c:34
       #7 0x5561bf80615f in run_test tests/builtin-test.c:238
       #8 0x5561bf806404 in test_and_print tests/builtin-test.c:267
       #9 0x5561bf807233 in __cmd_test tests/builtin-test.c:404
       #10 0x5561bf808702 in cmd_test tests/builtin-test.c:561
       #11 0x5561bf891a6f in run_builtin tools/perf/perf.c:323
       #12 0x5561bf891fe0 in handle_internal_command tools/perf/perf.c:377
       #13 0x5561bf8923a8 in run_argv tools/perf/perf.c:421
       #14 0x5561bf892910 in main tools/perf/perf.c:537

   SUMMARY: AddressSanitizer: heap-use-after-free tools/perf/util/maps.h:94 in maps__refcnt
   Shadow bytes around the buggy address:
     0x0c047fff89c0: fa fa 00 00 fa fa 00 fa fa fa 00 00 fa fa 00 fa
     0x0c047fff89d0: fa fa 00 fa fa fa 00 00 fa fa 00 02 fa fa 00 00
     0x0c047fff89e0: fa fa 00 fa fa fa 00 06 fa fa 00 03 fa fa 00 02
     0x0c047fff89f0: fa fa 00 05 fa fa 00 02 fa fa 00 fa fa fa 00 00
     0x0c047fff8a00: fa fa 00 00 fa fa 00 00 fa fa 00 00 fa fa 00 fa
   =>0x0c047fff8a10: fa fa 01 fa fa fa fd fa fa fa[fd]fa fa fa 03 fa
     0x0c047fff8a20: fa fa 00 fa fa fa 03 fa fa fa 00 fa fa fa 03 fa
     0x0c047fff8a30: fa fa 00 fa fa fa 03 fa fa fa 00 fa fa fa fd fd
     0x0c047fff8a40: fa fa 03 fa fa fa 00 fa fa fa fd fd fa fa 00 fa
     0x0c047fff8a50: fa fa 00 fa fa fa fa fa fa fa fa fa fa fa fa fa
     0x0c047fff8a60: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
   Shadow byte legend (one shadow byte represents 8 application bytes):
     Addressable:           00
     Partially addressable: 01 02 03 04 05 06 07 
     Heap left redzone:       fa
     Freed heap region:       fd
     Stack left redzone:      f1
     Stack mid redzone:       f2
     Stack right redzone:     f3
     Stack after return:      f5
     Stack use after scope:   f8
     Global redzone:          f9
     Global init order:       f6
     Poisoned by user:        f7
     Container overflow:      fc
     Array cookie:            ac
     Intra object redzone:    bb
     ASan internal:           fe
     Left alloca redzone:     ca
     Right alloca redzone:    cb
   ==465481==ABORTING
   test child finished with 1
   ---- end ----
   Share thread maps: FAILED!

So what we see in::

   READ of size 8 at 0x6020000050d0 thread T0
       #0 0x5561bf85504f in maps__refcnt tools/perf/util/maps.h:94
       #1 0x5561bf855a08 in test__thread_maps_share tests/thread-maps-share.c:80

is that the test is using a struct after it was put, the use is in the
header file accessor function but the test code thread-maps-share.c line
80 calls this. The put happening on the line before::

   freed by thread T0 here:
       #0 0x7fe6de4b76a8 in __interceptor_free libsanitizer/asan/asan_malloc_linux.cpp:52
       #1 0x5561bf992736 in maps__put util/maps.c:196
       #2 0x5561bf9b4d9e in thread__delete util/thread.c:92
       #3 0x5561bf9b50ff in thread__put util/thread.c:151
       #4 0x5561bf8559f9 in test__thread_maps_share tests/thread-maps-share.c:79

We can even see the location of the get::

   previously allocated by thread T0 here:
       #0 0x7fe6de4b89cf in __interceptor_malloc libsanitizer/asan/asan_malloc_linux.cpp:69
       #1 0x5561bf9924c2 in maps__new util/maps.c:168
       #2 0x5561bf9b4878 in thread__init_maps util/thread.c:27
       #3 0x5561bf97ccdf in ____machine__findnew_thread util/machine.c:658
       #4 0x5561bf97ce02 in __machine__findnew_thread util/machine.c:677
       #5 0x5561bf97ce72 in machine__findnew_thread util/machine.c:687
       #6 0x5561bf85518f in test__thread_maps_share tests/thread-maps-share.c:34

So we've diagnosed a use after put, we now just need to send a patch
making sure that the put happens later than the use.
