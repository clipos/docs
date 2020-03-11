.. Copyright © 2018 ANSSI.
   CLIP OS is a trademark of the French Republic.
   Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _build-steps:

Build steps
===========

.. important::

   This page details the purpose of each step in the project build process.
   Running steps manually is generally not needed for development as you may
   use directly the instructions from the :ref:`Building page <build>`.

Building the documentation
--------------------------

The project documentation is built with Sphinx. The documentation sources are
available in the ``docs-src`` directory.

To build the documentation and to open it in your browser, run:

.. code-block:: shell-session

   $ cosmk doc build
   $ cosmk doc open

SDK
---

To build the software components of CLIP OS, we use a SDK based on Gentoo
Hardened. The SDK container is created by importing the upstream `stage 3 root
filesystem <https://wiki.gentoo.org/wiki/Stage_tarball#Stage_3>`_ and updating
it with a current copy of the upstream Gentoo Portage tree to include various
utilities. If unavailable, the SDK is automatically imported from the
configured CI or build locally, and may be manually rebuild from scratch using:

.. code-block:: shell-session

   # Remove the current SDK container images
   $ podman rmi <registry>/clipos/sdk:<version>       # podman
   $ sudo docker rmi <registry>/clipos/sdk:<version>  # Docker

   # Set empty URLs for the 'registry' in the 'ci' section of the 'config.toml'
   $ $EDITOR config.toml

   # Bootstrap the SDK
   $ cosmk bootstrap sdk

Core
----

The main rootfs in CLIP OS is called Core, and can be built using:

.. code-block:: shell-session

   $ cosmk build core
   $ cosmk image core
   $ cosmk configure core
   $ cosmk bundle core

EFI boot partition
------------------

EFI boot is the only supported boot method. The content of the EFI boot
partition (bootloader, kernel image, etc.) is built using:

.. code-block:: shell-session

   $ cosmk build efiboot
   $ cosmk image efiboot
   $ cosmk configure efiboot
   $ cosmk bundle efiboot

Initial state for QEMU test image
---------------------------------

To test the resulting OS in a QEMU virtual machine, we generate a tarball with
the configuration to be installed in the system state partition:

.. code-block:: shell-session

   $ cosmk bundle qemu

QEMU image
----------

In order to test the resulting OS, we use ``libguestfs`` tools to assemble a
QEMU qcow2 disk image to boot inside a EFI enabled virtual machine using
``libvirt``. The qcow2 QEMU image may be assembled using:

.. code-block:: shell-session

   $ cosmk test qemu

Testbed environment setup
-------------------------

To setup the virtual testbed environment with ``Vagrant`` and ``libvirt``, use:

.. code-block:: shell-session

  $ cosmk test setup

Testing the QEMU image
----------------------

To setup a EFI & QEMU/KVM enabled virtual machine with ``libvirt``, use:

.. code-block:: shell-session

   $ cosmk test run

Caching and binary packages
---------------------------

To speed up the build process during development, we keep the output of each
build action in the ``cache`` and ``out`` folders. The ``cache`` directory
keeps binary packages and logs. The ``out`` directory keeps the intermediate
rootfs and temporary files that are safe to remove before a rebuild.

By default, the ``cosmk`` tool will clear the ``out`` folder and reuse cached
output (mainly packages) to speedup iterative development builds. To restart
everything from scratch:

.. code-block:: shell-session

   $ rm -rf out cache
   # If using Docker or rootfull podman
   $ sudo rm -rf out cache

.. admonition:: Pre-built binary packages by a continuous integration
                infrastructure (GitLab CI)
   :class: important

   We use GitLab CI to automatically build CLIP OS. The GitLab CI configuration
   and build scripts are tracked in the `clipos/ci
   <https://github.com/clipos/ci>`_ repository and the resulting artifacts are
   made available at `files.clip-os.org <https://files.clip-os.org/>`_.

To download CI-built binary packages from the last successful CI job, use:

.. code-block:: shell-session

   $ cosmk cache

SDKs are automatically downloaded from the CI if configured in ``config.toml``.

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
