Guidelines for writing code for tfmate
======================================

Project Overview
----------------

This is a a Python 3.10+ command-line tool.  Use python 3.10+ to write the code, preferably the latest version of stable python.

The project is organized into the following directories:

- ``doc/source/``: The documentation directory.
- ``tfmate/``: The main project directory.
- ``tfmate/services/``: The services directory.  This is where the code for the services is located.
- ``tfmate/tests/``: The tests directory.
- ``tfmate/exc.py``: The exceptions module.  Put any custom exceptions here.
- ``tfmate/settings.py``: The settings module.
- ``tfmate/types.py``: The types module.  Put any custom types here.
- ``tests/``: The tests directory.

Coding Guidelines
-----------------

General
^^^^^^^

Once you've created the ``.venv`` virtual environment by doing ``uv sync --dev``, ``cd`` out of the project directory and back in and it will be activated automatically, if you have `autoenv <https://github.com/hyperupcall/autoenv>`_ installed.

If you don't, then just run ``source .venv/bin/activate`` to activate the virtual environment.

Code
^^^^

- Use python 3.10+ type hints for all functions and classes.
- Document all functions and classes using docstrings in Sphinx Napoleon format.
- Use ``ruff`` to lint the code, according to the settings in ``pyproject.toml``. Try to fix the lint errors by either fixing the code or adding a ``# noqa: <rule>`` comment to the line.
- in ``try:`` blocks, include specific exceptions that are expected to be raised.  Allow ``Exception`` to be raised if it is expected to be raised.
- Use ``mypy`` to type check the code, according to the settings in ``pyproject.toml``.  For any imported modules that lack type hints, add a section like this to the appropriate place in the ``pyproject.toml`` file:

    .. code-block:: toml

        [[tool.mypy.overrides]]
        module = "<module_name>.*"
        ignore_missing_imports = true

- When documenting methods, put positional arguments in the ``Args:`` section, keyword arguments in the ``Keyword Arguments:`` section, include any exceptions raised in the ``Raises:`` section, and of course include a ``Returns:`` section.

Example:

.. code-block:: python

    def my_function(arg1: str, arg2: int, kwarg1: str = "default", kwarg2: int = 10) -> str:
        """
        My function description

        Args:
            arg1: The first argument
            arg2: The second argument

        Keyword Arguments:
            kwarg1: The first keyword argument
            kwarg2: The second keyword argument

        Raises:
            ValueError: If kwarg1 is not 'foo' or 'bar'
            ValueError: If kwarg2 is less than 0

        Returns:
            The result of the function

        """
        if kwarg1 not in ['foo', 'bar']:
            raise ValueError("kwarg1 must be 'foo' or 'bar'")
        if kwarg2 < 0:
            raise ValueError("kwarg2 must be greater than 0")

        return f"{arg1} {arg2}"

- To document class attributes, put the documentation on the lines above before the attribute, with each line beginning with ``#:``. Do not list the class attributes in an ``Attributes:`` section of the class docstring.
- If you need any main dependencies, add them using ``uv add``.
- If you need test dependencies, them using ``uv add --group=test``
- Use pydantic models instead of ``@dataclass`` models unless this is a DAO type model that just transfers data between bits of the code internally.  Use pydantic models for anything that enventually makes it to the user.
- Put any custom exceptions into a top-level ``exc.py`` module.
- Put any type aliases we create into a top-level ``types.py`` module.

Example:

    .. code-block:: python

        from pydantic import BaseModel, AnyUrl

        class AWSService(BaseModel):
            """
            Describes an AWS service, e.g. ``ecs``
            """
            #: The name of the service
            name: str
            #: The id of the service
            service_id: str
            #: The api version
            api_version: str
            #: A list of service endpoints
            endpoints: list[str]
            #: Where the service is documented
            documentation_url: AnyUrl | None = None

- If the code needs additional configuration, it should be added to ``tfmate.settings.Settings``

CLI Design
^^^^^^^^^^
- We use ``click`` for CLI implementation with proper command grouping and help text.
- Use ``rich`` for all output formatting, progress indicators, and user interface elements.
- Implement global options that apply to all commands:
  - ``--verbose`` / ``-v``: Enable verbose output with detailed logging
  - ``--quiet`` / ``-q``: Suppress all output except errors
  - ``--config-file``: Specify custom configuration file path
  - ``--output``: Control output format (json, table, text)
- Use ``rich.progress`` for progress indicators on long-running operations.
- Use ``rich.console`` for consistent output formatting and error handling.
- Implement proper error handling with user-friendly error messages and suggestions.
- Add command aliases and shortcuts for common operations.
- Use ``rich.table`` for tabular data output.
- Use ``rich.panel`` for highlighting important information.
- Use ``rich.syntax`` for syntax highlighting of configuration files and outputs.
- Implement consistent color schemes and styling across all commands.
- All error messages and progress indicators should be written to stderr.
- All desired output should be written to stdout.

Testing
^^^^^^^

- See the :doc:`/runbook/testing` guide for details on testing.

- Tests are in the ``tests/`` directory.
- Add tests for any new functionality.

Integration Tests
^^^^^^^^^^^^^^^^^

- The integration tests are in ``tests/test_integration.py``
- The integration tests that require an actual terraform project use the environment variable ``TFTEST_PROJECT_PATH`` to name the project.  This should be the top level directory of the .tf files.
- Integration tests should be flexible and not make assumptions about the specific state of the test project.
- Avoid testing specific resource counts, names, or values that may change.
- Use pytest markers to skip integration tests when the environment variable is not set.
- Provide clear error messages when integration test prerequisites are not met.

Project Documentation
^^^^^^^^^^^^^^^^^^^^^

- Use Sphinx to document the project in RestructuredText format.
- Use Napoleon to format the docstrings.
- Use the ``.. code-block:: python`` directive to format code examples.
- Use the ``.. code-block:: bash`` directive to format shell commands.
- Use the ``.. code-block:: text`` directive to format text.
- Use the ``.. code-block:: json`` directive to format JSON.
- And so on.
- The quickstart section to the in ``doc/source/overview/quickstart.rst`` tells the user how to quickly get moving using the tool.
- Any changes to configuration should go in ``doc/source/overview/installation.rst``.  Include installing via ``pip``, ``uv tool``, ``pipx``, and cloning the repo.
- Update the usage guide in ``doc/source/overview/usage.rst``.  Include how to use your new commands, or update the existing commands to reflect the changes you made.
- Configuration of tfmate is discussed in ``doc/source/overview/configuration.rst``.  Add any new configuration options to this file.
- Add a comprehensive FAQ guide to the in ``doc/source/overview/faq.rst``.  Consider adding new FAQs if you can anticipate them here.
