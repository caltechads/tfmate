.. _runbook__contributing:

Contributing
============

Instructions for contributors
-----------------------------

Make a clone of the bitbucket repo:

.. code-block:: shell

    git clone https://github.com/caltechads/tfmate.git

Workflow is pretty straightforward:

1. Make sure you are reading the latest version of this document.
2. Setup your machine with the required development environment
3. Make your changes.
4. Update the Sphinx documentation to reflect your changes.
5. ``cd doc; make clean && make html; open build/html/index.html``. Ensure the
   docs build without crashing and then review the docs for accuracy.
6. Commit changes to your branch.
7. Merge your changes into master.
8. ``make release``.


Preconditions for working on this project
-----------------------------------------

Python environment
^^^^^^^^^^^^^^^^^^

Install your Python environment with the following:

MacOS and Linux

    .. code-block:: shell

        curl -LsSf https://astral.sh/uv/install.sh | sh

    Now add the following to your ``~/.bash_profile`` or ``~/.zshrc``:

    .. code-block:: shell

        export PATH="$HOME/.local/bin:$PATH"

    Then run:

    .. code-block:: shell

        uv sync --dev

    This will install the dependencies for the project.

Coding Guidelines
^^^^^^^^^^^^^^^^^

See :doc:`guidelines` for details about how to write code, documentation and tests for this project.

Building the documentation
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block::

   $ cd doc; make clean && make html; open build/html/index.html

The docs hopefully will build without errors, and you can review the docs for
accuracy.  If there are critical build errors, fix them.

Review the docs for accuracy, and fix any Sphinx compile errors related to your
new module.

.. _runbook__contributing__commmit-message-format:

Pull request guidelines
-----------------------

When submitting a pull request, please follow these guidelines, otherwise we
will ask you to fix it in your fork before we will approve and  merge your
changes.

Use atomic commits
^^^^^^^^^^^^^^^^^^

- If you've added a new module, ensure you add the module code, the documentation, and the test code in the same commit.
- If you've touched multiple modules, commit each module separately.  If you've added multiple new functionalities to a single module, commit each functionality separately even if the touch the same module.  Again, ensure you add the module code, the documentation, and the test code in the same commit.
- Simiarly for refactoring, commit each refactoring separately with the ``REFACTORING`` prefix.
- ``DEV:`` commits can be more lax -- you can commit all your dev changes in a single commit.

Commit message format
^^^^^^^^^^^^^^^^^^^^^

.. important::

   If you do not follow the following commit message format, we will ask you to
   fix it before we can merge your changes.

Commit message format
^^^^^^^^^^^^^^^^^^^^^

First, prefix your commit message with one of the following prefixes:

- ``FEATURE:`` for new modules
- ``BUGFIX:`` for bug fixes
- ``DOCS:`` for documentation changes where the code is not changed
- ``UPDATE:`` updates to existing modules
- ``REFACTORING:`` refactoring of existing code without changing functionality
- ``MISC:`` for other changes: testing, etc.
- ``DEV:`` for development environment oriented changes, e.g. updating the ``Makefile`` or ``.gitignore``

Then, add a short description of the changes you made.

.. code-block::

   <prefix>: <module-name> <short description of changes>

   <(optional) more explanation of changes>

Examples:

   .. code-block::

      FEATURE: my-module: new module that creates my-resource

      This module creates a new `foobar` resource in AWS.

   .. code-block::

      BUGFIX: my-module: Fix bug in existing module

      The `foobar` resource was not being created with the wrong default values.

   .. code-block::

      UPDATE: my-module: Update existing module to use new feature

      Added support for the ``new-attribute`` parameter to ``aws_foobar_resource``.

   .. code-block::

      DOCS: my-module: Fixed documentation regarding <some feature>

