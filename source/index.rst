``perf:`` Linux profiling with performance counters
===================================================

*...More than just counters...*

Introduction
~~~~~~~~~~~~

This is the wiki page for the Linux ``perf`` command, also called
perf_events. ``perf`` is powerful: it can instrument CPU performance
counters, tracepoints, kprobes, and uprobes (dynamic tracing). It is
capable of lightweight profiling. It is also included in the Linux
kernel, under tools/perf, and is frequently updated and enhanced.

``perf`` began as a tool for using the performance counters subsystem in
Linux, and has had various enhancements to add tracing capabilities.

Performance counters are CPU hardware registers that count hardware
events such as instructions executed, cache-misses suffered, or branches
mispredicted. They form a basis for profiling applications to trace
dynamic control flow and identify hotspots. ``perf`` provides rich
generalized abstractions over hardware specific capabilities. Among
others, it provides per task, per CPU and per-workload counters,
sampling on top of these and source code event annotation.

Tracepoints are instrumentation points placed at logical locations in
code, such as for system calls, TCP/IP events, file system operations,
etc. These have negligible overhead when not in use, and can be enabled
by the ``perf`` command to collect information including timestamps and
stack traces. ``perf`` can also dynamically create tracepoints using the
kprobes and uprobes frameworks, for kernel and userspace dynamic
tracing. The possibilities with these are endless.

The userspace ``perf`` command present a simple to use interface with
commands like:

- :ref:`perf stat <counting_with_perf_stat>`: obtain event counts
- :ref:`perf record <sampling_with_perf_record>`: record events for later reporting
- :ref:`perf report <sample_analysis_with_perf_report>`: break down events by process, function, etc.
- :ref:`perf annotate <source_level_analysis_with_perf_annotate>`: annotate assembly or source code with event counts
- :ref:`perf top <live_analysis_with_perf_top>`: see live event count
- :ref:`perf bench <benchmarking_with_perf_bench>`: run different kernel microbenchmarks

To learn more, see the examples in the :doc:`Tutorial <tutorial>` or how
to do a :doc:`Top-Down Analysis <top_down_analysis>`.

To ask questions, report bugs/issues mail the `mailing list
<https://lore.kernel.org/linux-perf-users/>`__ or use `bugzilla
<https://bugzilla.kernel.org/buglist.cgi?bug_status=__open__&order=changeddate%20DESC%2Cpriority%2Cbug_severity&product=Tracing%2FProfiling&query_format=advanced>`__.

.. toctree::
   :maxdepth: 1

   tutorial
   top_down_analysis
   todo
   hardware_reference
   perf_tools_support_for_intel
   useful_links
   glossary
   manpages/index
   development

Google Summer of Code
~~~~~~~~~~~~~~~~~~~~~

As part of the `Linux Foundation <https://www.linuxfoundation.org/>`__
the perf tool has participated in the `Google Summer-of-Code
<https://summerofcode.withgoogle.com/>`__ since 2021. `Check out the
2024 process <https://wiki.linuxfoundation.org/gsoc/2024-gsoc-perf>`__.
