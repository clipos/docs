.. Copyright © 2018 ANSSI.
   CLIP OS is a trademark of the French Republic.
   Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _setup:

Environment setup
=================

This document will walk you through the steps to setup the environment required
to use the tools in the CLIP OS project.

Global environment configuration requirements
---------------------------------------------

Here is a requirements check-list about your global environment:

1. You must run a **64-bit x86 system** (a.k.a. ``AMD64``, ``Intel 64`` or
   ``x86_64``) architecture.

   .. admonition:: About other system architectures
      :class: tip

      Cross-compiling to other system architectures is not supported yet. No
      other host system architecture is supported yet.

2. You must run **Linux**. No other operating system is supported. Any
   relatively recent and stable kernel provided by a major Linux distribution
   should be compatible with the CLIP OS toolkit.

   .. admonition:: Supported Linux distributions
      :class: note

      The project is regularly tested and known to work with the following
      distributions:

        * Arch Linux
        * Debian 10 (stable)

      Although not regularly tested, the following distributions are supported
      (i.e. we will investigate and fix reported bugs) and the project should
      work on those:

        * Debian testing and unstable
        * Fedora 31 and later
        * Ubuntu 18.04 and later
        * CentOS 8 and later

   .. admonition:: About other Linux distributions
      :class: tip

      The project will likely work on other distributions but there is
      currently no plan to extend our support. If your distribution kernel is
      at least version 4.19 and supports all the features used by the CLIP OS
      toolkit (such as namespaces, capabilities, cgroups, OverlayFS, tmpfs,
      etc.), it is expected to work without issue.

3. Both your hardware and your kernel must support **KVM** (through Intel VT-x
   or AMD-V technologies) to run CLIP OS virtual machine images in the virtual
   testbed. Building without KVM support is supported but testing requires KVM
   support and access to an instance of **libvirt**.

4. **Podman** is required to spawn SDK containers to build CLIP OS. Rootless
   podman is supported and rootfull podman support requires **Super-user
   privileges** acquired through **sudo**. Running containers with **Docker**
   is also supported but not actively tested.

5. Make sure to have allocated a consequent size of **swap space** on your
   system as it might be required when working on large ephemeral SDK
   environments. Having a lot of RAM (16GB+) will also help.

   .. admonition:: Why this requirement?
      :class: tip

      SDK ephemeral containers make use of "in-memory" *tmpfs* OverlayFS
      layers. As a consequence, there are scenarios where memory usage may be
      large (typically when bootstrapping the CLIP OS SDK image) and may not
      fit the entire memory of the system.

6. Make sure to have **enough free storage space** before getting the source
   tree as it can take up to several gigabytes of storage, and even more when
   you will begin building CLIP OS images. 50GB should be a minimum.

7. Make sure to work in a **filesystem without any restricted features** such
   as `noexec` or `nodev` as it will cause undefined issues throughout the
   building process of some parts of the CLIP OS images.


Software dependencies
---------------------

.. admonition:: TL;DR
   :class: tip

   If your are using a distribution supported by the project, you may skip this
   part of the documentation and jump directly to the section
   :ref:`dependencies-installation-on-supported-linux-distributions`.
   Otherwise (or if you encounter issues with the above method), please
   continue reading this section.

To get a functional environment, you will need these software dependencies in
your userland:

- **Git** as all the source code is versioned through Git repositories.

- **Git LFS** (Git Large File Storage extension) is required to fetch Git
  repositories with a lot of large binary files.

  .. admonition:: Alternative way to install Git LFS
     :class: note

     If ``git-lfs`` is not provided through a package of your Linux
     distribution, you can follow instructions from the `Git LFS project pages
     <https://github.com/git-lfs/git-lfs/wiki/Installation>`_ to install it.

- **repo** (tool from the Android Open Source Project) is required to fetch
  the source tree.

  .. admonition:: Alternative way to install repo
     :class: note

     ``repo`` might be packaged by your Linux distribution. Otherwise you may
     have to get it and install it from source. To do so, follow the related
     instructions on `the Android Open Source project page regarding the setup
     of the environment for AOSP
     <https://source.android.com/setup/build/downloading#installing-repo>`_.

- **Bash 4.1 (or later)** is required for some toolkit helper scripts.

- The **Go** compiler (version **1.12 or above**) is required to build the
  ``cosmk`` tool.

  .. admonition:: Alternative way to install Go
     :class: note

     If your ditribution does not have a recent enough version of Go, you may
     download a pre-built official version from `golang.org
     <https://golang.org/>`_.

- **Podman** is required to launch SDK containers. SDK containers may be run
  rootless using podman or rootfull using podman. Running SDK containers
  rootfull will also require **sudo** for automatic privilege escalation (the
  current unprivileged user must be a ``sudoer`` to be able to gain those
  privileges). Running container with **Docker** is also supported and requires
  **sudo**.

- **libvirt with QEMU and KVM support** are required as the platform to run the
  CLIP OS virtual machines with QEMU with virtualized networks. The **Python 3
  module for libvirt** and a TPM emulator are also required.

  .. admonition:: Avoid running QEMU as root if not necessary
     :class: tip

     On some Linux distributions (e.g., Arch Linux), libvirt is provided with a
     default configuration which runs QEMU as root. If you intend to use
     libvirt only for the purpose of running CLIP OS QEMU images, you may want
     to run the QEMU processes launched by libvirt as your current user.

     To do so, edit the file ``/etc/libvirt/qemu.conf`` and change the values
     for the ``user`` and ``group`` as follows:

     .. code-block:: guess

        user = "myusername"  # replace with your current username
        group = "kvm"

- **Optionnal (but recommended):** **liguestfs tools** to build the disk image
  for the QEMU virtual machine. If unavailable, **Podman** or **Docker** will
  be used to run libguestfs inside a container.

- **Optionnal:** **Sphinx** and the **Read the Docs Theme** to build the
  documentation. If unavailable, **Podman** or **Docker** will be used to run
  Sphinx inside a container.

.. _dependencies-installation-on-supported-linux-distributions:

Dependencies installation on supported Linux-distributions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Ubuntu or Debian (with ``contrib`` sources enabled for Debian):

.. code-block:: shell-session

   $ sudo apt install \
          gnupg2 repo git git-lfs openssh-client golang jq zstd \
          qemu qemu-utils libvirt-dev libvirt-daemon python3-libvirt \
          libguestfs-tools virt-manager gir1.2-spiceclientglib-2.0 \
          gir1.2-spiceclientgtk-3.0

.. admonition:: Installing Podman or Docker
   :class: note

   On Debian and Ubuntu systems, you will have to choose between Podman
   (recommended) and Docker and install them on your system using the following
   guides:

   * `Podman for Ubuntu and Debian <https://github.com/containers/libpod/blob/master/install.md>`_
   * `Docker for Ubuntu <https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_
   * `Docker for Debian <https://docs.docker.com/install/linux/docker-ce/debian/>`_

.. admonition:: Installing libtpms and swtpm on Ubuntu or Debian
   :class: note

   As there is currently no official package for libtpms and swtpm on
   Ubuntu and Debian, you will have to follow the instructions from the
   ``INSTALL`` file on their respective GitHub repositories:
   `libtpms <https://github.com/stefanberger/libtpms>`_ and
   `swtpm <https://github.com/stefanberger/swtpm>`_.

On Fedora and CentOS:

.. code-block:: shell-session

   $ sudo dnf install \
          gnupg git git-lfs openssh-clients podman golang jq zstd \
          qemu libvirt-devel libvirt-daemon python3-libvirt libvirt-client \
          libguestfs-tools virt-manager

   # Vagrant setup for testbed environment
   $ sudo dnf install vagrant --setopt=install_weak_deps=False
   $ sudo dnf install @development-tools ruby-devel zlib-devel
   $ vagrant plugin install vagrant-libvirt

   # Virtual TPM support (Fedora only)
   $ sudo dnf install swtpm swtpm-tools

.. admonition:: Installing libtpms and swtpm on CentOS
   :class: note

   As there is currently no official package for libtpms and swtpm on CentOS,
   you will have to follow the instructions from the ``INSTALL`` file on their
   respective GitHub repositories:
   `libtpms <https://github.com/stefanberger/libtpms>`_ and
   `swtpm <https://github.com/stefanberger/swtpm>`_.

On Arch Linux:

.. code-block:: shell-session

   $ sudo pacman -Syu \
         gnupg repo git git-lfs openssh podman go jq zstd \
         qemu libvirt bridge-utils dnsmasq ebtables libvirt-python \
         virt-manager

   # Vagrant setup for testbed environment
   $ sudo pacman -Syu vagrant
   $ vagrant plugin install vagrant-libvirt

.. admonition:: Installing libguestfs, libtpms and swtpm on Arch Linux
   :class: note

   As there is currently no official package for those packages on Arch Linux,
   you will have to install them using AUR packages:
   `libguestfs <https://aur.archlinux.org/packages/libguestfs>`_,
   `libtpms <https://aur.archlinux.org/packages/libtpms>`_,
   `tpm-tools <https://aur.archlinux.org/packages/tpm-tools>`_ and
   `swtpm <https://aur.archlinux.org/packages/swtpm>`_.

How to fetch the entire source tree?
------------------------------------

The project source tree is split among several distinct repositories that are
managed together using ``repo``.

.. admonition:: Make sure the Git LFS filters are enabled
   :class: important

   To enable the Git LFS filters for your current user (changes will be made in
   ``~/.gitconfig``), please run:

   .. code-block:: shell-session

      $ git-lfs install --skip-repo

   This step should be done before synchronizing the whole CLIP OS source tree
   to automatically download the files stored with Git LFS.

.. admonition:: Watch out for unusual *umask* values!
   :class: error

   Due to the fact that we bind-mount the source tree within SDK containers,
   **please ensure to fetch and synchronize the entire source tree with a umask
   value keeping permissions to read files and traverse directories**
   (recommended *umask* value ``0022``).

   Failure to do so may lead to undefined issues when using the CLIP OS toolkit
   as all the file modes of this source tree are left unchanged when they are
   exposed within SDK containers. As a consequence, some unprivileged programs
   running in these containers might encounter a "Permission denied" error when
   trying to read files whose mode deny access for "others".

Then to get the entire source tree:

.. code-block:: shell-session

   $ mkdir clipos
   $ cd clipos
   $ umask 0022
   $ git lfs install --skip-repo
   $ repo init -u https://github.com/CLIPOS/manifest
   $ repo sync

This may take some time (several minutes at least, but this depends on your
network bandwidth) as several Git repositories need to be cloned, including
large Git repositories holding lots of contents and history, such as the Linux
kernel (``src/external/linux/``) or the Gentoo Portage tree
(``src/portage/gentoo/``).

.. admonition:: Quicker synchronization
   :class: tip

   You can instruct ``repo`` to synchronize all the sub-repositories
   concurrently by using multiple Git processes:

   .. code-block:: shell-session

      $ repo sync -j4

At this point, you should have successfully set up your environment and
fetched the whole source tree of the CLIP OS project.

.. admonition:: Quick fix for Git LFS issues
   :class: note

   If for any reason the Git LFS filters were not installed during ``repo
   sync``, you can still download the missing contents of the files backed by
   Git LFS (and therefore fix your current source tree checkout) by running
   this command:

   .. code-block:: shell-session

      $ repo forall -c 'git lfs install && git lfs pull'

Congratulations, you are now ready to start :ref:`building CLIP OS <build>`.

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
