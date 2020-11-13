.. Copyright © 2020 ANSSI.
   CLIP OS is a trademark of the French Republic.  Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _maintaining_sdk:

Maintaining the SDK
===================

.. admonition:: Before going further, make sure to update the Portage tree
   :class: important

   You should complete the :ref:`Portage tree update <maintaining_portage_tree>`
   steps before executing any command from this page.


Grabbing a new Gentoo stage 3 tarball
-------------------------------------

In order to obtain the latest Gentoo stage 3 tarball, use the ``vendor-gentoo-stage3``
helper script, which shall store it in ``assets/gentoo``:

.. code-block:: shell-session

  $ toolkit/helpers/vendor-gentoo-stage3.sh

Updating the SDK recipe
-----------------------

The SDK recipe then **needs** to be updated:

.. code-block:: shell-session

  # product/clipos/sdk/sdk.toml
  tag = <portage tree tag>
  rootfs = "assets/gentoo/stage3-amd64-hardened+nomultilib-<portage tree tag>.tar.xz"

Rebuilding the SDK from scratch
-------------------------------

* Remove all **distfiles** from the ``assets/distfiles`` directory (**and only
  distfiles**, do not delete ``README.md`` nor hidden configuration files)
* Delete the ``out/sdk`` and ``cache/sdk`` directories.
* **Disable** all instrumentation features for the SDK build and rebuild the
  SDK:

.. code-block:: shell-session

  $ cosmk bootstrap sdk

The resulting SDK can then be employed, for instance, to launch a full project
rebuild with the up-to-date Portage tree and the desired instrumentation
features.

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
