.. Copyright © 2019 ANSSI.
   CLIP OS is a trademark of the French Republic.
   Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _quick-try:

Quick try guide
===============

.. important::

   **Those files are provided AS IS for TESTING PURPOSES ONLY and MUST NOT be
   used in a PRODUCTION CONTEXT.**

To easily try the latest version (nightly build) of CLIP OS 5, you may download
pre-built packages, SDK and pre-installed QEMU images from `files.clip-os.org
<https://files.clip-os.org>`_.

Be aware that those packages and QEMU images are instrumented, which means they
allow full root access without any password on the system.

To run the CLIP OS system in a virtual machine, you will have to install QEMU.
It is also strongly recommended to have KVM support enabled for your current
user (i.e. your current user is part of the ``kvm`` group).

.. code-block:: shell-session

   # For Debian & Ubuntu
   $ sudo apt-get install qemu-system-x86

   # For Fedora & CentOS
   $ sudo dnf install -y qemu-system-x86

   # For Arch Linux
   $ sudo pacman -Syu qemu

Then, you can download and start a CLIP OS QEMU virtual machine with the
following commands:

.. code-block:: shell-session

   # Pick the latest successful build from https://gitlab.com/CLIPOS/ci/pipelines
   $ BUILD="$(curl 'https://gitlab.com/api/v4/projects/14752889/pipelines?scope=finished&status=success' | jq '.[0].id')"

   # Retrieve the QEMU image
   $ wget https://files.clip-os.org/$BUILD/qemu.tar.zst

   # Extract and enter directory
   $ tar xf qemu.tar.zst && cd clipos_*_qemu

   # Read the README
   $ cat README.md

   # Start CLIP OS with QEMU
   $ ./qemu.sh

   # Start CLIP OS with QEMU without KVM support (not supported, may have issues)
   # See: https://discuss.clip-os.org/t/qemu-system-x86-64-overcommit-invalid-option-at-boot/76/12
   $ ./qemu-nokvm.sh

You can login as root with no password.

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
