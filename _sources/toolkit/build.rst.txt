.. Copyright © 2018 ANSSI.
   CLIP OS is a trademark of the French Republic.
   Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _build:

Building
========

.. admonition:: Before going further, ensure to get a functional build
                environment setup
   :class: important

   You must complete the :ref:`environment setup <setup>` before executing any
   command from this page.

Building the CLIP OS toolkit (cosmk)
------------------------------------

To build the CLIP OS toolkit, run and follow the setup instructions from the
script:

.. code-block:: shell-session

   $ ./toolkit/setup.sh

Creating a ``config.toml`` for cosmk
------------------------------------

We now need to tell ``cosmk`` what it should build by writing a ``config.toml``
file at the repository root directory. A working default configuration to build
the CLIP OS project is provided in the ``config.toml.example`` file from the
``toolkit`` repository:

.. code-block:: shell-session

   $ cp toolkit/config.toml.example config.toml

Instrumentation features for testing
------------------------------------

Default builds of the project are meant to be set up with a production-ready
configuration. There is currently no password-based local login available in
this configuration. Thus if you want to test the project in QEMU, you will
have to enable some :ref:`instrumentation features <development>` in
``config.toml``. Here are some good defaults for development builds:

.. code-block:: shell-session

   $ sed -i '/#"instrumented-core"/s/#//g'        config.toml
   $ sed -i '/#"passwordless-root-login"/s/#//g'  config.toml
   $ sed -i '/#"allow-ssh-root-login"/s/#//g'     config.toml
   $ sed -i '/#"instrumented-initramfs"/s/#//g'   config.toml
   $ sed -i '/#"initramfs-no-require-tpm"/s/#//g' config.toml
   $ sed -i '/#"initramfs-no-tpm-lockout"/s/#//g' config.toml
   $ sed -i '/#"test-update"/s/#//g'              config.toml

This configuration will keep most system configuration unchanged, will install
common tools (vim, tar, grep, etc.) and will give you local password-less
`root` access.

.. note::

   Any change of instrumentation features requires a full project rebuild.

Building the full project
-------------------------

You will then be able to use ``cosmk`` to run commands to build CLIP OS.
Building the CLIP OS project requires multiple successive steps that are
described in TOML files called ``recipes``.

By default, to reduce compilation time, ``cosmk`` will download pre-built SDKs
from the public CLIP OS CI infrastructure. To further reduce compilation time,
you may also download binary packages from the public CLIP OS CI using:

.. code-block:: shell-session

   $ cosmk cache

To run all steps required to build CLIP OS:

.. code-block:: shell-session

   $ cosmk all

Virtual testbed setup
---------------------

In order to test some functionalities of a CLIP OS system, you will need a
virtual infrastructure acting as testbed. To setup this infrastructure, use:

.. code-block:: shell-session

   $ cosmk test setup

This will setup virtual networks using ``Vagrant`` with ``libvirt`` and create
a Debian virtual machine running the following services:

  * IPsec gateway (``strongSwan``)
  * Update server (``nginx``)

Building a QEMU image and running using QEMU/KVM
------------------------------------------------

.. admonition:: TPM emulation support
   :class: important

   TPM emulation support (see `libtpms
   <https://github.com/stefanberger/libtpms>`_ and `swtpm
   <https://github.com/stefanberger/swtpm>`_ setup in :ref:`Environment setup
   <setup>`) is required to test the project under QEMU in the test
   environment.

   Alternatively, you may enable the ``initramfs-no-require-tpm``
   instrumentation feature which will allow the initramfs to ask for a
   passphrase at bootup if TPM support is not available. The default passphrase
   is ``clipos`` (for old builds, it used to be ``core_state_key``).

To build a QCOW2 QEMU disk image and to setup a EFI & QEMU/KVM enabled virtual
machine with ``libvirt``, use:

.. code-block:: shell-session

   $ cosmk test qemu

.. admonition:: Local login disabled by default
   :class: important

   The default build configuration will create production images with root
   access disabled. See the `Instrumentation features for testing`_ paragraph
   for instructions to create an instrumented build.

Access to QEMU virtual machine over SSH
---------------------------------------

.. admonition:: Access disabled by default
   :class: important

   The default build configuration will create production images with SSH
   access available only over the IPsec tunnel. To enable SSH access from
   outside the IPsec tunnel, you must enable the ``allow-ssh-root-login``
   instrumentation feature. See the `Instrumentation features for testing`_
   paragraph for instructions to create an instrumented build.

To access a QEMU virtual machine over SSH, retrieve the IP address using
``virsh`` and use the SSH keys stored in the cache directory:

.. code-block:: shell-session

   $ virsh --connect qemu:///system domifaddr clipos-testbed_clipos-qemu
    Name       MAC address          Protocol     Address
   -------------------------------------------------------------------------------
    vnet2      XX:XX:XX:XX:XX:XX    ipv4         172.27.1.XX/24
   $ ssh -i cache/clipos/5.0.0-alpha.1/qemu/bundle/ssh_root \
         -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
         root@172.27.1.XX
   $ ssh -i cache/clipos/5.0.0-alpha.1/qemu/bundle/ssh_audit \
         -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
         audit@172.27.1.XX
   $ ssh -i cache/clipos/5.0.0-alpha.1/qemu/bundle/ssh_admin \
         -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
         admin@172.27.1.XX

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
