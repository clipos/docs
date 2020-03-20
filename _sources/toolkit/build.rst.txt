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

To run all the steps required to build CLIP OS:

.. code-block:: shell-session

   $ cosmk all

Congratulations, you are now ready to :ref:`test CLIP OS <test>`.

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
