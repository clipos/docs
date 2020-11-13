.. Copyright © 2020 ANSSI.
   CLIP OS is a trademark of the French Republic.  Content licensed under the Open License version 2.0 as published by Etalab
   (French task force for Open Data).

.. _maintaining_portage_tree:

Maintaining the Portage tree
============================

CLIP OS leverages the `portage-derive <https://github.com/clipos/src_platform_portage-derive>`_
tool to efficiently maintain its Portage tree.

Fetching the upstream Portage tree
----------------------------------

First, start by retrieving the latest Gentoo Portage tree using the
``fetch-upstreams`` helper script:

.. code-block:: shell-session

  $ repo forall src_portage_gentoo --command "fetch-upstreams.sh"

This will fetch upstream remote references and update the local
``upstream/gentoo/master`` branch.

Updating the Portage tree using portage-derive
----------------------------------------------

In compliance with the `portage-derive documentation
<https://github.com/clipos/src_platform_portage-derive/blob/master/README.md>`_,
an *artificial merge* should be started to keep track of the Git history:

.. code-block:: shell-session

  $ cd src/portage/gentoo
  $ git checkout autoclean
  $ git merge --no-commit --strategy=ours upstream/gentoo/master
  $ git read-tree -u -m upstream/gentoo/master

Next, **enter the SDK** and **equalize** the Portage tree:

.. code-block:: shell-session

  $ cosmk run sdk

  # cd /mnt/src/portage/gentoo
  # egencache --update "--jobs=$(($(nproc) + 1))" --repo=gentoo
  # portage-derive --portdir . \
    --profile /mnt/src/portage/clipos/profiles/clipos/arch/amd64/core     \
    --profile /mnt/src/portage/clipos/profiles/clipos/arch/amd64/efiboot  \
    --profile /mnt/src/portage/clipos/profiles/clipos/arch/amd64/sdk      \
    equalize

**Exit the SDK** and commit the merge:

.. code-block:: shell-session

  # exit

  $ git add --all
  $ git commit --message="Merge branch 'upstream/gentoo/master' into autoclean"

Finally, merge the *equalized* branch into the ``master`` branch, which
contains CLIP OS custom changes:

.. code-block:: shell-session

  $ git checkout master
  $ git merge autoclean

Manually deal with the remaining merge conflicts, if any.

.. admonition:: Before going further, make sure to bootstrap a new SDK
   :class: important

   Before rebuilding CLIP OS with the now up-to-date Portage tree, a new SDK
   should be :ref:`rebuilt <maintaining_sdk>`.

.. vim: set tw=79 ts=2 sts=2 sw=2 et:
