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

   (toolkit) $ just doc
   (toolkit) $ just open-doc

SDK
---

To build the software components of CLIP OS, we use a SDK based on Gentoo
Hardened. The SDK container is created by importing the upstream `stage 3 root
filesystem <https://wiki.gentoo.org/wiki/Stage_tarball#Stage_3>`_ and updating
it with a current copy of the upstream Gentoo Portage tree to include various
utilities. If unavailable, the SDK is automatically build, and may be manually
rebuild from scratch using:

.. code-block:: shell-session

   (toolkit) $ sujust products/clipos/sdk/bootstrap-from-scratch

Core
----

The main rootfs in CLIP OS is called Core, and can be built using:

.. code-block:: shell-session

   (toolkit) $ sujust products/clipos/core

EFI boot partition
------------------

EFI boot is the only supported boot method. The content of the EFI boot
partition (bootloader, kernel image, etc.) is built using:

.. code-block:: shell-session

   (toolkit) $ sujust products/clipos/efiboot

QEMU image
----------

In order to test the resulting OS, we use ``libguestfs`` tools to assemble a
QEMU qcow2 disk image to boot inside a EFI enabled virtual machine using
``libvirt``. The qcow2 QEMU image may be assembled using:

.. code-block:: shell-session

   (toolkit) $ ./testbed/create_qemu_image.sh

Testing the QEMU image
----------------------

To setup a EFI & QEMU/KVM enabled virtual machine with ``libvirt``, use:

.. code-block:: shell-session

   (toolkit) $ ./testbed/run_with_libvirt.py

Caching and binary packages
---------------------------

To speed up the build process during development, we keep the output of each
build action in the ``cache`` and ``out`` folders. The ``cache`` directory
keeps binary packages and SDK images. The ``cache`` directory keeps the
intermediate rootfs, logs and temporary files that are safe to remove before a
rebuild.

By default, the build commands will clear their ``out`` folder and reuse cached
output (mainly packages) to speedup iterative development builds. To restart
everything from scratch:

.. code-block:: shell-session

   (toolkit) $ sujust clean
   (toolkit) $ sujust clean-cache
   (toolkit) $ sujust all

.. admonition:: Pre-built binary packages by a continuous integration
                infrastructure (GitLab CI)
   :class: important

   We use GitLab CI to automatically build CLIP OS. The GitLab CI configuration
   and build scripts are tracked in the `clipos/ci
   <https://github.com/clipos/ci>`_ repository and the resulting artifacts are
   made available at `files.clip-os.org <https://files.clip-os.org/>`_.

To download CI-built binary packages and SDKs from the last successful CI job,
use:

.. code-block:: shell-session

   (toolkit) $ just get-cache

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
