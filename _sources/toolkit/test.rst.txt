.. Copyright © 2020 ANSSI.
   CLIP OS is a trademark of the French Republic.
   Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _test:

Testing
=======

.. admonition:: Prerequisite steps
   :class: important

   You must complete the :ref:`environment setup <setup>` and the :ref:`project
   build <build>` before executing any command from this page.

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
   passphrase at bootup if TPM support is not available:

   .. code-block:: shell-session

      $ sed -i '/#"initramfs-no-require-tpm"/s/#//g' config.toml

   The default passphrase is ``clipos`` (for old builds, it used to be
   ``core_state_key``).

   Any change of instrumentation features requires a full project rebuild.

To build a QCOW2 QEMU disk image and to setup a EFI & QEMU/KVM enabled virtual
machine with ``libvirt``, use:

.. code-block:: shell-session

   $ cosmk test qemu

.. admonition:: Local login disabled by default
   :class: important

   The default build configuration will create production images with root
   access disabled. To enable local passwordless root login, enable the
   ``passwordless-root-login`` instrumentation feature:

   .. code-block:: shell-session

      $ sed -i '/#"passwordless-root-login"/s/#//g' config.toml

   Any change of instrumentation features requires a full project rebuild.

.. admonition:: Workaround for SELinux on Fedora, CentOS and RHEL
   :class: important

   To allow QEMU virtual machines access to files in your home directory, you
   need to turn SELinux into permissive mode for the ``svirt_t`` domain:

   .. code-block:: shell-session

      $ sudo semanage permissive --add svirt_t

   This change can be revert with:

   .. code-block:: shell-session

      $ sudo semanage permissive --delete svirt_t

.. admonition:: Access to ``testbed/synced_folders`` for QEMU virtual machines
   :class: important

   Make sure that the directory ``testbed/synced_folders`` used by Vagrant is
   also accessible for everybody:

   .. code-block:: shell-session

      $ chmod o+x ~user{,/clipos{,/testbed{,/synced_folders}}}

Access to QEMU virtual machine over SSH
---------------------------------------

.. admonition:: Access disabled by default
   :class: important

   The default build configuration will create production images with SSH
   access available only over the IPsec tunnel. To enable SSH access from
   outside the IPsec tunnel, enable the ``allow-ssh-root-login``
   instrumentation feature:

   .. code-block:: shell-session

      $ sed -i '/#"allow-ssh-root-login"/s/#//g' config.toml

   Any change of instrumentation features requires a full project rebuild.

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

Access to vagrant virtual machine over SSH
------------------------------------------

To access a Vagrant virtual machine over SSH, you can use the ``vagrant ssh``
command in the testbed directory:

.. code-block:: shell-session
  $ vagrant ssh ipsec-gw
  $ sudo -i

Testing updates
---------------

.. admonition:: Test updates disabled by default
   :class: important

   The default build configuration will create production images that do not
   include the insecure key used to verify the integrity of test updates. This
   key must not be used in production as the private one is public (included in
   the testbed repository). To include this key in system images and build test
   updates, enable the ``test-update`` instrumentation feature:

   .. code-block:: shell-session

      $ sed -i '/#"test-update"/s/#//g' config.toml

   Any change of instrumentation features requires a full project rebuild.

Log in as ``root`` and start the update process with the following command:

.. code-block:: shell-session

  $ systemctl start updater

Testing chrony
--------------

In order to test the NTP communication between CLIP OS and the IPsec gateway,
log in as ``root`` in the Vagrant ipsec-gw virtual machine and list the clients connected to
the IPsec side chrony server:

.. code-block:: shell-session
  # chronyc -h /var/run/chrony-ipsec/chrony-ipsec.sock
  chronyc> clients
  Hostname                      NTP   Drop Int IntL Last     Cmd   Drop Int  Last
  ===============================================================================
  foo.example.net                12      0   6   -    23       0      0   -     -

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
