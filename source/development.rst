Development
===========

Installation Guide
------------------

You'll need to install 'perf' in your system. 'perf' uses CPU
performance counters( Performance Monitoring Counters or PMCs).

#. Install Required Packages like *libelf-dev* *libdw-dev*
   *binutils-dev* *libaudit-dev* by using the following command::

      sudo apt-get install build-essential libelf-dev libdw-dev \
        binutils-dev libaudit-dev libtraceevent-dev systemtap-sdt-dev \
        libunwind-dev libslang2-dev libperl-dev libzstd-dev \
        libbabeltrace-ctf-dev flex bison

#. Download the Source code for perf-tools v6.3 using the following
   command::

      git clone https://git.kernel.org/pub/scm/linux/kernel/git/perf/perf-tools-next.git

#. Install perf. Perf command is available in packages
   *linux-tools-common*, *linux-intel-iotg-tools-common*, etc. Hence,
   install any of these.

#. Run 'make' command to build the project

   #. It may show an error since the config file is not set and has to
      be changed. Run the following command::

          make xconfig

   #. We will also need to install qt5 for this. Install it using
      ::

          sudo apt-get install qtbase5-dev*

      or::

          sudo apt install qt5*

      and set the path using::

          export PKG_CONFIG_PATH=/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/share/pkgconfig

   #. Check if qt5 has been installed properly using
      ::

          qmake --version

   #. You might need to set ``kernel.perf_event_paranoid = -1`` because
      the ``perf record`` command may fail due to restricted access to
      performance monitoring features on your system
      ::

          sudo sysctl -w kernel.perf_event_paranoid=-1

      Note: this should not be done in production environment

    #. Now, Finally run ``make xconfig`` and it will work properly.

#. Go to the root directory of the kernel and run ``perf test``. You will
   see tests passing, skipping, or failing.

Maintainer Trees
----------------

Kernel
~~~~~~

`tip.git perf/core <https://git.kernel.org/pub/scm/linux/kernel/git/tip/tip.git/log/?h=perf/core>`__

Tool
~~~~

`perf-tools <https://git.kernel.org/pub/scm/linux/kernel/git/acme/linux.git/log/?h=perf-tools>`__ for the current release. Merged to linux-next.

`perf-tools-next <https://git.kernel.org/pub/scm/linux/kernel/git/acme/linux.git/log/?h=perf-tools-next>`__ staging for the next release.

Design
------

`design.txt <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/tools/perf/design.txt>`__

Latest source tar balls
-----------------------

`mirror <https://mirrors.edge.kernel.org/pub/linux/kernel/tools/perf/>`__

Cross Compilation
-----------------

:download:`arm64_cross_compilation.dockerfile <media/arm64_cross_compilation.dockerfile>` for an example of how to cross-compile for Arm on x86
