Perf tools support for Intel® Processor Trace
=============================================

Introduction
------------

What is Intel® Processor Trace
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Intel processors (Broadwell or newer, or Apollo Lake or newer) have a
performance analysis and debugging feature called Intel® Processor
Trace, or Intel® PT for short. Intel PT essentially provides **control
flow tracing**, and you can get all the technical details in the Intel
Processor Trace chapter in the Intel SDM (http://www.intel.com/sdm).

**Control flow tracing** is different from other kinds of performance
analysis and debugging. It provides fine-grained information on branches
taken in a program, but that means there can be a vast amount of trace
data. Such an enormous amount of trace data creates a number of
challenges, but it raises the central question: how to reduce the amount
of trace data that needs to be captured. That inverts the way
performance analysis is normally done. Instead of taking a test case and
creating a trace of it, you need first to create a test case that is
suitable for tracing.

Reducing and handling the massive amount of trace data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Intel PT can potentially produce hundreds of megabytes of trace data per
CPU per second. That can be at a faster rate than it can be recorded to
file (resulting in trace data loss), and sometimes faster even than can
be recording to memory (resulting in overflow packets).

``perf`` tools support output to memory buffers. CPU overhead is low,
but memory bandwidth consumption can be significant. ``perf`` tools do
not support output of Intel PT to **Intel® Trace Hub**.

Here are some ways for reducing and handling the massive amount of trace
data:

Shorten the tracing time
''''''''''''''''''''''''

Whereas statistical sampling can generally handle arbitrarily large test
cases, to reduce the massive amount Intel PT trace data, test cases need
to be created that provide a small representative set of operations to
trace.

Kernel-only tracing
'''''''''''''''''''

Typically, the kernel does not do particularly CPU intensive operations,
making it possible to trace for longer periods. Tracing the kernel-only
can be useful for analyzing latencies.

Snapshots
'''''''''

``perf`` tools support the ability to make snapshots of Intel PT trace.
A snapshot can be made at any time during recording by sending signal
USR2 to ``perf``. The snapshot size is configurable.

Sampling
''''''''

``perf`` tools support adding Intel PT traces (up to 60 KiB per sample)
onto samples of other events. The makes it possible, for example, to get
extended call chains or branch stacks.

Address filtering
'''''''''''''''''

``perf`` tools support specifying Intel PT address filters, refer to the
``--filter`` option of `perf record
<http://www.man7.org/linux/man-pages/man1/perf-record.1.html>`__.

Process only a fraction of the data collected
'''''''''''''''''''''''''''''''''''''''''''''

It is possible to decode only a fraction of a recorded trace by setting
time ranges (``--time`` option of ``perf script`` or ``perf report``) or
specifying CPUs (``--cpu`` option of ``perf script`` or ``perf
report``).

Intel PT in context
^^^^^^^^^^^^^^^^^^^

The following paragraphs provide some context for Intel PT:

Intel PT vs Performance Counters
''''''''''''''''''''''''''''''''

Normal performance analysis is done using performance counters and
performance monitoring events. Counters can be used to provide overall
statistics, which is what the :ref:`perf stat <counting_with_perf_stat>`
tool does, or to provide statistical sampling which is what :ref:`perf
record <sampling_with_perf_record>` / :ref:`perf report
<sample_analysis_with_perf_report>` do.

There are lots and lots of different performance events, not to mention
software events, probes and tracepoints.

By comparison, Intel PT fills a niche. People unfamiliar with normal
performance analysis and debugging, perhaps should not start their
learning with Intel PT.

Intel PT vs Function Profiling
''''''''''''''''''''''''''''''

Function profiling records function entry and exit, but usually requires
programs to be re-compiled for that purpose. It has the advantages of
flexible filtering and sophisticated tools.

Intel PT can be used to provide a call trace without re-compiling a
program, and can trace both user space and kernel space, but with the
challenges of massive trace data described above.

Intel PT vs Last Branch Record (LBR)
''''''''''''''''''''''''''''''''''''

LBR can store branches, filtering different branch types, and providing
finer timing than Intel PT. LBR can provide additional information that
Intel PT does not, such as branch prediction "misses". Intel PT,
however, can record significantly longer branch traces.

Intel PT vs Branch Trace Store (BTS)
''''''''''''''''''''''''''''''''''''

BTS could be considered the predecessor to Intel PT. It records taken
branches and other changes in control flow such as interrupts and
exceptions, but it has much greater overhead than Intel PT, and does not
provide timing information.

Intel PT miscellaneous abilities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Intel PT also has some miscellaneous abilities.

Virtualization
''''''''''''''

Intel PT can trace through VM Entries / Exits, ``perf`` tools have
support for tracing a host and guests. Refer :ref:`Intel PT man page
<intel_pt_man_page>`.

Intel® Transactional Synchronization Extensions (Intel® TSX)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Intel PT traces transactions including aborted transactions. That is
Intel PT will show the instructions in the incomplete transaction and
the subsequent transaction abort.

Power events and C-States
'''''''''''''''''''''''''

Some Intel Atom® processors support reporting C-state changes. All Intel
PT implementations support reporting of CPU frequency changes.

PEBS-via-PT
'''''''''''

Some Intel Atom® processors support recording adaptive PEBS records into
the Intel PT trace.

PTWRITE
'''''''

Hardware that supports it can write directly to the Intel PT trace using
an instruction, 'ptwrite'.

Can I use Intel PT
~~~~~~~~~~~~~~~~~~

Because Intel PT is a hardware feature, you need hardware that supports
it, and also a Linux kernel that has support. Support was added to Linux
in version 4.2 and nowadays, most Linux distributions have a kernel more
recent than that, so the simple way to tell whether you can use Intel PT
is to check whether the directory /sys/devices/intel_pt exists.

.. _intel_pt_man_page:

Intel PT man page
~~~~~~~~~~~~~~~~~

The online `perf Intel PT man page
<http://www.man7.org/linux/man-pages/man1/perf-intel-pt.1.html>`__ is
not necessarily the latest version, however the wiki has a copy which
may be more uptodate: :doc:`perf Intel PT man page
<manpages/perf-intel-pt>`. Also the man page source in
the Linux repository is quite readable: `perf Intel PT man page source
<http://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/perf/Documentation/perf-intel-pt.txt>`__

.. _other_tools:

Other tools
~~~~~~~~~~~

It is not necessary to use ``perf`` to use Intel PT. Here are some other
tools.

GDB
^^^

Get Intel PT branch traces within the GNU Debugger `GDB
<http://sourceware.org/gdb/current/onlinedocs/gdb/Process-Record-and-Replay.html#index-Intel-Processor-Trace>`__

Note GDB needs to be built with libipt (can be checked with "ldd \`which
gdb\` \| grep ipt"), unfortunately many are still not, but there is an
Intel version of GDB in Intel® System Studio for linux.

Fuzzers
^^^^^^^

Some feedback-driven fuzzers (such as `honggfuzz <http://honggfuzz.dev/>`__)
utilize Intel PT.

libipt
^^^^^^

`libipt <http://github.com/intel/libipt>`__ is an Intel® Processor Trace
decoder library that lets you build your own tools.

Intel® VTune™
^^^^^^^^^^^^^

`Intel® VTune™ <http://software.intel.com/en-us/get-started-with-vtune-linux-os>`__
Profiler for Linux.

SATT
^^^^

`SATT Software Analyze Trace Tool <http://github.com/intel/satt>`__ This
tool requires building and installing a custom kernel module

Other resources
~~~~~~~~~~~~~~~

`Cheat sheet for Intel Processor Trace with Linux perf and gdb
<http://halobates.de/blog/p/410>`__

`perf Intel PT man page
<http://www.man7.org/linux/man-pages/man1/perf-intel-pt.1.html>`__,
:doc:`perf Intel PT man page <manpages/perf-intel-pt>`
and `perf Intel PT man page source
<http://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/perf/Documentation/perf-intel-pt.txt>`__

`LWN article: Adding Processor Trace support to Linux
<http://lwn.net/Articles/648154/>`__

.. _getting_set_up:

Getting set up
--------------

``perf`` tools are packaged based on the kernel version, which means the
version of ``perf`` provided by Linux distributions is always quite old,
whereas updates to Intel PT support are happening all the time. That
means, for the latest Intel PT features, we really need to download and
build that latest ``perf``.

For other purposes, ``perf`` on modern Linux is usually fine.

Downloading and building the latest perf tools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The example below is for a Debian-based system, specifically Ubuntu
20.04 in this case, although there is a package list also for Fedora. We
will need 4G of disk space.

``git`` is needed::

   $ sudo apt-get install git

``perf`` is in the Linux source tree so get that::

   $ cd
   $ mkdir git
   $ cd git
   $ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

Get the minimum tools to build it::

   $ sudo apt-get install build-essential flex bison

It should be possible to build ``perf`` at this stage but it will be
missing essential features. Add some more development libraries::

   $ sudo apt-get install libelf-dev libnewt-dev libdw-dev libaudit-dev \
       libiberty-dev libunwind-dev libcap-dev libzstd-dev libnuma-dev \
       libssl-dev python3-dev python3-setuptools binutils-dev \
       gcc-multilib liblzma-dev

Note, with v5.19 and earlier, python3-distutils was used instead of
python3-setuptools, but that was changed in `this commit
<http://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=ee87a0841aa538ab7ad49cf5679ac5ea2682c909>`__.

Aside: example packages for Fedora instead of Ubuntu::

   $ sudo yum install flex bison gcc make elfutils-libelf-devel \
       elfutils-devel libunwind-devel python-devel libcap-devel \
       slang-devel binutils-devel numactl-devel openssl-devel

To build ``perf`` (with script bindings for python3 instead of python2)
and put it in ``~/bin/perf`` ::

   $ cd ~/git/linux
   $ PYTHON=python3 make -C tools/perf install

To use ``~/bin/perf``, ~/bin must be in $PATH. In Ubuntu, that is added
automatically when a user logs in if the ~/bin directory exists (refer
~/.bashrc). If it is not in $PATH, log out and in again. We can echo
$PATH to check::

   $ echo $PATH
   /home/user/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
   $ which perf
   /home/user/bin/perf

Permissions and limits
^^^^^^^^^^^^^^^^^^^^^^

Typically regular userids do not have permission to trace the kernel or
other processes. It is **essential** to understand `Perf Events and tool
security <http://www.kernel.org/doc/html/latest/admin-guide/perf-security.html>`__.

Intel PT benefits from large buffers which is controlled by the
RLIMIT_MEMLOCK limit or the perf_event_mlock_kb setting or the
CAP_IPC_LOCK capability. For kernel tracing with Intel PT, ``perf``
benefits from access to /proc/kcore.

The examples on this page use ``perf`` with extra privileges.

Adding capabilities to ``perf``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To give ``perf`` extra privileges (refer to `Perf Events and tool
security <http://www.kernel.org/doc/html/latest/admin-guide/perf-security.html>`__),
we can add capabilities to the perf executable. Note these capabilities
are not inherited by programs started by perf.

First, we can create a new group and add ourself. This only needs to be
done once::

   $ sudo groupadd perf_users
   $ sudo usermod -a -G perf_users $(whoami)

We will need to logout and login again to pick up the new ``perf_users``
group.

Now we can add capabilities, making ``perf`` executable by only ``root``
and ``perf_users``::

   $ sudo chown root ~/bin/perf
   $ sudo chgrp perf_users ~/bin/perf
   $ sudo chmod 550 ~/bin/perf
   $ sudo setcap "cap_ipc_lock,cap_sys_ptrace,cap_sys_admin,cap_syslog=ep" ~/bin/perf
   $ getcap ~/bin/perf
   ~/bin/perf = cap_ipc_lock,cap_sys_ptrace,cap_sys_admin,cap_syslog+ep

When not using ``perf``, we can remove the capabilities::

   $ sudo setcap -r ~/bin/perf

Updating perf tools
~~~~~~~~~~~~~~~~~~~

To fetch and update the tools again in the future, we can do the
following::

   $ cd ~/git/linux
   $ git pull
   $ rm tools/perf/PERF-VERSION-FILE
   $ make -C tools/perf install

Getting debug packages
~~~~~~~~~~~~~~~~~~~~~~

Debug packages are necessary to map addresses to function names. Ubuntu
provides -dbg and -dbgsym style packages, refer `Debug Symbol Packages -
Ubuntu Wiki <http://wiki.ubuntu.com/Debug%20Symbol%20Packages>`__.

Example: Tracing your own code : Hello World
--------------------------------------------

Before we start, if we haven't done it already, we will need to install
`Intel X86 Encoder Decoder (XED) <http://intelxed.github.io/>`__::

   $ cd ~/git
   $ git clone https://github.com/intelxed/mbuild.git mbuild
   $ git clone https://github.com/intelxed/xed
   $ cd xed
   $ ./mfile.py --share
   $ ./mfile.py examples
   $ sudo ./mfile.py --prefix=/usr/local install
   $ sudo ldconfig
   $ find . -type f -name xed
   ./obj/wkit/examples/obj/xed
   $ cp ./obj/wkit/examples/obj/xed /usr/local/bin

Then, we can start with a trivial "Hello World" program:

.. code-block:: cpp

   $ cat hello.c
   #include <stdio.h>

   int main()
   {
           printf("Hello World!\n");
           return 0;
   }

We can compile it with debugging information::

   $ gcc -g -o hello hello.c

We can use ``perf record`` with options:

- ``-e`` to select which events, i.e. the following:

- ``intel_pt/cyc,noretcomp/u`` to get Intel PT with cycle-accurate
  mode. We can add ``noretcomp`` to get a timing information at RET
  instructions.

- ``--filter 'filter main @ ./hello'`` specifies an address filter
  to trace only main()

- ``./hello`` is the workload.

::

   $ perf record -e intel_pt/cyc,noretcomp/u --filter 'filter main @ ./hello' ./hello
   Hello World!
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.012 MB perf.data ]

We can display an instruction trace with source line information and
source code::

   $ perf script --insn-trace --xed -F+srcline,+srccode
              hello 20444 [003] 28111.955861407:      5635beeae149 main+0x0 (/home/ahunter/git/linux/hello)
     hello.c:4             nop %edi, %edx
   |4        {
              hello 20444 [003] 28111.955861407:      5635beeae14d main+0x4 (/home/ahunter/git/linux/hello)
     hello.c:4             pushq  %rbp
              hello 20444 [003] 28111.955861407:      5635beeae14e main+0x5 (/home/ahunter/git/linux/hello)
     hello.c:4             mov %rsp, %rbp
              hello 20444 [003] 28111.955861407:      5635beeae151 main+0x8 (/home/ahunter/git/linux/hello)
     hello.c:5             leaq  0xeac(%rip), %rdi
   |5              printf("Hello World!\n");
              hello 20444 [003] 28111.955861407:      5635beeae158 main+0xf (/home/ahunter/git/linux/hello)
     hello.c:5             callq  0xfffffffffffffef8
              hello 20444 [003] 28111.955902827:      5635beeae15d main+0x14 (/home/ahunter/git/linux/hello)
     hello.c:6             mov $0x0, %eax
   |6              return 0;
              hello 20444 [003] 28111.955902827:      5635beeae162 main+0x19 (/home/ahunter/git/linux/hello)
     hello.c:7             popq  %rbp
   |7        }
              hello 20444 [003] 28111.955902938:      5635beeae163 main+0x1a (/home/ahunter/git/linux/hello)
     hello.c:7             retq

We can tidy that up a bit with awk::

   $ perf script --insn-trace --xed -F-dso,+srcline,+srccode |\
       awk '/hello / {printf("\n%-85s",$0)} /hello.c:/ {ln=$0;gsub("\t","  ",ln);printf("%-58s",ln)} /^\|/ {printf("%s",$0)}'

              hello 20444 [003] 28111.955861407:      5635beeae149 main+0x0               hello.c:4     nop %edi, %edx                            |4        {
              hello 20444 [003] 28111.955861407:      5635beeae14d main+0x4               hello.c:4     pushq  %rbp
              hello 20444 [003] 28111.955861407:      5635beeae14e main+0x5               hello.c:4     mov %rsp, %rbp
              hello 20444 [003] 28111.955861407:      5635beeae151 main+0x8               hello.c:5     leaq  0xeac(%rip), %rdi                   |5               printf("Hello World!\n");
              hello 20444 [003] 28111.955861407:      5635beeae158 main+0xf               hello.c:5     callq  0xfffffffffffffef8
              hello 20444 [003] 28111.955902827:      5635beeae15d main+0x14              hello.c:6     mov $0x0, %eax                            |6               return 0;
              hello 20444 [003] 28111.955902827:      5635beeae162 main+0x19              hello.c:7     popq  %rbp                                |7        }
              hello 20444 [003] 28111.955902938:      5635beeae163 main+0x1a              hello.c:7     retq

Example: Tracing short-running commands
---------------------------------------

It is possible to trace short-running commands entirely. A very simple
example is a trace of 'ls -l'.

First, we can get some debug symbols e.g.::

   $ find-dbgsym-packages `which ls`
   coreutils-dbgsym libpcre2-8-0-dbgsym libselinux1-dbgsym
   $ sudo apt-get install coreutils-dbgsym libpcre2-8-0-dbgsym \
       libselinux1-dbgsym libc6-dbg

The trace is more interesting if we can drop all file system caches
which will force the test case to do I/O instead of reading from a
cache. For how to use /proc/sys/vm/drop_caches, refer to the
/proc/sys/vm/drop_caches section in the manual page for `/proc
<http://man7.org/linux/man-pages/man5/proc.5.html>`__.

::

   $ sudo bash -c 'echo 3 > /proc/sys/vm/drop_caches'

This example includes kernel tracing, which requires administrator
privileges.

We can use ``perf record`` with options:

- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)

-  ``-e`` to select which events, i.e. the following:

-  ``intel_pt/cyc/`` to get Intel PT with cycle-accurate mode

-  ``ls -l`` is the workload to trace

::

   $ sudo perf record --kcore -e intel_pt/cyc/ ls -l
   total 832
   drwxrwxr-x  27 user user   4096 Jun  2 11:11 arch
   drwxrwxr-x   3 user user   4096 Jun  4 08:40 block
   drwxrwxr-x   2 user user   4096 May 23 15:47 certs
   -rw-rw-r--   1 user user    496 May 23 15:47 COPYING
   -rw-rw-r--   1 user user  99752 Jun  2 11:11 CREDITS
   drwxrwxr-x   4 user user   4096 Jun  2 11:11 crypto
   drwxrwxr-x  79 user user   4096 Jun  4 08:40 Documentation
   drwxrwxr-x 140 user user   4096 May 23 15:47 drivers
   drwxrwxr-x  79 user user   4096 Jun  4 08:40 fs
   drwxrwxr-x  10 user user   4096 Jun  4 08:39 heads
   drwxrwxr-x   3 user user   4096 Jun  4 17:11 hold
   drwxrwxr-x  30 user user   4096 Jun  2 12:37 include
   drwxrwxr-x   2 user user   4096 Jun  4 08:40 init
   drwxrwxr-x   2 user user   4096 Jun  4 08:40 ipc
   -rw-rw-r--   1 user user   1327 May 23 15:47 Kbuild
   -rw-rw-r--   1 user user    595 May 23 15:47 Kconfig
   drwxrwxr-x  18 user user   4096 Jun  4 08:40 kernel
   drwxrwxr-x  20 user user  12288 Jun  4 08:40 lib
   drwxrwxr-x   6 user user   4096 May 23 15:47 LICENSES
   -rw-rw-r--   1 user user 556326 Jun  4 08:40 MAINTAINERS
   -rw-rw-r--   1 user user  61844 Jun  2 11:11 Makefile
   drwxrwxr-x   3 user user   4096 Jun  4 08:40 mm
   drwxrwxr-x  72 user user   4096 Jun  4 08:40 net
   drwx------   3 user user   4096 Jun  4 17:30 perf.data
   -rw-rw-r--   1 user user    727 May 23 15:47 README
   drwxrwxr-x  30 user user   4096 Jun  2 11:11 samples
   drwxrwxr-x  16 user user   4096 Jun  4 08:40 scripts
   drwxrwxr-x  13 user user   4096 Jun  4 08:40 security
   drwxrwxr-x  26 user user   4096 May 23 15:47 sound
   drwxrwxr-x  37 user user   4096 Jun  4 12:11 tools
   drwxrwxr-x   3 user user   4096 May 23 15:47 usr
   drwxrwxr-x   4 user user   4096 May 23 15:47 virt
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 1.041 MB perf.data ]

Rather than look at the trace directly, we will instead create a GUI
call graph. To do that we will use 2 python scripts. The first,
``export-to-sqlite.py`` will export the trace data to a SQLite3
database. The second ``exported-sql-viewer.py`` will create a GUI
call graph.

We can install support for the script ``export-to-sqlite.py``, using
python3 (remember we built perf with python3 support not python2) as
follows::

   sudo apt-get install sqlite3 python3-pyside2.qtsql libqt5sql5-psql

Refer to the script `export-to-sqlite.py
<http://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/perf/scripts/python/export-to-sqlite.py>`__
for more information.

Then we can perform the export::

   $ perf script --itrace=bep \
       -s ~/libexec/perf-core/scripts/python/export-to-sqlite.py \
       ls-example.db branches calls
   2020-06-04 17:30:59.646366 Creating database ...
   2020-06-04 17:30:59.656860 Writing records...
   2020-06-04 17:31:21.215702 Adding indexes
   2020-06-04 17:31:21.408810 Dropping unused tables
   2020-06-04 17:31:21.427048 Done

We can install support for the script ``exported-sql-viewer.py``,
using python3 (remember we built perf with python3 support not python2)
as follows::

   $ sudo apt-get install python3-pyside2.qtcore python3-pyside2.qtgui \
       python3-pyside2.qtsql python3-pyside2.qtwidgets

We can see the call graph, running the script as below, and selecting
'Reports' then 'Context-Sensitive Call Graph'::

   $ python3 ~/libexec/perf-core/scripts/python/exported-sql-viewer.py ls-example.db

We can drill down the trace to see where most of the time is spent. In
this case, waiting on I/O in the lstat system call.

::

   Call Path                                                                         Object         Count   Time (ns)   Time (%)   Insn Cnt   Insn Cnt (%)   Cyc Cnt   Cyc Cnt (%)    IPC   Branch Count   Branch Count (%)
   ▶ perf
   ▼ ls
     ▼ 41672:41672
       ▶ setup_new_exec                                                              [kernel]           1         290        0.0        727            0.0      1230           0.0   0.59             68                0.0
       ▶ native_sched_clock                                                          [kernel]           1           0        0.0          0            0.0         0           0.0      0              1                0.0
       ▶ sched_clock                                                                 [kernel]           1           0        0.0          0            0.0         0           0.0      0              1                0.0
       ▶ sched_clock_cpu                                                             [kernel]           1           0        0.0          0            0.0         0           0.0      0              1                0.0
       ▶ local_clock                                                                 [kernel]           1           0        0.0         48            0.0        49           0.0   0.98              1                0.0
       ▶ __perf_event_header__init_id.isra.0                                         [kernel]           1           1        0.0         21            0.0         5           0.0   4.20              3                0.0
       ▶ perf_event_comm_output                                                      [kernel]           1         123        0.0        506            0.0       517           0.0   0.98             73                0.0
       ▶ perf_iterate_ctx                                                            [kernel]           1          65        0.0        199            0.0       273           0.0   0.73             23                0.0
       ▶ perf_iterate_sb                                                             [kernel]           1           0        0.0          7            0.0         3           0.0   2.33              1                0.0
       ▶ perf_event_comm                                                             [kernel]           1           0        0.0          9            0.0         8           0.0   1.13              1                0.0
       ▶ __set_task_comm                                                             [kernel]           1           0        0.0          7            0.0         2           0.0   3.50              1                0.0
       ▶ load_elf_binary                                                             [kernel]           1      454912        2.3     143405            1.6    255719           2.5   0.56          16318                1.6
       ▶ search_binary_handler                                                       [kernel]           1          11        0.0         28            0.0        46           0.0   0.61              6                0.0
       ▶ __do_execve_file.isra.0                                                     [kernel]           1         554        0.0        357            0.0      2319           0.0   0.15             38                0.0
       ▶ __x64_sys_execve                                                            [kernel]           1           0        0.0          6            0.0        17           0.0   0.35              1                0.0
       ▶ do_syscall_64                                                               [kernel]           1        5635        0.0       3227            0.0     23637           0.2   0.14            438                0.0
       ▶ entry_SYSCALL_64_after_hwframe                                              [kernel]           1         217        0.0         48            0.0       906           0.0   0.05              4                0.0
       ▼ _start                                                                      ld-2.31.so         1    19015524       97.6    8862084           98.4   9861482          97.2   0.90         980661               98.3
         ▶ page_fault                                                                [kernel]           1        3259        0.0       5925            0.1     13626           0.1   0.43            510                0.1
         ▶ _dl_start                                                                 ld-2.31.so         1      956115        5.0     915528           10.3   1665766          16.9   0.55          94430                9.6
         ▶ _dl_init                                                                  ld-2.31.so         1      298497        1.6     283430            3.2    600916           6.1   0.47          28996                3.0
         ▼ _start                                                                    ls                 1    17757387       93.4    7657198           86.4   7580061          76.9   1.01         856721               87.4
           ▶ page_fault                                                              [kernel]           1        2073        0.0       7214            0.1      8685           0.1   0.83            566                0.1
           ▼ __libc_start_main                                                       libc-2.31.so       1    17754971      100.0    7649972           99.9   7569935          99.9   1.01         856153               99.9
             ▶ __cxa_atexit                                                          libc-2.31.so       1        1836        0.0       5944            0.1      7699           0.1   0.77            503                0.1
             ▶ __libc_csu_init                                                       ls                 1        3685        0.0      12103            0.2     15466           0.2   0.78            971                0.1
             ▶ _setjmp                                                               libc-2.31.so       1         176        0.0         32            0.0       797           0.0   0.04              4                0.0
             ▼ main                                                                  ls                 1    17584228       99.0    7263996           95.0   7282964          96.2   1.00         815563               95.3
               ▶ set_program_name                                                    ls                 1        1980        0.0       2590            0.0      8318           0.1   0.31            306                0.0
               ▶ unknown                                                             ls                 1       79537        0.5     219668            3.0    332415           4.6   0.66          24704                3.0
               ▶ unknown                                                             ls                 1         446        0.0        297            0.0      1866           0.0   0.16             35                0.0
               ▶ unknown                                                             ls                 1         199        0.0        449            0.0       831           0.0   0.54             47                0.0
               ▶ atexit                                                              ls                 1          80        0.0          9            0.0       122           0.0   0.07              8                0.0
               ▶ unknown                                                             ls                 1        3420        0.0        829            0.0     14809           0.2   0.06            105                0.0
               ▶ set_quoting_style                                                   ls                 1           0        0.0          0            0.0         0           0.0      0              1                0.0
               ▶ unknown                                                             ls                 7        1023        0.0       3125            0.0      4292           0.1   0.73            501                0.1
               ▶ unknown                                                             ls                 1         512        0.0        488            0.0      2134           0.0   0.23             60                0.0
               ▶ unknown                                                             ls                 2        3091        0.0       6780            0.1     12494           0.2   0.54            642                0.1
               ▶ human_options                                                       ls                 1         445        0.0       1325            0.0      1868           0.0   0.71            217                0.0
               ▶ get_quoting_style                                                   ls                 1           0        0.0          0            0.0         0           0.0      0              1                0.0
               ▶ clone_quoting_options                                               ls                 2         444        0.0        548            0.0      1862           0.0   0.29             58                0.0
               ▶ set_char_quoting                                                    ls                 1           0        0.0          0            0.0         0           0.0      0              1                0.0
               ▶ argmatch                                                            ls                 1         360        0.0        211            0.0      1489           0.0   0.14             28                0.0
               ▶ hard_locale                                                         ls                 1         135        0.0         85            0.0       664           0.0   0.13              8                0.0
               ▶ unknown                                                             ls                 2       14056        0.1      49612            0.7     58942           0.8   0.84           4687                0.6
               ▶ abformat_init                                                       ls                 1       11578        0.1      58235            0.8     48048           0.7   1.21           6651                0.8
               ▶ tzalloc                                                             ls                 1          62        0.0        224            0.0       255           0.0   0.88             19                0.0
               ▶ xmalloc                                                             ls                 1        1642        0.0       2187            0.0      6877           0.1   0.32            214                0.0
               ▶ clear_files                                                         ls                 1         107        0.0         22            0.0       450           0.0   0.05              2                0.0
               ▶ queue_directory                                                     ls                 1         184        0.0        453            0.0       773           0.0   0.59             65                0.0
               ▼ print_dir                                                           ls                 1    17464204       99.3    6916440           95.2   6782904          93.1   1.02         777118               95.3
                 ▶ unknown                                                           ls                 1         121        0.0         39            0.0       509           0.0   0.08              2                0.0
                 ▶ unknown                                                           ls                 1        5298        0.0      12006            0.2     22207           0.3   0.54           1169                0.2
                 ▶ clear_files                                                       ls                 1           0        0.0          0            0.0         0           0.0      0              2                0.0
                 ▶ unknown                                                           ls                43       19841        0.1      54011            0.8     81698           1.2   0.66           5943                0.8
                 ▼ gobble_file.constprop.0                                           ls                32    10586230       60.6    4486659           64.9   4423411          65.2   1.01         496893               63.9
                   ▶ needs_quoting                                                   ls                32        5639        0.1      17998            0.4     23544           0.5   0.76           2421                0.5
                   ▶ unknown                                                         ls                96         732        0.0       2983            0.1      3359           0.1   0.89            290                0.1
                   ▼ unknown                                                         ls                32     8836035       83.5    3477158           77.5   2937626          66.4   1.18         382210               76.9
                     ▼ __lxstat64                                                    libc-2.31.so      32     8835858      100.0    3475998          100.0   2935941          99.9   1.18         382178              100.0
                       ▼ entry_SYSCALL_64                                            [kernel]          32     8828726       99.9    3466363           99.7   2905440          99.0   1.19         380853               99.7
                         ▼ do_syscall_64                                             [kernel]          32     8825966      100.0    3464667          100.0   2894453          99.6   1.20         380757              100.0
                           ▼ __x64_sys_newlstat                                      [kernel]          32     8816599       99.9    3462903           99.9   2866632          99.0   1.21         380458               99.9
                             ▼ __do_sys_newlstat                                     [kernel]          32     8816391      100.0    3460695           99.9   2854386          99.6   1.21         380330              100.0
                               ▼ vfs_statx                                           [kernel]          32     8814939      100.0    3453687           99.8   2848318          99.8   1.21         379722               99.8
                                 ▼ user_path_at_empty                                [kernel]          32     8806551       99.9    3436824           99.5   2816689          98.9   1.22         377734               99.5
                                   ▶ getname_flags                                   [kernel]          32        3471        0.0      13785            0.4     16445           0.6   0.84           1542                0.4
                                   ▼ filename_lookup                                 [kernel]          32     8803067      100.0    3422884           99.6   2800182          99.4   1.22         376096               99.6
                                     ▼ path_lookupat.isra.0                          [kernel]          32     8801121      100.0    3417967           99.9   2792250          99.7   1.22         375552               99.9
                                       ▶ path_init                                   [kernel]          32         603        0.0       3784            0.1      2854           0.1   1.33            192                0.1
                                       ▶ link_path_walk.part.0                       [kernel]          32         708        0.0       5794            0.2      3316           0.1   1.75            422                0.1
                                       ▼ walk_component                              [kernel]          32     8798631      100.0    3404427           99.6   2780753          99.6   1.22         374125               99.6
                                         ▶ lookup_fast                               [kernel]          32        4016        0.0       7158            0.2     16248           0.6   0.44            663                0.2
                                         ▼ lookup_slow                               [kernel]          31     8793533       99.9    3392561           99.7   2760139          99.3   1.23         372807               99.6
                                           ▶ down_read                               [kernel]          31         240        0.0       1009            0.0       499           0.0   2.02            186                0.0
                                           ▼ __lookup_slow                           [kernel]          31     8792986      100.0    3390994          100.0   2758348          99.9   1.23         372466               99.9
                                             ▶ d_alloc_parallel                      [kernel]          31        5966        0.1      13928            0.4     21691           0.8   0.64           1375                0.4
                                             ▼ ext4_lookup                           [kernel]          31     8786755       99.9    3376539           99.6   2736392          99.2   1.23         370998               99.6
                                               ▶ ext4_fname_prepare_lookup           [kernel]          31           0        0.0          0            0.0         0           0.0      0            124                0.0
                                               ▶ __ext4_find_entry                   [kernel]          31       11669        0.1      59063            1.7     49644           1.8   1.19           5687                1.5
                                               ▶ kfree                               [kernel]          62         426        0.0       1888            0.1      2724           0.1   0.69            124                0.0
                                               ▶ __brelse                            [kernel]          31         115        0.0        117            0.0       117           0.0   1.00             31                0.0
                                               ▼ __ext4_iget                         [kernel]          31     8770362       99.8    3301339           97.8   2663781          97.3   1.24         363327               97.9
                                                 ▶ iget_locked                       [kernel]          31       12625        0.1      28525            0.9     54273           2.0   0.53           3083                0.8
                                                 ▼ __ext4_get_inode_loc              [kernel]          31     8744348       99.7    3219494           97.5   2553678          95.9   1.26         353658               97.3
                                                   ▶ ext4_get_group_desc             [kernel]          31         717        0.0       2139            0.1      3006           0.1   0.71             62                0.0
                                                   ▶ ext4_inode_table                [kernel]          46          86        0.0          0            0.0         0           0.0      0             46                0.0
                                                   ▶ __getblk_gfp                    [kernel]          31       23459        0.3      77443            2.4     98450           3.9   0.79           8842                2.5
                                                   ▶ _cond_resched                   [kernel]          30          43        0.0         12            0.0        23           0.0   0.52            120                0.0
                                                   ▶ blk_start_plug                  [kernel]          15           0        0.0          0            0.0         0           0.0      0             30                0.0
                                                   ▶ ext4_itable_unused_count        [kernel]          15         198        0.0          0            0.0         0           0.0      0             15                0.0
                                                   ▶ __breadahead                    [kernel]         474      505000        5.8    2835900           88.1   2111610          82.7   1.34         310764               87.9
                                                   ▶ submit_bh                       [kernel]          15        4421        0.1      27069            0.8     18098           0.7   1.50           2720                0.8
                                                   ▶ blk_finish_plug                 [kernel]          15       42049        0.5     172041            5.3    175937           6.9   0.98          16287                4.6
                                                   ▼ __wait_on_buffer                [kernel]          15     8167641       93.4     102309            3.2    143500           5.6   0.71          13445                3.8
                                                     ▶ _cond_resched                 [kernel]          15         231        0.0        559            0.5      1122           0.8   0.50             60                0.4
                                                     ▼ out_of_line_wait_on_bit       [kernel]          15     8167384      100.0     101705           99.4    142275          99.1   0.71          13340               99.2
                                                       ▼ __wait_on_bit               [kernel]          15     8167361      100.0     101630           99.9    142183          99.9   0.71          13310               99.8
                                                         ▶ prepare_to_wait           [kernel]          15         812        0.0       1211            1.2      2828           2.0   0.43            105                0.8
                                                         ▼ bit_wait_io               [kernel]          15     8166063      100.0      99755           98.2    137321          96.6   0.73          13055               98.1
                                                           ▼ io_schedule             [kernel]          15     8165721      100.0      98826           99.1    135309          98.5   0.73          12980               99.4
                                                             ▶ io_schedule_prepare   [kernel]          15         461        0.0        375            0.4      1944           1.4   0.19             30                0.2
                                                             ▶ schedule              [kernel]          15     8165090      100.0      98301           99.5    132656          98.0   0.74          12905               99.4
                                                         ▶ finish_wait               [kernel]          15           0        0.0          0            0.0         0           0.0      0             45                0.3
                                                 ▶ crypto_shash_update               [kernel]          62         966        0.0       6204            0.2     12343           0.5   0.50            620                0.2
                                                 ▶ ext4_inode_csum.isra.0            [kernel]          31        3712        0.0      19437            0.6     15561           0.6   1.25           2697                0.7
                                                 ▶ make_kuid                         [kernel]          31         467        0.0          0            0.0         0           0.0      0            186                0.1
                                                 ▶ make_kgid                         [kernel]          31         186        0.0       4470            0.1      2661           0.1   1.68            186                0.1
                                                 ▶ make_kprojid                      [kernel]          31         306        0.0       1343            0.0      1313           0.0   1.02            186                0.1
                                                 ▶ set_nlink                         [kernel]          31         245        0.0       1017            0.0      1117           0.0   0.91             93                0.0
                                                 ▶ ext4_set_inode_flags              [kernel]          31         568        0.0       2213            0.1      2181           0.1   1.01            124                0.0
                                                 ▶ _raw_read_lock                    [kernel]          31         534        0.0         93            0.0       200           0.0   0.47             31                0.0
                                                 ▶ ext4_ext_check_inode              [kernel]          31        2147        0.0       9174            0.3     12353           0.5   0.74            654                0.2
                                                 ▶ __brelse                          [kernel]          31          73        0.0        557            0.0       670           0.0   0.83             31                0.0
                                                 ▶ unlock_new_inode                  [kernel]          31         608        0.0       1339            0.0      1578           0.1   0.85            248                0.1
                                                 ▶ ext4_set_aops                     [kernel]           7          95        0.0        384            0.0       397           0.0   0.97             21                0.0
                                               ▶ d_splice_alias                      [kernel]          31        3328        0.0      10072            0.3     12639           0.5   0.80           1178                0.3
                                           ▶ up_read                                 [kernel]          31         283        0.0          0            0.0         0           0.0      0             31                0.0
                                         ▶ follow_managed                            [kernel]          31           0        0.0          0            0.0         0           0.0      0             31                0.0
                                         ▶ dput                                      [kernel]          31         656        0.0       1890            0.1      2576           0.1   0.73            279                0.1
                                       ▶ complete_walk                               [kernel]          32         312        0.0        432            0.0      1605           0.1   0.27             77                0.0
                                       ▶ terminate_walk                              [kernel]          32         515        0.0       1170            0.0      2130           0.1   0.55            352                0.1
                                     ▶ restore_nameidata                             [kernel]          32         530        0.0        720            0.0      2693           0.1   0.27             96                0.0
                                     ▶ putname                                       [kernel]          32        1107        0.0        277            0.0       713           0.0   0.39            256                0.1
                                 ▶ vfs_getattr                                       [kernel]          32        4552        0.1       7924            0.2     18723           0.7   0.42            853                0.2
                                 ▶ path_put                                          [kernel]          32        3010        0.0       3250            0.1      5836           0.2   0.56           1007                0.3
                               ▶ cp_new_stat                                         [kernel]          32         740        0.0       5834            0.2      2933           0.1   1.99            512                0.1
                           ▶ fpregs_assert_state_consistent                          [kernel]          32         103        0.0        523            0.0       575           0.0   0.91             81                0.0
                           ▶ switch_fpu_return                                       [kernel]          15         422        0.0          0            0.0         0           0.0      0             45                0.0
                       ▶ irq_entries_start                                           [kernel]           1        6062        0.1       9283            0.3     25448           0.9   0.36           1260                0.3
                   ▶ rpl_lgetfilecon                                                 ls                32       45398        0.4     105988            2.4    191418           4.3   0.55          12026                2.4
                   ▶ unknown                                                         ls                32         257        0.0        288            0.0      1077           0.0   0.27             64                0.0
                   ▶ file_has_acl                                                    ls                32       47974        0.5     154946            3.5    199782           4.5   0.78          17666                3.6
                   ▶ human_readable                                                  ls                64        5859        0.1      17717            0.4     24550           0.6   0.72           1224                0.2
                   ▶ gnu_mbswidth                                                    ls                96        2885        0.0      12761            0.3     12586           0.3   1.01           1776                0.4
                   ▶ format_user_width                                               ls                32      511932        4.8     360256            8.0    669309          15.1   0.54          39985                8.0
                   ▶ getgroup                                                        ls                32     1120258       10.6     313835            7.0    321667           7.3   0.98          35905                7.2
                   ▶ umaxtostr                                                       ls                32         251        0.0        926            0.0       612           0.0   1.51             47                0.0
                   ▶ xstrdup                                                         ls                32        2645        0.0       8339            0.2     11308           0.3   0.74            986                0.2
                   ▶ page_fault                                                      [kernel]           2        2942        0.0       3786            0.1     12322           0.3   0.31            388                0.1
                 ▶ process_signals                                                   ls                42         248        0.0        688            0.0      1285           0.0   0.54            126                0.0
                 ▶ unknown                                                           ls                 1        3143        0.0       7469            0.1     13138           0.2   0.57            651                0.1
                 ▶ sort_files                                                        ls                 1        9564        0.1      47816            0.7     40449           0.6   1.18           4631                0.6
                 ▶ unknown                                                           ls                 1     4465654       25.6    1375195           19.9   1259338          18.6   1.09         155028               19.9
                 ▶ unknown                                                           ls                 2        4339        0.0       1922            0.0      6816           0.1   0.28            182                0.0
                 ▶ unknown                                                           ls                 2         122        0.0         61            0.0       290           0.0   0.21              6                0.0
                 ▶ unknown                                                           ls                 2       11181        0.1       4541            0.1     17820           0.3   0.25            561                0.1
                 ▶ human_readable                                                    ls                 1         641        0.0        258            0.0      1040           0.0   0.25             20                0.0
                 ▶ print_current_files                                               ls                 1     2356845       13.5     925664           13.4    913979          13.5   1.01         111603               14.4
               ▶ unknown                                                             ls                 3         383        0.0        167            0.0       610           0.0   0.27             15                0.0
             ▶ exit                                                                  libc-2.31.so       1      164956        0.9     367844            4.8    262690           3.5   1.40          39106                4.6

Example: Tracing power events and CPU frequency
-----------------------------------------------

Intel PT can record changes in CPU frequency.

This example includes kernel tracing, which requires administrator
privileges.

To trace power events, we can use ``perf record`` with options:

- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``-e`` to select which events, i.e. the following 2:
- ``intel_pt/branch=0/`` to get Intel PT but without control flow
  (branch) information
- ``power:cpu_idle`` to get the Intel CPU Idle driver tracepoint
- ``sleep 1`` is the workload. The tracing will stop when the
  workload finishes, so this is simply a way of tracing for about 1
  second.

Note, although only 2 events have been selected, we could add anything
else we are interested in::

   $ sudo perf record -a -e intel_pt/branch=0/,power:cpu_idle sleep 1
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.894 MB perf.data ]

To list the power events, use ``perf script`` with options:

- ``--itrace=ep`` to show errors (e) and power events (p)
- ``-F-ip`` to prevent showing the address i.e. instruction pointer
  (ip) register
- ``--ns`` to show the timestamp to nanoseconds instead of the
  default microseconds

The output shows the 10-character task command string, PID, CPU,
timestamp, and event::

   $ perf script --itrace=ep -F-ip --ns | head
               perf  4355 [000] 11253.350232603:                cbr:  cbr: 42 freq: 4219 MHz (156%)
            swapper     0 [000] 11253.350253949:     power:cpu_idle: state=6 cpu_id=0
               perf  4355 [001] 11253.350293104:                cbr:  cbr: 42 freq: 4219 MHz (156%)
            swapper     0 [001] 11253.350311546:     power:cpu_idle: state=8 cpu_id=1
               perf  4355 [002] 11253.350350478:                cbr:  cbr: 42 freq: 4219 MHz (156%)
            swapper     0 [002] 11253.350369240:     power:cpu_idle: state=8 cpu_id=2
               perf  4355 [003] 11253.350407645:                cbr:  cbr: 42 freq: 4219 MHz (156%)
            swapper     0 [003] 11253.350424940:     power:cpu_idle: state=6 cpu_id=3
               perf  4355 [004] 11253.350464191:                cbr:  cbr: 42 freq: 4219 MHz (156%)
            swapper     0 [004] 11253.350482739:     power:cpu_idle: state=8 cpu_id=4

To limit the output to a particular CPU, the ``-C`` option can be used e.g. for CPU 1

::

   $ perf script --itrace=ep -F-ip --ns -C 1 | head
               perf  4355 [001] 11253.350293104:                cbr:  cbr: 42 freq: 4219 MHz (156%)
            swapper     0 [001] 11253.350311546:     power:cpu_idle: state=8 cpu_id=1
            swapper     0 [001] 11253.569359111:                cbr:  cbr: 26 freq: 2612 MHz ( 96%)
            swapper     0 [001] 11253.569364879:     power:cpu_idle: state=4294967295 cpu_id=1
            swapper     0 [001] 11253.569424754:     power:cpu_idle: state=8 cpu_id=1
            swapper     0 [001] 11253.644214090:                cbr:  cbr: 23 freq: 2310 MHz ( 85%)
            swapper     0 [001] 11253.644220472:     power:cpu_idle: state=4294967295 cpu_id=1
            konsole  2033 [001] 11253.644436892:                cbr:  cbr: 20 freq: 2009 MHz ( 74%)
            swapper     0 [001] 11253.645046629:     power:cpu_idle: state=2 cpu_id=1
            swapper     0 [001] 11253.645074374:     power:cpu_idle: state=4294967295 cpu_id=1

To see some context, show context switch events (different trace to above)::

   $ perf script --itrace=ep -F-ip --ns -C 1 --show-switch-events | head -30
            swapper     0 [001] 15355.259304318: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid: 17393/17393
               perf 17393 [001] 15355.259305768: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:     0/0
               perf 17393 [001] 15355.259311919:                psb:  psb offs: 0
               perf 17393 [001] 15355.259311919:                cbr:  cbr: 42 freq: 4219 MHz (156%)
               perf 17393 [001] 15355.259322862: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid:    20/20
        migration/1    20 [001] 15355.259323762: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 17393/17393
        migration/1    20 [001] 15355.259330003: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid:     0/0
            swapper     0 [001] 15355.259330401: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:    20/20
            swapper     0 [001] 15355.259333457:     power:cpu_idle: state=8 cpu_id=1
            swapper     0 [001] 15355.349681141:                cbr:  cbr: 23 freq: 2310 MHz ( 85%)
            swapper     0 [001] 15355.349687604:     power:cpu_idle: state=4294967295 cpu_id=1
            swapper     0 [001] 15355.349711470: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid: 15823/15823
            konsole 15823 [001] 15355.349714239: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:     0/0
            konsole 15823 [001] 15355.349816414: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid:     0/0
            swapper     0 [001] 15355.349817827: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 15823/15823
            swapper     0 [001] 15355.349825977:     power:cpu_idle: state=8 cpu_id=1
            swapper     0 [001] 15355.390601064:                cbr:  cbr: 16 freq: 1607 MHz ( 59%)
            swapper     0 [001] 15355.390608944:     power:cpu_idle: state=4294967295 cpu_id=1
            swapper     0 [001] 15355.390649821: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid: 11264/11264
    kworker/1:0-mm_ 11264 [001] 15355.390652189: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:     0/0
    kworker/1:0-mm_ 11264 [001] 15355.390662452: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid:     0/0
            swapper     0 [001] 15355.390663434: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 11264/11264
            swapper     0 [001] 15355.390671955:     power:cpu_idle: state=8 cpu_id=1
            swapper     0 [001] 15355.422505769:     power:cpu_idle: state=4294967295 cpu_id=1
            swapper     0 [001] 15355.422538477: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid:    66/66
         kcompactd0    66 [001] 15355.422541064: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:     0/0
         kcompactd0    66 [001] 15355.422549431: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid:     0/0
            swapper     0 [001] 15355.422550337: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:    66/66
            swapper     0 [001] 15355.422557517:     power:cpu_idle: state=8 cpu_id=1
            swapper     0 [001] 15355.456474916:                cbr:  cbr: 15 freq: 1507 MHz ( 56%)

To see how to create a custom script refer to intel-pt-events.py
(https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/perf/scripts/python/intel-pt-events.py)

To trace with virtual machines::

   $ sudo ~/bin/perf record -a -e intel_pt/branch=0/,power:cpu_idle sleep 1
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 2.764 MB perf.data ]
   $ perf inject -i perf.data --vm-time-correlation=dry-run
   ERROR: Unknown TSC Offset for VMCS 0x24bad0
   VMCS: 0x25da39  TSC Offset 0xffffd5c7e6fbb0e0
   VMCS: 0x24bad0  TSC Offset 0xffffd5c0fd708df0
   VMCS: 0x1fd127  TSC Offset 0xffffd5c7e6fbb0e0
   VMCS: 0x24bb7d  TSC Offset 0xffffd5c0fd708df0
   VMCS: 0x2659c5  TSC Offset 0xffffd5c7e6fbb0e0
   ERROR: Unknown TSC Offset for VMCS 0x25dbc1
   VMCS: 0x25dbc1  TSC Offset 0xffffd5c0fd708df0
   ERROR: Unknown TSC Offset for VMCS 0x213002
   VMCS: 0x213002  TSC Offset 0xffffd5c0fd708df0
   VMCS: 0x1fd374  TSC Offset 0xffffd5c7e6fbb0e0
   $ perf inject -i perf.data --vm-time-correlation="dry-run 0xffffd5c7e6fbb0e0:0x25da39,0x1fd127,0x2659c5,0x1fd374 0xffffd5c0fd708df0:0x24bad0,0x24bb7d,0x25dbc1,0x213002"
   $ perf inject -i perf.data --vm-time-correlation="0xffffd5c7e6fbb0e0:0x25da39,0x1fd127,0x2659c5,0x1fd374 0xffffd5c0fd708df0:0x24bad0,0x24bb7d,0x25dbc1,0x213002"
   The input file would be updated in place, the --force option is required.
   $ perf inject -i perf.data --vm-time-correlation="0xffffd5c7e6fbb0e0:0x25da39,0x1fd127,0x2659c5,0x1fd374 0xffffd5c0fd708df0:0x24bad0,0x24bb7d,0x25dbc1,0x213002" --force
   $ perf script --itrace=ep -F-ip --ns --show-switch-events
   ...
      perf 18011 [004] 17398.037394927:                psb:  psb offs: 0
               perf 18011 [004] 17398.037394927:                cbr:  cbr: 42 freq: 4219 MHz (156%)
               perf 18011 [004] 17398.037408451: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid:    38/38
        migration/4    38 [004] 17398.037409609: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 18011/18011
          CPU 3/KVM 17819 [005] 17398.037417897: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid: 18011/18011
               perf 18011 [005] 17398.037421458: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 17809/17819
        migration/4    38 [004] 17398.037423366: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid:     0/0
            swapper     0 [004] 17398.037423908: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:    38/38
            swapper     0 [004] 17398.037426640:     power:cpu_idle: state=6 cpu_id=4
               perf 18011 [005] 17398.037427626:                psb:  psb offs: 0
               perf 18011 [005] 17398.037427626:                cbr:  cbr: 42 freq: 4219 MHz (156%)
               perf 18011 [005] 17398.037440808: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid:    44/44
        migration/5    44 [005] 17398.037441699: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 18011/18011
        migration/5    44 [005] 17398.037446582: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid: 17809/17819
          CPU 3/KVM 17819 [005] 17398.037448806: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:    44/44
            swapper     0 [006] 17398.037483980: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid: 18011/18011
               perf 18011 [006] 17398.037485458: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:     0/0
               perf 18011 [006] 17398.037491426:                psb:  psb offs: 0
               perf 18011 [006] 17398.037491426:                cbr:  cbr: 42 freq: 4219 MHz (156%)
               perf 18011 [006] 17398.037502683: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid:    50/50
        migration/6    50 [006] 17398.037503350: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid: 18011/18011
        migration/6    50 [006] 17398.037510565: PERF_RECORD_SWITCH_CPU_WIDE OUT          next pid/tid:     0/0
            swapper     0 [006] 17398.037510908: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:    50/50
            swapper     0 [006] 17398.037514870:     power:cpu_idle: state=8 cpu_id=6
          CPU 1/KVM 17817 [000] 17398.037533765:                cbr:  cbr: 41 freq: 4118 MHz (152%)
          CPU 0/KVM 17816 [001] 17398.037533766:                cbr:  cbr: 41 freq: 4118 MHz (152%)
          CPU 3/KVM 17819 [005] 17398.037533767:                cbr:  cbr: 41 freq: 4118 MHz (152%)
            swapper     0 [006] 17398.037533829:                cbr:  cbr: 41 freq: 4118 MHz (152%)
            swapper     0 [007] 17398.037541586: PERF_RECORD_SWITCH_CPU_WIDE OUT preempt  next pid/tid: 18011/18011
               perf 18011 [007] 17398.037544217: PERF_RECORD_SWITCH_CPU_WIDE IN           prev pid/tid:     0/0
               perf 18011 [007] 17398.037550910:                psb:  psb offs: 0
               perf 18011 [007] 17398.037550910:                cbr:  cbr: 41 freq: 4118 MHz (152%)
          CPU 1/KVM 17817 [000] 17398.037572496:                cbr:  cbr: 42 freq: 4219 MHz (156%)
          CPU 0/KVM 17816 [001] 17398.037572496:                cbr:  cbr: 42 freq: 4219 MHz (156%)
          CPU 3/KVM 17819 [005] 17398.037572498:                cbr:  cbr: 42 freq: 4219 MHz (156%)
               perf 18011 [007] 17398.037572508:                cbr:  cbr: 42 freq: 4219 MHz (156%)
          CPU 3/KVM 17819 [005] 17398.037644052:                cbr:  cbr: 41 freq: 4118 MHz (152%)
          CPU 1/KVM 17817 [000] 17398.037644053:                cbr:  cbr: 41 freq: 4118 MHz (152%)
          CPU 0/KVM 17816 [001] 17398.037644054:                cbr:  cbr: 41 freq: 4118 MHz (152%)
               perf 18011 [007] 17398.037644064:                cbr:  cbr: 41 freq: 4118 MHz (152%)
            swapper     0 [002] 17398.037646332:                cbr:  cbr: 41 freq: 4118 MHz (152%)
            swapper     0 [002] 17398.037647762:     power:cpu_idle: state=4294967295 cpu_id=2
   ...

Example: Tracing the NMI handler
--------------------------------

It is straight forward for Intel PT to trace the NMI handler using
snapshot mode.

This example includes kernel tracing, which requires administrator
privileges.

We can use ``perf record`` with options:

- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)
- ``-Se`` to make a snapshot when the workload ends
- ``-e`` to select which events, i.e. the following:
- ``intel_pt/cyc/k`` to get Intel PT with cycle-accurate mode,
  tracing the kernel only
- ``--`` is a separator, indicating that the rest of the options
  belong to the workload

To get NMIs, the workload itself is another ``perf record`` with
options:

- ``-o junk`` to output to a file named 'junk', since we are not
  interested in it, and it needs to be different from the first ``perf
  record``
- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``-e`` to select which events, i.e. the following:
- ``cycles`` to sample based on the CPU cycles counter
- ``--freq 100000`` to specify the sampling frequency (100kHz) which
  ensures that PEBS samples will cause NMIs (to avoid "large" PEBS)
- ``sleep 0.001`` is the workload. The tracing will stop when the
  workload finishes, so this is simply a way of tracing for about 1
  millisecond.

::

   $ sudo perf record -a --kcore -Se -e intel_pt/cyc/k -- \
       perf record -o junk -a -e cycles --freq 100000 sleep 0.001
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.915 MB junk (9 samples) ]
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 11.467 MB perf.data ]

We can see a call trace showing only the NMI handler with
instructions-per-cycle (IPC) infomation::

   $ perf script --call-trace --graph-function nmi -F+ipc,-dso
               perf  7148 [000] 18161.939140991: nmi            IPC: 0.01 (48/2416)
               perf  7148 [000] 18161.939140995:     paranoid_entry                             IPC: 0.19 (4/21)
               perf  7148 [000] 18161.939141087:     do_nmi                                     IPC: 0.18 (70/383)
               perf  7148 [000] 18161.939141183:         printk_nmi_enter
               perf  7148 [000] 18161.939141226:         rcu_nmi_enter
               perf  7148 [000] 18161.939141226:             rcu_dynticks_curr_cpu_in_eqs
               perf  7148 [000] 18161.939141229:         default_do_nmi                         IPC: 0.12 (77/594)
               perf  7148 [000] 18161.939141229:             nmi_handle
               perf  7148 [000] 18161.939141229:                 sched_clock
               perf  7148 [000] 18161.939141229:                     native_sched_clock
               perf  7148 [000] 18161.939141229:                 __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939141229:                     __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939141722:                         sched_clock            IPC: 0.07 (79/991)
               perf  7148 [000] 18161.939141722:                             native_sched_clock
               perf  7148 [000] 18161.939141722:                         __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939141722:                             __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939141728:                                 intel_bts_disable_local                        IPC: 0.05 (65/1093)
               perf  7148 [000] 18161.939141894:                                 __intel_pmu_disable_all                        IPC: 0.01 (8/694)
               perf  7148 [000] 18161.939141894:                                     native_write_msr
               perf  7148 [000] 18161.939141894:                                     intel_pmu_pebs_disable_all
               perf  7148 [000] 18161.939141943:                                 intel_pmu_drain_bts_buffer                     IPC: 0.15 (33/207)
               perf  7148 [000] 18161.939141943:                                 intel_bts_interrupt
               perf  7148 [000] 18161.939142030:                                 native_read_msr                                IPC: 0.17 (63/362)
               perf  7148 [000] 18161.939142030:                                 intel_pmu_lbr_read
               perf  7148 [000] 18161.939142030:                                 native_write_msr
               perf  7148 [000] 18161.939142193:                                 handle_pmi_common
               perf  7148 [000] 18161.939142193:                                     find_first_bit
               perf  7148 [000] 18161.939142220:                                     intel_pmu_save_and_restart
               perf  7148 [000] 18161.939142220:                                         x86_perf_event_update
               perf  7148 [000] 18161.939142230:                                             native_read_pmc
               perf  7148 [000] 18161.939142230:                                         x86_perf_event_set_period
               perf  7148 [000] 18161.939142262:                                             native_write_msr                   IPC: 0.21 (210/969)
               perf  7148 [000] 18161.939142362:                                             perf_event_update_userpage
               perf  7148 [000] 18161.939142362:                                                 calc_timer_values
               perf  7148 [000] 18161.939142362:                                                     sched_clock_cpu
               perf  7148 [000] 18161.939142362:                                                         sched_clock
               perf  7148 [000] 18161.939142362:                                                             native_sched_clock
               perf  7148 [000] 18161.939142376:                                                 __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939142376:                                                     __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939142395:                                                 arch_perf_update_userpage                                      IPC: 0.30 (170/556)
               perf  7148 [000] 18161.939142395:                                                     using_native_sched_clock
               perf  7148 [000] 18161.939142395:                                                     sched_clock_stable
               perf  7148 [000] 18161.939142395:                                                     cyc2ns_read_begin
               perf  7148 [000] 18161.939142403:                                                     cyc2ns_read_end                                            IPC: 2.00 (74/37)
               perf  7148 [000] 18161.939142411:                                     perf_event_overflow                        IPC: 1.56 (50/32)
               perf  7148 [000] 18161.939142411:                                         __perf_event_overflow
               perf  7148 [000] 18161.939142411:                                             __perf_event_account_interrupt
               perf  7148 [000] 18161.939142411:                                                 sched_clock_cpu
               perf  7148 [000] 18161.939142411:                                                     sched_clock
               perf  7148 [000] 18161.939142411:                                                         native_sched_clock
               perf  7148 [000] 18161.939142516:                                             __x86_indirect_thunk_rax           IPC: 0.19 (87/440)
               perf  7148 [000] 18161.939142516:                                                 __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939142599:                                                     perf_prepare_sample        IPC: 0.08 (30/346)
               perf  7148 [000] 18161.939142599:                                                         perf_misc_flags
               perf  7148 [000] 18161.939142936:                                                             __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939142936:                                                                 __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939143202:                                                         __perf_event_header__init_id.isra.0                    IPC: 0.02 (74/2522)
               perf  7148 [000] 18161.939143203:                                                             perf_event_pid_type                                IPC: 3.80 (19/5)
               perf  7148 [000] 18161.939143203:                                                                 __task_pid_nr_ns
               perf  7148 [000] 18161.939143205:                                                             perf_event_pid_type                                IPC: 3.90 (39/10)
               perf  7148 [000] 18161.939143205:                                                                 __task_pid_nr_ns
               perf  7148 [000] 18161.939143210:                                                             __x86_indirect_thunk_rax                           IPC: 3.41 (58/17)
               perf  7148 [000] 18161.939143210:                                                                 __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939143211:                                                                     sched_clock_cpu                            IPC: 2.25 (9/4)
               perf  7148 [000] 18161.939143211:                                                                         sched_clock
               perf  7148 [000] 18161.939143211:                                                                             native_sched_clock
               perf  7148 [000] 18161.939143227:                                                         perf_instruction_pointer                               IPC: 0.97 (66/68)
               perf  7148 [000] 18161.939143227:                                                             __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939143227:                                                                 __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939143359:                                                     perf_output_begin_forward                                  IPC: 0.11 (65/554)
               perf  7148 [000] 18161.939143895:                                                     perf_output_sample                                         IPC: 0.03 (78/2240)
               perf  7148 [000] 18161.939143895:                                                         perf_output_copy
               perf  7148 [000] 18161.939143895:                                                             memcpy
               perf  7148 [000] 18161.939143923:                                                         perf_output_copy
               perf  7148 [000] 18161.939143923:                                                             memcpy
               perf  7148 [000] 18161.939144052:                                                         perf_output_copy
               perf  7148 [000] 18161.939144052:                                                             memcpy
               perf  7148 [000] 18161.939144064:                                                         perf_output_copy                                       IPC: 0.23 (169/708)
               perf  7148 [000] 18161.939144064:                                                             memcpy
               perf  7148 [000] 18161.939144076:                                                         perf_output_copy                                       IPC: 0.98 (49/50)
               perf  7148 [000] 18161.939144076:                                                             memcpy
               perf  7148 [000] 18161.939144174:                                                         perf_output_copy
               perf  7148 [000] 18161.939144174:                                                             memcpy
               perf  7148 [000] 18161.939144207:                                                     perf_output_end                                            IPC: 0.20 (99/489)
               perf  7148 [000] 18161.939144207:                                                         perf_output_put_handle
               perf  7148 [000] 18161.939144245:                                     find_next_bit                              IPC: 0.41 (89/215)
               perf  7148 [000] 18161.939144260:                                 native_read_msr                                IPC: 0.55 (36/65)
               perf  7148 [000] 18161.939144284:                                 __intel_pmu_enable_all.constprop.0             IPC: 0.16 (16/99)
               perf  7148 [000] 18161.939144284:                                     intel_pmu_pebs_enable_all
               perf  7148 [000] 18161.939144284:                                     intel_pmu_lbr_enable_all
               perf  7148 [000] 18161.939144301:                                     native_write_msr                           IPC: 0.51 (37/72)
               perf  7148 [000] 18161.939144301:                                 intel_bts_enable_local
               perf  7148 [000] 18161.939144408:                                 __x86_indirect_thunk_rax                       IPC: 0.06 (31/446)
               perf  7148 [000] 18161.939144408:                                     __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939144420:                                         native_write_msr                       IPC: 0.34 (10/29)
               perf  7148 [000] 18161.939144457:                         sched_clock                                            IPC: 0.17 (31/179)
               perf  7148 [000] 18161.939144457:                             native_sched_clock
               perf  7148 [000] 18161.939144467:                         perf_sample_event_took
               perf  7148 [000] 18161.939144580:                 sched_clock                                                    IPC: 0.10 (53/512)
               perf  7148 [000] 18161.939144580:                     native_sched_clock
               perf  7148 [000] 18161.939144590:                 sched_clock
               perf  7148 [000] 18161.939144590:                     native_sched_clock
               perf  7148 [000] 18161.939144616:                 __x86_indirect_thunk_rax                                       IPC: 0.45 (68/149)
               perf  7148 [000] 18161.939144616:                     __x86_indirect_thunk_rax
               perf  7148 [000] 18161.939144681:                         nmi_cpu_backtrace                                      IPC: 0.04 (11/275)
               perf  7148 [000] 18161.939144954:                 sched_clock                                                    IPC: 0.02 (26/1141)
               perf  7148 [000] 18161.939144954:                     native_sched_clock
               perf  7148 [000] 18161.939144967:         rcu_nmi_exit                           IPC: 1.10 (61/55)
               perf  7148 [000] 18161.939144985:         printk_nmi_exit
        migration/0    12 [000] 18161.939155684: nmi        IPC: 0.06 (54/805)

Note, that was for a v5.4.33 based kernel i.e.::

   $ uname -a
   Linux 5.4.0-33-generic #37-Ubuntu SMP Thu May 21 12:53:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

Example: Tracing \__switch_to()
-------------------------------

It is straight forward for Intel PT to trace \__switch_to(), the kernel
function that switches tasks.

Before we start, if we haven't done it already, we will need to install
`Intel X86 Encoder Decoder (XED) <http://intelxed.github.io/>`__::

   $ cd ~/git
   $ git clone https://github.com/intelxed/mbuild.git mbuild
   $ git clone https://github.com/intelxed/xed
   $ cd xed
   $ ./mfile.py --share
   $ ./mfile.py examples
   $ sudo ./mfile.py --prefix=/usr/local install
   $ sudo ldconfig
   $ find . -type f -name xed
   ./obj/wkit/examples/obj/xed
   $ cp ./obj/wkit/examples/obj/xed /usr/local/bin

This example includes kernel tracing, which requires administrator
privileges.

We can use ``perf record`` with options:

- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)
- ``-e`` to select which events, i.e. the following:
- ``intel_pt/cyc,noretcomp/k`` to get Intel PT with cycle-accurate
  mode. We add ``noretcomp`` to get a Intel PT TIP packet from RET
  instructions, which has the side-effect of also getting a CYC timing
  packet, and consequently enables calculating IPC at that point.
- ``--filter 'filter __switch_to ,filter native_load_tls'``
  specifies address filters to trace only \__switch_to() and
  native_load_tls()
- ``--`` is a separator, indicating that the rest of the options
  belong to the workload
- ``sleep 1`` is the workload. The tracing will stop when the
  workload finishes, so this is simply a way of tracing for about 1
  second.

::

   $ sudo perf record -a --kcore -e intel_pt/cyc,noretcomp/k \
       --filter 'filter __switch_to ,filter native_load_tls' -- sleep 1

We can get an instruction trace with instructions-per-cycle (IPC)
information for CPU 0, as follows::

   $ perf script --insn-trace --xed -C0 -F-dso,-comm,-tid,+flags,+ipc | head -130
   [000]  2336.210811381:                        ffffffff9f02fdd0 __switch_to+0x0      pushq  %rbp
   [000]  2336.210811381:                        ffffffff9f02fdd1 __switch_to+0x1      movq  %gs:0x16bc0, %rax
   [000]  2336.210811381:                        ffffffff9f02fdda __switch_to+0xa      mov %rsp, %rbp
   [000]  2336.210811381:                        ffffffff9f02fddd __switch_to+0xd      pushq  %r15
   [000]  2336.210811381:                        ffffffff9f02fddf __switch_to+0xf      leaq  0x1340(%rsi), %r15
   [000]  2336.210811381:                        ffffffff9f02fde6 __switch_to+0x16         pushq  %r14
   [000]  2336.210811381:                        ffffffff9f02fde8 __switch_to+0x18         leaq  0x1340(%rdi), %r14
   [000]  2336.210811381:                        ffffffff9f02fdef __switch_to+0x1f         pushq  %r13
   [000]  2336.210811381:                        ffffffff9f02fdf1 __switch_to+0x21         mov %rdi, %r13
   [000]  2336.210811381:                        ffffffff9f02fdf4 __switch_to+0x24         pushq  %r12
   [000]  2336.210811381:                        ffffffff9f02fdf6 __switch_to+0x26         mov %rsi, %r12
   [000]  2336.210811381:                        ffffffff9f02fdf9 __switch_to+0x29         pushq  %rbx
   [000]  2336.210811381:                        ffffffff9f02fdfa __switch_to+0x2a         sub $0x10, %rsp
   [000]  2336.210811381:                        ffffffff9f02fdfe __switch_to+0x2e         movq  (%rax), %rdx
   [000]  2336.210811381:                        ffffffff9f02fe01 __switch_to+0x31         movl  %gs:0x60fe0548(%rip), %ebx
   [000]  2336.210811381:                        ffffffff9f02fe08 __switch_to+0x38         and $0x40, %dh
   [000]  2336.210811382:   jcc                  ffffffff9f02fe0b __switch_to+0x3b         jz 0xffffffff9f0300cd    IPC: 2.83 (17/6)
   [000]  2336.210811382:                        ffffffff9f0300cd __switch_to+0x2fd        testb  $0x20, 0x26(%rax)
   [000]  2336.210811382:   jcc                  ffffffff9f0300d1 __switch_to+0x301        jnz 0xffffffff9f02fe11
   [000]  2336.210811382:                        ffffffff9f0300d7 __switch_to+0x307        leaq  0x1400(%rdi), %rcx
   [000]  2336.210811382:                        ffffffff9f0300de __switch_to+0x30e        nopl  %eax, (%rax,%rax,1)
   [000]  2336.210811382:                        ffffffff9f0300e3 __switch_to+0x313        movl  0x181715e(%rip), %r9d
   [000]  2336.210811382:                        ffffffff9f0300ea __switch_to+0x31a        leaq  0x1440(%r13), %rdi
   [000]  2336.210811382:                        ffffffff9f0300f1 __switch_to+0x321        test %r9d, %r9d
   [000]  2336.210811382:   jcc                  ffffffff9f0300f4 __switch_to+0x324        jz 0xffffffff9f030235
   [000]  2336.210811382:                        ffffffff9f0300fa __switch_to+0x32a        mov $0xffffffff, %eax
   [000]  2336.210811382:                        ffffffff9f0300ff __switch_to+0x32f        mov %eax, %edx
   [000]  2336.210811382:                        ffffffff9f030101 __switch_to+0x331        xsaves64 (%rdi)
   [000]  2336.210811382:                        ffffffff9f030105 __switch_to+0x335        xor %eax, %eax
   [000]  2336.210811382:                        ffffffff9f030107 __switch_to+0x337        test %eax, %eax
   [000]  2336.210811382:   jcc                  ffffffff9f030109 __switch_to+0x339        jnz 0xffffffff9f03022e
   [000]  2336.210811382:                        ffffffff9f03010f __switch_to+0x33f        testb  $0xe0, 0x1640(%r13)
   [000]  2336.210811382:   jcc                  ffffffff9f030117 __switch_to+0x347        jnz 0xffffffff9f030207
   [000]  2336.210811382:                        ffffffff9f03011d __switch_to+0x34d        movl  %ebx, 0x1400(%r13)
   [000]  2336.210811382:                        ffffffff9f030124 __switch_to+0x354        nopl  %eax, (%rax,%rax,1)
   [000]  2336.210811382:   jmp                  ffffffff9f030129 __switch_to+0x359        jmp 0xffffffff9f02fe11
   [000]  2336.210811382:                        ffffffff9f02fe11 __switch_to+0x41         mov %fs, %ax
   [000]  2336.210811382:                        ffffffff9f02fe14 __switch_to+0x44         movw  %ax, 0x1364(%r13)
   [000]  2336.210811382:                        ffffffff9f02fe1c __switch_to+0x4c         mov %gs, %ax
   [000]  2336.210811382:                        ffffffff9f02fe1f __switch_to+0x4f         cmpw  $0x0, 0x1364(%r13)
   [000]  2336.210811382:                        ffffffff9f02fe28 __switch_to+0x58         movw  %ax, 0x1366(%r13)
   [000]  2336.210811382:   jcc                  ffffffff9f02fe30 __switch_to+0x60         jnz 0xffffffff9f0301a3
   [000]  2336.210811382:                        ffffffff9f02fe36 __switch_to+0x66         cmpw  $0x0, 0x1366(%r13)
   [000]  2336.210811428:   jcc                  ffffffff9f02fe3f __switch_to+0x6f         jnz 0xffffffff9f030185   IPC: 0.13 (27/195)
   [000]  2336.210811428:                        ffffffff9f02fe45 __switch_to+0x75         mov %ebx, %esi
   [000]  2336.210811428:                        ffffffff9f02fe47 __switch_to+0x77         mov %r15, %rdi
   [000]  2336.210811428:   call                 ffffffff9f02fe4a __switch_to+0x7a         callq  0xffffffff9f0786c0
   [000]  2336.210811428:                        ffffffff9f0786c0 native_load_tls+0x0      movq  (%rdi), %rdx
   [000]  2336.210811428:                        ffffffff9f0786c3 native_load_tls+0x3      mov %esi, %esi
   [000]  2336.210811428:                        ffffffff9f0786c5 native_load_tls+0x5      mov $0x9000, %rax
   [000]  2336.210811428:                        ffffffff9f0786cc native_load_tls+0xc      addq  -0x5fbbb680(,%rsi,8), %rax
   [000]  2336.210811428:                        ffffffff9f0786d4 native_load_tls+0x14         movq  %rdx, 0x60(%rax)
   [000]  2336.210811428:                        ffffffff9f0786d8 native_load_tls+0x18         movq  0x8(%rdi), %rdx
   [000]  2336.210811428:                        ffffffff9f0786dc native_load_tls+0x1c         movq  %rdx, 0x68(%rax)
   [000]  2336.210811428:                        ffffffff9f0786e0 native_load_tls+0x20         movq  0x10(%rdi), %rdx
   [000]  2336.210811428:                        ffffffff9f0786e4 native_load_tls+0x24         movq  %rdx, 0x70(%rax)
   [000]  2336.210811429:   return               ffffffff9f0786e8 native_load_tls+0x28         retq     IPC: 3.25 (13/4)
   [000]  2336.210811429:                        ffffffff9f02fe4f __switch_to+0x7f         data16 nop
   [000]  2336.210811429:                        ffffffff9f02fe51 __switch_to+0x81         mov %r12, %rdi
   [000]  2336.210811429:                        ffffffff9f02fe54 __switch_to+0x84         nopl  %eax, (%rax)
   [000]  2336.210811429:                        ffffffff9f02fe5b __switch_to+0x8b         mov %es, %ax
   [000]  2336.210811429:                        ffffffff9f02fe5e __switch_to+0x8e         movw  %ax, 0x20(%r14)
   [000]  2336.210811429:                        ffffffff9f02fe63 __switch_to+0x93         movzxw  0x1360(%r12), %eax
   [000]  2336.210811429:                        ffffffff9f02fe6c __switch_to+0x9c         mov %eax, %ebx
   [000]  2336.210811429:                        ffffffff9f02fe6e __switch_to+0x9e         orw  0x1360(%r13), %bx
   [000]  2336.210811430:   jcc                  ffffffff9f02fe76 __switch_to+0xa6         jnz 0xffffffff9f030195   IPC: 4.50 (9/2)
   [000]  2336.210811430:                        ffffffff9f02fe7c __switch_to+0xac         mov %ds, %ax
   [000]  2336.210811430:                        ffffffff9f02fe7f __switch_to+0xaf         movw  %ax, 0x22(%r14)
   [000]  2336.210811430:                        ffffffff9f02fe84 __switch_to+0xb4         movzxw  0x1362(%r12), %eax
   [000]  2336.210811430:                        ffffffff9f02fe8d __switch_to+0xbd         mov %eax, %ebx
   [000]  2336.210811430:                        ffffffff9f02fe8f __switch_to+0xbf         orw  0x1362(%r13), %bx
   [000]  2336.210811430:   jcc                  ffffffff9f02fe97 __switch_to+0xc7         jnz 0xffffffff9f03019c
   [000]  2336.210811430:                        ffffffff9f02fe9d __switch_to+0xcd         movq  0x1368(%r12), %rdx
   [000]  2336.210811430:                        ffffffff9f02fea5 __switch_to+0xd5         movzxw  0x1364(%r13), %ecx
   [000]  2336.210811430:                        ffffffff9f02fead __switch_to+0xdd         movzxw  0x1364(%r12), %eax
   [000]  2336.210811430:                        ffffffff9f02feb6 __switch_to+0xe6         cmp $0x3, %ax
   [000]  2336.210811430:   jcc                  ffffffff9f02feba __switch_to+0xea         jnbe 0xffffffff9f03016e
   [000]  2336.210811430:                        ffffffff9f02fec0 __switch_to+0xf0         test %rdx, %rdx
   [000]  2336.210811430:   jcc                  ffffffff9f02fec3 __switch_to+0xf3         jz 0xffffffff9f030085
   [000]  2336.210811430:   jmp                  ffffffff9f030085 __switch_to+0x2b5        jmp 0xffffffff9f03015c
   [000]  2336.210811430:                        ffffffff9f03015c __switch_to+0x38c        or %eax, %ecx
   [000]  2336.210811430:                        ffffffff9f03015e __switch_to+0x38e        movzx %cx, %ecx
   [000]  2336.210811430:                        ffffffff9f030161 __switch_to+0x391        orq  0x1368(%r13), %rcx
   [000]  2336.210811430:   jcc                  ffffffff9f030168 __switch_to+0x398        jz 0xffffffff9f02fee2
   [000]  2336.210811430:                        ffffffff9f03016e __switch_to+0x39e        mov %ax, %fs
   [000]  2336.210811430:   jmp                  ffffffff9f030170 __switch_to+0x3a0        jmp 0xffffffff9f02fee2
   [000]  2336.210811430:                        ffffffff9f02fee2 __switch_to+0x112        movq  0x1370(%r12), %r14
   [000]  2336.210811430:                        ffffffff9f02feea __switch_to+0x11a        movzxw  0x1366(%r13), %eax
   [000]  2336.210811430:                        ffffffff9f02fef2 __switch_to+0x122        movzxw  0x1366(%r12), %ebx
   [000]  2336.210811430:                        ffffffff9f02fefb __switch_to+0x12b        cmp $0x3, %bx
   [000]  2336.210811430:   jcc                  ffffffff9f02feff __switch_to+0x12f        jnbe 0xffffffff9f03014d
   [000]  2336.210811430:                        ffffffff9f02ff05 __switch_to+0x135        test %r14, %r14
   [000]  2336.210811439:   jcc                  ffffffff9f02ff08 __switch_to+0x138        jnz 0xffffffff9f0300a3   IPC: 0.67 (27/40)
   [000]  2336.210811439:   jmp                  ffffffff9f02ff0e __switch_to+0x13e        jmp 0xffffffff9f03013b
   [000]  2336.210811439:                        ffffffff9f03013b __switch_to+0x36b        or %ebx, %eax
   [000]  2336.210811439:                        ffffffff9f03013d __switch_to+0x36d        movzx %ax, %eax
   [000]  2336.210811439:                        ffffffff9f030140 __switch_to+0x370        orq  0x1370(%r13), %rax
   [000]  2336.210811439:   jcc                  ffffffff9f030147 __switch_to+0x377        jz 0xffffffff9f02ff29
   [000]  2336.210811439:                        ffffffff9f02ff29 __switch_to+0x159        movq  %r12, %gs:0x60fe6c8f(%rip)
   [000]  2336.210811439:                        ffffffff9f02ff31 __switch_to+0x161        movq  0x18(%r12), %rax
   [000]  2336.210811439:                        ffffffff9f02ff36 __switch_to+0x166        add $0x4000, %rax
   [000]  2336.210811439:                        ffffffff9f02ff3c __switch_to+0x16c        movq  %rax, %gs:0x60fd60c8(%rip)
   [000]  2336.210811439:                        ffffffff9f02ff44 __switch_to+0x174        movl  0x161efa6(%rip), %ebx
   [000]  2336.210811439:                        ffffffff9f02ff4a __switch_to+0x17a        movq  %gs:0x16bc0, %rax
   [000]  2336.210811439:                        ffffffff9f02ff53 __switch_to+0x183        lock orb  $0x40, 0x1(%rax)
   [000]  2336.210811439:   jmp                  ffffffff9f02ff58 __switch_to+0x188        jmp 0xffffffff9f02ff99
   [000]  2336.210811439:   jmp                  ffffffff9f02ff99 __switch_to+0x1c9        jmp 0xffffffff9f02ffb1
   [000]  2336.210811439:                        ffffffff9f02ffb1 __switch_to+0x1e1        movq  (%r12), %rdx
   [000]  2336.210811439:                        ffffffff9f02ffb5 __switch_to+0x1e5        movq  (%r13), %rax
   [000]  2336.210811439:                        ffffffff9f02ffb9 __switch_to+0x1e9        nopl  %eax, (%rax,%rax,1)
   [000]  2336.210811439:                        ffffffff9f02ffbe __switch_to+0x1ee        and $0x2418620, %edx
   [000]  2336.210811439:                        ffffffff9f02ffc4 __switch_to+0x1f4        and $0x2418e20, %eax
   [000]  2336.210811439:                        ffffffff9f02ffc9 __switch_to+0x1f9        or %rax, %rdx
   [000]  2336.210811439:   jcc                  ffffffff9f02ffcc __switch_to+0x1fc        jnz 0xffffffff9f030175
   [000]  2336.210811439:   jmp                  ffffffff9f02ffd2 __switch_to+0x202        jmp 0xffffffff9f02ffec
   [000]  2336.210811439:   jmp                  ffffffff9f02ffec __switch_to+0x21c        jmp 0xffffffff9f030001
   [000]  2336.210811439:   jmp                  ffffffff9f030001 __switch_to+0x231        jmp 0xffffffff9f030073
   [000]  2336.210811439:                        ffffffff9f030073 __switch_to+0x2a3        add $0x10, %rsp
   [000]  2336.210811439:                        ffffffff9f030077 __switch_to+0x2a7        mov %r13, %rax
   [000]  2336.210811439:                        ffffffff9f03007a __switch_to+0x2aa        popq  %rbx
   [000]  2336.210811439:                        ffffffff9f03007b __switch_to+0x2ab        popq  %r12
   [000]  2336.210811439:                        ffffffff9f03007d __switch_to+0x2ad        popq  %r13
   [000]  2336.210811439:                        ffffffff9f03007f __switch_to+0x2af        popq  %r14
   [000]  2336.210811439:                        ffffffff9f030081 __switch_to+0x2b1        popq  %r15
   [000]  2336.210811439:                        ffffffff9f030083 __switch_to+0x2b3        popq  %rbp
   [000]  2336.210811468:   tr end  return       ffffffff9f030084 __switch_to+0x2b4        retq     IPC: 0.27 (33/122)
   [000]  2336.210817292:                        ffffffff9f02fdd0 __switch_to+0x0      pushq  %rbp
   [000]  2336.210817292:                        ffffffff9f02fdd1 __switch_to+0x1      movq  %gs:0x16bc0, %rax
   [000]  2336.210817292:                        ffffffff9f02fdda __switch_to+0xa      mov %rsp, %rbp
   [000]  2336.210817292:                        ffffffff9f02fddd __switch_to+0xd      pushq  %r15

Note, that was for a v5.4.33 based kernel i.e.::

   $ uname -a
   Linux 5.4.0-33-generic #37-Ubuntu SMP Thu May 21 12:53:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

We can get source line information, but first we need the kernel debug
package::

   $ sudo apt-get install linux-image-5.4.0-33-generic-dbgsym

We can use it, as follows, to see how much it differs from the running
code::

   $ perf script --itrace=e --vmlinux /usr/lib/debug/boot/vmlinux-5.4.0-33-generic >/dev/null
   Warning:
   55 instruction trace errors

Really, we should be able to use kcore for object code, and
/usr/lib/debug/boot/vmlinux-5.4.0-33-generic for debug information, but
because we can't at present, we can reduce the decoding errors, by
copying from the copy of kcore into a copy of
/usr/lib/debug/boot/vmlinux-5.4.0-33-generic::

   $ cp /usr/lib/debug/boot/vmlinux-5.4.0-33-generic vmlinux-5.4.0-33-generic-plus-kcore
   $ dd if=perf.data/kcore_dir/kcore of=vmlinux-5.4.0-33-generic-plus-kcore bs=4096 skip=1 seek=512 count=3585 conv=nocreat,notrunc
   3585+0 records in
   3585+0 records out
   14684160 bytes (15 MB, 14 MiB) copied, 0.0143739 s, 1.0 GB/s
   $ perf script --itrace=e --vmlinux vmlinux-5.4.0-33-generic-plus-kcore >/dev/null

Note, 2 of the values above come from the readelf command below:

- skip=1 (4096-byte block) because that is the offset of the kernel code
  segment in perf.data/kcore_dir/kcore i.e. 0x1000 = 1 x 4096)
- count=3585 (4096-byte blocks) because that is the size (Filesiz) of
  the kernel code segment in perf.data/kcore_dir/kcore i.e. 0xe01000 =
  3585 x 4096

::

   $ readelf -l switch-example-3/kcore_dir/kcore | grep -C2 LOAD
     Type           Offset             VirtAddr           PhysAddr
                    FileSiz            MemSiz              Flags  Align
     LOAD           0x0000000000001000 0xffffffff9f000000 0x0000000000000000
                    0x0000000000e01000 0x0000000000e01000  RWE    0x1000
     LOAD           0x0000000000e02000 0xffffffffc025c000 0x0000000000000000
                    0x0000000000c99000 0x0000000000c99000  RWE    0x1000

The remaining value comes from the readelf command below:

- seek=512 (4096-byte blocks) because that is the offset of the kernel
  code segment in vmlinux-5.4.0-33-generic-plus-kcore i.e. 0x200000 =
  512 x 4096

::

   $ readelf -l vmlinux-5.4.0-33-generic-plus-kcore | grep -C2 LOAD
     Type           Offset             VirtAddr           PhysAddr
                    FileSiz            MemSiz              Flags  Align
     LOAD           0x0000000000200000 0xffffffff81000000 0x0000000001000000
                    0x00000000014d5000 0x00000000014d5000  R E    0x200000
     LOAD           0x0000000001800000 0xffffffff82600000 0x0000000002600000
                    0x000000000026f000 0x000000000026f000  RW     0x200000
     LOAD           0x0000000001c00000 0x0000000000000000 0x000000000286f000
                    0x000000000002d000 0x000000000002d000  RW     0x200000
     LOAD           0x0000000001c9c000 0xffffffff8289c000 0x000000000289c000
                    0x0000000000d64000 0x0000000000d64000  RWE    0x200000
     NOTE           0x0000000001000eb4 0xffffffff81e00eb4 0x0000000001e00eb4

Then, we can show source line information::

   $ perf script --vmlinux ./vmlinux-5.4.0-33-generic-plus-kcore --insn-trace --xed -C0 -F-dso,-comm,-tid,+flags,+ipc,+srcline | head
   [000]  2336.210811381:                        ffffffff9f02fdd0 __switch_to+0x0
     process_64.c:505              pushq  %rbp
   [000]  2336.210811381:                        ffffffff9f02fdd1 __switch_to+0x1
     current.h:15          movq  %gs:0x16bc0, %rax
   [000]  2336.210811381:                        ffffffff9f02fdda __switch_to+0xa
     process_64.c:505              mov %rsp, %rbp
   [000]  2336.210811381:                        ffffffff9f02fddd __switch_to+0xd
     process_64.c:505              pushq  %r15
   [000]  2336.210811381:                        ffffffff9f02fddf __switch_to+0xf
     process_64.c:507              leaq  0x1340(%rsi), %r15

We can tidy that up a bit with awk::

   $ perf script --vmlinux ./vmlinux-5.4.0-33-generic-plus-kcore \
   >   --insn-trace --xed -C0 -F-dso,-comm,-tid,+flags,+ipc,+srcline | \
   > awk '/^\[/ {printf("\n%-85s",$0)} /^ / {ln=$0;gsub("\t","  ",ln);printf("%-58s",ln)}' | head

   [000]  2336.210811381:                        ffffffff9f02fdd0 __switch_to+0x0         process_64.c:505     pushq  %rbp
   [000]  2336.210811381:                        ffffffff9f02fdd1 __switch_to+0x1         current.h:15     movq  %gs:0x16bc0, %rax
   [000]  2336.210811381:                        ffffffff9f02fdda __switch_to+0xa         process_64.c:505     mov %rsp, %rbp
   [000]  2336.210811381:                        ffffffff9f02fddd __switch_to+0xd         process_64.c:505     pushq  %r15
   [000]  2336.210811381:                        ffffffff9f02fddf __switch_to+0xf         process_64.c:507     leaq  0x1340(%rsi), %r15
   [000]  2336.210811381:                        ffffffff9f02fde6 __switch_to+0x16        process_64.c:505     pushq  %r14
   [000]  2336.210811381:                        ffffffff9f02fde8 __switch_to+0x18        process_64.c:506     leaq  0x1340(%rdi), %r14
   [000]  2336.210811381:                        ffffffff9f02fdef __switch_to+0x1f        process_64.c:505     pushq  %r13
   [000]  2336.210811381:                        ffffffff9f02fdf1 __switch_to+0x21        process_64.c:505     mov %rdi, %r13

We can add the source code as well, but first we need the matching linux
kernel source, taking care to get the right version::

   $ sudo apt-get install linux-source-5.4.0=5.4.0-33.37

We can see, from below, that the source code needs to be in directory
/build/linux-3STWY5::

   $ perf script --vmlinux ./vmlinux-5.4.0-33-generic-plus-kcore --insn-trace --xed -C0 -F-dso,-comm,-tid,+flags,+ipc,+srcline --full-source-path | awk '/^\[/ {printf("\n%-85s",$0)} /^ / {ln=$0;gsub("\t","  ",ln);printf("%-58s",ln)}' | head -4

   [000]  2336.210811381:                        ffffffff9f02fdd0 __switch_to+0x0         /build/linux-3STWY5/linux-5.4.0/arch/x86/kernel/process_64.c:505     pushq  %rbp
   [000]  2336.210811381:                        ffffffff9f02fdd1 __switch_to+0x1         /build/linux-3STWY5/linux-5.4.0/arch/x86/include/asm/current.h:15     movq  %gs:0x16bc0, %rax
   [000]  2336.210811381:                        ffffffff9f02fdda __switch_to+0xa         /build/linux-3STWY5/linux-5.4.0/arch/x86/kernel/process_64.c:505     mov %rsp, %rbp

We can unpackage the source tarball into that directory, rename it, and
remove the package::

   $ sudo mkdir -p /build/linux-3STWY5
   $ sudo tar -xjf  /usr/src/linux-source-5.4.0/linux-source-5.4.0.tar.bz2 --directory=/build/linux-3STWY5
   $ sudo mv /build/linux-3STWY5/linux-source-5.4.0 /build/linux-3STWY5/linux-5.4.0
   $ sudo apt-get remove linux-source-5.4.0

Then, we can show source code::

   $ perf script --vmlinux ./vmlinux-5.4.0-33-generic-plus-kcore --insn-trace --xed -C0 -F-dso,-comm,-tid,+flags,+ipc,+srccode,+srcline | head
   [000]  2336.210811381:                        ffffffff9f02fdd0 __switch_to+0x0
     process_64.c:505              pushq  %rbp
   |505      {
   [000]  2336.210811381:                        ffffffff9f02fdd1 __switch_to+0x1
     current.h:15          movq  %gs:0x16bc0, %rax
   |515            if (!test_thread_flag(TIF_NEED_FPU_LOAD))
   [000]  2336.210811381:                        ffffffff9f02fdda __switch_to+0xa
     process_64.c:505              mov %rsp, %rbp
   |505      {
   [000]  2336.210811381:                        ffffffff9f02fddd __switch_to+0xd

We can tidy that up a bit with awk::

   $ perf script -i switch-example-3 --vmlinux ./vmlinux-5.4.0-33-generic-plus-kcore --insn-trace --xed -C0 -F-dso,-comm,-tid,+flags,+ipc,+srccode,+srcline | \
   > awk '/^\[/ {printf("\n%-85s",$0)} /^ / {ln=$0;gsub("\t","  ",ln);printf("%-58s",ln)} /^\|/ {printf("%s",$0)}' | head -138

   [000]  2336.210811381:                        ffffffff9f02fdd0 __switch_to+0x0         process_64.c:505     pushq  %rbp                        |505      {
   [000]  2336.210811381:                        ffffffff9f02fdd1 __switch_to+0x1         current.h:15     movq  %gs:0x16bc0, %rax                |515             if (!test_thread_flag(TIF_NEED_FPU_LOAD))
   [000]  2336.210811381:                        ffffffff9f02fdda __switch_to+0xa         process_64.c:505     mov %rsp, %rbp                     |505      {
   [000]  2336.210811381:                        ffffffff9f02fddd __switch_to+0xd         process_64.c:505     pushq  %r15
   [000]  2336.210811381:                        ffffffff9f02fddf __switch_to+0xf         process_64.c:507     leaq  0x1340(%rsi), %r15           |507             struct thread_struct *next = &next_p->thread;
   [000]  2336.210811381:                        ffffffff9f02fde6 __switch_to+0x16        process_64.c:505     pushq  %r14                        |505      {
   [000]  2336.210811381:                        ffffffff9f02fde8 __switch_to+0x18        process_64.c:506     leaq  0x1340(%rdi), %r14           |506             struct thread_struct *prev = &prev_p->thread;
   [000]  2336.210811381:                        ffffffff9f02fdef __switch_to+0x1f        process_64.c:505     pushq  %r13                        |505      {
   [000]  2336.210811381:                        ffffffff9f02fdf1 __switch_to+0x21        process_64.c:505     mov %rdi, %r13
   [000]  2336.210811381:                        ffffffff9f02fdf4 __switch_to+0x24        process_64.c:505     pushq  %r12
   [000]  2336.210811381:                        ffffffff9f02fdf6 __switch_to+0x26        process_64.c:505     mov %rsi, %r12
   [000]  2336.210811381:                        ffffffff9f02fdf9 __switch_to+0x29        process_64.c:505     pushq  %rbx
   [000]  2336.210811381:                        ffffffff9f02fdfa __switch_to+0x2a        process_64.c:505     sub $0x10, %rsp
   [000]  2336.210811381:                        ffffffff9f02fdfe __switch_to+0x2e        bitops.h:207     movq  (%rax), %rdx                     |515             if (!test_thread_flag(TIF_NEED_FPU_LOAD))
   [000]  2336.210811381:                        ffffffff9f02fe01 __switch_to+0x31        process_64.c:510     movl  %gs:0x60fe0548(%rip), %ebx   |510             int cpu = smp_processor_id();
   [000]  2336.210811381:                        ffffffff9f02fe08 __switch_to+0x38        process_64.c:515     and $0x40, %dh                     |515             if (!test_thread_flag(TIF_NEED_FPU_LOAD))
   [000]  2336.210811382:   jcc                  ffffffff9f02fe0b __switch_to+0x3b        process_64.c:515     jz 0x2c2    IPC: 2.83 (17/6)
   [000]  2336.210811382:                        ffffffff9f0300cd __switch_to+0x2fd       internal.h:574     testb  $0x20, 0x26(%rax)             |516                     switch_fpu_prepare(prev_fpu, cpu);
   [000]  2336.210811382:   jcc                  ffffffff9f0300d1 __switch_to+0x301       internal.h:574     jnz 0xfffffffffffffd40
   [000]  2336.210811382:                        ffffffff9f0300d7 __switch_to+0x307       process_64.c:508     leaq  0x1400(%rdi), %rcx           |508             struct fpu *prev_fpu = &prev->fpu;
   [000]  2336.210811382:                        ffffffff9f0300de __switch_to+0x30e       cpufeature.h:175     nopl  %eax, (%rax,%rax,1)          |516                     switch_fpu_prepare(prev_fpu, cpu);
   [000]  2336.210811382:                        ffffffff9f0300e3 __switch_to+0x313       internal.h:327     movl  0x181715e(%rip), %r9d
   [000]  2336.210811382:                        ffffffff9f0300ea __switch_to+0x31a       internal.h:420     leaq  0x1440(%r13), %rdi
   [000]  2336.210811382:                        ffffffff9f0300f1 __switch_to+0x321       internal.h:327     test %r9d, %r9d
   [000]  2336.210811382:   jcc                  ffffffff9f0300f4 __switch_to+0x324       internal.h:327     jz 0x141
   [000]  2336.210811382:                        ffffffff9f0300fa __switch_to+0x32a       internal.h:329     mov $0xffffffff, %eax
   [000]  2336.210811382:                        ffffffff9f0300ff __switch_to+0x32f       internal.h:329     mov %eax, %edx
   [000]  2336.210811382:                        ffffffff9f030101 __switch_to+0x331       internal.h:329     xsaves64 (%rdi)
   [000]  2336.210811382:                        ffffffff9f030105 __switch_to+0x335       internal.h:329     xor %eax, %eax
   [000]  2336.210811382:                        ffffffff9f030107 __switch_to+0x337       internal.h:332     test %eax, %eax
   [000]  2336.210811382:   jcc                  ffffffff9f030109 __switch_to+0x339       internal.h:332     jnz 0x125
   [000]  2336.210811382:                        ffffffff9f03010f __switch_to+0x33f       internal.h:426     testb  $0xe0, 0x1640(%r13)
   [000]  2336.210811382:   jcc                  ffffffff9f030117 __switch_to+0x347       internal.h:426     jnz 0xf0
   [000]  2336.210811382:                        ffffffff9f03011d __switch_to+0x34d       internal.h:578     movl  %ebx, 0x1400(%r13)
   [000]  2336.210811382:                        ffffffff9f030124 __switch_to+0x354       jump_label.h:25     nopl  %eax, (%rax,%rax,1)
   [000]  2336.210811382:   jmp                  ffffffff9f030129 __switch_to+0x359       jump_label.h:34     jmp 0xfffffffffffffce8
   [000]  2336.210811382:                        ffffffff9f02fe11 __switch_to+0x41        process_64.c:201     mov %fs, %ax                       |523             save_fsgs(prev_p);
   [000]  2336.210811382:                        ffffffff9f02fe14 __switch_to+0x44        process_64.c:201     movw  %ax, 0x1364(%r13)
   [000]  2336.210811382:                        ffffffff9f02fe1c __switch_to+0x4c        process_64.c:202     mov %gs, %ax
   [000]  2336.210811382:                        ffffffff9f02fe1f __switch_to+0x4f        process_64.c:164     cmpw  $0x0, 0x1364(%r13)
   [000]  2336.210811382:                        ffffffff9f02fe28 __switch_to+0x58        process_64.c:202     movw  %ax, 0x1366(%r13)
   [000]  2336.210811382:   jcc                  ffffffff9f02fe30 __switch_to+0x60        process_64.c:164     jnz 0x373
   [000]  2336.210811382:                        ffffffff9f02fe36 __switch_to+0x66        process_64.c:164     cmpw  $0x0, 0x1366(%r13)
   [000]  2336.210811428:   jcc                  ffffffff9f02fe3f __switch_to+0x6f        process_64.c:164     jnz 0x346    IPC: 0.13 (27/195)
   [000]  2336.210811428:                        ffffffff9f02fe45 __switch_to+0x75        paravirt.h:271     mov %ebx, %esi                       |529             load_TLS(next, cpu);
   [000]  2336.210811428:                        ffffffff9f02fe47 __switch_to+0x77        paravirt.h:271     mov %r15, %rdi
   [000]  2336.210811428:   call                 ffffffff9f02fe4a __switch_to+0x7a        paravirt.h:271     callq  0x48876
   [000]  2336.210811428:                        ffffffff9f0786c0 native_load_tls+0x0     desc.h:282     movq  (%rdi), %rdx                       |282                     gdt[GDT_ENTRY_TLS_MIN + i] = t->tls_array[i];
   [000]  2336.210811428:                        ffffffff9f0786c3 native_load_tls+0x3     desc.h:57     mov %esi, %esi                            |278             struct desc_struct *gdt = get_cpu_gdt_rw(cpu);
   [000]  2336.210811428:                        ffffffff9f0786c5 native_load_tls+0x5     desc.h:57     mov $0x9000, %rax
   [000]  2336.210811428:                        ffffffff9f0786cc native_load_tls+0xc     desc.h:57     addq  -0x5fbbb680(,%rsi,8), %rax
   [000]  2336.210811428:                        ffffffff9f0786d4 native_load_tls+0x14    desc.h:282     movq  %rdx, 0x60(%rax)                   |282                     gdt[GDT_ENTRY_TLS_MIN + i] = t->tls_array[i];
   [000]  2336.210811428:                        ffffffff9f0786d8 native_load_tls+0x18    desc.h:282     movq  0x8(%rdi), %rdx
   [000]  2336.210811428:                        ffffffff9f0786dc native_load_tls+0x1c    desc.h:282     movq  %rdx, 0x68(%rax)
   [000]  2336.210811428:                        ffffffff9f0786e0 native_load_tls+0x20    desc.h:282     movq  0x10(%rdi), %rdx
   [000]  2336.210811428:                        ffffffff9f0786e4 native_load_tls+0x24    desc.h:282     movq  %rdx, 0x70(%rax)
   [000]  2336.210811429:   return               ffffffff9f0786e8 native_load_tls+0x28    desc.h:283     retq      IPC: 3.25 (13/4)               |283      }
   [000]  2336.210811429:                        ffffffff9f02fe4f __switch_to+0x7f        paravirt.h:271     data16 nop                           |529             load_TLS(next, cpu);
   [000]  2336.210811429:                        ffffffff9f02fe51 __switch_to+0x81        paravirt.h:611     mov %r12, %rdi                       |536             arch_end_context_switch(next_p);
   [000]  2336.210811429:                        ffffffff9f02fe54 __switch_to+0x84        paravirt.h:611     nopl  %eax, (%rax)
   [000]  2336.210811429:                        ffffffff9f02fe5b __switch_to+0x8b        process_64.c:552     mov %es, %ax                       |552             savesegment(es, prev->es);
   [000]  2336.210811429:                        ffffffff9f02fe5e __switch_to+0x8e        process_64.c:552     movw  %ax, 0x20(%r14)
   [000]  2336.210811429:                        ffffffff9f02fe63 __switch_to+0x93        process_64.c:553     movzxw  0x1360(%r12), %eax         |553             if (unlikely(next->es | prev->es))
   [000]  2336.210811429:                        ffffffff9f02fe6c __switch_to+0x9c        process_64.c:553     mov %eax, %ebx
   [000]  2336.210811429:                        ffffffff9f02fe6e __switch_to+0x9e        process_64.c:553     orw  0x1360(%r13), %bx
   [000]  2336.210811430:   jcc                  ffffffff9f02fe76 __switch_to+0xa6        process_64.c:553     jnz 0x31f    IPC: 4.50 (9/2)
   [000]  2336.210811430:                        ffffffff9f02fe7c __switch_to+0xac        process_64.c:556     mov %ds, %ax                       |556             savesegment(ds, prev->ds);
   [000]  2336.210811430:                        ffffffff9f02fe7f __switch_to+0xaf        process_64.c:556     movw  %ax, 0x22(%r14)
   [000]  2336.210811430:                        ffffffff9f02fe84 __switch_to+0xb4        process_64.c:557     movzxw  0x1362(%r12), %eax         |557             if (unlikely(next->ds | prev->ds))
   [000]  2336.210811430:                        ffffffff9f02fe8d __switch_to+0xbd        process_64.c:557     mov %eax, %ebx
   [000]  2336.210811430:                        ffffffff9f02fe8f __switch_to+0xbf        process_64.c:557     orw  0x1362(%r13), %bx
   [000]  2336.210811430:   jcc                  ffffffff9f02fe97 __switch_to+0xc7        process_64.c:557     jnz 0x305
   [000]  2336.210811430:                        ffffffff9f02fe9d __switch_to+0xcd        process_64.c:283     movq  0x1368(%r12), %rdx           |560             x86_fsgsbase_load(prev, next);
   [000]  2336.210811430:                        ffffffff9f02fea5 __switch_to+0xd5        process_64.c:283     movzxw  0x1364(%r13), %ecx
   [000]  2336.210811430:                        ffffffff9f02fead __switch_to+0xdd        process_64.c:284     movzxw  0x1364(%r12), %eax
   [000]  2336.210811430:                        ffffffff9f02feb6 __switch_to+0xe6        process_64.c:236     cmp $0x3, %ax
   [000]  2336.210811430:   jcc                  ffffffff9f02feba __switch_to+0xea        process_64.c:236     jnbe 0x2b4
   [000]  2336.210811430:                        ffffffff9f02fec0 __switch_to+0xf0        process_64.c:241     test %rdx, %rdx
   [000]  2336.210811430:   jcc                  ffffffff9f02fec3 __switch_to+0xf3        process_64.c:241     jz 0x1c2
   [000]  2336.210811430:   jmp                  ffffffff9f030085 __switch_to+0x2b5       cpufeature.h:175     jmp 0xd7
   [000]  2336.210811430:                        ffffffff9f03015c __switch_to+0x38c       process_64.c:262     or %eax, %ecx
   [000]  2336.210811430:                        ffffffff9f03015e __switch_to+0x38e       process_64.c:262     movzx %cx, %ecx
   [000]  2336.210811430:                        ffffffff9f030161 __switch_to+0x391       process_64.c:262     orq  0x1368(%r13), %rcx
   [000]  2336.210811430:   jcc                  ffffffff9f030168 __switch_to+0x398       process_64.c:262     jz 0xfffffffffffffd7a
   [000]  2336.210811430:                        ffffffff9f03016e __switch_to+0x39e       segment.h:349     mov %ax, %fs
   [000]  2336.210811430:   jmp                  ffffffff9f030170 __switch_to+0x3a0       segment.h:356     jmp 0xfffffffffffffd72
   [000]  2336.210811430:                        ffffffff9f02fee2 __switch_to+0x112       process_64.c:285     movq  0x1370(%r12), %r14
   [000]  2336.210811430:                        ffffffff9f02feea __switch_to+0x11a       process_64.c:285     movzxw  0x1366(%r13), %eax
   [000]  2336.210811430:                        ffffffff9f02fef2 __switch_to+0x122       process_64.c:286     movzxw  0x1366(%r12), %ebx
   [000]  2336.210811430:                        ffffffff9f02fefb __switch_to+0x12b       process_64.c:236     cmp $0x3, %bx
   [000]  2336.210811430:   jcc                  ffffffff9f02feff __switch_to+0x12f       process_64.c:236     jnbe 0x24e
   [000]  2336.210811430:                        ffffffff9f02ff05 __switch_to+0x135       process_64.c:241     test %r14, %r14
   [000]  2336.210811439:   jcc                  ffffffff9f02ff08 __switch_to+0x138       process_64.c:241     jnz 0x19b    IPC: 0.67 (27/40)
   [000]  2336.210811439:   jmp                  ffffffff9f02ff0e __switch_to+0x13e       cpufeature.h:175     jmp 0x22d
   [000]  2336.210811439:                        ffffffff9f03013b __switch_to+0x36b       process_64.c:262     or %ebx, %eax
   [000]  2336.210811439:                        ffffffff9f03013d __switch_to+0x36d       process_64.c:262     movzx %ax, %eax
   [000]  2336.210811439:                        ffffffff9f030140 __switch_to+0x370       process_64.c:262     orq  0x1370(%r13), %rax
   [000]  2336.210811439:   jcc                  ffffffff9f030147 __switch_to+0x377       process_64.c:262     jz 0xfffffffffffffde2
   [000]  2336.210811439:                        ffffffff9f02ff29 __switch_to+0x159       process_64.c:565     movq  %r12, %gs:0x60fe6c8f(%rip)   |565             this_cpu_write(current_task, next_p);
   [000]  2336.210811439:                        ffffffff9f02ff31 __switch_to+0x161       process_64.c:566     movq  0x18(%r12), %rax             |566             this_cpu_write(cpu_current_top_of_stack, task_top_of_stack(next_p));
   [000]  2336.210811439:                        ffffffff9f02ff36 __switch_to+0x166       process_64.c:566     add $0x4000, %rax
   [000]  2336.210811439:                        ffffffff9f02ff3c __switch_to+0x16c       process_64.c:566     movq  %rax, %gs:0x60fd60c8(%rip)
   [000]  2336.210811439:                        ffffffff9f02ff44 __switch_to+0x174       internal.h:595     movl  0x161efa6(%rip), %ebx          |568             switch_fpu_finish(next_fpu);
   [000]  2336.210811439:                        ffffffff9f02ff4a __switch_to+0x17a       current.h:15     movq  %gs:0x16bc0, %rax
   [000]  2336.210811439:                        ffffffff9f02ff53 __switch_to+0x183       bitops.h:55     lock orb  $0x40, 0x1(%rax)
   [000]  2336.210811439:   jmp                  ffffffff9f02ff58 __switch_to+0x188       cpufeature.h:175     jmp 0x41
   [000]  2336.210811439:   jmp                  ffffffff9f02ff99 __switch_to+0x1c9       cpufeature.h:175     jmp 0x18                           |571             update_task_stack(next_p);
   [000]  2336.210811439:                        ffffffff9f02ffb1 __switch_to+0x1e1       process.h:16     movq  (%r12), %rdx                     |573             switch_to_extra(prev_p, next_p);
   [000]  2336.210811439:                        ffffffff9f02ffb5 __switch_to+0x1e5       process.h:17     movq  (%r13), %rax
   [000]  2336.210811439:                        ffffffff9f02ffb9 __switch_to+0x1e9       jump_label.h:41     nopl  %eax, (%rax,%rax,1)
   [000]  2336.210811439:                        ffffffff9f02ffbe __switch_to+0x1ee       process.h:36     and $0x2418620, %edx
   [000]  2336.210811439:                        ffffffff9f02ffc4 __switch_to+0x1f4       process.h:36     and $0x2418e20, %eax
   [000]  2336.210811439:                        ffffffff9f02ffc9 __switch_to+0x1f9       process.h:36     or %rax, %rdx
   [000]  2336.210811439:   jcc                  ffffffff9f02ffcc __switch_to+0x1fc       process.h:36     jnz 0x1a9
   [000]  2336.210811439:   jmp                  ffffffff9f02ffd2 __switch_to+0x202       cpufeature.h:175     jmp 0x1a                           |581             if (unlikely(static_cpu_has(X86_FEATURE_XENPV) &&
   [000]  2336.210811439:   jmp                  ffffffff9f02ffec __switch_to+0x21c       cpufeature.h:175     jmp 0x15                           |586             if (static_cpu_has_bug(X86_BUG_SYSRET_SS_ATTRS)) {
   [000]  2336.210811439:   jmp                  ffffffff9f030001 __switch_to+0x231       jump_label.h:41     jmp 0x72                            |615             resctrl_sched_in();
   [000]  2336.210811439:                        ffffffff9f030073 __switch_to+0x2a3       process_64.c:618     add $0x10, %rsp                    |618      }
   [000]  2336.210811439:                        ffffffff9f030077 __switch_to+0x2a7       process_64.c:618     mov %r13, %rax
   [000]  2336.210811439:                        ffffffff9f03007a __switch_to+0x2aa       process_64.c:618     popq  %rbx
   [000]  2336.210811439:                        ffffffff9f03007b __switch_to+0x2ab       process_64.c:618     popq  %r12
   [000]  2336.210811439:                        ffffffff9f03007d __switch_to+0x2ad       process_64.c:618     popq  %r13
   [000]  2336.210811439:                        ffffffff9f03007f __switch_to+0x2af       process_64.c:618     popq  %r14
   [000]  2336.210811439:                        ffffffff9f030081 __switch_to+0x2b1       process_64.c:618     popq  %r15
   [000]  2336.210811439:                        ffffffff9f030083 __switch_to+0x2b3       process_64.c:618     popq  %rbp
   [000]  2336.210811468:   tr end  return       ffffffff9f030084 __switch_to+0x2b4       process_64.c:618     retq      IPC: 0.27 (33/122)
   [000]  2336.210817292:                        ffffffff9f02fdd0 __switch_to+0x0         process_64.c:505     pushq  %rbp                        |505      {
   [000]  2336.210817292:                        ffffffff9f02fdd1 __switch_to+0x1         current.h:15     movq  %gs:0x16bc0, %rax                |515             if (!test_thread_flag(TIF_NEED_FPU_LOAD))
   [000]  2336.210817292:                        ffffffff9f02fdda __switch_to+0xa         process_64.c:505     mov %rsp, %rbp                     |505      {
   [000]  2336.210817292:                        ffffffff9f02fddd __switch_to+0xd         process_64.c:505     pushq  %r15
   [000]  2336.210817292:                        ffffffff9f02fddf __switch_to+0xf         process_64.c:507     leaq  0x1340(%rsi), %r15           |507             struct thread_struct *next = &next_p->thread;
   [000]  2336.210817292:                        ffffffff9f02fde6 __switch_to+0x16        process_64.c:505     pushq  %r14                        |505      {
   [000]  2336.210817292:                        ffffffff9f02fde8 __switch_to+0x18        process_64.c:506     leaq  0x1340(%rdi), %r14           |506             struct thread_struct *prev = &prev_p->thread;
   [000]  2336.210817292:                        ffffffff9f02fdef __switch_to+0x1f        process_64.c:505     pushq  %r13                        |505      {
   [000]  2336.210817292:                        ffffffff9f02fdf1 __switch_to+0x21        process_64.c:505     mov %rdi, %r13
   [000]  2336.210817292:                        ffffffff9f02fdf4 __switch_to+0x24        process_64.c:505     pushq  %r12
   [000]  2336.210817292:                        ffffffff9f02fdf6 __switch_to+0x26        process_64.c:505     mov %rsi, %r12

Example: Tracing GUI program ``kcalc``
------------------------------------------

Tracing a GUI program produces a very large trace. In this case, a
simple calculator program ``kcalc`` is started and immediately
closed.

First we can get some debug packages::

   $ debug_packages=$(find-dbgsym-packages $(which kcalc))
   dpkg-query: no path found matching pattern /lib/x86_64-linux-gnu/libXau.so.6
   W: Cannot find debug package for /lib/x86_64-linux-gnu/libXau.so.6 (f387ef05bc753c7843a31067224d3d3e92b7190e)
   dpkg-query: no path found matching pattern /lib/x86_64-linux-gnu/libstdc++.so.6
   W: Cannot find debug package for /lib/x86_64-linux-gnu/libstdc++.so.6 (48f1a714f64ac85caa1496bcb14275c8ff0aeace)
   dpkg-query: no path found matching pattern /lib/x86_64-linux-gnu/libogg.so.0
   W: Cannot find debug package for /lib/x86_64-linux-gnu/libogg.so.0 (861e2c0ca7f6c2ddfdffaf0ea9831e9ae84a44ac)
   $ echo $debug_packages
   kcalc-dbgsym lib32stdc++6-10-dbg lib64stdc++6-10-dbg libbsd0-dbgsym libbz2-1.0-dbgsym libcanberra0-dbgsym libdbus-1-3-dbgsym libdbusmenu-qt5-2-dbgsym libdouble-conversion3-dbgsym libfam0-dbgsym libfreetype6-dbgsym libgcrypt20-dbgsym libgl1-dbgsym libglib2.0-0-dbgsym libglvnd0-dbgsym libglx0-dbgsym libgmp10-dbgsym libgpg-error0-dbgsym libgraphite2-3-dbgsym libharfbuzz0b-dbgsym libicu66-dbgsym libkf5archive5-dbgsym libkf5attica5-dbgsym libkf5authcore5-dbgsym libkf5codecs5-dbgsym libkf5configcore5-dbgsym libkf5configgui5-dbgsym libkf5configwidgets5-dbgsym libkf5coreaddons5-dbgsym libkf5crash5-dbgsym libkf5globalaccel5-dbgsym libkf5guiaddons5-dbgsym libkf5i18n5-dbgsym libkf5iconthemes5-dbgsym libkf5itemviews5-dbgsym libkf5notifications5-dbgsym libkf5widgetsaddons5-dbgsym libkf5windowsystem5-dbgsym libkf5xmlgui5-dbgsym libltdl7-dbgsym liblz4-1-dbgsym liblzma5-dbgsym libmpfr6-dbgsym libpcre2-16-0-dbgsym libpcre3-dbg libpng16-16-dbgsym libqt5core5a-dbgsym libqt5dbus5-dbgsym libqt5gui5-dbgsym libqt5network5-dbgsym libqt5printsupport5-dbgsym libqt5svg5-dbgsym libqt5texttospeech5-dbgsym libqt5widgets5-dbgsym libqt5x11extras5-dbgsym libqt5xml5-dbgsym libstdc++6-10-dbg libsystemd0-dbgsym libtdb1-dbgsym libvorbis0a-dbgsym libvorbisfile3-dbgsym libx11-6-dbgsym libx32stdc++6-10-dbg libxcb-keysyms1-dbgsym libxcb1-dbgsym libxdmcp6-dbg zlib1g-dbgsym

find-dbgsym-packages fails in 3 cases because Ubuntu mixes up /lib and
/usr/lib, but we can find them if we put in the correct names::

   $ more_debug_packages=$(find-dbgsym-packages /usr/lib/x86_64-linux-gnu/libXau.so.6 /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib/x86_64-linux-gnu/libogg.so.0)
   $ echo $more_debug_packages
   lib32stdc++6-10-dbg lib64stdc++6-10-dbg libogg-dbg libstdc++6-10-dbg libx32stdc++6-10-dbg libxau6-dbg

We can try to install those::

   $ sudo apt-get install $debug_packages $more_debug_packages
   [sudo] password for ahunter:
   Reading package lists... Done
   Building dependency tree
   Reading state information... Done
   Some packages could not be installed. This may mean that you have
   requested an impossible situation or if you are using the unstable
   distribution that some required packages have not yet been created
   or been moved out of Incoming.
   The following information may help to resolve the situation:

   The following packages have unmet dependencies:
    lib64stdc++6-10-dbg:i386 : Depends: lib64stdc++6:i386 (>= 10-20200411-0ubuntu1) but it is not going to be installed
                               Depends: lib64gcc-s1:i386 (>= 4.2) but it is not going to be installed
                               Depends: libc6-amd64:i386 (>= 2.18) but it is not going to be installed
    liblzma5-dbgsym : Depends: liblzma5 (= 5.2.4-1ubuntu1) but 5.2.4-1 is to be installed
   E: Unable to correct problems, you have held broken packages.

But it seems the dependencies are wrong, so we leave out those ones and
get the rest::

   $ sudo apt-get install kcalc-dbgsym lib32stdc++6-10-dbg libbsd0-dbgsym libbz2-1.0-dbgsym libcanberra0-dbgsym libdbus-1-3-dbgsym libdbusmenu-qt5-2-dbgsym libdouble-conversion3-dbgsym libfam0-dbgsym libfreetype6-dbgsym libgcrypt20-dbgsym libgl1-dbgsym libglib2.0-0-dbgsym libglvnd0-dbgsym libglx0-dbgsym libgmp10-dbgsym libgpg-error0-dbgsym libgraphite2-3-dbgsym libharfbuzz0b-dbgsym libicu66-dbgsym libkf5archive5-dbgsym libkf5attica5-dbgsym libkf5authcore5-dbgsym libkf5codecs5-dbgsym libkf5configcore5-dbgsym libkf5configgui5-dbgsym libkf5configwidgets5-dbgsym libkf5coreaddons5-dbgsym libkf5crash5-dbgsym libkf5globalaccel5-dbgsym libkf5guiaddons5-dbgsym libkf5i18n5-dbgsym libkf5iconthemes5-dbgsym libkf5itemviews5-dbgsym libkf5notifications5-dbgsym libkf5widgetsaddons5-dbgsym libkf5windowsystem5-dbgsym libkf5xmlgui5-dbgsym libltdl7-dbgsym liblz4-1-dbgsym libmpfr6-dbgsym libogg-dbg libpcre2-16-0-dbgsym libpcre3-dbg libpng16-16-dbgsym libqt5core5a-dbgsym libqt5dbus5-dbgsym libqt5gui5-dbgsym libqt5network5-dbgsym libqt5printsupport5-dbgsym libqt5svg5-dbgsym libqt5texttospeech5-dbgsym libqt5widgets5-dbgsym libqt5x11extras5-dbgsym libqt5xml5-dbgsym libstdc++6-10-dbg libsystemd0-dbgsym libtdb1-dbgsym libvorbis0a-dbgsym libvorbisfile3-dbgsym libx11-6-dbgsym libx32stdc++6-10-dbg libxau6-dbg libxcb1-dbgsym libxcb-keysyms1-dbgsym libxdmcp6-dbg zlib1g-dbgsym

This example uses ``perf record`` with extra privileges, as described in
the "Adding capabilities to ``perf``" section.

We can use ``perf record`` with options:

- ``-m,64M`` to set the trace buffer size to 64 MiB. This is needed
  to avoid trace data loss. Note the comma is needed. Also **be careful
  setting large buffer sizes**. With per-cpu tracing (the default), one
  buffer per CPU will be allocated. In our case we have 8 CPUs so that
  means 512 MiB. However when tracing with per-task contexts, there will
  be one buffer per task, which might be far more than anticipated.
- ``-e`` to select which events, i.e. the following:
- ``intel_pt//u`` to get Intel PT tracing userspace only
- ``kcalc`` is the workload.

::

   $ perf record -m,64M -e intel_pt//u kcalc
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 76.373 MB perf.data ]

Using ``perf report``, we can get a profile with call chains without
needing any special options::

   $ perf report | head -156
   # To display the perf.data header info, please use --header/--header-only options.
   #
   #
   # Total Lost Samples: 0
   #
   # Samples: 0  of event 'intel_pt//u'
   # Event count (approx.): 0
   #
   # Children      Self  Command  Shared Object  Symbol
   # ........  ........  .......  .............  ......
   #


   # Samples: 0  of event 'dummy:u'
   # Event count (approx.): 0
   #
   # Children      Self  Command  Shared Object  Symbol
   # ........  ........  .......  .............  ......
   #


   # Samples: 0  of event 'dummy:u'
   # Event count (approx.): 0
   #
   # Children      Self  Command  Shared Object  Symbol
   # ........  ........  .......  .............  ......
   #


   # Samples: 1K of event 'instructions:u'
   # Event count (approx.): 1030926311
   #
   # Children      Self  Command          Shared Object                  Symbol
   # ........  ........  ...............  .............................  .....................................................................................................................................................................................................................................
   #
       41.00%     0.00%  kcalc            libQt5Gui.so.5.12.8            [.] QFontDatabase::findFont
               |
               ---QFontDatabase::findFont
                  |
                  |--36.29%--loadEngine
                  |          QFontconfigDatabase::fontEngine
                  |          |
                  |           --35.82%--QFontconfigDatabase::setupFontEngine
                  |                     |
                  |                     |--27.98%--FcFontMatch
                  |                     |          0x7f0068874c6d
                  |                     |          |
                  |                     |           --27.62%--0x7f0068874b42
                  |                     |                     |
                  |                     |                      --22.71%--0x7f0068874845
                  |                     |                                |
                  |                     |                                |--15.92%--0x7f006887469f
                  |                     |                                |          |
                  |                     |                                |          |--7.57%--0x7f006887c198
                  |                     |                                |          |          |
                  |                     |                                |          |           --3.03%--0x7f006887be36
                  |                     |                                |          |                     |
                  |                     |                                |          |                      --2.21%--__strchr_avx2
                  |                     |                                |          |
                  |                     |                                |           --6.99%--0x7f006887c18b
                  |                     |                                |                     |
                  |                     |                                |                      --2.70%--0x7f006887be36
                  |                     |                                |                                |
                  |                     |                                |                                 --2.23%--__strchr_avx2
                  |                     |                                |
                  |                     |                                |--1.75%--0x7f006887447b
                  |                     |                                |          FcStrCmpIgnoreCase
                  |                     |                                |
                  |                     |                                 --1.36%--0x7f006887464f
                  |                     |
                  |                      --7.84%--FcConfigSubstituteWithPat
                  |                                |
                  |                                |--4.29%--0x7f00688639f7
                  |                                |          |
                  |                                |          |--2.35%--0x7f006887c198
                  |                                |          |          |
                  |                                |          |           --0.66%--0x7f006887be36
                  |                                |          |                     |
                  |                                |          |                      --0.56%--__strchr_avx2
                  |                                |          |
                  |                                |           --1.31%--0x7f006887c18b
                  |                                |                     |
                  |                                |                      --0.51%--0x7f006887be36
                  |                                |                                __strchr_avx2
                  |                                |
                  |                                 --0.54%--0x7f0068863a1c
                  |                                           FcStrStrIgnoreCase
                  |
                   --4.71%--initializeDb
                             QFontconfigDatabase::populateFontDatabase
                             |
                             |--2.01%--FcFontList
                             |          FcFontSetList
                             |          |
                             |           --0.52%--0x7f006887c240
                             |
                             |--1.66%--populateFromPattern
                             |          |
                             |           --1.03%--FcLangSetHasLang
                             |
                              --0.88%--0x7f006886318d
                                        0x7f0068863157
                                        |
                                         --0.51%--0x7f00688711db

       36.29%     0.00%  kcalc            libQt5Gui.so.5.12.8            [.] loadEngine
               |
               ---loadEngine
                  QFontconfigDatabase::fontEngine
                  |
                   --35.82%--QFontconfigDatabase::setupFontEngine
                             |
                             |--27.98%--FcFontMatch
                             |          0x7f0068874c6d
                             |          |
                             |           --27.62%--0x7f0068874b42
                             |                     |
                             |                      --22.71%--0x7f0068874845
                             |                                |
                             |                                |--15.92%--0x7f006887469f
                             |                                |          |
                             |                                |          |--7.57%--0x7f006887c198
                             |                                |          |          |
                             |                                |          |           --3.03%--0x7f006887be36
                             |                                |          |                     |
                             |                                |          |                      --2.21%--__strchr_avx2
                             |                                |          |
                             |                                |           --6.99%--0x7f006887c18b
                             |                                |                     |
                             |                                |                      --2.70%--0x7f006887be36
                             |                                |                                |
                             |                                |                                 --2.23%--__strchr_avx2
                             |                                |
                             |                                |--1.75%--0x7f006887447b
                             |                                |          FcStrCmpIgnoreCase
                             |                                |
                             |                                 --1.36%--0x7f006887464f
                             |
                              --7.84%--FcConfigSubstituteWithPat
                                        |
                                        |--4.29%--0x7f00688639f7
                                        |          |
                                        |          |--2.35%--0x7f006887c198
                                        |          |          |
                                        |          |           --0.66%--0x7f006887be36
                                        |          |                     |
                                        |          |                      --0.56%--__strchr_avx2
                                        |          |
                                        |           --1.31%--0x7f006887c18b
                                        |                     |
                                        |                      --0.51%--0x7f006887be36
                                        |                                __strchr_avx2
                                        |
                                         --0.54%--0x7f0068863a1c
                                                   FcStrStrIgnoreCase

Example: Using SQL to analyze latencies
---------------------------------------

In this example, eMMC driver latencies are analyzed.

On an otherwise idle test system, we can run a small test with dd and
trace it with Intel PT. In this case we will trace as root.

The test case is reading from the eMMC because reading is faster and
therefore is affected more by latencies.

First we can drop caches::

   # sync
   # echo 3 > /proc/sys/vm/drop_caches
   [  398.282985] sh (207): drop_caches: 3

We can start an open-ended trace, then run the test case ``dd`` command,
then kill ``perf`` to end the trace.

We can use ``perf record`` with options:

- ``-o t4`` to name the output t4 as it was, in fact, our the 4th
  test
- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)
- ``--no-bpf-event`` to avoid unwanted bpf events because they were
  making perf segfault.
- ``-e`` to select which events, i.e. the following:
- ``intel_pt/cyc,mtc=0/k`` to get Intel PT with cycle-accurate mode.
  We also avoid MTC packets which was found to give a better trace in
  this case.

::

   # perf record -o t4 -a --kcore --no-bpf-event -e intel_pt/cyc,mtc=0/k &
   # dd if=/mnt/mmc/afile of=/dev/null bs=4096
   4096+0 records in
   4096+0 records out
   16777216 bytes (17 MB, 16 MiB) copied, 0.0758838 s, 221 MB/s
   # kill %1
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 1.923 MB t4 ]
   [1]+  Terminated                 perf record -o t4 -a --kcore --no-bpf-event -e intel_pt/cyc,mtc=0/k

Note, because the test system is otherwise idle, and the test case is
small and I/O bound, the resulting trace is really quite small.

For a kernel trace with the ``--kcore`` option, the output directory
(called t4 in this case) and everything in it, is all that we need, so
we transfer it to another machine for analysis.

There, we can export it to a SQLite3 database::

   $ perf script -i t4 --itrace=bep -s ~/libexec/perf-core/scripts/python/export-to-sqlite.py t4.db branches calls
   2020-06-09 11:17:12.507846 Creating database ...
   2020-06-09 11:17:12.512662 Writing records...
   2020-06-09 11:18:13.610538 Adding indexes
   2020-06-09 11:18:14.168248 Dropping unused tables
   2020-06-09 11:18:14.177895 Done

To make sense of the trace we need some kernel knowledge, as follows:

- The function that performs the 'read' system call is called vfs_read
- The function that issues eMMC I/O is called mmc_blk_mq_issue_rq
- The function that completes eMMC I/O is called mmc_blk_mq_complete

The database contains a table of function calls which shows the time
that each function started (call_time) and the time that it ended
(return_time). That table has a 'view' named 'calls_view' which is
easier to deal with.

The SQL below is fairly straight forward, and it is likely even someone
without SQL knowledge can understand most of it. There are 2 tricks.
First, sqlite3 doesn't size columns correctly so we need to artificially
make the 'symbol' column wider by renaming it with a long name
``"symbol name            "``. Secondly, to show the call_time
difference to the previous row, the LAG function is used. That doesn't
make sense on the first row, but on the second row, the value is
411032602702 - 411032546128 = 56574.

::

   $ sqlite3  -column -header t4.db \
   > 'SELECT call_id,command,symbol AS "symbol name            ",call_time,call_time - LAG(call_time,1,0) OVER (ORDER BY id) call_time_lag,
   > return_time,elapsed_time,branch_count,insn_count,cyc_count,IPC,flags
   > FROM calls_view WHERE (command = "dd" AND symbol = "vfs_read") OR symbol IN ("mmc_blk_mq_issue_rq", "mmc_blk_mq_complete") ORDER BY call_time;' | head -100
   call_id     command     symbol name              call_time     call_time_lag  return_time   elapsed_time  branch_count  insn_count  cyc_count   IPC         flags
   ----------  ----------  -----------------------  ------------  -------------  ------------  ------------  ------------  ----------  ----------  ----------  ----------
   182536      dd          vfs_read                 411032546128  411032546128   411032550680  4552          293           2573        8890        0.29
   191117      dd          vfs_read                 411032602702  56574          411032605663  2961          285           2397        5726        0.42
   199364      dd          vfs_read                 411032652150  49448          411032655372  3222          282           2499        6233        0.4
   256863      dd          vfs_read                 411033324366  672216         411035014934  1690568       2456          19464       78746       0.25
   434468      kworker/3:  mmc_blk_mq_issue_rq      411034185597  861231         411034195025  9428          117           1005        18042       0.06
   436485      swapper     mmc_blk_mq_complete      411035001225  815628         411035009951  8726          464           3630        16785       0.22        jump
   437997      dd          vfs_read                 411035018460  17235          411035041818  23358         2194          17360       27335       0.64
   440393      kworker/3:  mmc_blk_mq_issue_rq      411035034183  15723          411035038508  4325          95            727         1785        0.41
   441291      dd          vfs_read                 411035043009  8826           411035044052  1043          119           1169        2080        0.56
   441496      dd          vfs_read                 411035044594  1585           411035045478  884           119           1169        1722        0.68
   441701      dd          vfs_read                 411035045885  1291           411035343621  297736        3981          36177       30929       1.17
   445677      kworker/3:  mmc_blk_mq_issue_rq      411035061376  15491          411035065210  3834          95            848         1079        0.79
   447604      swapper     mmc_blk_mq_complete      411035337501  276125         411035340344  2843          340           2692        5469        0.49        jump
   448875      dd          vfs_read                 411035344722  7221           411035345661  939           119           1169        1846        0.63
   449080      dd          vfs_read                 411035346106  1384           411035347080  974           119           1169        1894        0.62
   449285      dd          vfs_read                 411035347486  1380           411035348326  840           119           1169        1636        0.71
   449490      dd          vfs_read                 411035348733  1247           411035349616  883           119           1169        1724        0.68
   449695      dd          vfs_read                 411035350151  1418           411035350869  718           119           1169        1402        0.83
   449900      dd          vfs_read                 411035351288  1137           411035352137  849           119           1169        1657        0.71
   450105      dd          vfs_read                 411035352538  1250           411035353358  820           119           1169        1601        0.73
   450310      dd          vfs_read                 411035353758  1220           411035716494  362736        6102          50580       39418       1.28
   456386      kworker/3:  mmc_blk_mq_issue_rq      411035373641  19883          411035377812  4171          112           1078        8037        0.13
   457553      swapper     mmc_blk_mq_complete      411035710799  337158         411035713234  2435          368           2956        4685        0.63        jump
   458851      dd          vfs_read                 411035717565  6766           411035718671  1106          119           1169        2207        0.53
   459056      dd          vfs_read                 411035719152  1587           411035719880  728           119           1169        1420        0.82
   459261      dd          vfs_read                 411035720292  1140           411035721143  851           119           1169        1661        0.7
   459466      dd          vfs_read                 411035721544  1252           411035722345  801           119           1169        1560        0.75
   459671      dd          vfs_read                 411035722750  1206           411035723805  1055          119           1169        2051        0.57
   459876      dd          vfs_read                 411035724202  1452           411035725094  892           119           1169        1738        0.67
   460081      dd          vfs_read                 411035725494  1292           411035726204  710           119           1169        1390        0.84
   460286      dd          vfs_read                 411035726612  1118           411035727364  752           119           1169        1471        0.79
   460491      dd          vfs_read                 411035727769  1157           411035728705  936           119           1169        1821        0.64
   460696      dd          vfs_read                 411035729098  1329           411035729929  831           119           1169        1621        0.72
   460901      dd          vfs_read                 411035730327  1229           411035731267  940           119           1169        1831        0.64
   461106      dd          vfs_read                 411035731667  1340           411035732415  748           119           1169        1461        0.8
   461311      dd          vfs_read                 411035732827  1160           411035733667  840           119           1169        1638        0.71
   461516      dd          vfs_read                 411035734066  1239           411035734884  818           119           1169        1598        0.73
   461721      dd          vfs_read                 411035735284  1218           411035736118  834           119           1169        1627        0.72
   461926      dd          vfs_read                 411035736515  1231           411036291139  554624        6669          60095       46591       1.29
   468611      kworker/3:  mmc_blk_mq_issue_rq      411035760954  24439          411035765009  4055          112           1078        7813        0.14
   474012      swapper     mmc_blk_mq_complete      411036285461  524507         411036288423  2962          448           3710        5697        0.65        jump
   475391      dd          vfs_read                 411036292157  6696           411036293361  1204          121           1032        2343        0.44
   475598      dd          vfs_read                 411036293848  1691           411036294695  847           121           1032        1612        0.64
   475805      dd          vfs_read                 411036295112  1264           411036295943  831           121           1032        1578        0.65
   476012      dd          vfs_read                 411036296352  1240           411036297270  918           121           1032        1749        0.59
   476219      dd          vfs_read                 411036297672  1320           411036298519  847           121           1032        1610        0.64
   476426      dd          vfs_read                 411036298929  1257           411036299781  852           121           1032        1620        0.64
   476633      dd          vfs_read                 411036300180  1251           411036300978  798           121           1032        1515        0.68
   476840      dd          vfs_read                 411036301379  1199           411036302205  826           121           1032        1573        0.66
   477047      dd          vfs_read                 411036302605  1226           411036303510  905           121           1032        1723        0.6
   477254      dd          vfs_read                 411036303908  1303           411036304798  890           121           1032        1696        0.61
   477461      dd          vfs_read                 411036305195  1287           411036306098  903           121           1032        1718        0.6
   477668      dd          vfs_read                 411036306504  1309           411036307202  698           121           1032        1324        0.78
   477875      dd          vfs_read                 411036307618  1114           411036308522  904           121           1032        1721        0.6
   478082      dd          vfs_read                 411036308920  1302           411036309859  939           121           1032        1790        0.58
   478289      dd          vfs_read                 411036310258  1338           411036311112  854           121           1032        1624        0.64
   478496      dd          vfs_read                 411036311513  1255           411036312423  910           121           1032        1733        0.6
   478703      dd          vfs_read                 411036312822  1309           411036313592  770           121           1032        1462        0.71
   478910      dd          vfs_read                 411036314007  1185           411036314864  857           121           1032        1633        0.63
   479117      dd          vfs_read                 411036315262  1255           411036316193  931           121           1032        1773        0.58
   479324      dd          vfs_read                 411036316595  1333           411036317407  812           121           1032        1548        0.67
   479531      dd          vfs_read                 411036317809  1214           411036318730  921           121           1032        1752        0.59
   479738      dd          vfs_read                 411036319136  1327           411036319845  709           121           1032        1348        0.77
   479945      dd          vfs_read                 411036320259  1123           411036321429  1170          121           1032        2232        0.46
   480152      dd          vfs_read                 411036321833  1574           411036322748  915           121           1032        1744        0.59
   480359      dd          vfs_read                 411036323151  1318           411036323889  738           121           1032        1401        0.74
   480566      dd          vfs_read                 411036324307  1156           411036325164  857           121           1032        1634        0.63
   480773      dd          vfs_read                 411036325563  1256           411036326289  726           121           1032        1378        0.75
   480980      dd          vfs_read                 411036326696  1133           411036327544  848           121           1032        1614        0.64
   481187      dd          vfs_read                 411036328079  1383           411036328981  902           121           1032        1716        0.6
   481394      dd          vfs_read                 411036329382  1303           411036330224  842           121           1032        1604        0.64
   481601      dd          vfs_read                 411036330622  1240           411036331534  912           121           1032        1735        0.59
   481808      dd          vfs_read                 411036331931  1309           411036866337  534406        6223          51811       38456       1.35
   488019      kworker/3:  mmc_blk_mq_issue_rq      411036351419  19488          411036355435  4016          112           1078        7740        0.14
   489216      swapper     mmc_blk_mq_complete      411036860963  509544         411036863938  2975          437           3562        4657        0.76        jump
   490583      dd          vfs_read                 411036867336  6373           411036868323  987           121           1032        1884        0.55
   490790      dd          vfs_read                 411036868742  1406           411036869660  918           121           1032        1751        0.59
   490997      dd          vfs_read                 411036870070  1328           411036871041  971           121           1032        1847        0.56
   491204      dd          vfs_read                 411036871454  1384           411036872335  881           121           1032        1678        0.62
   491411      dd          vfs_read                 411036872734  1280           411036873536  802           121           1032        1523        0.68
   491618      dd          vfs_read                 411036873936  1202           411036874802  866           121           1032        1642        0.63
   491825      dd          vfs_read                 411036875207  1271           411036876030  823           121           1032        1564        0.66
   492032      dd          vfs_read                 411036876429  1222           411036877279  850           121           1032        1619        0.64
   492239      dd          vfs_read                 411036877677  1248           411036878577  900           121           1032        1712        0.6
   492446      dd          vfs_read                 411036878976  1299           411036879801  825           121           1032        1571        0.66
   492653      dd          vfs_read                 411036880199  1223           411036881130  931           121           1032        1769        0.58
   492860      dd          vfs_read                 411036881526  1327           411036882247  721           121           1032        1326        0.78
   493067      dd          vfs_read                 411036882653  1127           411036883546  893           121           1032        1700        0.61
   493274      dd          vfs_read                 411036883946  1293           411036884759  813           121           1032        1547        0.67
   493481      dd          vfs_read                 411036885167  1221           411036885989  822           121           1032        1563        0.66
   493688      dd          vfs_read                 411036886387  1220           411036887255  868           121           1032        1651        0.63
   493895      dd          vfs_read                 411036887652  1265           411036888461  809           121           1032        1538        0.67
   494102      dd          vfs_read                 411036888862  1210           411036889721  859           121           1032        1632        0.63
   494309      dd          vfs_read                 411036890119  1257           411036891039  920           121           1032        1751        0.59
   494516      dd          vfs_read                 411036891439  1320           411036892243  804           121           1032        1528        0.68
   494723      dd          vfs_read                 411036892641  1202           411036893542  901           121           1032        1716        0.6
   494930      dd          vfs_read                 411036893941  1300           411036894625  684           121           1032        1296        0.8
   495137      dd          vfs_read                 411036895162  1221           411036896011  849           121           1032        1615        0.64

The number of vfs_read between each eMMC I/O increases because of
readahead. It grows to 32 because the dd block size was 4096 and 32 \*
4096 = 128 KiB which is the readahead value determined on the test
system as follows::

   # cat /sys/block/mmcblk0/queue/read_ahead_kb
   128

When vfs_read gets cached data, it takes less that a microsecond.
However the vfs_read that triggers I/O settles at about 550 us. The
latency between the system call and the I/O being issued is about 20 to
25 us.

Of interest is the very first vfs_read that causes I/O. It takes about
1.7 ms. About half of that is spent before the I/O is even issued. One
possibility is runtime power management - resuming to a ready state from
a low power state. So we can add the runtime power management function
\__pm_runtime_resume to the SQL query and see if that fits::

   $ sqlite3  -column -header t4.db \
   > 'SELECT call_id,command,symbol AS "symbol name            ",call_time,call_time - LAG(call_time,1,0) OVER (ORDER BY id) call_time_lag,
   > return_time,elapsed_time,branch_count,insn_count,cyc_count,IPC,flags
   > FROM calls_view WHERE (command = "dd" AND symbol = "vfs_read") OR
   > symbol IN ("mmc_blk_mq_issue_rq", "mmc_blk_mq_complete", "__pm_runtime_resume", "sdhci_pci_runtime_resume", "sdhci_pci_runtime_suspend") ORDER BY call_time;' | head -30
   call_id     command          symbol name              call_time     call_time_lag  return_time   elapsed_time  branch_count  insn_count  cyc_count   IPC         flags
   ----------  ---------------  -----------------------  ------------  -------------  ------------  ------------  ------------  ----------  ----------  ----------  ----------
   63682       kworker/3:2-eve  __pm_runtime_resume      408067236586  408067236586   408067237097  511           13            135         1789        0.08
   114034      kworker/3:2-eve  __pm_runtime_resume      410051235334  1983998748     410051235411  77            13            135         245         0.55
   182536      dd               vfs_read                 411032546128  981310794      411032550680  4552          293           2573        8890        0.29
   191117      dd               vfs_read                 411032602702  56574          411032605663  2961          285           2397        5726        0.42
   199364      dd               vfs_read                 411032652150  49448          411032655372  3222          282           2499        6233        0.4
   256863      dd               vfs_read                 411033324366  672216         411035014934  1690568       2456          19464       78746       0.25
   259175      kworker/3:1H-kb  __pm_runtime_resume      411033363271  38905          411034184239  820968        173371        676401      1544084     0.44
   259256      kworker/3:1H-kb  __pm_runtime_resume      411033366884  3613           411034181988  815104        173144        674814      1532805     0.44
   433515      kworker/3:1H-kb  sdhci_pci_runtime_resum  411034083899  717015         411034180654  96755         658           6972        186013      0.04        jump
   434406      kworker/3:1H-kb  __pm_runtime_resume      411034184429  100530         411034184544  115           13            101         222         0.45
   434468      kworker/3:1H-kb  mmc_blk_mq_issue_rq      411034185597  1168           411034195025  9428          117           1005        18042       0.06
   436485      swapper          mmc_blk_mq_complete      411035001225  815628         411035009951  8726          464           3630        16785       0.22        jump
   437997      dd               vfs_read                 411035018460  17235          411035041818  23358         2194          17360       27335       0.64
   440326      kworker/3:1H-kb  __pm_runtime_resume      411035033461  15001          411035033692  231           13            124         444         0.28
   440367      kworker/3:1H-kb  __pm_runtime_resume      411035033929  468            411035034079  150           13            0           0           0.0
   440393      kworker/3:1H-kb  mmc_blk_mq_issue_rq      411035034183  254            411035038508  4325          95            727         1785        0.41
   441291      dd               vfs_read                 411035043009  8826           411035044052  1043          119           1169        2080        0.56
   441496      dd               vfs_read                 411035044594  1585           411035045478  884           119           1169        1722        0.68
   441701      dd               vfs_read                 411035045885  1291           411035343621  297736        3981          36177       30929       1.17
   445677      kworker/3:1H-kb  mmc_blk_mq_issue_rq      411035061376  15491          411035065210  3834          95            848         1079        0.79
   447604      swapper          mmc_blk_mq_complete      411035337501  276125         411035340344  2843          340           2692        5469        0.49        jump
   448875      dd               vfs_read                 411035344722  7221           411035345661  939           119           1169        1846        0.63
   449080      dd               vfs_read                 411035346106  1384           411035347080  974           119           1169        1894        0.62
   449285      dd               vfs_read                 411035347486  1380           411035348326  840           119           1169        1636        0.71
   449490      dd               vfs_read                 411035348733  1247           411035349616  883           119           1169        1724        0.68
   449695      dd               vfs_read                 411035350151  1418           411035350869  718           119           1169        1402        0.83
   449900      dd               vfs_read                 411035351288  1137           411035352137  849           119           1169        1657        0.71
   450105      dd               vfs_read                 411035352538  1250           411035353358  820           119           1169        1601        0.73

Yes, that shows that nearly all the extra latency before the I/O is
issued is due to runtime pm.

More analysis would be needed to determine whether the rest of the 1.7
ms is due to the eMMC or interrupt latency.

For now, we will just look at the time between the hardware interrupt,
and the driver interrupt handler. For this we need to know that the
interrupt handler is called sdhci_irq and that it was running on CPU 1.

Below we can see that the first interrupt at 411034993801 is between the
first mmc_blk_mq_issue_rq (411034185597) and first mmc_blk_mq_complete
(411035001225). The previous hardware interrupt is 20 us earlier, which
is a surprisingly long time, but still much less than the I/O time of
815us.

::

   $ sqlite3 -column -header t4.db \
   > 'SELECT id,cpu,time,time - LAG (time,1,0) OVER (ORDER BY id) time_lag,symbol,sym_offset,to_symbol,to_sym_offset,branch_type_name FROM samples_view
   > WHERE (cpu = 1) AND ( ( to_symbol = "sdhci_irq" AND to_sym_offset = 0 ) OR (branch_type_name = "hardware interrupt")) ;' | head -30
   id          cpu         time          time_lag      symbol                sym_offset  to_symbol             to_sym_offset  branch_type_name
   ----------  ----------  ------------  ------------  --------------------  ----------  --------------------  -------------  ------------------
   55161       1           407843243644  407843243644  tick_nohz_idle_enter  51          reschedule_interrupt  0              hardware interrupt
   63715       1           408067314572  224070928     mwait_idle            143         irq_entries_start     40             hardware interrupt
   72957       1           408347228661  279914089     mwait_idle            143         apic_timer_interrupt  0              hardware interrupt
   110047      1           409843244054  1496015393    __sched_text_start    643         reschedule_interrupt  0              hardware interrupt
   114067      1           410051312542  208068488     mwait_idle            143         irq_entries_start     40             hardware interrupt
   116528      1           410347228668  295916126     mwait_idle            143         apic_timer_interrupt  0              hardware interrupt
   436300      1           411034973294  687744626     mwait_idle            143         irq_entries_start     16             hardware interrupt
   436388      1           411034993801  20507         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   447419      1           411035311825  318024        mwait_idle            143         irq_entries_start     16             hardware interrupt
   447507      1           411035332130  20305         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   457368      1           411035685301  353171        mwait_idle            143         irq_entries_start     16             hardware interrupt
   457456      1           411035705179  19878         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   473827      1           411036260355  555176        mwait_idle            143         irq_entries_start     16             hardware interrupt
   473915      1           411036279988  19633         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   489031      1           411036835523  555535        mwait_idle            143         irq_entries_start     16             hardware interrupt
   489119      1           411036855487  19964         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   505392      1           411037410996  555509        mwait_idle            143         irq_entries_start     16             hardware interrupt
   505480      1           411037430723  19727         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   520611      1           411037986623  555900        mwait_idle            143         irq_entries_start     16             hardware interrupt
   520699      1           411038006568  19945         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   536927      1           411038562401  555833        mwait_idle            143         irq_entries_start     16             hardware interrupt
   537015      1           411038582223  19822         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   552098      1           411039137734  555511        mwait_idle            143         irq_entries_start     16             hardware interrupt
   552186      1           411039157598  19864         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   568254      1           411039712835  555237        mwait_idle            143         irq_entries_start     16             hardware interrupt
   568342      1           411039732767  19932         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump
   586569      1           411040288423  555656        mwait_idle            143         irq_entries_start     16             hardware interrupt
   586657      1           411040307873  19450         __x86_indirect_thunk  0           sdhci_irq             0              unconditional jump

We can look at the 20 us between the hardware interrupt and sdhci_irq.
It shows that there are 2 other interrupt handlers being serviced that
consume most of that time: idma64_irq and i801_isr::

   $ perf script -i t4 --itrace=b --time 411.034973294,411.034993801 -F-comm,-tid,-period,-cpu,-event,+flags,+callindent,+addr,-dso --ns -C 1 | \
   > grep -v " jcc " | \
   > awk -F: 'BEGIN {p=0} {ns=1000000000*$1;if (!p) p=ns;printf("%10.0f %s\n",ns-p,$0);p=ns}'
            0   411.034973294:   hw int                  irq_entries_start            ffffffffa66db33f mwait_idle+0x8f => ffffffffa6800220 irq_entries_start+0x10
            0   411.034973294:   jmp                                                  ffffffffa6800222 irq_entries_start+0x12 => ffffffffa6800a00 common_interrupt+0x0
            0   411.034973294:   call                        interrupt_entry          ffffffffa6800a05 common_interrupt+0x5 => ffffffffa6800910 interrupt_entry+0x0
           37   411.034973331:   return                      interrupt_entry          ffffffffa68009cf interrupt_entry+0xbf => ffffffffa6800a0a common_interrupt+0xa
            0   411.034973331:   call                        do_IRQ                   ffffffffa6800a0a common_interrupt+0xa => ffffffffa68017e0 do_IRQ+0x0
            0   411.034973331:   call                            irq_enter            ffffffffa68017fa do_IRQ+0x1a => ffffffffa5c6ddd0 irq_enter+0x0
            0   411.034973331:   call                                rcu_irq_enter    ffffffffa5c6ddd0 irq_enter+0x0 => ffffffffa5cd81a0 rcu_irq_enter+0x0
            0   411.034973331:   call                                    rcu_dynticks_eqs_exit                        ffffffffa5cd826e rcu_irq_enter+0xce => ffffffffa5cd21d0 rcu_dynticks_eqs_exit+0x0
           80   411.034973411:   return                                  rcu_dynticks_eqs_exit                        ffffffffa5cd21f9 rcu_dynticks_eqs_exit+0x29 => ffffffffa5cd8273 rcu_irq_enter+0xd3
            0   411.034973411:   jmp                                                                                  ffffffffa5cd8295 rcu_irq_enter+0xf5 => ffffffffa5cd8201 rcu_irq_enter+0x61
            0   411.034973411:   return                              rcu_irq_enter                                    ffffffffa5cd821f rcu_irq_enter+0x7f => ffffffffa5c6ddd5 irq_enter+0x5
            0   411.034973411:   call                                tick_irq_enter                                   ffffffffa5c6de09 irq_enter+0x39 => ffffffffa5cf2050 tick_irq_enter+0x0
            0   411.034973411:   call                                    tick_check_oneshot_broadcast_this_cpu        ffffffffa5cf205c tick_irq_enter+0xc => ffffffffa5cf0ec0 tick_check_oneshot_broadcast_this_cpu+0x0
           88   411.034973499:   return                                  tick_check_oneshot_broadcast_this_cpu        ffffffffa5cf0ee8 tick_check_oneshot_broadcast_this_cpu+0x28 => ffffffffa5cf2061 tick_irq_enter+0x11
            0   411.034973499:   call                                    ktime_get                                    ffffffffa5cf2075 tick_irq_enter+0x25 => ffffffffa5ce46d0 ktime_get+0x0
            0   411.034973499:   call                                        __x86_indirect_thunk_rax                 ffffffffa5ce4704 ktime_get+0x34 => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
            4   411.034973503:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa5c27580 read_tsc+0x0
           21   411.034973524:   return                                      read_tsc                                 ffffffffa5c2758c read_tsc+0xc => ffffffffa5ce4709 ktime_get+0x39
            0   411.034973524:   return                                  ktime_get                                    ffffffffa5ce4754 ktime_get+0x84 => ffffffffa5cf207a tick_irq_enter+0x2a
            0   411.034973524:   call                                    update_ts_time_stats                         ffffffffa5cf20cc tick_irq_enter+0x7c => ffffffffa5cf1430 update_ts_time_stats+0x0
            0   411.034973524:   call                                        nr_iowait_cpu                            ffffffffa5cf1479 update_ts_time_stats+0x49 => ffffffffa5c98170 nr_iowait_cpu+0x0
            0   411.034973524:   return                                      nr_iowait_cpu                            ffffffffa5c98189 nr_iowait_cpu+0x19 => ffffffffa5cf147e update_ts_time_stats+0x4e
           12   411.034973536:   jmp                                                                                  ffffffffa5cf1497 update_ts_time_stats+0x67 => ffffffffa5cf143f update_ts_time_stats+0xf
            0   411.034973536:   return                                  update_ts_time_stats                         ffffffffa5cf1468 update_ts_time_stats+0x38 => ffffffffa5cf20d1 tick_irq_enter+0x81
            0   411.034973536:   call                                    sched_clock_idle_wakeup_event                ffffffffa5cf20d5 tick_irq_enter+0x85 => ffffffffa5c9a220 sched_clock_idle_wakeup_event+0x0
            0   411.034973536:   return                                  sched_clock_idle_wakeup_event                ffffffffa5c9a227 sched_clock_idle_wakeup_event+0x7 => ffffffffa5cf20da tick_irq_enter+0x8a
            0   411.034973536:   jmp                                                                                  ffffffffa5cf20e2 tick_irq_enter+0x92 => ffffffffa5cf2083 tick_irq_enter+0x33
           37   411.034973573:   return                              tick_irq_enter                                   ffffffffa5cf20b8 tick_irq_enter+0x68 => ffffffffa5c6de0e irq_enter+0x3e
            0   411.034973573:   call                                _local_bh_enable                                 ffffffffa5c6de0e irq_enter+0x3e => ffffffffa5c6d760 _local_bh_enable+0x0
            0   411.034973573:   return                              _local_bh_enable                                 ffffffffa5c6d779 _local_bh_enable+0x19 => ffffffffa5c6de13 irq_enter+0x43
            0   411.034973573:   jmp                                                                                  ffffffffa5c6de13 irq_enter+0x43 => ffffffffa5c6ddf2 irq_enter+0x22
            0   411.034973573:   return                          irq_enter                                            ffffffffa5c6ddfd irq_enter+0x2d => ffffffffa68017ff do_IRQ+0x1f
            0   411.034973573:   call                            __x86_indirect_thunk_rax                             ffffffffa680181c do_IRQ+0x3c => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
          193   411.034973766:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa5cc6cf0 handle_fasteoi_irq+0x0
            0   411.034973766:   call                                _raw_spin_lock                                   ffffffffa5cc6d00 handle_fasteoi_irq+0x10 => ffffffffa66dbbd0 _raw_spin_lock+0x0
          183   411.034973949:   return                              _raw_spin_lock                                   ffffffffa66dbbdd _raw_spin_lock+0xd => ffffffffa5cc6d05 handle_fasteoi_irq+0x15
            0   411.034973949:   call                                irq_may_run                                      ffffffffa5cc6d08 handle_fasteoi_irq+0x18 => ffffffffa5cc6a70 irq_may_run+0x0
            0   411.034973949:   return                              irq_may_run                                      ffffffffa5cc6a81 irq_may_run+0x11 => ffffffffa5cc6d0d handle_fasteoi_irq+0x1d
           16   411.034973965:   call                                handle_irq_event                                 ffffffffa5cc6d5c handle_fasteoi_irq+0x6c => ffffffffa5cc3090 handle_irq_event+0x0
            0   411.034973965:   call                                    handle_irq_event_percpu                      ffffffffa5cc30ad handle_irq_event+0x1d => ffffffffa5cc3020 handle_irq_event_percpu+0x0
            0   411.034973965:   call                                        __handle_irq_event_percpu                ffffffffa5cc3046 handle_irq_event_percpu+0x26 => ffffffffa5cc2ea0 __handle_irq_event_percpu+0x0
            0   411.034973965:   call                                            __x86_indirect_thunk_rax             ffffffffa5cc2ed6 __handle_irq_event_percpu+0x36 => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
          158   411.034974123:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa60d0270 idma64_irq+0x0
         8285   411.034982408:   call                                                _raw_spin_lock                   ffffffffa60d0339 idma64_irq+0xc9 => ffffffffa66dbbd0 _raw_spin_lock+0x0
            0   411.034982408:   return                                              _raw_spin_lock                   ffffffffa66dbbdd _raw_spin_lock+0xd => ffffffffa60d033e idma64_irq+0xce
            0   411.034982408:   call                                                _raw_spin_lock                   ffffffffa60d0339 idma64_irq+0xc9 => ffffffffa66dbbd0 _raw_spin_lock+0x0
         3358   411.034985766:   return                                              _raw_spin_lock                   ffffffffa66dbbdd _raw_spin_lock+0xd => ffffffffa60d033e idma64_irq+0xce
            0   411.034985766:   return                                          idma64_irq                           ffffffffa60d02d4 idma64_irq+0x64 => ffffffffa5cc2edb __handle_irq_event_percpu+0x3b
          112   411.034985878:   call                                            __x86_indirect_thunk_rax             ffffffffa5cc2ed6 __handle_irq_event_percpu+0x36 => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
            1   411.034985879:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa6377370 i801_isr+0x0
            0   411.034985879:   call                                                pci_read_config_word             ffffffffa637739f i801_isr+0x2f => ffffffffa6036a70 pci_read_config_word+0x0
          191   411.034986070:   jmp                                                                                  ffffffffa6036a87 pci_read_config_word+0x17 => ffffffffa6035cb0 pci_bus_read_config_word+0x0
            0   411.034986070:   call                                                    __x86_indirect_thunk_rax     ffffffffa6035cea pci_bus_read_config_word+0x3a => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
           21   411.034986091:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa6454cc0 pci_read+0x0
            0   411.034986091:   jmp                                                                                  ffffffffa6454cde pci_read+0x1e => ffffffffa6454c80 raw_pci_read+0x0
            2   411.034986093:   jmp                                                                                  ffffffffa6454c9b raw_pci_read+0x1b => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
            2   411.034986095:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa6452220 pci_conf1_read+0x0
            0   411.034986095:   call                                                        _raw_spin_lock_irqsave   ffffffffa645226d pci_conf1_read+0x4d => ffffffffa66dbb40 _raw_spin_lock_irqsave+0x0
            0   411.034986095:   return                                                      _raw_spin_lock_irqsave   ffffffffa66dbb55 _raw_spin_lock_irqsave+0x15 => ffffffffa6452272 pci_conf1_read+0x52
          191   411.034986286:   jmp                                                                                  ffffffffa64522fa pci_conf1_read+0xda => ffffffffa64522c5 pci_conf1_read+0xa5
            0   411.034986286:   call                                                        __lock_text_start        ffffffffa64522cc pci_conf1_read+0xac => ffffffffa66db980 __lock_text_start+0x0
            0   411.034986286:   return                                                      __lock_text_start        ffffffffa66db985 __lock_text_start+0x5 => ffffffffa64522d1 pci_conf1_read+0xb1
            0   411.034986286:   return                                                  pci_conf1_read               ffffffffa64522db pci_conf1_read+0xbb => ffffffffa6035cef pci_bus_read_config_word+0x3f
            0   411.034986286:   return                                              pci_bus_read_config_word         ffffffffa6035d0b pci_bus_read_config_word+0x5b => ffffffffa63773a4 i801_isr+0x34
         7363   411.034993649:   return                                          i801_isr                             ffffffffa63773f5 i801_isr+0x85 => ffffffffa5cc2edb __handle_irq_event_percpu+0x3b
            0   411.034993649:   call                                            __x86_indirect_thunk_rax             ffffffffa5cc2ed6 __handle_irq_event_percpu+0x36 => ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0
          152   411.034993801:   jmp                                                                                  ffffffffa6a00fc0 __x86_indirect_thunk_rax+0x0 => ffffffffa63e14a0 sdhci_irq+0x0
            0   411.034993801:   call                                                _raw_spin_lock                   ffffffffa63e14da sdhci_irq+0x3a => ffffffffa66dbbd0 _raw_spin_lock+0x0

Example: Looking at Intel PT trace packets
------------------------------------------

There are 2 ways to see the trace packets. To begin, for this example,
we can make a trivial trace::

   $ perf record -e intel_pt//u uname
   Linux
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.026 MB perf.data ]

The first way is to use the ``perf script`` option to display a dump of
trace data (``-D`` or ``--dump-raw-trace``). For example, to
show the first 25 lines of PERF_RECORD_AUXTRACE\* events::

   $ perf script -D | grep -A25 PERF_RECORD_AUXTRACE
   0 0 0x2e0 [0x98]: PERF_RECORD_AUXTRACE_INFO type: 1
     PMU Type            8
     Time Shift          31
     Time Muliplier      791841207
     Time Zero           18446744057222810652
     Cap Time Zero       1
     TSC bit             0x400
     NoRETComp bit       0x800
     Have sched_switch   3
     Snapshot mode       0
     Per-cpu maps        1
     MTC bit             0x200
     TSC:CTC numerator   226
     TSC:CTC denominator 2
     CYC bit             0x2
     Max non-turbo ratio 27
     Filter string len.  0
     Filter string

   0x378 [0x28]: event: 73
   .
   . ... raw event: size 40 bytes
   .  0000:  49 00 00 00 00 00 28 00 01 00 00 00 00 00 00 00  I.....(.........
   .  0010:  08 38 00 00 00 00 00 00 00 00 00 00 00 00 00 00  .8..............
   .  0020:  00 00 00 00 00 00 00 00                          ........

   --
   0 0 0x1568 [0x30]: PERF_RECORD_AUXTRACE size: 0x5390  offset: 0  ref: 0x51f87fa233f  idx: 2  tid: 14344  cpu: 2
   .
   . ... Intel Processor Trace data: size 21392 bytes
   .  00000000:  02 82 02 82 02 82 02 82 02 82 02 82 02 82 02 82 PSB
   .  00000010:  00 00 00 00 00 00                               PAD
   .  00000016:  19 1d fe e8 87 1f 05 00                         TSC 0x51f87e8fe1d
   .  0000001e:  00 00 00 00 00 00 00 00                         PAD
   .  00000026:  02 73 d9 7e 00 54 00 00                         TMA CTC 0x7ed9 FC 0x54
   .  0000002e:  00 00                                           PAD
   .  00000030:  02 03 2a 00                                     CBR 0x2a
   .  00000034:  02 23                                           PSBEND
   .  00000036:  59 dc                                           MTC 0xdc
   .  00000038:  59 dd                                           MTC 0xdd
   .  0000003a:  59 de                                           MTC 0xde
   .  0000003c:  59 df                                           MTC 0xdf
   .  0000003e:  59 e0                                           MTC 0xe0
   .  00000040:  59 e1                                           MTC 0xe1
   .  00000042:  59 e2                                           MTC 0xe2
   .  00000044:  59 e3                                           MTC 0xe3
   .  00000046:  59 e4                                           MTC 0xe4
   .  00000048:  59 e5                                           MTC 0xe5
   .  0000004a:  59 e6                                           MTC 0xe6
   .  0000004c:  59 e7                                           MTC 0xe7
   .  0000004e:  59 e8                                           MTC 0xe8
   .  00000050:  59 e9                                           MTC 0xe9
   .  00000052:  59 ea                                           MTC 0xea

The second way is to use ``perf script`` option to create a decoder
debug log (``--itrace=d``). This will create a file named
``intel_pt.log`` (beware it will overwrite any previous file)::

   $ perf script --itrace=d
   $ cat intel_pt.log | head -60
   TSC frequency 2712013000
   Maximum non-turbo ratio 27
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc0374f16 len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 22
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc0384224 len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 23
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc03865f9 len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 24
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc0440ba9 len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 25
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc045239d len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 26
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc0454f83 len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 27
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc068eae8 len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 28
   event 17: cpu 0 time 0 tsc 0 PERF_RECORD_KSYMBOL addr ffffffffc06901ce len 61 type 1 flags 0x0 name bpf_prog_6deef7357e7b4530
   event 18: cpu 0 time 0 tsc 0 PERF_RECORD_BPF_EVENT type 1, flags 0, id 29
   event 19: cpu 0 time 0 tsc 0 PERF_RECORD_CGROUP cgroup: 4294967297 /
   event 3: cpu 0 time 0 tsc 0 PERF_RECORD_COMM: perf:14344/14344
   timestamp: mtc_shift 3
   timestamp: tsc_ctc_ratio_n 226
   timestamp: tsc_ctc_ratio_d 2
   timestamp: tsc_ctc_mult 113
   timestamp: tsc_slip 0x10000
   queue 2 getting timestamp
   queue 2 decoding cpu 2 pid -1 tid 14344
   Scanning for PSB
   Getting more data
   Reference timestamp 0x51f87fa233f
   Scanning for PSB
     00000000:  02 82 02 82 02 82 02 82 02 82 02 82 02 82 02 82 PSB
     00000010:  00 00 00 00 00 00                               PAD
     00000016:  19 1d fe e8 87 1f 05 00                         TSC 0x51f87e8fe1d
   Setting timestamp to 0x51f87e8fe1d
     0000001e:  00 00 00 00 00 00 00 00                         PAD
     00000026:  02 73 d9 7e 00 54 00 00                         TMA CTC 0x7ed9 FC 0x54
   CTC timestamp 0x51f87e8fd58 last MTC 0xdb  CTC rem 0x1
     0000002e:  00 00                                           PAD
     00000030:  02 03 2a 00                                     CBR 0x2a
     00000034:  02 23                                           PSBEND
   Scanning for full IP
     00000036:  59 dc                                           MTC 0xdc
   Setting timestamp to 0x51f87e900e0
     00000038:  59 dd                                           MTC 0xdd
   Setting timestamp to 0x51f87e90468
     0000003a:  59 de                                           MTC 0xde
   Setting timestamp to 0x51f87e907f0
     0000003c:  59 df                                           MTC 0xdf
   Setting timestamp to 0x51f87e90b78
     0000003e:  59 e0                                           MTC 0xe0
   Setting timestamp to 0x51f87e90f00
     00000040:  59 e1                                           MTC 0xe1
   Setting timestamp to 0x51f87e91288
     00000042:  59 e2                                           MTC 0xe2
   Setting timestamp to 0x51f87e91610
     00000044:  59 e3                                           MTC 0xe3
   Setting timestamp to 0x51f87e91998
     00000046:  59 e4                                           MTC 0xe4
   Setting timestamp to 0x51f87e91d20

The debug log can be very big, but it can be reduced in size by setting
time ranges (``--time`` option) or specifying CPUs (``--cpu``).
When tracing with per-cpu contexts (which is the default), the debug log
is much easier to understand if it is limited to one CPU. The example
below shows how a 2G log can be trimmed to 681K when reduced to one CPU
and a 1 ms time range, and with sideband events stripped by ``grep
-v``::

   $ sudo ~/bin/perf record -a -e intel_pt//u -- sleep 1
   [ perf record: Woken up 3 times to write data ]
   [ perf record: Captured and wrote 12.302 MB perf.data ]
   $ sudo chgrp perf_users perf.data
   $ sudo chmod g+r perf.data
   $ perf script --itrace=d
   $ ls -lh intel_pt.log
   -rw-rw-r-- 1 user user 2.0G Jun 16 10:22 intel_pt.log
   $ perf script --itrace=d --cpu 0
   $ ls -lh intel_pt.log
   -rw-rw-r-- 1 user user 109M Jun 16 10:22 intel_pt.log
   $ perf script --itrace=i10ms
           kwin_x11  1042 [007]  4760.598466:     922319 instructions:u:      7f54fdd1b090 QTextEngine::itemize+0x690 (/usr/lib/x86_64-linux-gnu/libQt5Gui.so.5.12.8)
            konsole 13784 [002]  4760.617025:    4048473 instructions:u:      7f34f0282003 QString::reallocData+0x93 (/usr/lib/x86_64-linux-gnu/libQt5Core.so.5.12.8)
               Xorg   875 [003]  4760.796426:   23859371 instructions:u:      562d6a00ab7b [unknown] (/usr/lib/xorg/Xorg)
        plasmashell  1048 [001]  4761.155230:   15828349 instructions:u:      7f2da6ead8bb __powf_fma+0x4b (/usr/lib/x86_64-linux-gnu/libm-2.31.so)
           kwin_x11  1042 [002]  4761.248024:   16194616 instructions:u:      7f54e56673e3 [unknown] (/usr/lib/x86_64-linux-gnu/dri/iris_dri.so)
   $ perf script --itrace=d --cpu 0 --time 4760.598,4760.599
   $ ls -lh intel_pt.log
   -rw-rw-r-- 1 user user 4.7M Jun 16 10:27 intel_pt.log
   $ grep -v "PERF_RECORD\|context_switch" intel_pt.log > intel_pt.log-no-sideband
   $ ls -lh intel_pt.log-no-sideband
   -rw-rw-r-- 1 user user 681K Jun 16 10:27 intel_pt.log-no-sideband

Example: Unknown symbols
------------------------

The current version of ``perf`` (v5.8-rc1) shows unknown symbols::

   $ perf record -e intel_pt//u uname
   Linux
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.027 MB perf.data ]
   $ perf script --itrace=b -Fip,sym,dso | grep unknown | sort -u
                   0 [unknown] ([unknown])
        55e2455b7304 [unknown] (/usr/bin/uname)
        55e2455b7364 [unknown] (/usr/bin/uname)
        55e2455b7374 [unknown] (/usr/bin/uname)
        55e2455b7384 [unknown] (/usr/bin/uname)
        55e2455b7394 [unknown] (/usr/bin/uname)
        55e2455b73e4 [unknown] (/usr/bin/uname)
        55e2455b7404 [unknown] (/usr/bin/uname)
        55e2455b7414 [unknown] (/usr/bin/uname)
        55e2455b7424 [unknown] (/usr/bin/uname)
        55e2455b7464 [unknown] (/usr/bin/uname)
        55e2455b74a4 [unknown] (/usr/bin/uname)
        55e2455b74d4 [unknown] (/usr/bin/uname)
        55e2455b74f4 [unknown] (/usr/bin/uname)
        55e2455b7514 [unknown] (/usr/bin/uname)
        55e2455b7564 [unknown] (/usr/bin/uname)
        7f4661556006 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556314 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556334 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556394 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f46615563b4 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556424 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556454 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556464 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556474 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f46615564d4 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556514 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556574 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f4661556584 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f46615565a4 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f46615565e4 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
        7f466173f084 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
        7f466173f094 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
        7f466173f0a4 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
        7f466173f0c4 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
   $ perf script --itrace=b -Faddr,sym,dso | grep unknown | sort -u
    =>                0 [unknown] ([unknown])
    =>     55e2455b7300 [unknown] (/usr/bin/uname)
    =>     55e2455b7360 [unknown] (/usr/bin/uname)
    =>     55e2455b7370 [unknown] (/usr/bin/uname)
    =>     55e2455b7380 [unknown] (/usr/bin/uname)
    =>     55e2455b7390 [unknown] (/usr/bin/uname)
    =>     55e2455b73e0 [unknown] (/usr/bin/uname)
    =>     55e2455b7400 [unknown] (/usr/bin/uname)
    =>     55e2455b7410 [unknown] (/usr/bin/uname)
    =>     55e2455b7420 [unknown] (/usr/bin/uname)
    =>     55e2455b7460 [unknown] (/usr/bin/uname)
    =>     55e2455b74a0 [unknown] (/usr/bin/uname)
    =>     55e2455b74d0 [unknown] (/usr/bin/uname)
    =>     55e2455b74f0 [unknown] (/usr/bin/uname)
    =>     55e2455b7510 [unknown] (/usr/bin/uname)
    =>     55e2455b7560 [unknown] (/usr/bin/uname)
    =>     7f4661556000 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556310 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556330 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556390 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f46615563b0 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556420 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556450 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556460 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556470 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f46615564d0 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556510 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556570 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f4661556580 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f46615565a0 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f46615565e0 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
    =>     7f466173f080 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
    =>     7f466173f090 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
    =>     7f466173f0a0 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
    =>     7f466173f0c0 [unknown] (/usr/lib/x86_64-linux-gnu/ld-2.31.so)

To find out what they are we need to look at MMAP events::

   $ perf script --no-itrace --show-mmap-events
              uname  2385 [007]   291.223573: PERF_RECORD_MMAP2 2385/2385: [0x55e2455b7000(0x4000) @ 0x2000 08:02 4326740 2619663641]: r-xp /usr/bin/uname
              uname  2385 [007]   291.223584: PERF_RECORD_MMAP2 2385/2385: [0x7f466173f000(0x23000) @ 0x1000 08:02 4332741 3243916115]: r-xp /usr/lib/x86_64-linux-gnu/ld-2.31.so
              uname  2385 [007]   291.223591: PERF_RECORD_MMAP2 2385/2385: [0x7ffd1ef71000(0x1000) @ 0 00:00 0 0]: r-xp [vdso]
              uname  2385 [007]   291.223674: PERF_RECORD_MMAP2 2385/2385: [0x7f4661556000(0x178000) @ 0x25000 08:02 4333341 2513837035]: r-xp /usr/lib/x86_64-linux-gnu/libc-2.31.so

The details of each MMAP event for a process (PID 2385 in this case) is
in the form ``[start(size) @ offset device inode generation]: protection
pathname``. To calculate a file offset from an address::

   file_offset = address - start + offset

Unknown addresses in the range from 0x55e2455b7300 to 0x55e2455b7564 are
from /usr/bin/uname, from file offset 0x2300 to 0x2564. We can look at
the section headers to find out more::

   $ readelf -S -W /usr/bin/uname
   There are 30 section headers, starting at offset 0x91f8:

   Section Headers:
     [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
     [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
     [ 1] .interp           PROGBITS        0000000000000318 000318 00001c 00   A  0   0  1
     [ 2] .note.gnu.property NOTE            0000000000000338 000338 000020 00   A  0   0  8
     [ 3] .note.gnu.build-id NOTE            0000000000000358 000358 000024 00   A  0   0  4
     [ 4] .note.ABI-tag     NOTE            000000000000037c 00037c 000020 00   A  0   0  4
     [ 5] .gnu.hash         GNU_HASH        00000000000003a0 0003a0 0000a8 00   A  6   0  8
     [ 6] .dynsym           DYNSYM          0000000000000448 000448 000648 18   A  7   1  8
     [ 7] .dynstr           STRTAB          0000000000000a90 000a90 00033a 00   A  0   0  1
     [ 8] .gnu.version      VERSYM          0000000000000dca 000dca 000086 02   A  6   0  2
     [ 9] .gnu.version_r    VERNEED         0000000000000e50 000e50 000060 00   A  7   1  8
     [10] .rela.dyn         RELA            0000000000000eb0 000eb0 0003d8 18   A  6   0  8
     [11] .rela.plt         RELA            0000000000001288 001288 000438 18  AI  6  25  8
     [12] .init             PROGBITS        0000000000002000 002000 00001b 00  AX  0   0  4
     [13] .plt              PROGBITS        0000000000002020 002020 0002e0 10  AX  0   0 16
     [14] .plt.got          PROGBITS        0000000000002300 002300 000010 10  AX  0   0 16
     [15] .plt.sec          PROGBITS        0000000000002310 002310 0002d0 10  AX  0   0 16
     [16] .text             PROGBITS        00000000000025e0 0025e0 003492 00  AX  0   0 16
     [17] .fini             PROGBITS        0000000000005a74 005a74 00000d 00  AX  0   0  4
     [18] .rodata           PROGBITS        0000000000006000 006000 00114c 00   A  0   0 32
     [19] .eh_frame_hdr     PROGBITS        000000000000714c 00714c 00028c 00   A  0   0  4
     [20] .eh_frame         PROGBITS        00000000000073d8 0073d8 000be8 00   A  0   0  8
     [21] .init_array       INIT_ARRAY      00000000000099d0 0089d0 000008 08  WA  0   0  8
     [22] .fini_array       FINI_ARRAY      00000000000099d8 0089d8 000008 08  WA  0   0  8
     [23] .data.rel.ro      PROGBITS        00000000000099e0 0089e0 000278 00  WA  0   0 32
     [24] .dynamic          DYNAMIC         0000000000009c58 008c58 0001f0 10  WA  7   0  8
     [25] .got              PROGBITS        0000000000009e48 008e48 0001a8 08  WA  0   0  8
     [26] .data             PROGBITS        000000000000a000 009000 0000a0 00  WA  0   0 32
     [27] .bss              NOBITS          000000000000a0a0 0090a0 000198 00  WA  0   0 32
     [28] .gnu_debuglink    PROGBITS        0000000000000000 0090a0 000034 00      0   0  4
     [29] .shstrtab         STRTAB          0000000000000000 0090d4 00011d 00      0   0  1
   Key to Flags:
     W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
     L (link order), O (extra OS processing required), G (group), T (TLS),
     C (compressed), x (unknown), o (OS specific), E (exclude),
     l (large), p (processor specific)

We can see the unknown symbols are from the .plt.got and .plt.sec
sections. If we were to look also at
/usr/lib/x86_64-linux-gnu/libc-2.31.so and
/usr/lib/x86_64-linux-gnu/ld-2.31.so, we would see the same thing. That
is, the unknown symbols are from the .plt, .plt.got or .plt.sec sections
of those files. ``perf`` has a function named
dso\__synthesize_plt_symbols() to get PLT symbols, but it gets it wrong.

BFD also provides a function, named bfd_get_synthetic_symtab(), to get
PLT symbols. It is used by ``objdump`` when disassembling. Here are the
PLT symbols from /usr/bin/uname, /usr/lib/x86_64-linux-gnu/ld-2.31.so
and /usr/lib/x86_64-linux-gnu/libc.so.6::

   $ objdump -d /usr/bin/uname -j .plt -j .plt.got -j .plt.sec | grep ^0
   0000000000002020 <.plt>:
   0000000000002300 <__cxa_finalize@plt>:
   0000000000002310 <free@plt>:
   0000000000002320 <abort@plt>:
   0000000000002330 <__errno_location@plt>:
   0000000000002340 <strncmp@plt>:
   0000000000002350 <_exit@plt>:
   0000000000002360 <__fpending@plt>:
   0000000000002370 <textdomain@plt>:
   0000000000002380 <fclose@plt>:
   0000000000002390 <bindtextdomain@plt>:
   00000000000023a0 <dcgettext@plt>:
   00000000000023b0 <__ctype_get_mb_cur_max@plt>:
   00000000000023c0 <strlen@plt>:
   00000000000023d0 <__stack_chk_fail@plt>:
   00000000000023e0 <getopt_long@plt>:
   00000000000023f0 <mbrtowc@plt>:
   0000000000002400 <__overflow@plt>:
   0000000000002410 <strrchr@plt>:
   0000000000002420 <uname@plt>:
   0000000000002430 <lseek@plt>:
   0000000000002440 <memset@plt>:
   0000000000002450 <memcmp@plt>:
   0000000000002460 <fputs_unlocked@plt>:
   0000000000002470 <calloc@plt>:
   0000000000002480 <strcmp@plt>:
   0000000000002490 <memcpy@plt>:
   00000000000024a0 <fileno@plt>:
   00000000000024b0 <fgets_unlocked@plt>:
   00000000000024c0 <malloc@plt>:
   00000000000024d0 <fflush@plt>:
   00000000000024e0 <nl_langinfo@plt>:
   00000000000024f0 <__freading@plt>:
   0000000000002500 <realloc@plt>:
   0000000000002510 <setlocale@plt>:
   0000000000002520 <__printf_chk@plt>:
   0000000000002530 <error@plt>:
   0000000000002540 <fseeko@plt>:
   0000000000002550 <fopen@plt>:
   0000000000002560 <__cxa_atexit@plt>:
   0000000000002570 <exit@plt>:
   0000000000002580 <fwrite@plt>:
   0000000000002590 <__fprintf_chk@plt>:
   00000000000025a0 <mbsinit@plt>:
   00000000000025b0 <iswprint@plt>:
   00000000000025c0 <strstr@plt>:
   00000000000025d0 <__ctype_b_loc@plt>:
   $ objdump -d /usr/lib/x86_64-linux-gnu/ld-2.31.so -j .plt -j .plt.got -j .plt.sec | grep ^0
   0000000000001000 <.plt>:
   0000000000001080 <free@plt>:
   0000000000001090 <_dl_catch_exception@plt>:
   00000000000010a0 <malloc@plt>:
   00000000000010b0 <_dl_signal_exception@plt>:
   00000000000010c0 <calloc@plt>:
   00000000000010d0 <realloc@plt>:
   00000000000010e0 <_dl_signal_error@plt>:
   00000000000010f0 <_dl_catch_error@plt>:
   $ objdump -d /usr/lib/x86_64-linux-gnu/libc.so.6 -j .plt -j .plt.got -j .plt.sec | grep ^0
   0000000000025000 <.plt>:
   0000000000025300 <__libpthread_freeres@plt>:
   0000000000025310 <malloc@plt>:
   0000000000025320 <__libdl_freeres@plt>:
   0000000000025330 <free@plt>:
   0000000000025340 <*ABS*+0xa3600@plt>:
   0000000000025350 <*ABS*+0xa27f0@plt>:
   0000000000025360 <*ABS*+0xbf960@plt>:
   0000000000025370 <realloc@plt>:
   0000000000025380 <*ABS*+0xa3a20@plt>:
   0000000000025390 <*ABS*+0xa4e10@plt>:
   00000000000253a0 <*ABS*+0xbfea0@plt>:
   00000000000253b0 <*ABS*+0xa3870@plt>:
   00000000000253c0 <__tls_get_addr@plt>:
   00000000000253d0 <*ABS*+0xa38d0@plt>:
   00000000000253e0 <*ABS*+0xa2c60@plt>:
   00000000000253f0 <*ABS*+0xbfab0@plt>:
   0000000000025400 <memalign@plt>:
   0000000000025410 <_dl_exception_create@plt>:
   0000000000025420 <*ABS*+0xa3550@plt>:
   0000000000025430 <*ABS*+0xa39d0@plt>:
   0000000000025440 <*ABS*+0xabd20@plt>:
   0000000000025450 <__tunable_get_val@plt>:
   0000000000025460 <*ABS*+0xa27b0@plt>:
   0000000000025470 <*ABS*+0xa2280@plt>:
   0000000000025480 <*ABS*+0xa29a0@plt>:
   0000000000025490 <*ABS*+0xbf9e0@plt>:
   00000000000254a0 <*ABS*+0xbff30@plt>:
   00000000000254b0 <*ABS*+0xbff30@plt>:
   00000000000254c0 <*ABS*+0xc10d0@plt>:
   00000000000254d0 <*ABS*+0xa3ad0@plt>:
   00000000000254e0 <*ABS*+0xa2350@plt>:
   00000000000254f0 <*ABS*+0xbfe60@plt>:
   0000000000025500 <*ABS*+0xa3980@plt>:
   0000000000025510 <_dl_find_dso_for_object@plt>:
   0000000000025520 <*ABS*+0xa23b0@plt>:
   0000000000025530 <*ABS*+0xa27f0@plt>:
   0000000000025540 <*ABS*+0xbf960@plt>:
   0000000000025550 <calloc@plt>:
   0000000000025560 <*ABS*+0xa36c0@plt>:
   0000000000025570 <*ABS*+0xa22d0@plt>:
   0000000000025580 <*ABS*+0xa2890@plt>:
   0000000000025590 <*ABS*+0xa3590@plt>:
   00000000000255a0 <*ABS*+0xa3760@plt>:
   00000000000255b0 <*ABS*+0xbf9a0@plt>:
   00000000000255c0 <*ABS*+0xbfe60@plt>:
   00000000000255d0 <*ABS*+0xa4dd0@plt>:
   00000000000255e0 <*ABS*+0xa2960@plt>:
   00000000000255f0 <*ABS*+0xa2220@plt>:
   0000000000025600 <*ABS*+0xa3930@plt>:
   0000000000025610 <*ABS*+0xa2900@plt>:
   0000000000025620 <*ABS*+0xa3600@plt>:

Many of the PLT symbol names in libc are not particularly meaningful,
and there are also some missing from 0x7f4661556000 to 0x7f46615562ff
(file offset 0x25000 to 0x25300). However, with a branch trace, we can
see where they are going anyway::

   $ perf script --itrace=be -F+flags,+addr,-period,-event --ns | grep -B 1 -A 2 7f4661556[012]
              uname  2385 [007]   291.223778433:   call                     7f46616932ab _dl_addr+0x3b (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661556510 _dl_find_dso_for_object@plt+0x0 (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
              uname  2385 [007]   291.223778433:   jmp                      7f4661556514 _dl_find_dso_for_object@plt+0x4 (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f46615561e0 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
              uname  2385 [007]   291.223778433:   jmp                      7f46615561e9 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661556000 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
              uname  2385 [007]   291.223778766:   jmp                      7f4661556006 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661756bb0 _dl_runtime_resolve_xsavec+0x0 (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
              uname  2385 [007]   291.223778766:   call                     7f4661756c29 _dl_runtime_resolve_xsavec+0x79 (/usr/lib/x86_64-linux-gnu/ld-2.31.so) =>     7f466174f0b0 _dl_fixup+0x0 (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
              uname  2385 [007]   291.223778766:   call                     7f466174f182 _dl_fixup+0xd2 (/usr/lib/x86_64-linux-gnu/ld-2.31.so) =>     7f466174a0d0 _dl_lookup_symbol_x+0x0 (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
   --
              uname  2385 [007]   291.223797766:   call                     7f46615cd275 ptmalloc_init.part.0+0xa5 (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661556450 __tunable_get_val@plt+0x0 (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
              uname  2385 [007]   291.223797766:   jmp                      7f4661556454 __tunable_get_val@plt+0x4 (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661556120 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
              uname  2385 [007]   291.223797766:   jmp                      7f4661556129 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661556000 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so)
              uname  2385 [007]   291.223797766:   jmp                      7f4661556006 [unknown] (/usr/lib/x86_64-linux-gnu/libc-2.31.so) =>     7f4661756bb0 _dl_runtime_resolve_xsavec+0x0 (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
              uname  2385 [007]   291.223797766:   call                     7f4661756c29 _dl_runtime_resolve_xsavec+0x79 (/usr/lib/x86_64-linux-gnu/ld-2.31.so) =>     7f466174f0b0 _dl_fixup+0x0 (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
              uname  2385 [007]   291.223797766:   call                     7f466174f182 _dl_fixup+0xd2 (/usr/lib/x86_64-linux-gnu/ld-2.31.so) =>     7f466174a0d0 _dl_lookup_symbol_x+0x0 (/usr/lib/x86_64-linux-gnu/ld-2.31.so)
   $

Example: Tracing suspend and resume
-----------------------------------

Tracing suspend and resume can be problematic. For example, with a NUC
runing Ubuntu 20.04 the following:

- during suspend, non-boot CPUs (CPUs 1 to 7) are disabled and Intel PT
  is stopped, but not restarted during resume
- on CPU 0, while suspended, Intel PT is reset and also not restarted
- TSC is reset, which would mess up the timing information, even if
  Intel PT had not been disabled

The NUC and OS support "freeze" (suspend-to-idle) which does not have
the issues above, however another alternative is to use `pm_test
<http://www.kernel.org/doc/html/latest/power/basic-pm-debugging.html>`__
which is what we will do in this example.

Before tracing, we can do a trick to reduce some trace errors. Because
there can be JIT-compiled eBPF in between modules, we need to load
another module to delineate the used memory. A good choice is zfs
because it is not already loaded and it is big so it will be added at
the end.

::

   $ sudo cat /proc/kallsyms | sort | tail
   ffffffffc0f7e1b8 d __UNIQUE_ID_ddebug323.71328  [rfcomm]
   ffffffffc0f7e200 d __this_module        [rfcomm]
   ffffffffc0f7e580 b __key.72085  [rfcomm]
   ffffffffc0f7e580 b rfcomm_dlc_debugfs   [rfcomm]
   ffffffffc0f7e588 b rfcomm_thread        [rfcomm]
   ffffffffc0f7e590 b l2cap_ertm   [rfcomm]
   ffffffffc0f7e591 b disable_cfc  [rfcomm]
   ffffffffc0f7e5a0 b rfcomm_sock_debugfs  [rfcomm]
   ffffffffc0f7e5b0 b rfcomm_sk_list       [rfcomm]
   ffffffffc0f7e5c8 b rfcomm_tty_driver    [rfcomm]
   $ sudo modprobe zfs
   $ sudo cat /proc/kallsyms | sort | tail
   ffffffffc1439980 b zvol_request_sync    [zfs]
   ffffffffc1439984 b zvol_inhibit_dev     [zfs]
   ffffffffc1439988 b __key.65694  [zfs]
   ffffffffc1439988 b __key.65702  [zfs]
   ffffffffc1439988 b __key.65999  [zfs]
   ffffffffc1439990 b zvol_ida     [zfs]
   ffffffffc14399a0 b zvol_htable  [zfs]
   ffffffffc14399c0 b zvol_state_list      [zfs]
   ffffffffc14399e0 b zvol_state_lock      [zfs]
   ffffffffc1439a10 b zvol_taskq   [zfs]

We can enable pm_test as follows::

   $ sudo cat /sys/power/pm_test
   [none] core processors platform devices freezer
   $ sudo bash -c 'echo platform > /sys/power/pm_test'
   $ sudo cat /sys/power/pm_test
   none core processors [platform] devices freezer

This example includes kernel tracing, which requires administrator
privileges.

We can start an open-ended trace, then run the suspend platform test,
then kill ``perf`` to end the trace.

We can use ``perf record`` with options:

- ``-o pt-mem-test`` to name the output pt-mem-test
- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)
- ``-m,128M`` to set the trace buffer size to 128 MiB. This is
  needed to avoid trace data loss. Note the comma is needed. Also **be
  careful setting large buffer sizes**. With per-cpu tracing (the
  default), one buffer per CPU will be allocated. In our case we have 8
  CPUs so that means 1024 MiB. However when tracing with per-task
  contexts, there will be one buffer per task, which might be far more
  than anticipated.
- ``-e`` to select which events, i.e. the following:
- ``intel_pt//k`` to get Intel PT tracing the kernel only

::

   $ sudo ~/bin/perf record -o pt-mem-test -a --kcore -m,128M -e intel_pt//k &
   [1] 2186
   $ sudo bash -c 'echo mem > /sys/power/state'
   $ sudo kill 2186
   $ [ perf record: Woken up 9 times to write data ]
   [ perf record: Captured and wrote 148.521 MB pt-mem-test ]
   [1]+  Terminated              sudo ~/bin/perf record -o pt-mem-test -a --kcore -m,128M -e intel_pt//k

Now, we can disable pm_test as follows::

   $ sudo bash -c 'echo none > /sys/power/pm_test'
   $ sudo cat /sys/power/pm_test
   [none] core processors platform devices freezer

We can allow perf_users to access the trace::

   $ sudo chgrp -R perf_users pt-mem-test
   $ sudo chmod -R g+r pt-mem-test
   $ sudo chmod g+rx pt-mem-test pt-mem-test/kcore_dir

We can check to see how many trace errors there are::

   $ perf script -i pt-mem-test --itrace=e
   Warning:
   29 instruction trace errors
    instruction trace error type 1 time 2854.304038029 cpu 3 pid 0 tid 0 ip 0xffffffffae0001b0 code 10: Never-ending loop
    instruction trace error type 1 time 2854.519908646 cpu 3 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2854.615877328 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2855.951848561 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2855.971847161 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2855.983848121 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2856.391842767 cpu 2 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2856.827831320 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.059827217 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.071827844 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.143826938 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.163825205 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.659815892 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.671817186 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2857.971810190 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.359804569 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.591800132 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.651797933 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.663798894 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.683797160 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.695798454 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.795795122 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2858.987791818 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2859.111791073 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2859.323785703 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2859.435783331 cpu 0 pid 0 tid 0 ip 0xffffffffaded55b6 code 7: Overflow packet
    instruction trace error type 1 time 2859.474475203 cpu 0 pid 0 tid 0 ip 0xffffffffaded55a1 code 7: Overflow packet
    instruction trace error type 1 time 2859.517865726 cpu 6 pid 0 tid 0 ip 0xffffffffad547ec9 code 7: Overflow packet
    instruction trace error type 1 time 2859.519310388 cpu 4 pid 2189 tid 2189 ip 0xffffffffad4d9acc code 7: Overflow packet

Most of the overflows are at 0xffffffffaded55b6, which is in mwait_idle i.e.::

   $ grep ffffffffaded55b6 pt-mem-test/kcore_dir/kallsyms
   $ grep ffffffffaded55b pt-mem-test/kcore_dir/kallsyms
   $ grep ffffffffaded55 pt-mem-test/kcore_dir/kallsyms
   ffffffffaded5530 t mwait_idle
   $ cat pt-mem-test/kcore_dir/kallsyms | sort | grep -A 1 ffffffffaded55
   ffffffffaded5530 t mwait_idle
   ffffffffaded5700 T acpi_processor_ffh_cstate_enter

It is not uncommon to get overflows when transitioning to a C-state, so
these errors are not significant.

Overflows last relatively short periods, and there are very few errors
compared with the size of the trace, so we can ignore them.

To reduce the time ranges that we look at, we can find the time of the
state_store function which got called when we did ``'echo mem >
/sys/power/state'``::

   $ perf script -i pt-mem-test --itrace=be --ns | grep state_store
               bash  2189 [002]  2854.298249382:          1  branches:k:  ffffffffae200cf0 __x86_indirect_thunk_rax+0x10 ([kernel.kallsyms]) => ffffffffad507580 state_store+0x0 ([kernel.kallsyms])
               bash  2189 [002]  2854.298249382:          1  branches:k:  ffffffffad5075ac state_store+0x2c ([kernel.kallsyms]) => ffffffffadeb9480 memchr+0x0 ([kernel.kallsyms])
               bash  2189 [002]  2854.298249715:          1  branches:k:  ffffffffadeb949c memchr+0x1c ([kernel.kallsyms]) => ffffffffad5075b1 state_store+0x31 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075df state_store+0x5f ([kernel.kallsyms]) => ffffffffadeb93a0 strlen+0x0 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffadeb93b4 strlen+0x14 ([kernel.kallsyms]) => ffffffffad5075e4 state_store+0x64 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075f5 state_store+0x75 ([kernel.kallsyms]) => ffffffffad5075c8 state_store+0x48 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075d6 state_store+0x56 ([kernel.kallsyms]) => ffffffffad5075ed state_store+0x6d ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075f5 state_store+0x75 ([kernel.kallsyms]) => ffffffffad5075c8 state_store+0x48 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075df state_store+0x5f ([kernel.kallsyms]) => ffffffffadeb93a0 strlen+0x0 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffadeb93b4 strlen+0x14 ([kernel.kallsyms]) => ffffffffad5075e4 state_store+0x64 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075eb state_store+0x6b ([kernel.kallsyms]) => ffffffffad50761d state_store+0x9d ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad507623 state_store+0xa3 ([kernel.kallsyms]) => ffffffffadeb9600 strncmp+0x0 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffadeb9627 strncmp+0x27 ([kernel.kallsyms]) => ffffffffad507628 state_store+0xa8 ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad507638 state_store+0xb8 ([kernel.kallsyms]) => ffffffffad5075fa state_store+0x7a ([kernel.kallsyms])
               bash  2189 [002]  2854.298250048:          1  branches:k:  ffffffffad5075fd state_store+0x7d ([kernel.kallsyms]) => ffffffffad508bd0 pm_suspend+0x0 ([kernel.kallsyms])
   ^C$

For other time ranges, we can use the kernel messages::

   $ dmesg | tail -60
   [    5.553665] iwlwifi 0000:00:14.3: BIOS contains WGDS but no WRDS
   [    6.820682] Bluetooth: RFCOMM TTY layer initialized
   [    6.820688] Bluetooth: RFCOMM socket layer initialized
   [    6.820691] Bluetooth: RFCOMM ver 1.11
   [   10.290428] e1000e: eno1 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: Rx/Tx
   [   10.290585] IPv6: ADDRCONF(NETDEV_CHANGE): eno1: link becomes ready
   [ 2345.150557] usb 1-2: new low-speed USB device number 4 using xhci_hcd
   [ 2345.305251] usb 1-2: New USB device found, idVendor=1a2c, idProduct=2124, bcdDevice= 1.10
   [ 2345.305255] usb 1-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
   [ 2345.305257] usb 1-2: Product: USB Keyboard
   [ 2345.305258] usb 1-2: Manufacturer: SEM
   [ 2345.309691] input: SEM USB Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-2/1-2:1.0/0003:1A2C:2124.0002/input/input12
   [ 2345.366828] hid-generic 0003:1A2C:2124.0002: input,hidraw1: USB HID v1.10 Keyboard [SEM USB Keyboard] on usb-0000:00:14.0-2/input0
   [ 2345.369299] input: SEM USB Keyboard Consumer Control as /devices/pci0000:00/0000:00:14.0/usb1/1-2/1-2:1.1/0003:1A2C:2124.0003/input/input13
   [ 2345.426807] input: SEM USB Keyboard System Control as /devices/pci0000:00/0000:00:14.0/usb1/1-2/1-2:1.1/0003:1A2C:2124.0003/input/input14
   [ 2345.427152] hid-generic 0003:1A2C:2124.0003: input,hidraw2: USB HID v1.10 Device [SEM USB Keyboard] on usb-0000:00:14.0-2/input1
   [ 2633.489054] zlua: loading out-of-tree module taints kernel.
   [ 2633.489058] zlua: module license 'MIT' taints kernel.
   [ 2633.489059] Disabling lock debugging due to kernel taint
   [ 2635.463839] ZFS: Loaded module v0.8.3-1ubuntu12, ZFS pool version 5000, ZFS filesystem version 5
   [ 2854.298251] PM: suspend entry (deep)
   [ 2854.302058] Filesystems sync: 0.003 seconds
   [ 2854.303263] Freezing user space processes ... (elapsed 0.001 seconds) done.
   [ 2854.305258] OOM killer disabled.
   [ 2854.305259] Freezing remaining freezable tasks ... (elapsed 0.001 seconds) done.
   [ 2854.306481] printk: Suspending console(s) (use no_console_suspend to debug)
   [ 2854.307753] e1000e: EEE TX LPI TIMER: 00000011
   [ 2854.332049] sd 2:0:0:0: [sda] Synchronizing SCSI cache
   [ 2854.332204] sd 2:0:0:0: [sda] Stopping disk
   [ 2854.502891] ACPI: EC: interrupt blocked
   [ 2854.525496] ACPI: Preparing to enter system sleep state S3
   [ 2854.527608] ACPI: EC: event blocked
   [ 2854.527609] ACPI: EC: EC stopped
   [ 2854.527610] PM: Saving platform NVS memory
   [ 2854.527621] PM: suspend debug: Waiting for 5 second(s).
   [ 2859.465807] ACPI: EC: EC started
   [ 2859.465809] ACPI: Waking up from system sleep state S3
   [ 2859.472814] ACPI: EC: interrupt unblocked
   [ 2859.517524] ACPI: EC: event unblocked
   [ 2859.526934] iwlwifi 0000:00:14.3: Applying debug destination EXTERNAL_DRAM
   [ 2859.528470] sd 2:0:0:0: [sda] Starting disk
   [ 2859.675539] iwlwifi 0000:00:14.3: Applying debug destination EXTERNAL_DRAM
   [ 2859.743970] iwlwifi 0000:00:14.3: FW already configured (0) - re-configuring
   [ 2859.754123] iwlwifi 0000:00:14.3: BIOS contains WGDS but no WRDS
   [ 2859.844031] ata3: SATA link up 6.0 Gbps (SStatus 133 SControl 300)
   [ 2859.845676] ata3.00: ACPI cmd ef/10:06:00:00:00:00 (SET FEATURES) succeeded
   [ 2859.845680] ata3.00: ACPI cmd f5/00:00:00:00:00:00 (SECURITY FREEZE LOCK) filtered out
   [ 2859.845682] ata3.00: ACPI cmd b1/c1:00:00:00:00:00 (DEVICE CONFIGURATION OVERLAY) filtered out
   [ 2859.847186] ata3.00: ACPI cmd ef/10:06:00:00:00:00 (SET FEATURES) succeeded
   [ 2859.847189] ata3.00: ACPI cmd f5/00:00:00:00:00:00 (SECURITY FREEZE LOCK) filtered out
   [ 2859.847192] ata3.00: ACPI cmd b1/c1:00:00:00:00:00 (DEVICE CONFIGURATION OVERLAY) filtered out
   [ 2859.847451] ata3.00: configured for UDMA/133
   [ 2859.847457] ahci 0000:00:17.0: port does not support device sleep
   [ 2859.847570] ata3.00: Enabling discard_zeroes_data
   [ 2859.888491] acpi LNXPOWER:04: Turning OFF
   [ 2859.888592] OOM killer enabled.
   [ 2859.888593] Restarting tasks ... done.
   [ 2859.895729] video LNXVIDEO:00: Restoring backlight state
   [ 2859.895741] PM: suspend exit
   [ 2864.837431] e1000e: eno1 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: Rx/Tx

We can use the following times:

- suspend start time 2854.298249382 from time of state_store
- suspend end time 2854.527622 from time of "PM: suspend debug: Waiting
  for 5 second(s)." plus 1 us
- resume start time 2859.465807 from time of "ACPI: EC: EC started"
- resume end time 2859.895742 from time of "PM: suspend exit" plus 1 us

For analysis, we can export to a database, but because we have a lot of
data we will use PostgreSQL because it is faster than SQLite3 for large
data sets.

We can install PostgreSQL and add userid "user" as follows::

   $ sudo apt-get install postgresql
   $ sudo su - postgres
   $ createuser -s user
   $ exit

We can stop PostgreSQL and stop it from starting at boot up::

   $ sudo systemctl stop postgresql
   $ sudo systemctl disable postgresql

Now, we can manually start PostgreSQL when we need it::

   $ sudo systemctl start postgresql

We can make one database for the suspend::

   $ perf script -i pt-mem-test --itrace=bp --time 2854.298249382,2854.527622 -s \
       ~/libexec/perf-core/scripts/python/export-to-postgresql.py \
       pt_mem_test_suspend branches calls
   2020-06-20 20:12:47.806519 Creating database...
   2020-06-20 20:12:47.919280 Writing to intermediate files...
   2020-06-20 20:12:57.584499 Copying to database...
   2020-06-20 20:13:10.366941 Removing intermediate files...
   2020-06-20 20:13:10.458730 Adding primary keys
   2020-06-20 20:13:13.194662 Adding foreign keys
   2020-06-20 20:13:23.763399 Dropping unused tables
   2020-06-20 20:13:23.784527 Done

And one database for the resume::

   $ perf script -i pt-mem-test --itrace=bp --time 2859.465807,2859.895742 \
       -s ~/libexec/perf-core/scripts/python/export-to-postgresql.py \
       pt_mem_test_resume branches calls
   2020-06-20 20:56:33.427551 Creating database...
   2020-06-20 20:56:33.639840 Writing to intermediate files...
   2020-06-20 20:57:54.092614 Copying to database...
   2020-06-20 20:59:45.300365 Removing intermediate files...
   2020-06-20 20:59:45.715793 Adding primary keys
   2020-06-20 21:00:36.798973 Adding foreign keys
   2020-06-20 21:01:49.617689 Dropping unused tables
   2020-06-20 21:01:49.648594 Done

We can do a crude analysis by aggregating and sorting by function
elapsed time. Note the elapsed time is summed across all CPUs and will
be inaccurate for functions where a call or return was not found.

::

   $ psql pt_mem_test_suspend -c 'SELECT COUNT(symbol_id),symbol_id,(SELECT name FROM symbols WHERE id = symbol_id) AS symbol,SUM(elapsed_time) AS tot_elapsed_time,SUM(branch_count) AS tot_branch_count FROM calls_view GROUP BY symbol_id ORDER BY tot_elapsed_time DESC LIMIT 140;' | cat
    count | symbol_id |               symbol                | tot_elapsed_time | tot_branch_count
   -------+-----------+-------------------------------------+------------------+------------------
    51753 |       155 | __switch_to_asm                     |      25643692943 |          8084655
     2247 |       118 | __schedule                          |       3384905721 |          1392743
     1049 |       117 | schedule                            |       3073233872 |           655725
       57 |       232 | worker_thread                       |       2251468533 |          1611634
     2027 |       141 | __perf_event_task_sched_out         |       2179969857 |           469787
     3657 |       142 | perf_iterate_sb                     |       2111444662 |           820888
     7109 |       144 | perf_event_switch_output            |       2024431937 |           763784
      765 |       161 | do_idle                             |       1801033690 |          1121148
        8 |       162 | cpu_startup_entry                   |       1799810214 |          1117125
      929 |       189 | cpuidle_enter_state                 |       1512404441 |           372312
      951 |       186 | call_cpuidle                        |       1512291784 |           374355
      929 |       187 | cpuidle_enter                       |       1512260439 |           372467
      816 |       191 | intel_idle                          |       1506383402 |            11860
       54 |      1572 | ret_from_fork                       |       1236671564 |           405507
    32212 |       150 | perf_output_copy                    |       1140597616 |           236324
    32622 |        19 | memcpy                              |       1037897766 |            98232
    33010 |        20 | memcpy_erms                         |       1037897766 |            65712
     6858 |       151 | __perf_event__output_id_sample      |       1020294283 |           238528
      124 |       740 | schedule_timeout                    |        833043903 |            80630
      333 |       234 | process_one_work                    |        818013956 |          1492462
     1617 |       156 | __switch_to                         |        806190983 |           230558
      302 |      1388 | async_run_entry_fn                  |        805990273 |          1332080
      935 |      1878 | __device_suspend                    |        602465683 |           795242
      106 |      1905 | async_suspend                       |        598335358 |           711555
       21 |      1575 | kthread                             |        511135637 |           654753
      453 |      1674 | wait_for_completion                 |        462718168 |            27924
     1805 |      1896 | dpm_wait                            |        462391503 |            27536
     2752 |      1879 | dpm_wait_for_subordinate            |        462184824 |           127442
     6458 |       147 | __perf_event_header__init_id.isra.0 |        453504285 |           207450
    17677 |        27 | sched_clock_cpu                     |        443029861 |           104431
    17678 |        28 | sched_clock                         |        442861857 |            69342
    17677 |        29 | native_sched_clock                  |        442769188 |            23308
     6483 |       148 | local_clock                         |        442429154 |            73876
     2765 |      1880 | device_for_each_child               |        434117232 |           106147
     1801 |      1895 | dpm_wait_fn                         |        432735602 |            33496
        7 |      1754 | smpboot_thread_fn                   |        383439061 |            27473
      459 |      1891 | dpm_run_callback                    |        352449547 |          1209376
       20 |      2179 | msleep                              |        284584388 |            15571
      475 |       553 | schedule_hrtimeout_range_clock      |        224993536 |           303536
      475 |       554 | schedule_hrtimeout_range            |        224959544 |           303188
      306 |      1767 | usleep_range                        |        215422913 |           297674
        2 |      2920 | tpm_transmit                        |        204055552 |           192763
       15 |      2039 | pci_set_power_state                 |        203058833 |           313354
       14 |      2257 | pci_raw_set_power_state             |        201300666 |            22320
     6488 |       149 | perf_output_begin                   |        200342196 |            53228
       76 |      3752 | default_idle_call                   |        197153681 |             9395
       65 |      3754 | mwait_idle                          |        197150348 |             9130
       64 |      3753 | arch_cpu_idle                       |        197150348 |             9250
      914 |      3763 | __device_suspend_noirq              |        177018409 |           539319
     6410 |       152 | perf_output_end                     |        172279083 |            26373
     6406 |       153 | perf_output_put_handle              |        171087750 |            10655
       97 |      3766 | async_suspend_noirq                 |        170761099 |           481677
       19 |      3767 | pci_pm_suspend_noirq                |        169819437 |           461954
        2 |      2377 | tpm_pm_suspend                      |        157369145 |           157176
        1 |      2918 | tpm2_shutdown                       |        156458481 |           150632
        1 |      2919 | tpm_transmit_cmd                    |        156456147 |           150552
        9 |      3768 | pci_prepare_to_sleep                |        150865833 |           314568
     1125 |       154 | enter_lazy_tlb                      |        143645190 |             8102
       21 |      1975 | pci_pm_suspend                      |        130362569 |           672764
      758 |       160 | schedule_idle                       |         89123399 |           466833
        4 |      2157 | usb_port_resume                     |         68498773 |             7301
        3 |       744 | rcu_gp_kthread                      |         68089441 |            14204
        1 |      2372 | e1000e_pm_suspend                   |         58136474 |           509370
       37 |      1977 | __pm_runtime_resume                 |         42501525 |            53958
       39 |      1978 | rpm_resume                          |         42495862 |            53684
        8 |      1981 | rpm_callback                        |         42488526 |            53097
        8 |      1982 | __rpm_callback                      |         42488526 |            53093
      912 |      3681 | __device_suspend_late               |         35785898 |           102222
       95 |      3684 | async_suspend_late                  |         35475218 |            46241
        9 |      1459 | async_synchronize_cookie_domain     |         34602552 |            16471
        4 |      1983 | pci_pm_runtime_resume               |         31412230 |            45884
        1 |      2398 | e1000e_pm_freeze                    |         27508908 |            20420
     5930 |       525 | mutex_lock                          |         27491910 |            45500
        3 |      1988 | rtnl_lock                           |         27271909 |             2585
        2 |      2399 | __mutex_lock_slowpath               |         27270243 |             2558
        2 |      2400 | __mutex_lock.isra.0                 |         27270243 |             2554
        2 |      2401 | schedule_preempt_disabled           |         27270243 |             2518
        2 |      1784 | suspend_devices_and_enter           |         26722578 |           313122
        1 |      3111 | __e1000_shutdown                    |         26479579 |           418542
        3 |      3679 | suspend_enter                       |         26475246 |           509770
        7 |      2273 | scsi_bus_suspend_common             |         25100250 |             4560
      335 |      3117 | e1000e_write_phy_reg_mdic           |         24979374 |           424874
        2 |      2275 | scsi_device_quiesce                 |         24933585 |             1607
        6 |      2272 | scsi_bus_suspend                    |         24932584 |             1556
        1 |      2277 | blk_mq_freeze_queue                 |         24924917 |              823
        1 |      2281 | blk_mq_freeze_queue_wait            |         24923584 |              768
      595 |      2575 | delay_tsc                           |         24782071 |           483853
      597 |      2574 | __const_udelay                      |         24782071 |           485042
      678 |      1999 | raw_pci_read                        |         23076242 |           165159
      659 |      1998 | pci_read                            |         23032911 |           167502
      147 |      3121 | __e1000_write_phy_reg_hv            |         21554264 |           372409
      579 |      2002 | pci_conf1_read                      |         21341669 |           161095
        1 |      2368 | pci_legacy_suspend                  |         20570598 |             5401
        1 |      2369 | rtsx_pci_suspend                    |         20570265 |             5384
        1 |      3788 | hcd_pci_suspend_noirq               |         20541265 |            53751
       19 |      2402 | pci_save_state                      |         19793934 |           150800
    23936 |        33 | _raw_spin_lock_irqsave              |         19035782 |           182998
        4 |      1984 | pci_restore_standard_config         |         18696937 |            31582
      787 |       794 | native_queued_spin_lock_slowpath    |         18328613 |           163490
        1 |      3680 | dpm_suspend_late                    |         18081940 |            86213
        9 |      1927 | usb_suspend_both                    |         17712607 |            14462
       19 |      3685 | pci_pm_suspend_late                 |         17650940 |            32293
       25 |      3686 | pm_generic_suspend_late             |         17647941 |            31934
        1 |      3690 | i915_pm_suspend_late                |         17642941 |            31880
        1 |      3691 | i915_drm_suspend_late               |         17642608 |            31876
       18 |      2121 | usb_control_msg                     |         17637609 |            14910
       19 |      2124 | usb_start_wait_urb                  |         17631277 |            14520
        6 |      1840 | async_synchronize_full              |         17585608 |             4762
        6 |      2119 | generic_suspend                     |         17220276 |             5745
        5 |      2120 | usb_port_suspend                    |         17017944 |             4109
       20 |      2125 | usb_submit_urb                      |         16906944 |             6050
       20 |      2126 | usb_hcd_submit_urb                  |         16903944 |             5908
        2 |      2011 | __ieee80211_suspend                 |         16900277 |            16924
       12 |      2159 | rh_call_control                     |         16895943 |             5360
       12 |      2161 | xhci_hub_control                    |         16875276 |             2647
        2 |      2287 | set_port_feature                    |         16800944 |             2587
      459 |      1997 | pci_bus_read_config_dword           |         16082280 |           116781
      449 |      2395 | pci_read_config_dword               |         15771950 |           115817
      118 |      3330 | e1000_access_phy_wakeup_reg_bm      |         15113286 |           269309
      115 |      3336 | e1000_write_phy_reg_page_hv         |         14123620 |           256960
        5 |      2381 | crb_wait_for_reg_32.constprop.0     |         13730621 |           116395
        1 |      2550 | ieee80211_stop_device               |         13666621 |             7922
        1 |      2558 | drv_stop                            |         13661621 |             7674
        1 |      2559 | iwl_mvm_mac_stop                    |         13661288 |             7666
        1 |      2560 | __iwl_mvm_mac_stop                  |         13659621 |             7519
        1 |      2994 | azx_runtime_resume                  |         12872624 |             8900
        1 |      2995 | __azx_runtime_resume                |         12868957 |             8894
        1 |      2566 | iwl_trans_pcie_stop_device          |         12811624 |             5928
        1 |      2567 | _iwl_trans_pcie_stop_device         |         12809957 |             5902
        2 |      2589 | iwl_trans_pcie_sw_reset             |         12121294 |             1713
        1 |      2961 | linkwatch_event                     |         11315629 |             1838
        1 |      2950 | e1000e_down                         |         11221963 |            15148
    25465 |       145 | perf_event_pid_type                 |         11045680 |           166961
    25467 |       146 | __task_pid_nr_ns                    |         10978670 |           102056
        1 |      2441 | mei_me_pm_runtime_resume            |         10954631 |              859
        1 |      2443 | mei_me_d0i3_exit_sync               |         10953630 |              842
        1 |      2442 | mei_me_pg_exit_sync                 |         10953630 |              844
        1 |         3 | state_store                         |         10033967 |           242098
        1 |         1 | kobj_attr_store                     |         10033967 |           242099
        1 |         7 | pm_suspend                          |         10033301 |           242064
   (140 rows)

   $ psql pt_mem_test_resume -c 'SELECT COUNT(symbol_id),symbol_id,(SELECT name FROM symbols WHERE id = symbol_id) AS symbol,SUM(elapsed_time) AS tot_elapsed_time,SUM(branch_count) AS tot_branch_count FROM calls_view GROUP BY symbol_id ORDER BY tot_elapsed_time DESC LIMIT 140;' | cat
    count  | symbol_id |                  symbol                   | tot_elapsed_time | tot_branch_count
   --------+-----------+-------------------------------------------+------------------+------------------
     85530 |        91 | __switch_to_asm                           |     285351925524 |         96433888
      6011 |        56 | __schedule                                |      14948664243 |          7784999
      1851 |        96 | schedule                                  |      14653524942 |          6639494
      2871 |        92 | __switch_to                               |       8977603169 |          5737396
       178 |       193 | worker_thread                             |       6680804308 |          1576838
      3317 |        77 | __perf_event_task_sched_out               |       4600197729 |           793879
      6008 |        78 | perf_iterate_sb                           |       4449766476 |          1390741
     11533 |        80 | perf_event_switch_output                  |       4094624656 |          1295510
       627 |       703 | schedule_timeout                          |       3725284572 |           599231
      3245 |       142 | do_idle                                   |       3470715806 |         21221047
        11 |       264 | cpu_startup_entry                         |       3411701625 |         21217880
     46407 |      1210 | call_cpuidle                              |       2804113112 |         11364705
     46313 |      1212 | cpuidle_enter                             |       2803665450 |         11271883
     46313 |      1214 | cpuidle_enter_state                       |       2803575784 |         11087380
       488 |       194 | process_one_work                          |       2667120505 |          1326501
       344 |       805 | async_run_entry_fn                        |       2590109758 |           820705
      1802 |      1216 | intel_idle                                |       2453148165 |            18285
       287 |       725 | wait_for_completion                       |       2300947973 |           102703
       498 |       808 | dpm_wait_for_superior                     |       2300522704 |           130369
       501 |       834 | dpm_wait                                  |       2299873032 |            98477
     53293 |        86 | perf_output_copy                          |       2095119404 |           384340
       944 |      1314 | device_resume                             |       1895560721 |           407787
     11148 |        87 | __perf_event__output_id_sample            |       1876404517 |           403447
       124 |      1326 | async_resume                              |       1659448842 |           266326
         9 |        97 | smpboot_thread_fn                         |       1335019903 |            63663
     69170 |        15 | memcpy_erms                               |       1259056878 |           118574
     68671 |        14 | memcpy                                    |       1259056878 |           187115
        50 |      1065 | kthread                                   |       1210197649 |           234278
       124 |      1062 | ret_from_fork                             |       1119255949 |           472199
     10741 |        83 | __perf_event_header__init_id.isra.0       |        835974127 |           361423
         4 |      1713 | irq_thread                                |        821502272 |           135505
    209977 |        18 | sched_clock_cpu                           |        768117692 |          1085558
    209971 |        19 | sched_clock                               |        767738618 |           664497
    209968 |        20 | native_sched_clock                        |        766809889 |           222311
     10783 |        84 | local_clock                               |        763929233 |           131320
        24 |       872 | msleep                                    |        716006959 |            70580
       941 |       807 | device_resume_noirq                       |        689453373 |           623511
       123 |       806 | async_resume_noirq                        |        605174322 |           485002
     13995 |       502 | mutex_lock                                |        419992763 |           167568
       477 |       732 | __mutex_lock_slowpath                     |        418528939 |            65987
       477 |       733 | __mutex_lock.isra.0                       |        418528605 |            65061
        14 |       737 | schedule_preempt_disabled                 |        416323953 |             9538
         5 |       714 | acpi_hotplug_work_fn                      |        416319618 |            16566
         4 |       730 | acpi_device_hotplug                       |        416307618 |            15386
        24 |      1327 | pci_pm_resume                             |        407527495 |          2376699
         2 |      1313 | dpm_resume                                |        368244112 |           157625
         1 |      1805 | e1000e_pm_resume                          |        367856112 |          2264968
     10743 |        85 | perf_output_begin                         |        360460322 |            91341
     44514 |      1734 | poll_idle                                 |        344075848 |          9752486
      3241 |       187 | schedule_idle                             |        341011283 |           760522
         5 |       957 | kthreadd                                  |        334471558 |           107715
        24 |      1466 | ata_msleep                                |        325523584 |            19652
        18 |      2077 | __scsi_execute                            |        320628269 |            27053
        18 |      2091 | blk_execute_rq                            |        320596603 |            23104
        20 |      2112 | io_schedule_timeout                       |        320546936 |            15539
        18 |      2111 | wait_for_completion_io_timeout            |        320546604 |            15342
         1 |      2066 | scsi_dev_type_resume                      |        319120274 |             1816
         1 |      2065 | async_sdev_resume                         |        319120274 |             1820
         1 |      2067 | do_scsi_resume                            |        319116940 |             1689
         1 |      2068 | sd_resume                                 |        319116940 |             1685
         1 |      2076 | sd_start_stop_device                      |        319112274 |             1143
         4 |      2011 | scsi_error_handler                        |        317802944 |           289791
         4 |      2012 | ata_scsi_error                            |        317782945 |           287891
         4 |      2015 | ata_scsi_port_error_handler               |        317759613 |           287682
         2 |      2046 | ata_eh_recover                            |        317745944 |           288063
         2 |      2039 | sata_pmp_error_handler                    |        317713279 |           286708
         2 |      2038 | ahci_error_handler                        |        317712945 |           286676
         2 |      2044 | sata_pmp_eh_recover                       |        317712279 |           286647
         1 |      2047 | ata_eh_reset                              |        315574953 |            19949
         1 |      2054 | ata_do_reset                              |        315560620 |            19303
         1 |      2056 | ahci_do_hardreset                         |        315560286 |            19287
         1 |      2055 | ahci_hardreset                            |        315560286 |            19291
         1 |      2058 | sata_link_hardreset                       |        315554953 |            19259
         1 |      2115 | sata_link_resume                          |        314494623 |            18406
         1 |      2710 | __e1000_resume                            |        312765962 |          1386473
       411 |      1264 | schedule_hrtimeout_range_clock            |        310439884 |           326474
       409 |      1263 | schedule_hrtimeout_range                  |        310434220 |           326579
       228 |       141 | default_idle_call                         |        309502540 |            13169
       212 |       140 | arch_cpu_idle                             |        309499872 |            12727
       212 |       139 | mwait_idle                                |        309499872 |            12317
       464 |       841 | dpm_run_callback                          |        292866210 |           893629
         2 |      2943 | rcu_gp_kthread                            |        288285043 |            10491
       213 |      1262 | usleep_range                              |        286360633 |           179582
      2336 |        90 | enter_lazy_tlb                            |        256199483 |            10612
         1 |      2711 | e1000_resume_workarounds_pchlan           |        242773528 |          1209790
         1 |      2712 | e1000_init_phy_workarounds_pchlan         |        241693198 |          1199741
        22 |       854 | pci_power_up                              |        202384994 |           240416
        22 |       851 | pci_pm_resume_noirq                       |        186022716 |           645188
      2628 |        94 | finish_task_switch                        |        185356047 |           632666
      2623 |        95 | __perf_event_task_sched_in                |        183479485 |           611197
        21 |       861 | pci_raw_set_power_state                   |        174633751 |            89570
      3283 |      1268 | __const_udelay                            |        156769618 |          2992055
      3281 |      1269 | delay_tsc                                 |        156769618 |          2985493
         3 |      1141 | async_synchronize_cookie_domain           |        132836559 |             2419
         3 |      1140 | async_synchronize_full                    |        132836559 |             2425
         4 |      2746 | iwl_wait_notification                     |        128951238 |             4668
       202 |      1833 | e1000e_write_phy_reg_mdic                 |        128042966 |          2451760
         8 |      2748 | usb_port_resume                           |        113942621 |            16391
         1 |      3483 | sata_link_debounce                        |        108159308 |            17521
         2 |      1749 | iwl_mvm_up                                |        106032315 |          1828345
         4 |      1333 | i915_drm_resume                           |         96838063 |           925008
        83 |      1933 | __e1000_write_phy_reg_hv                  |         89115984 |          1677443
         2 |      2714 | e1000_toggle_lanphypc_pch_lpt             |         86013323 |             3235
         1 |      2454 | intel_display_resume                      |         84957385 |           864298
         1 |      2459 | __intel_display_resume                    |         84955051 |           863955
         1 |      2489 | drm_atomic_helper_commit_duplicated_state |         84910385 |           858672
         1 |      2490 | drm_atomic_commit                         |         84910051 |           858658
         2 |      2623 | intel_atomic_commit                       |         84909051 |           858119
         1 |      2665 | intel_atomic_commit_tail                  |         84869385 |           851826
        11 |      1925 | i2c_transfer                              |         84185721 |           238582
        11 |      1927 | __i2c_transfer                            |         84183386 |           238363
        11 |      1928 | __i2c_transfer.part.0                     |         84183053 |           238334
     10599 |        88 | perf_output_end                           |         82322046 |            44321
     10598 |        89 | perf_output_put_handle                    |         82288381 |            17948
         3 |      1944 | iwl_mvm_load_ucode_wait_alive             |         81495062 |           153583
         1 |      2680 | skl_update_crtcs                          |         68021108 |           843191
         1 |      2682 | intel_update_crtc                         |         68020774 |           843159
         1 |      2684 | haswell_crtc_enable                       |         68002107 |           842352
     42409 |        81 | perf_event_pid_type                       |         67962101 |           293906
         1 |      2692 | intel_encoders_pre_enable.isra.0          |         67831108 |           835383
         1 |      2693 | intel_ddi_pre_enable                      |         67830775 |           835376
         1 |      3679 | e1000e_reset                              |         66664445 |           147625
        53 |      1851 | usb_control_msg                           |         66284776 |            40820
        53 |      1854 | usb_start_wait_urb                        |         66272777 |            38835
        38 |      1930 | drm_dp_i2c_do_msg                         |         65810781 |           206511
     42534 |        82 | __task_pid_nr_ns                          |         65224567 |           175448
         8 |      1929 | drm_dp_i2c_xfer                           |         63746788 |           201668
         1 |      3681 | e1000_reset_hw_ich8lan                    |         62712125 |           119427
        51 |       339 | acpi_ps_parse_aml                         |         60445802 |          7921658
         1 |      3167 | i915_hpd_poll_init_work                   |         56777812 |           170866
         1 |      3168 | drm_helper_hpd_irq_event                  |         56777478 |           170808
         3 |      3169 | drm_helper_probe_detect_ctx               |         56777145 |           170726
         2 |      1942 | iwl_run_init_mvm_ucode                    |         56027147 |            70531
        74 |       236 | acpi_ns_evaluate                          |         55670815 |          3550574
        73 |        50 | acpi_evaluate_object                      |         55557155 |          3444264
         8 |      1836 | usb_resume_both                           |         55403483 |            19838
         8 |      1835 | usb_resume                                |         55397150 |            19714
         8 |      1834 | usb_dev_resume                            |         55375484 |            19520
         7 |      1837 | generic_resume                            |         55328150 |            10526
         2 |      1838 | hcd_bus_resume                            |         55138485 |             2692
   (140 rows)

While this analysis is crude, we can still pick out some interesting
items.

Important subsystem and device driver functions are usually prefixed by
their identifier, so we can readily see the longest to suspend are tpm
and e1000 while the longest to resume are e1000 and scsi devices.

The presence of \__mutex_lock_slowpath indicates contended mutexes,
which is not ideal.

The presence of delay or sleep functions msleep, usleep_range,
\__const_udelay indicates polling, which is not ideal.

The 209977 calls to sched_clock_cpu seems excessive, but is probably
caused by perf.

Finally, we can stop PostgreSQL since we are not using it anymore.

   $ sudo systemctl stop postgresql

Example: Detecting System Management Mode (SMM)
-----------------------------------------------

Intel PT is automatically disabled by hardware when entering System
Management Mode (SMM), so it is possible to use Intel PT to detect
whether SMM might have run. Control-flow packet generation is also
disabled in secure enclaves.

If those do not occur, and there is no trace data loss, then system wide
tracing will show only one "trace start" and one "trace end" for each
CPU. Additional "trace start" or "trace end" branches would need another
explanation i.e. SMM, secure enclave or trace data loss.

To record system wide with 64M buffers for 3 seconds, we can use ``perf
record`` with options:

- ``-m,64M`` to set the trace buffer size to 64 MiB. This is needed
  to avoid trace data loss. Note the comma is needed. Also **be careful
  setting large buffer sizes**. With per-cpu tracing (the default), one
  buffer per CPU will be allocated. In our case we have 8 CPUs so that
  means 512 MiB. However when tracing with per-task contexts, there will
  be one buffer per task, which might be far more than anticipated.
- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``-e`` to select which events, i.e. the following:
- ``intel_pt//`` to get Intel PT
- ``sleep 3`` is the workload. The tracing will stop when the
  workload finishes, so this is simply a way of tracing for about 3
  seconds.

::

   $ sudo perf record -m,64M -a -e intel_pt// -- sleep 3
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 4.524 MB perf.data ]

To show "trace start" and "trace end", use ``perf script`` with options:

- ``--itrace=qbe`` to show quicker (less detailed) decoding (q),
  branches (b) and errors (e)
- ``-F+flags`` to show branch flags such as ``tr strt`` and ``tr
  end``
- ``--ns`` to show the timestamp to nanoseconds instead of the
  default microseconds

.. note::

   the itrace 'q' option is new from v5.9. Use ``perf version`` to check
   the version.

::

   $ sudo perf script --itrace=qbe -F+flags --ns | grep 'tr strt\|tr end'
               perf  7212 [000] 14733.245070483:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [001] 14733.245128490:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [002] 14733.245182361:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [003] 14733.245237624:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [004] 14733.245292674:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [005] 14733.245351116:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [006] 14733.245404620:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [007] 14733.245458096:          1   branches:   tr strt                             0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu) => ffffffff81016aa8 pt_config_start+0x68 ([kernel.kallsyms])
               perf  7212 [000] 14736.246883705:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [001] 14736.246987037:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [002] 14736.247082703:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [003] 14736.247180369:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [004] 14736.247277368:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [005] 14736.247370701:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [006] 14736.247465034:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)
               perf  7212 [007] 14736.247558700:          1   branches:   tr end               ffffffff81016eb6 pt_config_stop+0x66 ([kernel.kallsyms]) =>                0 __per_cpu_start+0x0 ([kernel.kallsyms].data..percpu)

In this case there were no unexplained ``tr strt`` or ``tr end``.

Example: rdtsc vs Intel PT
--------------------------

The rdtsc instruction has long been used to time functions. Consider:

- How to Benchmark Code Execution Times on Intel ®IA-32 and IA-64
  Instruction Set Architectures
  https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/ia-32-ia-64-benchmark-code-execution-paper.pdf

- RDTSC the only way to benchmark.
  https://medium.com/geekculture/rdtsc-the-only-way-to-benchmark-fc84562ef734

But how does it compare to Intel PT. In particular, Intel PT in
cycle-accurate mode and configured to use an address filter to trace a
single function.

Let's create a test program with a trivial function f() to trace. Here
is the program (named rdtsc-vs-intel-pt.c):

.. code-block:: cpp

   #include <stdio.h>
   #include <stdint.h>
   #include <stdlib.h>
   #include <unistd.h>

   static inline uint64_t rdtsc(void)
   {
           unsigned int low, high;

           asm volatile("rdtsc" : "=a" (low), "=d" (high));

           return low | ((uint64_t)high) << 32;
   }

   static inline uint64_t rdtsc_before(void)
   {
           unsigned int low, high;

           asm volatile("rdtsc" : "=a" (low), "=d" (high));
       asm volatile("lfence");

           return low | ((uint64_t)high) << 32;
   }

   static inline uint64_t rdtsc_after(void)
   {
           unsigned int low, high;

       asm volatile("lfence");
           asm volatile("rdtsc" : "=a" (low), "=d" (high));

           return low | ((uint64_t)high) << 32;
   }

   static inline uint64_t rdtscp(void)
   {
           unsigned int low, high, p;

           asm volatile("rdtscp" : "=a" (low), "=d" (high), "=c"(p));

           return low | ((uint64_t)high) << 32;
   }

   static inline uint64_t rdtscp_before(void)
   {
           unsigned int low, high, p;

           asm volatile("rdtscp" : "=a" (low), "=d" (high), "=c"(p));
       asm volatile("lfence");

           return low | ((uint64_t)high) << 32;
   }

   static void prt(const char *msg, unsigned long long val)
   {
       printf("%60s: %llu TSC ticks\n", msg, val);
   }

   volatile int x = 0;

   static void __attribute__((noinline)) f(void)
   {
       x = x * x + x + 1;
   }

   int main(int argc, char *argv[])
   {
       unsigned long long tsc1, tsc2;

       if (argc > 1) {
           int secs = atoi(argv[1]);

           if (secs)
               sleep(secs);
       }

       f();

       f();

       tsc1 = rdtsc();
       f();
       tsc2 = rdtsc();
       prt("rdtsc with no ordering, measured f() time", tsc2 - tsc1);

       tsc1 = rdtscp();
       f();
       tsc2 = rdtscp();
       prt("rdtscp with no ordering before, measured f() time", tsc2 - tsc1);

       tsc1 = rdtscp_before();
       f();
       tsc2 = rdtscp();
       prt("rdtscp with ordering, measured f() time", tsc2 - tsc1);

       tsc1 = rdtsc_before();
       f();
       tsc2 = rdtsc_after();
       prt("rdtsc with ordering, measured f() time", tsc2 - tsc1);

       return 0;
   }

There are some important points to note.

First, the compiler must not inline the function f(), so
\__attribute\_\_((noinline)) is employed.

Secondly, rdtsc is not a serializing instruction. Here is what the Intel
SDM (https://www.intel.com/sdm) has to say about it:

   The RDTSC instruction is not a serializing instruction. It does not
   necessarily wait until all previous instructions have been executed
   before reading the counter. Similarly, subsequent instructions may
   begin execution before the read operation is performed. The following
   items may guide software seeking to order executions of RDTSC:

   - If software requires RDTSC to be executed only after all previous
     instructions have executed and all previous loads are globally
     visible, 1 it can execute LFENCE immediately before RDTSC.

   - If software requires RDTSC to be executed only after all previous
     instructions have executed and all previous loads and stores are
     globally visible, it can execute the sequence MFENCE;LFENCE
     immediately before RDTSC.

   - If software requires RDTSC to be executed prior to execution of any
     subsequent instruction (including any memory accesses), it can
     execute the sequence LFENCE immediately after RDTSC.

So to serialize rdtsc, the program defines functions rdtsc_before() and
rdtsc_after().

Thirdly, the newer alternative instruction rdtscp can be used. However
it is not entirely serialized either. Here is what the Intel SDM
(https://www.intel.com/sdm) has to say about it:

   The RDTSCP instruction is not a serializing instruction, but it does
   wait until all previous instructions have executed and all previous
   loads are globally visible.1 But it does not wait for previous stores
   to be globally visible, and subsequent instructions may begin
   execution before the read operation is performed. The following items
   may guide software seeking to order executions of RDTSCP:

   - If software requires RDTSCP to be executed only after all previous
     stores are globally visible, it can execute MFENCE immediately
     before RDTSCP.

   - If software requires RDTSCP to be executed prior to execution of
     any subsequent instruction (including any memory accesses), it can
     execute LFENCE immediately after RDTSCP.

So to serialize rdtscp, the program defines function rdtscp_before().

Finally, the program can take an extra parameter that causes sleep for
that number of seconds. This parameters will come in useful later.

Before continuing, we need to consider, if rdtsc and rdtscp need some
extra serialization, what about Intel PT. Here is what the Intel SDM
(https://www.intel.com/sdm) has to say about it:

   Intel PT can be run in a cycle-accurate mode which enables CYC
   packets (see Section 32.4.2.14) that provide low- level information
   in the processor core clock domain. This cycle counter data in CYC
   packets can be used to compute IPC (Instructions Per Cycle), or to
   track wall-clock time on a fine-grain level.

   Cycle-accurate mode adheres to the following protocol:

   - All packets that precede a CYC packet represent instructions or
     events that took place before the CYC time.

   - All packets that follow a CYC packet represent instructions or
     events that took place at the same time as, or after, the CYC time.

   - The CYC-eligible packet that immediately follows a CYC packet
     represents an instruction or event that took place at the same time
     as the CYC time.

To compile the test program::

   $ gcc -Wall -Wextra -O3 -o rdtsc-vs-intel-pt rdtsc-vs-intel-pt.c

And run it::

   $ ./rdtsc-vs-intel-pt
                      rdtsc with no ordering, measured f() time: 95 TSC ticks
              rdtscp with no ordering before, measured f() time: 288 TSC ticks
                        rdtscp with ordering, measured f() time: 176 TSC ticks
                         rdtsc with ordering, measured f() time: 177 TSC ticks

And trace it with Intel PT:

We can use ``perf record`` with options:

- ``-e`` to select which events, i.e. the following:
- ``intel_pt/cyc/u`` to get Intel PT with cycle-accurate mode, user
  space only.
- ``--filter 'filter f @ ./rdtsc-vs-intel-pt'`` specifies an address
  filter to trace only f()
- ``./rdtsc-vs-intel-pt`` is the workload.

::

   $ perf record -e intel_pt/cyc/u --filter 'filter f @ ./rdtsc-vs-intel-pt' ./rdtsc-vs-intel-pt
                      rdtsc with no ordering, measured f() time: 26 TSC ticks
              rdtscp with no ordering before, measured f() time: 73 TSC ticks
                        rdtscp with ordering, measured f() time: 47 TSC ticks
                         rdtsc with ordering, measured f() time: 47 TSC ticks
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.015 MB ]

   perf script --itrace=bep --ns -F+ipc,-period,-dso,-comm,+flags,+addr,-pid,-tid
   [000] 12434.509024343:            psb:                          psb offs: 0                                     0                0 [unknown]
   [000] 12434.509024343:            cbr:                          cbr: 42 freq: 4219 MHz (156%)                   0                0 [unknown]
   [000] 12434.509252555:    branches:uH:   tr strt                               0 [unknown] =>     7f45b77032f0 f+0x0
   [000] 12434.509252568:    branches:uH:   tr end  return             7f45b770330f f+0x1f =>     7f45b77030b3 main+0x13    IPC: 0.12 (7/54)
   [000] 12434.509252568:    branches:uH:   tr strt                               0 [unknown] =>     7f45b77032f0 f+0x0
   [000] 12434.509252570:    branches:uH:   tr end  return             7f45b770330f f+0x1f =>     7f45b77030b8 main+0x18    IPC: 0.87 (7/8)
   [000] 12434.509252571:    branches:uH:   tr strt                               0 [unknown] =>     7f45b77032f0 f+0x0
   [000] 12434.509252572:    branches:uH:   tr end  return             7f45b770330f f+0x1f =>     7f45b77030c8 main+0x28    IPC: 1.75 (7/4)
   [000] 12434.509306045:    branches:uH:   tr strt                               0 [unknown] =>     7f45b77032f0 f+0x0
   [000] 12434.509306065:    branches:uH:   tr end  return             7f45b770330f f+0x1f =>     7f45b7703104 main+0x64    IPC: 0.08 (7/81)
   [000] 12434.509307757:    branches:uH:   tr strt                               0 [unknown] =>     7f45b77032f0 f+0x0
   [000] 12434.509307763:    branches:uH:   tr end  return             7f45b770330f f+0x1f =>     7f45b7703144 main+0xa4    IPC: 0.28 (7/25)
   [000] 12434.509308965:    branches:uH:   tr strt                               0 [unknown] =>     7f45b77032f0 f+0x0
   [000] 12434.509308970:    branches:uH:   tr end  return             7f45b770330f f+0x1f =>     7f45b7703183 main+0xe3    IPC: 0.31 (7/22)

The trace shows the CPU, timestamp in nanoseconds, and event
information: "psb" is the Intel PT synchronization event. "cbr"
(core-to-bus ratio) shows the CPU frequency. The "branches" events show
the 6 invocations of f() as:

- entry to f() "0 [unknown] => 7f45b77032f0 f+0x0" (Note the offset is 0
  i.e. the start of f())
- return from f() "7f45b770330f f+0x1f => 7f45b77030b3 main+0x13"

Branches are annotated with IPC (instructions-per-cpu-cycle)
information. The number of instructions is always 7. The number of CPU
cycles varies. Note that the frequency of CPU cycles changes whereas the
TSC frequency (2712 MHz in this case) is constant.

But the times are much smaller when traced with Intel PT, so what
happened. Let's add a 1 second sleep by passing 1 as the first
parameter::

   $ perf record -e intel_pt/cyc/u --filter 'filter f @ ./rdtsc-vs-intel-pt' ./rdtsc-vs-intel-pt 1
                      rdtsc with no ordering, measured f() time: 359 TSC ticks
              rdtscp with no ordering before, measured f() time: 234 TSC ticks
                        rdtscp with ordering, measured f() time: 184 TSC ticks
                         rdtsc with ordering, measured f() time: 193 TSC ticks
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.123 MB perf.data ]
   $ perf script --itrace=bep --ns -F+ipc,-period,-dso,-comm,+flags,+addr,-pid,-tid
   [005] 12915.734605958:            psb:                          psb offs: 0                                     0                0 [unknown]
   [005] 12915.734605958:            cbr:                          cbr: 42 freq: 4219 MHz (156%)                   0                0 [unknown]
   [005] 12916.735108407:            cbr:                          cbr: 12 freq: 1205 MHz ( 44%)                   0                0 [unknown]
   [005] 12916.735120793:    branches:uH:   tr strt                               0 [unknown] =>     7f67047c42f0 f+0x0
   [005] 12916.735120937:    branches:uH:   tr end  return             7f67047c430f f+0x1f =>     7f67047c40b3 main+0x13    IPC: 0.04 (7/173)
   [005] 12916.735120938:    branches:uH:   tr strt                               0 [unknown] =>     7f67047c42f0 f+0x0
   [005] 12916.735120946:    branches:uH:   tr end  return             7f67047c430f f+0x1f =>     7f67047c40b8 main+0x18    IPC: 0.70 (7/10)
   [005] 12916.735120952:    branches:uH:   tr strt                               0 [unknown] =>     7f67047c42f0 f+0x0
   [005] 12916.735120955:    branches:uH:   tr end  return             7f67047c430f f+0x1f =>     7f67047c40c8 main+0x28    IPC: 1.75 (7/4)
   [005] 12916.735338404:    branches:uH:   tr strt                               0 [unknown] =>     7f67047c42f0 f+0x0
   [005] 12916.735338465:    branches:uH:   tr end  return             7f67047c430f f+0x1f =>     7f67047c4104 main+0x64    IPC: 0.09 (7/73)
   [005] 12916.735344207:    branches:uH:   tr strt                               0 [unknown] =>     7f67047c42f0 f+0x0
   [005] 12916.735344233:    branches:uH:   tr end  return             7f67047c430f f+0x1f =>     7f67047c4144 main+0xa4    IPC: 0.22 (7/31)
   [005] 12916.735348485:    branches:uH:   tr strt                               0 [unknown] =>     7f67047c42f0 f+0x0
   [005] 12916.735348511:    branches:uH:   tr end  return             7f67047c430f f+0x1f =>     7f67047c4183 main+0xe3    IPC: 0.22 (7/31)

Now we can see what happened. The activity of 'perf' resulted in the CPU
frequency increasing to 4219 MHz. When the program went to sleep, the
frequency dropped to 1205 MHz and the TSC timings went back up.

This shows a powerful feature of Intel PT. **Intel PT can show what the
CPU frequency actually is.**

Let's get that output again, but with only what is interesting and using
"--deltatime" to get the change in time::

   $ perf script --itrace=bep --ns -F+ipc,-period,-dso,-comm,-pid,-tid,-cpu,-ip,-event --deltatime | grep IPC
       0.000000144:         IPC: 0.04 (7/173)
       0.000000008:         IPC: 0.70 (7/10)
       0.000000003:         IPC: 1.75 (7/4)
       0.000000061:         IPC: 0.09 (7/73)
       0.000000026:         IPC: 0.22 (7/31)
       0.000000026:         IPC: 0.22 (7/31)

The last 4 correspond to the 4 TSC values, so we have::

        ns                                 TSC ticks / ns
       144         IPC: 0.04 (7/173)
         8         IPC: 0.70 (7/10)
         3         IPC: 1.75 (7/4)         359 / 132   rdtsc with no ordering
        61         IPC: 0.09 (7/73)        234 /  86   rdtscp with no ordering before
        26         IPC: 0.22 (7/31)        184 /  68   rdtscp with ordering
        26         IPC: 0.22 (7/31)        193 /  71   rdtsc with ordering

"rdtsc with no ordering" at 132ns does not match Intel PT result of 3ns
at all. "rdtscp with ordering" and "rdtsc with ordering" seem about the
same.

However the presence of TSC overhead is clearly visible. Perhaps around
50ns in this case. That isn't bad, but Intel PT seems the winner in this
case.

Example: Using PTWRITE to measure the time of an individual instruction
-----------------------------------------------------------------------

In "Example: rdtsc vs Intel PT" above, it was seen how Intel PT can be
used to measure the time of a function.

In general it is not possible to measure the time of an individual
instruction with Intel PT, but with some changes to the program code and
using the PTWRITE instruction, it can be done.

Some Atom and Hybrid CPUs support PTWRITE. To check::

   # cat /sys/bus/event_source/devices/intel_pt/caps/ptwrite
   1

1 means PTWRITE is supported, 0 (or no file at all) means not supported.

The reason for using PTWRITE is that it produces a CYC-eligible Intel PT
packet which means we know the cycle count at that point. Sandwich an
instruction between 2 PTWRITE instructions and we can see how long that
instruction took.

Let's measure the time of xsave and xrestore instructions in the Linux
kernel.

First patch the kernel to insert PTWRITE instruction's in the right places. For this, the instruction used is "ptwrite (%rsp)" because it will always work and we don't care what the PTWRITE payload value is.

Here is the patch diff:

.. code-block:: diff

   diff --git a/arch/x86/kernel/fpu/xstate.h b/arch/x86/kernel/fpu/xstate.h
   index d22ace092ca2..87b192548621 100644
   --- a/arch/x86/kernel/fpu/xstate.h
   +++ b/arch/x86/kernel/fpu/xstate.h
   @@ -83,14 +83,19 @@ static inline u64 xfeatures_mask_independent(void)
    #define XRSTOR     ".byte " REX_PREFIX "0x0f,0xae,0x2f"
    #define XRSTORS        ".byte " REX_PREFIX "0x0f,0xc7,0x1f"

   +/* ptwrite (%rsp) */
   +#define PTWRITE        " .byte 0xf3, 0x48, 0x0f, 0xae, 0x24, 0x24\n"
   +
    /*
     * After this @err contains 0 on success or the trap number when the
     * operation raises an exception.
     */
    #define XSTATE_OP(op, st, lmask, hmask, err)               \
   -   asm volatile("1:" op "\n\t"                 \
   +   asm volatile(PTWRITE                        \
   +            "1:" op "\n\t"                 \
                "xor %[err], %[err]\n"             \
                "2:\n\t"                       \
   +            PTWRITE                        \
                _ASM_EXTABLE_TYPE(1b, 2b, EX_TYPE_FAULT_MCE_SAFE)  \
                : [err] "=a" (err)                 \
                : "D" (st), "m" (*st), "a" (lmask), "d" (hmask)    \
   @@ -111,12 +116,14 @@ static inline u64 xfeatures_mask_independent(void)
     * address of the instruction where we might get an exception at.
     */
    #define XSTATE_XSAVE(st, lmask, hmask, err)                \
   -   asm volatile(ALTERNATIVE_2(XSAVE,               \
   +   asm volatile(PTWRITE                        \
   +            ALTERNATIVE_2(XSAVE,               \
                      XSAVEOPT, X86_FEATURE_XSAVEOPT,  \
                      XSAVES,   X86_FEATURE_XSAVES)    \
                "\n"                       \
                "xor %[err], %[err]\n"             \
                "3:\n"                     \
   +            PTWRITE                        \
                _ASM_EXTABLE_TYPE_REG(661b, 3b, EX_TYPE_EFAULT_REG, %[err]) \
                : [err] "=r" (err)                 \
                : "D" (st), "m" (*st), "a" (lmask), "d" (hmask)    \
   @@ -127,10 +134,12 @@ static inline u64 xfeatures_mask_independent(void)
     * XSAVE area format.
     */
    #define XSTATE_XRESTORE(st, lmask, hmask)              \
   -   asm volatile(ALTERNATIVE(XRSTOR,                \
   +   asm volatile(PTWRITE                        \
   +            ALTERNATIVE(XRSTOR,                \
                    XRSTORS, X86_FEATURE_XSAVES)       \
                "\n"                       \
                "3:\n"                     \
   +            PTWRITE                        \
                _ASM_EXTABLE_TYPE(661b, 3b, EX_TYPE_FPU_RESTORE)   \
                :                          \
                : "D" (st), "m" (*st), "a" (lmask), "d" (hmask)    \

After building and installing the new kernel, we can run a trace:

We can use ``perf record`` with options:

- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)
- ``-e`` to select which events, i.e. the following:
- ``intel_pt/cyc,ptw,fup_on_ptw/k`` to get Intel PT with
  cycle-accurate mode, PTWRITE+FUP, kernel space only.
- **:literal:`sh -c 'for i in \`seq 1 10\`;do sleep 0.01;done'`** is the
  workload.

::

   # perf record -a --kcore -e intel_pt/cyc,ptw,fup_on_ptw/k -- sh -c 'for i in `seq 1 10`;do sleep 0.01;done'
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 2.599 MB perf.data ]

And see the results.

Let's see xsaves on CPU 5:

We can use ``perf script`` with options:

- ``--deltatime`` to show time differences
- ``--insn-trace`` to show all instructions
- ``--xed`` to disassemble instructions using XED
- ``-C 5`` to decode only CPU 5

This shows times varying from 23ns to 163ns::

   # perf script --deltatime --insn-trace --xed -C 5 | grep -A 2 xsave
        migration/5    36 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
        migration/5    36 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
        migration/5    36 [005]     0.000000023:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000031:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000033:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000048:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000121:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000154:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000122:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000148:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000122:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000160:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)
   --
            swapper     0 [005]     0.000000000:  ffffffffafea8f04 save_fpregs_to_fpstate+0x44 ([kernel.kallsyms])                 xsaves64 (%rdi)
            swapper     0 [005]     0.000000000:  ffffffffafea8f08 save_fpregs_to_fpstate+0x48 ([kernel.kallsyms])                 xor %edi, %edi
            swapper     0 [005]     0.000000163:  ffffffffafea8f0a save_fpregs_to_fpstate+0x4a ([kernel.kallsyms])                 ptwriteq  (%rsp)

Similarly, let's see xrestores on CPU 5:

This shows times varying from 32ns to 855ns::

   # perf script --deltatime --insn-trace --xed -C 5 | grep -A 1 xrst
                 sh  1167 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
                 sh  1167 [005]     0.000000117:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
              sleep  1167 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
              sleep  1167 [005]     0.000000032:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
                 sh  1169 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
                 sh  1169 [005]     0.000000095:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
              sleep  1169 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
              sleep  1169 [005]     0.000000047:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
                 sh  1170 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
                 sh  1170 [005]     0.000000344:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
              sleep  1170 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
              sleep  1170 [005]     0.000000135:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
                 sh  1174 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
                 sh  1174 [005]     0.000000317:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
              sleep  1174 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
              sleep  1174 [005]     0.000000134:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
                 sh  1176 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
                 sh  1176 [005]     0.000000855:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
              sleep  1176 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
              sleep  1176 [005]     0.000000536:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)
   --
               perf  1164 [005]     0.000000000:  ffffffffafea9072 restore_fpregs_from_fpstate+0x52 ([kernel.kallsyms])            xrstors64 (%rdi)
               perf  1164 [005]     0.000000198:  ffffffffafea9076 restore_fpregs_from_fpstate+0x56 ([kernel.kallsyms])            ptwriteq  (%rsp)

Example: Tracing \__schedule()
------------------------------

We want to observe some task switches, so in this example, a kernel
compile is started in the background to act as a workload::

   $ cd ~/git/linux
   $ make -j9 >/dev/null &
   [1] 77175

This example includes kernel tracing, which requires administrator
privileges.

We can use ``perf record`` with options:

- ``-Se`` snapshot mode with a snapshot at the end. This is a way to
  control the amount of data collected. The root user default buffer
  size is 4MiB so with a single snapshot at the end, and 8 CPUs, data is
  limited to about 32MiB.
- ``-a`` to trace system wide i.e. all tasks, all CPUs
- ``--kcore`` to copy kernel object code from the /proc/kcore image
  (helps avoid decoding errors due to kernel self-modifying code)
- ``-e`` to select which events, i.e. the following:
- ``intel_pt/cyc,noretcomp,mtc_period=9/k`` to get Intel PT with
  cycle-accurate mode. We add ``noretcomp`` to get a Intel PT TIP
  packet from RET instructions, which has the side-effect of also
  getting a CYC timing packet, and consequently enables calculating a
  timestamp at that point. The ``mtc_period`` is set as high as
  possible because MTC packets are collected continuously which bloats
  the trace.
- ``--filter 'filter __schedule'`` specifies an address filter to
  trace only \__schedule()
- ``--no-switch-events`` switch events will divide up \__schedule(),
  so disable them
- ``--`` is a separator, indicating that the rest of the options
  belong to the workload
- ``sleep 3`` is the workload. The tracing will stop when the
  workload finishes, so this is simply a way of tracing for about 3
  seconds.

::

   $ sudo ~/bin/perf record -Se -a --kcore -e intel_pt/cyc,noretcomp,mtc_period=9/k --filter 'filter __schedule' --no-switch-events -- sleep 3
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 34.200 MB perf.data ]

Don't need the workload anymore so kill it::

   $ kill %1
   [1]+  Terminated              make -j9 > /dev/null

Get extra packages for ``export-to-sqlite.py`` script::

   sudo apt-get install sqlite3 python3-pyside2.qtsql libqt5sql5-psql

Refer to the script `export-to-sqlite.py
<http://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/perf/scripts/python/export-to-sqlite.py>`__
for more information.

Export to SQLite3 database::

   $ perf script --itrace=be -s export-to-sqlite.py pt.db branches calls
   2022-09-30 16:34:08.522415 Creating database ...
   2022-09-30 16:34:08.526911 Writing records...
   Warning:
   2 instruction trace errors
   2022-09-30 16:34:11.191507 Adding indexes
   2022-09-30 16:34:11.215882 Dropping unused tables
   2022-09-30 16:34:11.230945 Done

Have a look at the errors::

   $ perf script --itrace=e
   Warning:
   2 instruction trace errors
    instruction trace error type 1 time 31840.557348019 cpu 4 pid 83275 tid 83275 ip 0xffffffffb43a6c0e code 7: Overflow packet
    instruction trace error type 1 time 31840.557316000 cpu 0 pid 83275 tid 83275 ip 0xffffffffb43a6c0e code 7: Overflow packet

Overflows are not unexpected, and there are only 2, so they can be
ignored in this case.

Start the viewer to look at a call graph::

   $ python3 ~/libexec/perf-core/scripts/python/exported-sql-viewer.py pt.db

The call graph looks like this::

   Call Path                            Object     Count   Time (ns)   Time (%)
   ▼ perf
     ▼ 83275:83275
       ▼ __schedule                     [kernel]    7013    47684493      100.0
         ▶ rcu_note_context_switch     [kernel]    4773       29498        0.1
         ▶ raw_spin_rq_lock_nested     [kernel]    4773       62676        0.1
         ▶ update_rq_clock             [kernel]    4773       83770        0.2
         ▶ pick_next_task              [kernel]    4773     2129516        4.5
         ▶ psi_task_switch             [kernel]    4719     2336791        4.9
         ▶ prepare_task_switch         [kernel]    4719      288511        0.6
         ▶ enter_lazy_tlb              [kernel]    1117       15066        0.0
         ▶ __switch_to_asm             [kernel]    4719    38735454       81.2
         ▶ finish_task_switch.isra.0   [kernel]    4164     1214343        2.5
         ▶ dequeue_task                [kernel]    2854     1183437        2.5
         ▶ switch_mm_irqs_off          [kernel]    3602      529887        1.1
         ▶ raw_spin_rq_unlock          [kernel]      54         515        0.0
         ▶ asm_sysvec_reschedule_ipi   [kernel]       7        2250        0.0

Example: How to create an application that can trace itself
-----------------------------------------------------------

An application may be able to detect unexpected conditions that warrent
making a trace. Here is an example of a program that does that::

   $ cat self-trace-example.c

.. code-block:: cpp

   #define _GNU_SOURCE
   #include <sys/types.h>
   #include <sys/stat.h>
   #include <unistd.h>
   #include <fcntl.h>
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <stdbool.h>
   #include <stddef.h>

   static char perf_temp_dir[] = "/tmp/perf-control-XXXXXX";
   static char *perf_control_name;
   static char *perf_ack_name;
   static char *perf_pid_name;
   static int perf_control = -1;
   static int perf_ack = -1;
   static int perf_pid;

   static int control_perf(const char *cmd)
   {
           ssize_t ret;
           ssize_t sz = strlen(cmd);
           char buf[16];

           printf("sending cmd '%s'\n", cmd);
           ret = write(perf_control, cmd, sz);
           if (ret != sz)
                   return -1;
           buf[0] = 0;
           ret = read(perf_ack, buf, sizeof(buf));
           if (ret < 0)
                   return ret;
           if (ret < 4 || strncmp("ack\n", buf, 4))
                   return -1;
           printf("Ack'ed\n");
           return 0;
   }

   static int mk_perf_fifo(char *dir_name, char *name, char **fifo_name, int *fd)
   {
           if (asprintf(fifo_name, "%s/%s", dir_name, name) < 0)
                   return -1;
           mkfifo(*fifo_name, 0600);
           *fd = open(*fifo_name, O_RDWR);
           return *fd < 0 ? *fd : 0;
   }

   static int init_perf(void)
   {
           char *dir_name;
           int err;

           dir_name = mkdtemp(perf_temp_dir);
           if (!dir_name)
                   return -1;
           if (asprintf(&perf_pid_name, "%s/perf_pid", dir_name) < 0)
                   return -1;
           err = mk_perf_fifo(dir_name, "perf.control", &perf_control_name, &perf_control);
           if (err < 0)
                   return err;
           return mk_perf_fifo(dir_name, "perf.ack", &perf_ack_name, &perf_ack);
   }

   static int read_perf_pid(void)
   {
           FILE *f = fopen(perf_pid_name, "r");

           return (f && fscanf(f, "%d", &perf_pid) == 1) ? 0 : -1;
   }

   static void waitfor_perf_pid(void)
   {
           char buf[256];

           snprintf(buf, sizeof(buf), "/proc/%d", perf_pid);
           while (!access(buf, F_OK))
                   usleep(1000);
   }

   static int start_perf(void)
   {
           static bool initialized;
           char *cmd;
           int err;

           if (!initialized) {
                   err = init_perf();
                   if (err)
                           return err;
                   initialized = true;
           }
           err = asprintf(&cmd, "perf record --control fifo:%s,%s -S -e intel_pt//u -p %d & echo $! > %s", perf_control_name, perf_ack_name, getpid(), perf_pid_name);
           if (err < 0)
                   return err;
           printf("cmd is '%s'\n", cmd);
           err = system(cmd);
           free(cmd);
           if (err)
                   return err;
           err = control_perf("ping");
           if (err)
                   return err;
           return read_perf_pid();
   }

   /* noinline so it shows in a call trace */
   static __attribute__((noinline)) int perf_snapshot(void)
   {
           return control_perf("snapshot");
   }

   volatile int dummy_var;

   /* noinline so it shows in a call trace */
   static __attribute__((noinline)) void work(void)
   {
           dummy_var += 1; /* Stop work() being optimized away */
   }

   int main()
   {
           int err;

           err = start_perf();
           if (err)
                   goto out;
           work();
           work();
           work();
           err = perf_snapshot();
           if (err)
                   goto out;
           err = control_perf("stop");
           if (err)
                   goto out;
           waitfor_perf_pid();
   out:
           printf("Done, error %d\n", err);
           return 0;
   }

::

   $ gcc -Wall -Wextra -g -O3 -o self-trace-example self-trace-example.c
   $ ./self-trace-example
   cmd is 'perf record --control fifo:/tmp/perf-control-FgaFCP/perf.control,/tmp/perf-control-FgaFCP/perf.ack -S -e intel_pt//u -p 13341 & echo $! > /tmp/perf-control-FgaFCP/perf_pid'
   sending cmd 'ping'
   Ack'ed
   sending cmd 'snapshot'
   Ack'ed
   sending cmd 'stop'
   Ack'ed
   [ perf record: Woken up 3 times to write data ]
   [ perf record: Captured and wrote 0.015 MB perf.data ]
   Done, error 0
   $ perf script --call-trace
    self-trace-exam 13341 [005] 17238.631440463:  psb offs: 0
    self-trace-exam 13341 [005] 17238.631440463:  cbr: 41 freq: 4105 MHz (146%)
    self-trace-exam 13341 [005] 17238.631448567: (/home/ahunter/git/self-trace-example/self-trace-e )              5617e25ba190
    self-trace-exam 13341 [005] 17238.631448983: (/home/ahunter/git/self-trace-example/self-trace-e )              5617e25ba1b0
    self-trace-exam 13341 [005] 17238.631448983: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  7fbbdde28490
    self-trace-exam 13341 [005] 17238.631449817: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631449817: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      7fbbdde283e0
    self-trace-exam 13341 [005] 17238.631450233: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              __memcmpeq_sse2
    self-trace-exam 13341 [005] 17238.631450442: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __memchr_sse2
    self-trace-exam 13341 [005] 17238.631450442: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __strcmp_sse2_unaligned
    self-trace-exam 13341 [005] 17238.631450442: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          rcmd_af
    self-trace-exam 13341 [005] 17238.631450442: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
    self-trace-exam 13341 [005] 17238.631455233: (/home/ahunter/git/self-trace-example/self-trace-e )              5617e25ba280
    self-trace-exam 13341 [005] 17238.631455442: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  7fbbdde28380
    self-trace-exam 13341 [005] 17238.631455858: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  ____wcstold_l_internal
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __memcmp_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __GI___strncasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __GI___strncasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631456275: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      rcmd_af
    self-trace-exam 13341 [005] 17238.631456483: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
    self-trace-exam 13341 [005] 17238.631462317: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __memcmp_sse2
    self-trace-exam 13341 [005] 17238.631462525: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __wcsxfrm_l
    self-trace-exam 13341 [005] 17238.631462525: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          7fbbdde28600
    self-trace-exam 13341 [005] 17238.631462525: (/home/ahunter/git/self-trace-example/self-trace-e )              5617e25ba1a0
    self-trace-exam 13341 [005] 17238.631462733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              _IO_file_seekoff@@GLIBC_2.2.5
    self-trace-exam 13341 [005] 17238.631462942: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___strncasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631463150: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631463150: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __memchr_sse2
    self-trace-exam 13341 [005] 17238.631463150: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631463150: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                              __malloc_usable_size
    self-trace-exam 13341 [005] 17238.631463150: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                                  __strcmp_sse2_unaligned
    self-trace-exam 13341 [005] 17238.631463150: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
    self-trace-exam 13341 [005] 17238.631464400: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                                      7fbbdde28380
    self-trace-exam 13341 [005] 17238.631464400: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                                      ____wcstold_l_internal
    self-trace-exam 13341 [005] 17238.631464608: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                                      ____wcstold_l_internal
    self-trace-exam 13341 [005] 17238.631466900: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                                  __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631466900: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          __memcmpeq_sse2
    self-trace-exam 13341 [005] 17238.631466900: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631467108: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
    self-trace-exam 13341 [005] 17238.631469400: (/usr/lib/x86_64-linux-gnu/libc.so.6               )              _IO_file_xsgetn_mmap
    self-trace-exam 13341 [005] 17238.631471692: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631471900: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___strcasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631471900: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __vfwscanf_internal
    self-trace-exam 13341 [005] 17238.631472108: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___strncasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631472317: (/home/ahunter/git/self-trace-example/self-trace-e )          work
    self-trace-exam 13341 [005] 17238.631472317: (/home/ahunter/git/self-trace-example/self-trace-e )          work
    self-trace-exam 13341 [005] 17238.631472317: (/home/ahunter/git/self-trace-example/self-trace-e )          work
    self-trace-exam 13341 [005] 17238.631472317: (/home/ahunter/git/self-trace-example/self-trace-e )          perf_snapshot
    self-trace-exam 13341 [005] 17238.631472317: (/home/ahunter/git/self-trace-example/self-trace-e )                  5617e25ba1e0
    self-trace-exam 13341 [005] 17238.631472317: (/home/ahunter/git/self-trace-example/self-trace-e )                  5617e25ba250
    self-trace-exam 13341 [005] 17238.631472317: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                  __GI___pthread_tpp_change_priority
    self-trace-exam 13341 [005] 17238.631472525: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          7fbbdde284d0
    self-trace-exam 13341 [005] 17238.631472525: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __GI___strncasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                              7fbbdde283e0
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          7fbbdde28490
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                              7fbbdde283e0
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          7fbbdde284d0
    self-trace-exam 13341 [005] 17238.631472733: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __strncmp_sse2
    self-trace-exam 13341 [005] 17238.631472942: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                              7fbbdde283e0
    self-trace-exam 13341 [005] 17238.631472942: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                          __memrchr_sse2
    self-trace-exam 13341 [005] 17238.631472942: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                              __strcmp_sse2_unaligned
    self-trace-exam 13341 [005] 17238.631472942: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                                  rcmd_af
    self-trace-exam 13341 [005] 17238.631472942: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
    self-trace-exam 13341 [005] 17238.631473567: (/usr/lib/x86_64-linux-gnu/libc.so.6               )                      __GI___strncasecmp_l_sse2
    self-trace-exam 13341 [005] 17238.631473567: (/home/ahunter/git/self-trace-example/self-trace-e )                  5617e25ba1c0
    self-trace-exam 13341 [005] 17238.631473775: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
    self-trace-exam 13341 [005] 17238.631475442: (/home/ahunter/git/self-trace-example/self-trace-e )                  5617e25ba210
    self-trace-exam 13341 [005] 17238.631475442: (/usr/lib/x86_64-linux-gnu/libc.so.6               )
   $

Points to note:

- The program is fairly crude and is primarily an example of how to
  control perf using FIFOs.

- Snapshot mode (option -S) is used. To also make a snapshot if the
  application terminates, the option -Se can be used instead. Also the
  Intel PT snapshot size in bytes can be specified e.g. -Se100000

- Default buffer sizes are used, but that can be changed with the -m
  option.

- The program makes a single snapshot then stops perf and waits for it
  to finish. The program could instead have made more snapshots and
  could stop and start a new perf process any time to enable the perf
  output file to be collected for later analysis or deleted.

- perf output file name is not specified and so will use (and overwrite)
  "perf.data".

- Intel PT config terms like mtc_period, psb_period, cyc, cyc_threshold,
  and noretcomp can affect the Intel PT overhead.

Example: How to read perf clock
-------------------------------

perf tools can use different clock IDs but Intel PT only supports the
default perf clock. While regular clocks can be read using POSIX
clock_gettime() API, perf clock can be calculated from TSC, however the
conversion parameters need to be read from a perf event. This is a
little awkward, but does offer the advantage of being a bit faster
because it does not have to call into the kernel to get the clock value.
Here is an example::

   $ cat perf-clock.c

.. code-block:: cpp

   #include <linux/perf_event.h>
   #include <sys/syscall.h>
   #include <sys/mman.h>
   #include <unistd.h>
   #include <stddef.h>
   #include <stdio.h>
   #include <errno.h>

   #define rmb() asm volatile("lfence" ::: "memory")

   struct perf_tsc_conversion {
           __u16 time_shift;
           __u32 time_mult;
           __u64 time_zero;
   };

   static struct perf_tsc_conversion perf_tsc_conversion;

   static int perf_read_tsc_conversion(const struct perf_event_mmap_page *pc,
                                       struct perf_tsc_conversion *tc)
   {
           int cap_user_time_zero;
           __u32 seq;
           int i = 0;

           while (1) {
                   seq = pc->lock;
                   rmb();
                   tc->time_mult = pc->time_mult;
                   tc->time_shift = pc->time_shift;
                   tc->time_zero = pc->time_zero;
                   cap_user_time_zero = pc->cap_user_time_zero;
                   rmb();
                   if (pc->lock == seq && !(seq & 1))
                           break;
                   if (++i > 10000) {
                           errno = EINVAL;
                           return -1;
                   }
           }

           if (!cap_user_time_zero) {
                   errno = EOPNOTSUPP;
                   return -1;
           }

           return 0;
   }

   static int perf_clock_init(void)
   {
           struct perf_event_attr attr = {
                   .type   = PERF_TYPE_SOFTWARE,
                   .size = sizeof(attr),
                   .config = PERF_COUNT_SW_DUMMY,
                   .disabled = 1,
                   .exclude_kernel = 1,
                   .exclude_hv = 1,
           };
           long page_size = sysconf(_SC_PAGE_SIZE);
           struct perf_event_mmap_page *pc;
           size_t len;
           int ret;
           int fd;

           if (page_size < 0)
                   return -1;
           fd = syscall(__NR_perf_event_open, &attr, 0, 0, -1, 0);
           if (fd < 0)
                   return -1;
           len = page_size * 2;
           pc = mmap(NULL, len, PROT_READ, MAP_SHARED, fd, 0);
           close(fd);
           if (pc == MAP_FAILED)
                   return -1;
           ret = perf_read_tsc_conversion(pc, &perf_tsc_conversion);
           munmap(pc, len);

           return ret;
   }

   static __u64 rdtsc_ordered(void)
   {
           unsigned int low, high;

           asm volatile("rdtscp" : "=a" (low), "=d" (high));
           asm volatile("lfence");
           return low | ((__u64)high) << 32;
   }

   static __u64 tsc_to_perf_time(__u64 cyc, const struct perf_tsc_conversion *tc)
   {
           __u64 quot, rem;

           quot = cyc >> tc->time_shift;
           rem  = cyc & (((__u64)1 << tc->time_shift) - 1);
           return tc->time_zero + quot * tc->time_mult +
                  ((rem * tc->time_mult) >> tc->time_shift);
   }

   __u64 perf_clock(void)
   {
           return tsc_to_perf_time(rdtsc_ordered(), &perf_tsc_conversion);
   }

   #define CNT 4

   int main()
   {
           unsigned long long t[CNT];
           int i;

           if (perf_clock_init()) {
                   fprintf(stderr, "Failed!\n");
                   return 1;
           }
           for (i = 0; i < CNT; i++)
                   t[i] = perf_clock();
           for (i = 0; i < CNT; i++)
                   printf("%llu\n", t[i]);
           return 0;
   }

::

   $ gcc -Wall -Wextra -o perf-clock perf-clock.c
   $ ./perf-clock
   2411326242381
   2411326242491
   2411326242570
   2411326242648
   $

Points to note:

- For systems that support Intel PT, the conversion parameters should
  not change while the system is running. Notable exceptions are
  Hibernation (because it actually involves a reboot), and inside a
  Virtual Machine (because TSC might be virtualized).

Example: Tracing kernel raw spin locks
--------------------------------------

In a v6.2 kernel, the raw spin lock functions are conveniently
together::

   $ sudo cat /proc/kallsyms | sort | grep -C30 ' _raw_spin_lock'
   ffffffffacd2e940 t native_safe_halt
   ffffffffacd2e960 t __pfx_native_halt
   ffffffffacd2e970 t native_halt
   ffffffffacd2e990 t __pfx_cpu_idle_poll.isra.0
   ffffffffacd2e9a0 t cpu_idle_poll.isra.0
   ffffffffacd2eaa0 T __pfx_default_idle_call
   ffffffffacd2eab0 T default_idle_call
   ffffffffacd2ebb0 t __pfx_intel_idle_s2idle
   ffffffffacd2ebc0 t intel_idle_s2idle
   ffffffffacd2ec40 t __pfx_intel_idle_xstate
   ffffffffacd2ec50 t intel_idle_xstate
   ffffffffacd2eca0 t __pfx_intel_idle_irq
   ffffffffacd2ecb0 t intel_idle_irq
   ffffffffacd2ed10 t __pfx_intel_idle
   ffffffffacd2ed20 t intel_idle
   ffffffffacd2ed70 t __pfx_intel_idle_ibrs
   ffffffffacd2ed80 t intel_idle_ibrs
   ffffffffacd2ee40 t __pfx_acpi_idle_do_entry
   ffffffffacd2ee50 t acpi_idle_do_entry
   ffffffffacd2eeb0 t __pfx_acpi_idle_enter_bm
   ffffffffacd2eec0 t acpi_idle_enter_bm
   ffffffffacd2f0c0 t __pfx_acpi_idle_enter
   ffffffffacd2f0d0 t acpi_idle_enter
   ffffffffacd2f240 t __pfx_acpi_idle_enter_s2idle
   ffffffffacd2f250 t acpi_idle_enter_s2idle
   ffffffffacd2f330 t __pfx_poll_idle
   ffffffffacd2f340 t poll_idle
   ffffffffacd2f3fa T __cpuidle_text_end
   ffffffffacd2f400 T __lock_text_start
   ffffffffacd2f400 T __pfx__raw_spin_lock_irqsave
   ffffffffacd2f410 T _raw_spin_lock_irqsave
   ffffffffacd2f470 T __pfx__raw_spin_trylock
   ffffffffacd2f480 T _raw_spin_trylock
   ffffffffacd2f4e0 T __pfx__raw_spin_unlock_irqrestore
   ffffffffacd2f4f0 T _raw_spin_unlock_irqrestore
   ffffffffacd2f540 T __pfx__raw_read_trylock
   ffffffffacd2f550 T _raw_read_trylock
   ffffffffacd2f5c0 T __pfx__raw_write_trylock
   ffffffffacd2f5d0 T _raw_write_trylock
   ffffffffacd2f630 T __pfx__raw_read_unlock
   ffffffffacd2f640 T _raw_read_unlock
   ffffffffacd2f680 T __pfx__raw_write_unlock
   ffffffffacd2f690 T _raw_write_unlock
   ffffffffacd2f6c0 T __pfx__raw_read_unlock_irq
   ffffffffacd2f6d0 T _raw_read_unlock_irq
   ffffffffacd2f710 T __pfx__raw_write_unlock_irq
   ffffffffacd2f720 T _raw_write_unlock_irq
   ffffffffacd2f760 T __pfx__raw_read_unlock_irqrestore
   ffffffffacd2f770 T _raw_read_unlock_irqrestore
   ffffffffacd2f7c0 T __pfx__raw_write_unlock_irqrestore
   ffffffffacd2f7d0 T _raw_write_unlock_irqrestore
   ffffffffacd2f810 T __pfx__raw_read_unlock_bh
   ffffffffacd2f820 T _raw_read_unlock_bh
   ffffffffacd2f840 T __pfx__raw_write_unlock_bh
   ffffffffacd2f850 T _raw_write_unlock_bh
   ffffffffacd2f870 T __pfx__raw_read_lock_irqsave
   ffffffffacd2f880 T _raw_read_lock_irqsave
   ffffffffacd2f8e0 T __pfx__raw_write_lock_irqsave
   ffffffffacd2f8f0 T _raw_write_lock_irqsave
   ffffffffacd2f950 T __pfx__raw_spin_trylock_bh
   ffffffffacd2f960 T _raw_spin_trylock_bh
   ffffffffacd2f9b0 T __pfx__raw_write_lock
   ffffffffacd2f9c0 T _raw_write_lock
   ffffffffacd2fa00 T __pfx__raw_read_lock
   ffffffffacd2fa10 T _raw_read_lock
   ffffffffacd2fa50 T __pfx__raw_read_lock_bh
   ffffffffacd2fa60 T _raw_read_lock_bh
   ffffffffacd2faa0 T __pfx__raw_write_lock_bh
   ffffffffacd2fab0 T _raw_write_lock_bh
   ffffffffacd2faf0 T __pfx__raw_write_lock_nested
   ffffffffacd2fb00 T _raw_write_lock_nested
   ffffffffacd2fb40 T __pfx__raw_read_lock_irq
   ffffffffacd2fb50 T _raw_read_lock_irq
   ffffffffacd2fb90 T __pfx__raw_write_lock_irq
   ffffffffacd2fba0 T _raw_write_lock_irq
   ffffffffacd2fbe0 T __pfx__raw_spin_lock
   ffffffffacd2fbf0 T _raw_spin_lock
   ffffffffacd2fc30 T __pfx__raw_spin_lock_bh
   ffffffffacd2fc40 T _raw_spin_lock_bh
   ffffffffacd2fc80 T __pfx__raw_spin_lock_irq
   ffffffffacd2fc90 T _raw_spin_lock_irq
   ffffffffacd2fce0 T __pfx__raw_spin_unlock_bh
   ffffffffacd2fcf0 T _raw_spin_unlock_bh
   ffffffffacd2fd10 T __pfx__raw_spin_unlock
   ffffffffacd2fd20 T _raw_spin_unlock
   ffffffffacd2fd60 T __pfx__raw_spin_unlock_irq
   ffffffffacd2fd70 T _raw_spin_unlock_irq
   ffffffffacd2fdb0 T __pfx___raw_callee_save___pv_queued_spin_unlock_slowpath
   ffffffffacd2fdc0 T __raw_callee_save___pv_queued_spin_unlock_slowpath
   ffffffffacd2fdf0 T __pfx___raw_callee_save___pv_queued_spin_unlock
   ffffffffacd2fe00 T __raw_callee_save___pv_queued_spin_unlock
   ffffffffacd2fe1a t .slowpath
   ffffffffacd2fe30 T __pfx_native_queued_spin_lock_slowpath
   ffffffffacd2fe40 T native_queued_spin_lock_slowpath
   ffffffffacd30160 T __pfx___pv_queued_spin_lock_slowpath
   ffffffffacd30170 T __pv_queued_spin_lock_slowpath
   ffffffffacd30530 T __pfx___pv_queued_spin_unlock_slowpath
   ffffffffacd30540 T __pv_queued_spin_unlock_slowpath
   ffffffffacd306c0 T __pfx_queued_read_lock_slowpath
   ffffffffacd306d0 T queued_read_lock_slowpath
   ffffffffacd30800 T __pfx_queued_write_lock_slowpath
   ffffffffacd30810 T queued_write_lock_slowpath
   ffffffffacd30942 T __lock_text_end
   ffffffffacd30950 T __kprobes_text_end
   ffffffffacd30950 T __kprobes_text_start
   ffffffffacd30950 T __pfx___do_softirq
   ffffffffacd30950 T __softirqentry_text_start
   ffffffffacd30960 T __do_softirq
   ffffffffacd30ccd T __indirect_thunk_start
   ffffffffacd30ccd T __softirqentry_text_end
   ffffffffacd30ce0 T __x86_indirect_thunk_array

So an address filter can be used from \__lock_text_start to
\__lock_text_end.

For this example, fio will be used as a workload, with a runtime of just
1 second::

   $ cat fio-rand-read.fio
   [global]
   name=fio-rand-read
   filename=fio-read
   allow_file_create=0
   rw=randread
   bs=4K
   direct=1
   numjobs=8
   time_based=1
   runtime=1
   group_reporting=1

   [file1]
   size=1G
   ioengine=libaio
   iodepth=128

To reduce overflows, the mtc_period is increased to maximum (9 in this
case), and to get accurate timing, cycle accurate mode is used.

The NMI handler (asm_exc_nmi in this case) is also traced to see if
there are NMIs during the spin locking.

And the lock contention trace points are added to show up lock
contention::

   $ sudo perf record --kcore -a  -e intel_pt/cyc,mtc_period=9/k --filter 'filter __lock_text_start / __lock_text_end , filter asm_exc_nmi' -e lock:*  -- fio ./fio-rand-read.fio
   file1: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
   ...
   fio-3.28
   Starting 4 processes
   Jobs: 4 (f=4)
   file1: (groupid=0, jobs=4): err= 0: pid=9051: Sun Apr  2 13:29:59 2023
     read: IOPS=92.7k, BW=362MiB/s (380MB/s)(365MiB/1007msec)
       slat (nsec): min=945, max=183448, avg=1376.99, stdev=1321.13
       clat (usec): min=734, max=13668, avg=5512.61, stdev=891.87
        lat (usec): min=739, max=13670, avg=5514.05, stdev=891.74
       clat percentiles (usec):
        |  1.00th=[ 2868],  5.00th=[ 4293], 10.00th=[ 4621], 20.00th=[ 4883],
        | 30.00th=[ 5145], 40.00th=[ 5276], 50.00th=[ 5473], 60.00th=[ 5669],
        | 70.00th=[ 5866], 80.00th=[ 6128], 90.00th=[ 6521], 95.00th=[ 6915],
        | 99.00th=[ 7767], 99.50th=[ 8356], 99.90th=[11076], 99.95th=[11731],
        | 99.99th=[12649]
      bw (  KiB/s): min=370784, max=372152, per=100.00%, avg=371468.00, stdev=233.09, samples=8
      iops        : min=92696, max=93038, avg=92867.00, stdev=58.27, samples=8
     lat (usec)   : 750=0.01%, 1000=0.02%
     lat (msec)   : 2=0.31%, 4=2.55%, 10=96.93%, 20=0.19%
     cpu          : usr=1.99%, sys=6.98%, ctx=84145, majf=0, minf=564
     IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=99.7%
        submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
        complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
        issued rwts: total=93375,0,0,0 short=0,0,0,0 dropped=0,0,0,0
        latency   : target=0, window=0, percentile=100.00%, depth=128

   Run status group 0 (all jobs):
      READ: bw=362MiB/s (380MB/s), 362MiB/s-362MiB/s (380MB/s-380MB/s), io=365MiB (382MB), run=1007-1007msec

   Disk stats (read/write):
     nvme0n1: ios=83164/0, merge=0/0, ticks=456685/0, in_queue=456685, util=89.88%
   [ perf record: Woken up 147 times to write data ]
   [ perf record: Captured and wrote 109.608 MB perf.data ]

In this case 109 MB of data is manageable to process.

Check the errors::

   $ sudo perf script --itrace=e | grep error
    instruction trace error type 1 time 17126.002595342 cpu 0 pid 0 tid 0 ip 0xffffffffabdb5ac5 code 7: Overflow packet
    instruction trace error type 1 time 17126.203301006 cpu 7 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
    instruction trace error type 1 time 17126.240656254 cpu 4 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
    instruction trace error type 1 time 17126.394446279 cpu 5 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
    instruction trace error type 1 time 17126.424700733 cpu 5 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
    instruction trace error type 1 time 17126.615614220 cpu 7 pid 82 tid 82 ip 0xffffffffacc7597e code 7: Overflow packet
    instruction trace error type 1 time 17126.788347553 cpu 5 pid 1115 tid 1320 ip 0xffffffffc062f8c2 code 7: Overflow packet
    instruction trace error type 1 time 17126.863253778 cpu 0 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
    instruction trace error type 1 time 17126.906410486 cpu 2 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
    instruction trace error type 1 time 17127.091705727 cpu 2 pid 0 tid 0 ip 0xffffffffabdb5944 code 7: Overflow packet
   Warning:
   10 instruction trace errors
   $

All the errors are overflows. The overflows represent small losses of
trace data, so we are not seeing a perfect picture, however overflows
clear fast (e.g. 10us) so it is not too bad.

Because spin lock functions run to completion on the same CPU, a
relatively simple script can be used to analyze the times:

.. code-block:: python

   from perf_trace_context import perf_set_itrace_options

   glb_last = {}
   glb_data = []
   glb_count = 0

   def get_optional(perf_dict, field, dflt):
       if field in perf_dict:
           return perf_dict[field]
       return dflt

   def process_event(param_dict):
       name = param_dict["ev_name"]
       if name[0:8] != "branches":
           return
       sample      = param_dict["sample"]
       flags       = get_optional(sample, "flags", "")
       addr_symbol = get_optional(sample, "addr_symbol", "[unknown]")
       addr_symoff = get_optional(sample, "addr_symoff", "[unknown]")
       symbol      = get_optional(param_dict, "symbol", "[unknown]")
       ts          = sample["time"]
       cpu         = sample["cpu"]
       if "B" in flags and addr_symoff == 0:
           # Address filter beginning trace, so record call information
           glb_last[cpu] = (ts, addr_symbol)
       elif "E" in flags and "r" in flags and cpu in glb_last:
           call_ts, call_sym = glb_last[cpu]
           if call_sym == symbol:
               # Address filter ending trace, so record return information
               glb_data.append((cpu, call_ts, ts, symbol, addr_symbol, addr_symoff))
               del glb_last[cpu]
       global glb_count
       glb_count += 1
       if glb_count % 100000 == 0:
           print("\rProcessed", glb_count, end = "")

   def trace_begin():
       perf_set_itrace_options(perf_script_context, "be")

   def timestamp(x):
       s = str(x)
       r = s[-9:]
       l = s[:len(s) - len(r)]
       if len(l):
           return l + "." + ("000000000" + r)[-9:]
       return "0." + ("000000000" + r)[-9:]

   def trace_end():
       print("\rProcessed", glb_count)
       print("Aggregating...")
       syms = {}
       least = 0
       topn = []
       N = 10
       # Aggregate by symbol
       for cpu, call_ts, ret_ts, symbol, from_sym, from_symoff in glb_data:
           if symbol not in syms:
               syms[symbol] = (0, 0, 0, -1)
           tot, cnt, max, min = syms[symbol]
           dur = ret_ts - call_ts
           tot += dur
           cnt += 1
           if dur > max:
               max = dur
           if min == -1 or dur < min:
               min = dur
           syms[symbol] = (tot, cnt, max, min)
           if dur > least:
               topn.append((dur, cpu, call_ts, ret_ts, symbol, from_sym, from_symoff))
               topn = sorted(topn, key=lambda x: x[0])
               n = len(topn)
               if n > N:
                   del topn[0]
               least = topn[0][0]
       print("\nFunction duration:")
       print("Symbol                                        Count     Avg (ns)   Max (ns)   Min (ns)")
       for x in sorted(syms.keys()):
           tot, cnt, max, min = syms[x]
           print("%-40s %10u %12.1f %10u %10u" % (x, cnt, tot / cnt, max, min))
       print("\nTop", N, ":")
       print("Symbol                                Duration (ns)  CPU            Start              End  Returns to")
       topn.reverse()
       for d in topn:
           dur, cpu, call_ts, ret_ts, symbol, from_sym, from_symoff = d
           call_ts = str(call_ts)
           call_ts = call_ts
           print("%-40s %10u %4u %16s %16s  %s+%#x" % (symbol, dur, cpu, timestamp(call_ts), timestamp(ret_ts), from_sym, from_symoff))

   def auxtrace_error(typ, code, cpu, pid, tid, ip, ts, msg, cpumode, *x):
       if cpu in glb_last:
           del glb_last[cpu]

The results are::

   $ time sudo perf script fn-durations.py
   ProcessedWarning:0
   10 instruction trace errors
   Processed 15804761
   Aggregating...

   Function duration:
   Symbol                                        Count     Avg (ns)   Max (ns)   Min (ns)
   _raw_read_lock                               242755         23.2      14119          8
   _raw_read_lock_bh                                60         35.5        140          8
   _raw_read_lock_irqsave                          122         45.5        338          7
   _raw_read_unlock                             242778          8.2       2050          5
   _raw_read_unlock_irqrestore                     122         10.5        113          5
   _raw_spin_lock                               767672         11.8      19064          4
   _raw_spin_lock_bh                                62         19.6        135          9
   _raw_spin_lock_irq                            36660         15.7       2467          5
   _raw_spin_lock_irqsave                       808035         15.5      93655          6
   _raw_spin_trylock                             32357         11.2       1371          5
   _raw_spin_unlock                             799630          3.9       1946          1
   _raw_spin_unlock_irq                          31435          5.8        506          2
   _raw_spin_unlock_irqrestore                  813481          4.6       3790          1
   _raw_write_lock                               26842         11.8        266          7
   _raw_write_lock_bh                               34         15.2         42          6
   _raw_write_lock_irq                             175         26.7        354          9
   _raw_write_unlock                             26841          3.3        289          1
   _raw_write_unlock_irq                           177          6.7        194          2

   Top 10 :
   Symbol                                Duration (ns)  CPU            Start              End  Returns to
   _raw_spin_lock_irqsave                        93655    5  17127.007313673  17127.007407328  fwtable_write32+0x63
   _raw_spin_lock                                19064    2  17126.745592947  17126.745612011  tick_do_update_jiffies64+0x25
   _raw_spin_lock                                14544    1  17127.151604551  17127.151619095  __queue_work+0x18c
   _raw_spin_lock                                14363    1  17127.087603311  17127.087617674  handle_irq_event+0x51
   _raw_spin_lock                                14123    5  17127.084610854  17127.084624977  raw_spin_rq_lock_nested+0x23
   _raw_read_lock                                14119    2  17127.085605857  17127.085619976  ext4_es_lookup_extent+0x55
   _raw_spin_lock_irqsave                        13484    1  17127.073606372  17127.073619856  hrtimer_next_event_without+0x43
   _raw_spin_lock                                11660    4  17126.615990730  17126.616002390  handle_edge_irq+0x1f
   _raw_spin_lock_irqsave                        11256    0  17126.014614808  17126.014626064  hrtimer_get_next_event+0x3e
   _raw_spin_lock                                 9598    1  17126.617729578  17126.617739176  handle_edge_irq+0x1f

   real    1m50.786s
   user    0m0.005s
   sys     0m0.012s

Note the processing of 109 MB took nearly 2 minutes.

The average lock times are very low, and consequently not very
interesting.

To look at the worst case::

   $ sudo perf script --itrace=crpe -F+flags,-dso,+addr,-comm,-tid,-period --ns  --time 17127.007313673,17127.007407328 -C 5
   [005] 17127.007313673:                         cbr:                          cbr: 41 freq: 4105 MHz (146%)    ffffffffacd2f410                0 [unknown]
   [005] 17127.007313673:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2f410 _raw_spin_lock_irqsave+0x0
   [005] 17127.007313675:                  branches:k:   tr end  call           ffffffffacd2f433 _raw_spin_lock_irqsave+0x23 => ffffffffabd168b0 preempt_count_add+0x0
   [005] 17127.007313677:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2f438 _raw_spin_lock_irqsave+0x28
   [005] 17127.007313709:                  branches:k:   call                   ffffffffacd2f455 _raw_spin_lock_irqsave+0x45 => ffffffffacd2fe40 native_queued_spin_lock_slowpath+0x0
   [005] 17127.007407328:                  branches:k:   return                 ffffffffacd2fed1 native_queued_spin_lock_slowpath+0x91 => ffffffffacd2f45a _raw_spin_lock_irqsave+0x4a
   [005] 17127.007407328:                  branches:k:   tr end  return         ffffffffacd2f461 _raw_spin_lock_irqsave+0x51 => ffffffffc07ed873 fwtable_write32+0x63

The call to native_queued_spin_lock_slowpath() indicates a contended
spin lock. Interestingly, the lock contention tracepoint is missing in
the case. A review of the source code of queued_spin_lock_slowpath()
shows why: the tracepoints are only triggered in the "queue:" path.
Checking the address shows it spinning instead here when the "queue:"
path has not been taken:

.. code-block:: cpp

       /*
        * We're pending, wait for the owner to go away.
        *
        * 0,1,1 -> 0,1,0
        *
        * this wait loop must be a load-acquire such that we match the
        * store-release that clears the locked bit and create lock
        * sequentiality; this is because not all
        * clear_pending_set_locked() implementations imply full
        * barriers.
        */
       if (val & _Q_LOCKED_MASK)
           atomic_cond_read_acquire(&lock->val, !(VAL & _Q_LOCKED_MASK));

A look at the next worst case shows it was interrupted by an NMI::

   $ sudo perf script --itrace=crpe -F+flags,-dso,+addr,-comm,-tid,-period --ns  --time 17126.745592947,17126.745612011 -C 2
   [002] 17126.745592947:                         cbr:                          cbr:  4 freq:  400 MHz ( 14%)    ffffffffacd2fbf0                0 [unknown]
   [002] 17126.745592947:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fbf0 _raw_spin_lock+0x0
   [002] 17126.745592952:                  branches:k:   tr end  call           ffffffffacd2fc02 _raw_spin_lock+0x12 => ffffffffabd168b0 preempt_count_add+0x0
   [002] 17126.745592957:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fc07 _raw_spin_lock+0x17
   [002] 17126.745594638:                         psb:                          psb offs: 0x800190                              0 ffffffffacd2fc12 _raw_spin_lock+0x22
   [002] 17126.745596360:                  branches:k:   hw int                 ffffffffacd2fc12 _raw_spin_lock+0x22 => fffffffface01cd0 asm_exc_nmi+0x0
   [002] 17126.745596430:                  branches:k:   tr end  jcc            fffffffface01df4 first_nmi+0x0 => fffffffface01df4 first_nmi+0x0
   [002] 17126.745611938:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fc12 _raw_spin_lock+0x22
   [002] 17126.745612011:                  branches:k:   tr end

A look at the next worst case shows it was undergoing a change in CPU
frequency from 400 MHz to 1301 MHz::

   $ sudo perf script --itrace=crpe -F+flags,-dso,+addr,-comm,-tid,-period --ns  --time 17127.151604551,17127.151619095 -C 1
   [001] 17127.151604551:                         cbr:                          cbr:  4 freq:  400 MHz ( 14%)    ffffffffacd2fbf0                0 [unknown]
   [001] 17127.151604551:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fbf0 _raw_spin_lock+0x0
   [001] 17127.151604581:                  branches:k:   tr end  call           ffffffffacd2fc02 _raw_spin_lock+0x12 => ffffffffabd168b0 preempt_count_add+0x0
   [001] 17127.151604618:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fc07 _raw_spin_lock+0x17
   [001] 17127.151612757:                         cbr:   call                   cbr: 13 freq: 1301 MHz ( 46%)                   0 ffffffffacd2fc07 _raw_spin_lock+0x17
   [001] 17127.151619095:                  branches:k:   tr end  return         ffffffffacd2fc15 _raw_spin_lock+0x25 => ffffffffabcf900c __queue_work+0x18c

A look at the next worst case shows it really did take that long::

   $ sudo perf script --itrace=bpe -F+flags,-dso,+addr,-comm,-tid,-period --ns  --time 17127.087603311,17127.087617674
   [001] 17127.087603311:                         cbr:                          cbr: 41 freq: 4105 MHz (146%)    ffffffffacd2fbf0                0 [unknown]
   [001] 17127.087603311:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fbf0 _raw_spin_lock+0x0
   [001] 17127.087603312:                  branches:k:   tr end  call           ffffffffacd2fc02 _raw_spin_lock+0x12 => ffffffffabd168b0 preempt_count_add+0x0
   [001] 17127.087603447:                  branches:k:   tr strt                               0 [unknown] => ffffffffacd2fc07 _raw_spin_lock+0x17
   [001] 17127.087617674:                  branches:k:   tr end  return         ffffffffacd2fc15 _raw_spin_lock+0x25 => ffffffffabd72381 handle_irq_event+0x51

Looking at the instructions, presumably it was the atomic operation
"lock cmpxchgl %edx, (%rbx)" that took unusually long::

   $ sudo perf script --itrace=i0nse  -F-dso,+addr,-comm,-tid,-period,+insn --xed --ns  --time 17127.087603311,17127.087617674
   [001] 17127.087603311:              instructions:k:                0 ffffffffacd2fbf0 _raw_spin_lock+0x0                nop %edi, %edx
   [001] 17127.087603311:              instructions:k:                0 ffffffffacd2fbf4 _raw_spin_lock+0x4                nopl  %eax, (%rax,%rax,1)
   [001] 17127.087603311:              instructions:k:                0 ffffffffacd2fbf9 _raw_spin_lock+0x9                pushq  %rbx
   [001] 17127.087603311:              instructions:k:                0 ffffffffacd2fbfa _raw_spin_lock+0xa                mov %rdi, %rbx
   [001] 17127.087603311:              instructions:k:                0 ffffffffacd2fbfd _raw_spin_lock+0xd                mov $0x1, %edi
   [001] 17127.087603312:              instructions:k: ffffffffabd168b0 ffffffffacd2fc02 _raw_spin_lock+0x12               callq  0xffffffffaacfd55e
   [001] 17127.087603447:              instructions:k:                0 ffffffffacd2fc07 _raw_spin_lock+0x17               xor %eax, %eax
   [001] 17127.087603447:              instructions:k:                0 ffffffffacd2fc09 _raw_spin_lock+0x19               mov $0x1, %edx
   [001] 17127.087603447:              instructions:k:                0 ffffffffacd2fc0e _raw_spin_lock+0x1e               lock cmpxchgl  %edx, (%rbx)
   [001] 17127.087617673:              instructions:k:                0 ffffffffacd2fc12 _raw_spin_lock+0x22               jnz 0xffffffffacd2fc1a
   [001] 17127.087617673:              instructions:k:                0 ffffffffacd2fc14 _raw_spin_lock+0x24               popq  %rbx
   [001] 17127.087617674:              instructions:k: ffffffffabd72381 ffffffffacd2fc15 _raw_spin_lock+0x25               retq

Example: How to run perf on a different CPU to the workload
-----------------------------------------------------------

The taskset utility can be used to run perf on a different CPU to the
workload. For example, in a 8 CPU system, to run perf on CPU 0 and the
workload on CPUs 1 to 7::

   $ taskset --cpu 0 perf record -e intel_pt//u -- taskset --cpu 1-7 uname
   Linux
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.049 MB perf.data ]

Here it can be seen perf is restricted to CPU 0 but uname is running on
CPU 2::

   $ perf script --itrace=bieqq
          perf-exec 11353 [000] 26597.875924:          1 instructions:  ffffffffabca5248 native_write_msr+0x8 ([kernel.kallsyms])
            taskset 11353 [000] 26597.876033:          1 instructions:  ffffffffac302b6d ima_file_mmap+0x2d ([kernel.kallsyms])
            taskset 11353 [000] 26597.876130:          1 instructions:  fffffffface01264 asm_exc_page_fault+0x4 ([kernel.kallsyms])
            taskset 11353 [000] 26597.876201:          1 instructions:      7f3f4be379cc _nl_intern_locale_data+0xdc (/usr/lib/x86_64-linux-gnu/libc.so.6)
            taskset 11353 [002] 26597.876280:          1 instructions:  ffffffffabca5248 native_write_msr+0x8 ([kernel.kallsyms])
            taskset 11353 [002] 26597.876386:          1 instructions:  ffffffffabd92645 __rcu_read_unlock+0x25 ([kernel.kallsyms])
            taskset 11353 [002] 26597.876460:          1 instructions:  ffffffffabf2a9ba folio_mark_accessed+0x2a ([kernel.kallsyms])
              uname 11353 [002] 26597.876547:          1 instructions:      7f8dcb95f66f __GI___tunables_init+0x10f (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2)
              uname 11353 [002] 26597.876663:          1 instructions:      7f8dcb9594e8 _dl_relocate_object+0x788 (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2)
              uname 11353 [002] 26597.876752:          1 instructions:  ffffffffabf196d1 filemap_map_pages+0x651 ([kernel.kallsyms])
              uname 11353 [002] 26597.876817:          1 instructions:  ffffffffabf2a9ba folio_mark_accessed+0x2a ([kernel.kallsyms])

Example: Intel PT Event Trace
-----------------------------

Some processors like Alder Lake-N support Intel PT Event Trace. Whether
it is supported can be determined by the event_trace capability::

   $ cat /sys/bus/event_source/devices/intel_pt/caps/event_trace
   1

"1" means it is supported. "0" means it is not supported. If the file is
missing, then the kernel does not have support.

Intel PT Event Trace : Asynchronous Events
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Intel PT Event Trace records asynchronous events like interrupts,
exceptions, faults, and NMI. Branch tracing can be done at the same time
but is not necessary.

We can use ``perf record`` with options:

- ``-e`` to select which events, i.e. the following:
- ``intel_pt/event,branch=0/u`` to get Intel PT Event Trace and
  disable branch tracing, user space only.
- ``uname`` is the workload.

::

   $ perf record -e intel_pt/event,branch=0/u uname
   Linux
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.031 MB perf.data ]

To list the events, use ``perf script`` with options:

-  ``--itrace=eI`` to show errors (e) and events (I)
-  ``--ns`` to show the timestamp to nanoseconds instead of the default microseconds

The output shows INTR for interrupts, exceptions, faults, and NMI. In
this case the PFA (Page Fault Linear Address) indicates that they are
page faults. IRET indicates a return to user space.

::

   $ perf script --itrace=Ie --ns | head -10
              uname   253 [007]  1719.062059042:                       evt:  cfe: IRET IP: 0 vector: 0                        0 [unknown] ([unknown])
              uname   253 [007]  1719.062059042:                     iflag:  IFLAG: 0->1 non branch                7f7371368059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062060708:                       evt:  cfe: INTR IP: 1 vector: 14 PFA: 0x7f7371368059     7f7371368059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062065292:                       evt:  cfe: IRET IP: 0 vector: 0             7f7371368059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062068000:                       evt:  cfe: INTR IP: 1 vector: 14 PFA: 0x7f7371366004     7f737136bbdb [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062070083:                       evt:  cfe: IRET IP: 0 vector: 0             7f737136bbdb [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062071333:                       evt:  cfe: INTR IP: 1 vector: 14 PFA: 0x7f737136dff8     7f737136bc42 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062073417:                       evt:  cfe: IRET IP: 0 vector: 0             7f737136bc42 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062078417:                       evt:  cfe: INTR IP: 1 vector: 14 PFA: 0x7f737136de88     7f737136bcd3 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   253 [007]  1719.062084250:                       evt:  cfe: IRET IP: 0 vector: 0             7f737136bcd3 [unknown] (/lib/ld64-uClibc-1.0.39.so)

The timestamps are accurate only to the MTC period. By default,
mtc_period is 3 which means 8 (2^3) times the period of ART (Always
Running Timer). We can get the information needed to determine the ART
frequency as follows::

   $ cat /sys/bus/event_source/devices/intel_pt/tsc_art_ratio
   84:2
   $ dmesg | grep TSC
   [    0.000000] tsc: Detected 1612.800 MHz TSC
   [    0.018522] TSC deadline timer available

ART frequency is 1612.800 MHz / 42. MTC period is 8 / ART frequency,
which is 208 ns. So, in this case, the timestamps are accurate to +/-208
ns. It is important to note that this cannot be improved by using
Cycle-Accurate Mode alone. That is because the CFE and EVT packets are
not CYC-eligible. To get more accurate timestamps, we need to do a
branch trace also, but we do not need TNT packets to get timestamps on
asynchronous or indirect branches, so the ``perf record`` command
becomes::

   $ perf record -e intel_pt/event,cyc,notnt/u uname
   Linux
   [ perf record: Woken up 2 times to write data ]
   [ perf record: Captured and wrote 0.036 MB perf.data ]

Now the corresponding branches can be displayed also, with more accurate
timestamps.

::

   $ perf script --itrace=eIb --ns | head -11
              uname   274 [003]  4513.329990775:                        evt:  cfe: IRET IP: 0 vector: 0                        0 [unknown] ([unknown])
              uname   274 [003]  4513.329990794:                      iflag:  IFLAG: 0->1 via branch                           0 [unknown] ([unknown])
              uname   274 [003]  4513.329990794:          1                branches:uH:                 0 [unknown] ([unknown]) =>     7fdd2d39b059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   274 [003]  4513.329991489:                        evt:  cfe: INTR IP: 1 vector: 14 PFA: 0x7fdd2d39b059     7fdd2d39b059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   274 [003]  4513.329991489:          1                branches:uH:      7fdd2d39b059 [unknown] (/lib/ld64-uClibc-1.0.39.so) =>                0 [unknown] ([unknown])
              uname   274 [003]  4513.329993275:                        evt:  cfe: IRET IP: 0 vector: 0             7fdd2d39b059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   274 [003]  4513.329993391:          1                branches:uH:                 0 [unknown] ([unknown]) =>     7fdd2d39b059 [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   274 [003]  4513.329994447:                        evt:  cfe: INTR IP: 1 vector: 14 PFA: 0x7ffe3db37de0     7fdd2d39eb6b [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   274 [003]  4513.329994447:          1                branches:uH:      7fdd2d39eb6b [unknown] (/lib/ld64-uClibc-1.0.39.so) =>                0 [unknown] ([unknown])
              uname   274 [003]  4513.329995775:                        evt:  cfe: IRET IP: 0 vector: 0             7fdd2d39eb6b [unknown] (/lib/ld64-uClibc-1.0.39.so)
              uname   274 [003]  4513.329995950:          1                branches:uH:                 0 [unknown] ([unknown]) =>     7fdd2d39eb6b [unknown] (/lib/ld64-uClibc-1.0.39.so)

Tracing JIT-compiled code like Java™ with OpenJDK
-------------------------------------------------

Intel PT decoding needs to walk the object code to reconstruct control
flow from trace packets, so how does this work if the code is created at
runtime by a JIT compiler.

The answer is that some JIT compilers like OpenJDK Java provide an
interface to record the JIT-compiled code. For Java it is called Java™
Virtual Machine Tool Interface (JVM TI). ``perf`` provides library
libperf-jvmti.so to make use of the interface. To make it work:

- java option ``-agentpath`` is used to tell java to use
  libperf-jvmti.so which produces a **jitdump file**
- ``perf inject`` must be used to inject information from the
  **jitdump file** into the output perf.data file

Below is a small example using HelloWorldApp from The Java™ Tutorials.
Note that the recorded perf.data file is called java.perf.data and the
injected perf.data file is called java.perf.data.jitted::

   $ cat HelloWorldApp.java

.. code-block:: java

   /**
    * The HelloWorldApp class implements an application that
    * simply prints "Hello World!" to standard output.
    */
   class HelloWorldApp {
       public static void main(String[] args) {
           System.out.println("Hello World!"); // Display the string.
       }
   }

::

   $ javac HelloWorldApp.java
   $ perf record -m,16M -e intel_pt//u -o java.perf.data java -agentpath:$HOME/lib64/libperf-jvmti.so HelloWorldApp
   java: jvmti: jitdump in /home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jit-162584.dump
   Hello World!
   [ perf record: Woken up 3 times to write data ]
   [ perf record: Captured and wrote 15.469 MB java.perf.data ]
   $ perf inject -i java.perf.data --jit -o java.perf.data.jitted
   $ perf script --itrace=qqi -i java.perf.data.jitted | grep jitted | head
               java  162586 [006] 779113.252541:          1 instructions:uH:      7f7bc860008b flush_icache_stub+0xb (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-1.so)
               java  162586 [006] 779113.257527:          1 instructions:uH:      7f7bc8614007 Interpreter+0xb487 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259374:          1 instructions:uH:      7f7bc8611617 Interpreter+0x8a97 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259391:          1 instructions:uH:      7f7bc8618c3d Interpreter+0x100bd (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259411:          1 instructions:uH:      7f7bc86146a7 Interpreter+0xbb27 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259426:          1 instructions:uH:      7f7bc861a823 Interpreter+0x11ca3 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259444:          1 instructions:uH:      7f7bc8618b3f Interpreter+0xffbf (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259458:          1 instructions:uH:      7f7bc8621514 Interpreter+0x18994 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259473:          1 instructions:uH:      7f7bc860f684 Interpreter+0x6b04 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
               java  162586 [006] 779113.259489:          1 instructions:uH:      7f7bc861e401 Interpreter+0x15881 (/home/ahunter/.debug/jit/java-jit-20230920.XXVLk6o0/jitted-162584-29.so)
   $

**Note** that OpenJDK does **not** provide all JIT-compiled code via
JVMTI so there are still decode errors::

   $ perf script --itrace=e -i java.perf.data.jitted | head
    instruction trace error type 1 time 779113.256448669 cpu 6 pid 162584 tid 162586 ip 0x7f7bc8657180 code 5: Failed to get instruction
    instruction trace error type 1 time 779113.256449353 cpu 6 pid 162584 tid 162586 ip 0x7f7be7902b8c code 6: Trace doesn't match instruction
    instruction trace error type 1 time 779113.256562002 cpu 6 pid 162584 tid 162586 ip 0x7f7bc8657180 code 5: Failed to get instruction
    instruction trace error type 1 time 779113.256567103 cpu 6 pid 162584 tid 162586 ip 0x7f7bc860fa2e code 6: Trace doesn't match instruction
    instruction trace error type 1 time 779113.256567276 cpu 6 pid 162584 tid 162586 ip 0x7f7be75739df code 6: Trace doesn't match instruction
    instruction trace error type 1 time 779113.256567282 cpu 6 pid 162584 tid 162586 ip 0x7f7be7e3797e code 6: Trace doesn't match instruction
    instruction trace error type 1 time 779113.256567458 cpu 6 pid 162584 tid 162586 ip 0x7f7be787eeb8 code 6: Trace doesn't match instruction
    instruction trace error type 1 time 779113.256590771 cpu 6 pid 162584 tid 162586 ip 0x7f7bc8657180 code 5: Failed to get instruction
    instruction trace error type 1 time 779113.256666104 cpu 6 pid 162584 tid 162586 ip 0x7f7bc8657180 code 5: Failed to get instruction
    instruction trace error type 1 time 779113.256666673 cpu 6 pid 162584 tid 162586 ip 0x7f7be7b8159b code 6: Trace doesn't match instruction

Using perf dlfilter interface to disassemble executed instructions
------------------------------------------------------------------

perf provides an API to link a user-provided library for filtering or
processing of data. Such a library is refered to as a dlfilter. Refer to
the :doc:`perf dlfilter <manpages/perf-dlfilter>` man page for details.
The advantage of using a dlfilter is that it can be much faster than
processing with the perf Python API.

Below is a perf dlfilter program to disassemble Intel PT traces. It
could be modified to filter based on particular kinds of instructions,
but at the moment just prints a disassembly. The `Intel X86 Encoder
Decoder (XED) <http://intelxed.github.io/>`__ is used. Refer to the
instructions above for how to install XED.

.. code-block:: cpp

   // SPDX-License-Identifier: GPL-2.0
   /*
    * perf dlfilter to disassemble Intel PT trace
    */

   #include <stdio.h>
   #include <stdbool.h>
   #include <string.h>

   #include <perf/perf_dlfilter.h>

   #include <xed/xed-interface.h>

   #define MAX_CPU 4096

   static __u64 last_branch_addr[MAX_CPU];

   #define BUFSZ 65536

   static __u8 ibuf[BUFSZ];

   #define BRANCHES_EVENT "branches"

   static const char *branches = BRANCHES_EVENT;
   static const size_t branches_len = (sizeof(BRANCHES_EVENT) - 1);

   struct perf_dlfilter_fns perf_dlfilter_fns;

   int start(void **data, void *ctx)
   {
       xed_tables_init();
       return 0;
   }

   static void print_instruction(xed_decoded_inst_t *inst, __u64 from_addr)
   {
       char obuf[256];

       if (!xed_format_context(XED_SYNTAX_ATT, inst, obuf, sizeof(obuf), from_addr, NULL, NULL)) {
           printf("xed_format_context failed\n");
           return;
       }

       printf("%16llx  %s\n", from_addr, obuf);
   }

   static void disassemble_buffer(__u64 from_addr, __u64 to_addr, bool is_64_bit, bool inclusive, __u8 *buf, __s32 len)
   {
       xed_error_enum_t xed_ret;
       xed_decoded_inst_t inst;
       __u8 *p = buf;

       xed_decoded_inst_zero(&inst);

       if (is_64_bit)
           xed_decoded_inst_set_mode(&inst, XED_MACHINE_MODE_LONG_64, XED_ADDRESS_WIDTH_64b);
       else
           xed_decoded_inst_set_mode(&inst, XED_MACHINE_MODE_LEGACY_32, XED_ADDRESS_WIDTH_32b);

       /* Bump up 'to_addr' to make the loop include that instruction also */
       if (inclusive)
           to_addr += 1;

       while (len > 0 && from_addr < to_addr) {
           xed_uint_t ilen;

           xed_decoded_inst_zero_keep_mode(&inst);

           xed_ret = xed_decode(&inst, p, len);
           if (xed_ret != XED_ERROR_NONE) {
               printf("xed_decode failed, error %d\n", xed_ret);
               return;
           }

           print_instruction(&inst, from_addr);

           ilen = xed_decoded_inst_get_length(&inst);

           p += ilen;
           len -= ilen;
           from_addr += ilen;
       }
   }

   static void disassemble_range(void *ctx, __u64 from_addr, __u64 to_addr, bool is_64_bit, bool inclusive, __u8 *buf, size_t buf_len)
   {
       __u64 sz;
       __u64 read_sz;
       __s32 nr_bytes;

       sz = to_addr - from_addr;
       if (sz == 0 && !inclusive)
           return;

       /* 'inclusive' means also read last instruction */
       read_sz = inclusive ? sz + XED_MAX_INSTRUCTION_BYTES : sz;

       if (read_sz > buf_len) {
           printf("Cannot disassemble from %llx to %llx, buffer too small\n", from_addr, to_addr);
           return;
       }

       /* Read object code, return numbers of bytes read */
       nr_bytes = perf_dlfilter_fns.object_code(ctx, from_addr, ibuf, read_sz);
       if (nr_bytes < sz) {
           printf("Failed to read object code\n");
           return;
       }

       disassemble_buffer(from_addr, to_addr, is_64_bit, inclusive, buf, nr_bytes);
   }

   int filter_event_early(void *data, const struct perf_dlfilter_sample *sample, void *ctx)
   {
       const struct perf_dlfilter_al *al;
       __u64 from_addr, to_addr;
       bool async;

       /* Only process per-cpu branch events */
       if (strncmp(sample->event, branches, branches_len) ||
           sample->cpu < 0 ||
           !sample->addr_correlates_sym)
           return 0;

       from_addr = last_branch_addr[sample->cpu];
       to_addr = sample->ip;

       if (!from_addr || !to_addr)
           goto out;

       if (to_addr < from_addr) {
           printf("Cannot disassemble from %llx to %llx\n", from_addr, to_addr);
           goto out;
       }

       al = perf_dlfilter_fns.resolve_ip(ctx);
       if (!al) {
           printf("Cannot resolve address %llx\n", sample->ip);
           goto out;
       }

       /* Asynchronous means the instruction at the branch address was not executed */
       async = sample->flags & PERF_DLFILTER_FLAG_ASYNC;

       disassemble_range(ctx, from_addr, to_addr, al->is_64_bit, !async, ibuf, sizeof(ibuf));

   out:
       if (sample->flags & PERF_DLFILTER_FLAG_TRACE_END)
           last_branch_addr[sample->cpu] = 0;
       else
           last_branch_addr[sample->cpu] = sample->addr;

       return 0;
   }

   const char *filter_description(const char **long_description)
   {
       static char *long_desc = "Requires an Intel PT branch trace (not config term branch=0).  "
           "Per-thread recording is not supported.  Example: "
           "perf record -e intel_pt//u uname ; "
           "perf script --itrace=be --dlfilter dlfilter-x86-disasm.so --ns -F-period,-event,+addr,+flags";

       *long_description = long_desc;
       return "Disassemble Intel PT trace";
   }

To compile the dlfilter::

   $ gcc -c -o dlfilter-x86-disasm.o -I ~/include -fpic dlfilter-x86-disasm.c
   $ gcc -shared -o dlfilter-x86-disasm.so dlfilter-x86-disasm.o -lxed

To record a simple trace and disassemble::

   $ perf record -e intel_pt//u uname
   Linux
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.044 MB perf.data ]
   $ perf script --itrace=be --dlfilter dlfilter-x86-disasm.so --ns -F-period,-event,+addr,+flags | head
              uname    7834 [000] 15899.089119450:   tr strt                               0 [unknown] ([unknown]) =>     7fb0daf7b2f0 _start+0x0 (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2)
              uname    7834 [000] 15899.089119659:   tr end  async              7fb0daf7b2f0 _start+0x0 (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2) =>                0 [unknown] ([unknown])
              uname    7834 [000] 15899.089122367:   tr strt                               0 [unknown] ([unknown]) =>     7fb0daf7b2f0 _start+0x0 (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2)
       7fb0daf7b2f0  mov %rsp, %rdi
       7fb0daf7b2f3  callq  0x7fb0daf7c090
              uname    7834 [000] 15899.089122367:   call                       7fb0daf7b2f3 _start+0x3 (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2) =>     7fb0daf7c090 _dl_start+0x0 (/usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2)
       7fb0daf7c090  nop %edi, %edx
       7fb0daf7c094  pushq  %rbp
       7fb0daf7c095  mov %rsp, %rbp
       7fb0daf7c098  pushq  %r15

Tracing Intel User Interrupts with Intel Processor Trace (Intel PT)
-------------------------------------------------------------------

RFC patches to add Linux kernel support for Intel User Interrupts were
submitted in 2021 `LWN Article <https://lwn.net/Articles/871113/>`__,
`discussion <https://lore.kernel.org/linux-api/20210913200132.3396598-1-sohil.mehta@intel.com/#r>`__,
but have not progressed since.

So this example uses a custom kernel based on the provided
`uintr-linux-kernel git repository <https://github.com/intel/uintr-linux-kernel>`__ branch uintr-next
commit 9bbbb4b7fb89fbc3bde8aaed57a0eabc43a2dc64.

Importantly, User Interrupt support must be enabled in kernel config
i.e. ``CONFIG_X86_USER_INTERRUPTS=y``

Once installed, check the kernel version is what is expected e.g.::

   $ uname -a
   Linux spr2 6.0.0-00019-g9bbbb4b7fb89 #2 SMP PREEMPT_DYNAMIC Tue Jan 16 18:38:27 EET 2024 x86_64 GNU/Linux

And check for the User Interrupts feature::

   $ cat /proc/cpuinfo | head -30 | grep -c uintr
   1

The developer provided benchmark programs `uintr-ipc-bench
<https://github.com/intel/uintr-ipc-bench>`__. In this example, branch
master commit 6696170577b964a16e770e2bca2c03ce898d4608. Build and run
instructions in README.md

Benchamark program build/source/uintrfd/uintrfd-uni will be used. It
repeatedly sends a user interrupt (senduipi) to another thread which
will invoke a registered handler function ui_handler(). The time taken
from sending to receiving is measured::

   $ cat source/uintrfd/uintrfd-uni.c

.. code-block:: cpp

   #include <stdint.h>
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <x86gprintrin.h>

   #define __USE_GNU
   #include <pthread.h>
   #include <sched.h>

   #include "common/common.h"

   #ifndef __NR_uintr_register_handler
   #define __NR_uintr_register_handler 471
   #define __NR_uintr_unregister_handler   472
   #define __NR_uintr_create_fd        473
   #define __NR_uintr_register_sender  474
   #define __NR_uintr_unregister_sender    475
   #define __NR_uintr_wait         476
   #endif

   #define uintr_register_handler(handler, flags)  syscall(__NR_uintr_register_handler, handler, flags)
   #define uintr_unregister_handler(flags)     syscall(__NR_uintr_unregister_handler, flags)
   #define uintr_create_fd(vector, flags)      syscall(__NR_uintr_create_fd, vector, flags)
   #define uintr_register_sender(fd, flags)    syscall(__NR_uintr_register_sender, fd, flags)
   #define uintr_unregister_sender(ipi_idx, flags) syscall(__NR_uintr_unregister_sender, ipi_idx, flags)
   #define uintr_wait(flags)           syscall(__NR_uintr_wait, flags)

   volatile unsigned long uintr_received;
   volatile unsigned int uintr_count = 0;
   int descriptor;

   struct Benchmarks bench;

   void __attribute__ ((interrupt))
        __attribute__((target("general-regs-only", "inline-all-stringops")))
        ui_handler(struct __uintr_frame *ui_frame,
           unsigned long long vector) {

       benchmark(&bench);
       uintr_count++;
       uintr_received = 1;
   }

   void __attribute__ ((noinline)) senduipi(int uipi_index)
   {
       _senduipi(uipi_index);
   }

   void *client_communicate(void *arg) {

       struct Arguments* args = (struct Arguments*)arg;
       int loop;

       int uipi_index = uintr_register_sender(descriptor, 0);
       if (uipi_index < 0)
           throw("Sender register error\n");

       for (loop = args->count; loop > 0; --loop) {

           uintr_received = 0;
           bench.single_start = now();

           // Send User IPI
           senduipi(uipi_index);

           while (!uintr_received){
               // Keep spinning until this user interrupt is received.
           }
       }

       return NULL;
   }

   void server_communicate(int descriptor, struct Arguments* args) {

       while (uintr_count < args->count) {
           //Keep spinning until all user interrupts are delivered.
       }

       // The message size is always one (it's just a signal)
       args->size = 1;
       evaluate(&bench, args);
   }

   void communicate(int descriptor, struct Arguments* args) {

       pthread_t pt;

       setup_benchmarks(&bench);

       // Create another thread
       if (pthread_create(&pt, NULL, &client_communicate, args)) {
           throw("Error creating sender thread");
       }

       server_communicate(descriptor, args);

       close(descriptor);
   }

   int main(int argc, char* argv[]) {

       struct Arguments args;

       if (uintr_register_handler(ui_handler, 0))
           throw("Interrupt handler register error\n");

       // Create a new uintrfd object and get the corresponding
       // file descriptor.
       descriptor = uintr_create_fd(0, 0);
       if (descriptor < 0)
           throw("Interrupt vector allocation error\n");

       // Enable interrupts
       _stui();

       parse_arguments(&args, argc, argv);

       communicate(descriptor, &args);

       return EXIT_SUCCESS;
   }

The benchmark is using timespec_get() which is not ideal for comparing
time measurements because it uses CLOCK_REALTIME which is not monotonic.
For comparing times on different machines, CLOCK_MONOTONIC can be used.
For comparing times on the same machine, CLOCK_MONTONIC or
CLOCK_MONTONIC_RAW can be used. For comparing with Intel PT,
CLOCK_MONTONIC_RAW will be a close match to perf time, so that is used
here.

Also to get precise timing, Intel PT address filtering can be used, but
in that case senduipi needs to be a separate, non-inline function.

Here are the changes::

   $ git diff

.. code-block:: diff

   diff --git a/source/common/benchmarks.c b/source/common/benchmarks.c
   index 6c3fe92..6619cd9 100644
   --- a/source/common/benchmarks.c
   +++ b/source/common/benchmarks.c
   @@ -13,7 +13,7 @@ bench_t now() {
       return ((double)clock()) / CLOCKS_PER_SEC * 1e9;
    #else
       struct timespec ts;
   -   timespec_get(&ts, TIME_UTC);
   +   clock_gettime(CLOCK_MONOTONIC_RAW, &ts);

       return ts.tv_sec * 1e9 + ts.tv_nsec;

   diff --git a/source/uintrfd/uintrfd-uni.c b/source/uintrfd/uintrfd-uni.c
   index cb62dbb..c26f9d2 100644
   --- a/source/uintrfd/uintrfd-uni.c
   +++ b/source/uintrfd/uintrfd-uni.c
   @@ -42,6 +42,11 @@ void __attribute__ ((interrupt))
       uintr_received = 1;
    }

   +void __attribute__ ((noinline)) senduipi(int uipi_index)
   +{
   +   _senduipi(uipi_index);
   +}
   +
    void *client_communicate(void *arg) {

       struct Arguments* args = (struct Arguments*)arg;
   @@ -57,7 +62,7 @@ void *client_communicate(void *arg) {
           bench.single_start = now();

           // Send User IPI
   -       _senduipi(uipi_index);
   +       senduipi(uipi_index);

           while (!uintr_received){
               // Keep spinning until this user interrupt is received.

Run the benchmark program, tracing with Intel PT and cycle-accurate
mode, using address filters for the sending function senduipi() and
receiving function ui_handler()::

   $ perf record -e intel_pt/cyc/u --filter 'filter senduipi #1 @ build/source/uintrfd/uintrfd-uni , filter ui_handler #1 @ build/source/uintrfd/uintrfd-uni' -- build/source/uintrfd/uintrfd-uni
   [ perf record: Woken up 1 times to write data ]
   [ perf record: Captured and wrote 0.131 MB uintr-tfr2/perf.data ]

   ============ RESULTS ================
   Message size:       1
   Message count:      1000
   Total duration:     2.416   ms
   Average duration:   1.348   us
   Minimum duration:   1.280   us
   Maximum duration:   7.608   us
   Standard deviation: 0.235   us
   Message rate:       413926  msg/s
   =====================================

A python script is needed to compute statistics from the Intel PT
trace::

   $ cat uintr-uni-stats.py

.. code-block:: python

   import statistics
   import numpy

   def trace_begin():
       print("uintr-uni statistics")
       print("====================")

   def get_optional(perf_dict, field, dflt):
       if field in perf_dict:
           return perf_dict[field]
       return dflt

   glb_start = 0
   glb_data = []

   def process_event(param_dict):
       name        = param_dict["ev_name"]
       if not name.startswith("branches"):
           return
       sample      = param_dict["sample"]
       pid         = sample["pid"]
       tid         = sample["tid"]
       cpu         = sample["cpu"]
       ts          = sample["time"]
       addr_symbol = get_optional(sample, "addr_symbol", "[unknown]")
       addr_symoff = get_optional(sample, "addr_symoff", "[unknown]")
       symbol      = get_optional(param_dict, "symbol", "[unknown]")
       symoff      = get_optional(param_dict, "symoff", 1)
       global glb_start
       global glb_data
       if addr_symbol == "senduipi" and addr_symoff == 0:
           if glb_start:
               print(cpu, pid, tid, ts, name, symbol, symoff, addr_symbol, addr_symoff, "START AFTER START")
           glb_start = ts
       elif addr_symbol == "ui_handler" and addr_symoff == 0:
           if glb_start:
               latency = (ts - glb_start) / 1000
               glb_data.append(latency)
               glb_start = 0
           else:
               print(cpu, pid, tid, ts, name, symbol, symoff, addr_symbol, addr_symoff, "END NO START")

   def trace_end():
       print()
       print("Count:              %u" % len(glb_data))
       print("Average:            %.3f" % statistics.mean(glb_data))
       print("Minimum:            %.3f" % min(glb_data))
       print("Maximum:            %.3f" % max(glb_data))
       print("Standard deviation: %.3f" % statistics.stdev(glb_data))
       h = numpy.histogram(glb_data, bins=10)
       print("\nHistogram:")
       print("  Range          Count")
       for i in range(10):
           print("  %.3f - %.3f  %u" % (h[1][i], h[1][i+1], h[0][i]))

First check for errors::

   $ perf script --itrace=e
   $

No errors, so run the script::

   $ perf script --itrace=bep -s uintr-uni-stats.py
   uintr-uni statistics
   ====================

   Count:              1000
   Average:            1.259
   Minimum:            1.201
   Maximum:            7.484
   Standard deviation: 0.216

   Histogram:
     Range          Count
     1.201 - 1.829  998
     1.829 - 2.458  1
     2.458 - 3.086  0
     3.086 - 3.714  0
     3.714 - 4.342  0
     4.342 - 4.971  0
     4.971 - 5.599  0
     5.599 - 6.227  0
     6.227 - 6.856  0
     6.856 - 7.484  1

The Intel PT results are slighly lower than the benchmark program
produced itself, because they do not included the overhead of getting
the time (via clock_gettime).

Appendix: Intel PT capabilities on different processors
-------------------------------------------------------

Generally, processors with the same CPU core microarchitecture can be
expected to have the same Intel PT capabilities. The Linux Intel PT
driver provides capability flags in sysfs, refer :ref:`Intel PT man page
<intel_pt_man_page>`. Below are some examples:

.. _th_gen_core_skylake:

9th Gen Core (Skylake)
~~~~~~~~~~~~~~~~~~~~~~

::

   $ grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:3fff
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:0
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:0
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

Intel Atom (Apollo Lake)
~~~~~~~~~~~~~~~~~~~~~~~~

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:ffff
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:1
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:0
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:0
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

Intel Atom (Gemini Lake)
~~~~~~~~~~~~~~~~~~~~~~~~

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:ffff
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:1
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:1
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:0
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

10th Gen Core (Ice Lake)
~~~~~~~~~~~~~~~~~~~~~~~~

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:1fff
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:0
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:0
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:0
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

11th Gen Core (Tiger Lake)
~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   $ grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:1fff
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:0
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:0
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

Intel Atom (Jasper Lake)
~~~~~~~~~~~~~~~~~~~~~~~~

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:ffff
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:1
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:1
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:0
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

12th Gen Core (Alder Lake)
~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:3f
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:0
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:1
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:0
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

N-Series (Alder Lake-N)
~~~~~~~~~~~~~~~~~~~~~~~

Alder Lake-N has only E-Cores and more Intel PT features like Event
Trace and TNT Disable.

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:ffff
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:1
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:1
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:1
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

13th Gen Core (Raptor Lake)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # grep -H . /sys/bus/event_source/devices/intel_pt/caps/*
   /sys/bus/event_source/devices/intel_pt/caps/cr3_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/cycle_thresholds:3f
   /sys/bus/event_source/devices/intel_pt/caps/event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/ip_filtering:1
   /sys/bus/event_source/devices/intel_pt/caps/max_subleaf:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc:1
   /sys/bus/event_source/devices/intel_pt/caps/mtc_periods:249
   /sys/bus/event_source/devices/intel_pt/caps/num_address_ranges:2
   /sys/bus/event_source/devices/intel_pt/caps/output_subsys:0
   /sys/bus/event_source/devices/intel_pt/caps/payloads_lip:0
   /sys/bus/event_source/devices/intel_pt/caps/power_event_trace:0
   /sys/bus/event_source/devices/intel_pt/caps/psb_cyc:1
   /sys/bus/event_source/devices/intel_pt/caps/psb_periods:3f
   /sys/bus/event_source/devices/intel_pt/caps/ptwrite:1
   /sys/bus/event_source/devices/intel_pt/caps/single_range_output:1
   /sys/bus/event_source/devices/intel_pt/caps/tnt_disable:0
   /sys/bus/event_source/devices/intel_pt/caps/topa_multiple_entries:1
   /sys/bus/event_source/devices/intel_pt/caps/topa_output:1

   # echo -n "Intel PT can be used in VMX operation (VMX_MISC MSR 0x485 bit 14) : " ; rdmsr -p 0 -f 14:14 0x485
   Intel PT can be used in VMX operation (VMX_MISC MSR 0x485 bit 14) : 1

   # echo -n "PEBS output to Intel PT Capability (PERF_CAPABILITIES MSR 0x345 bit 16) : " ;  rdmsr -p 0 -f 16:16 0x345
   PEBS output to Intel PT Capability (PERF_CAPABILITIES MSR 0x345 bit-16) : 0
