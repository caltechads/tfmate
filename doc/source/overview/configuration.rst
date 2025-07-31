Configuration: Command Line Tool
================================

This guide covers all configuration options for the ``tfmate`` command line
tool, including configuration files, environment variables, and command-line
options.

``tfmate`` is a Terraform maintenance tool that provides analysis capabilities
for Terraform configurations and state files. The default configuration should
work for most use cases, but you can customize behavior through various
configuration methods.

Configuration Methods
---------------------

The ``tfmate`` command line tool supports multiple configuration methods,
loaded in order of priority:

1. **Command-line options** (highest priority)
2. **Environment variables**
3. **Configuration files**
4. **Default values** (lowest priority)

Configuration Files
-------------------

File Locations
~~~~~~~~~~~~~~

Configuration files are searched in this order:

1. **Global configuration**: ``/etc/tfmate/config.toml`` (Unix/Linux) or ``C:/ProgramData/tfmate/config.toml`` (Windows)
2. **User configuration**: ``~/.config/tfmate/config.toml``
3. **Local configuration**: ``./tfmate.toml`` (current directory)
4. **Explicit configuration**: Path specified with ``--config-file`` option

File Format
~~~~~~~~~~~

Configuration files use TOML format:

.. code-block:: toml

    # Application settings
    app_name = "tfmate"
    app_version = "0.1.0"

    # Output settings
    default_output_format = "table"
    enable_colors = true
    quiet_mode = false

    # AWS settings
    aws_default_region = "us-west-2"
    aws_default_profile = "default"

    # Terraform settings
    terraform_timeout = 30
    terraform_max_retries = 3

    # Logging settings
    log_level = "INFO"
    log_file = "/var/log/tfmate.log"

Configuration Options
~~~~~~~~~~~~~~~~~~~~~

**Application Settings**
    - **app_name**: Application name (default: ``tfmate``)
    - **app_version**: Application version (default: ``0.1.0``)

**Output Settings**
    - **default_output_format**: Default output format - ``table``, ``json``, or ``text`` (default: ``table``)
    - **enable_colors**: Enable colored output (default: ``true``)
    - **quiet_mode**: Enable quiet mode (default: ``false``)

**AWS Settings**
    - **aws_default_region**: Default AWS region (default: ``None``)
    - **aws_default_profile**: Default AWS profile (default: ``None``)

**Terraform Settings**
    - **terraform_timeout**: Terraform operation timeout in seconds (default: ``30``)
    - **terraform_max_retries**: Maximum retries for Terraform operations (default: ``3``)

**Logging Settings**
    - **log_level**: Logging level - ``DEBUG``, ``INFO``, ``WARNING``, ``ERROR`` (default: ``INFO``)
    - **log_file**: Log file path (default: ``None``)

Environment Variables
---------------------

You can set configuration using environment variables. Environment variables
follow the pattern ``tfmate_<SETTING_NAME>``:

.. code-block:: bash

    # Set AWS region
    export tfmate_AWS_DEFAULT_REGION="us-west-2"

    # Set output format
    export tfmate_DEFAULT_OUTPUT_FORMAT="json"

    # Set Terraform timeout
    export tfmate_TERRAFORM_TIMEOUT="60"

    # Set log level
    export tfmate_LOG_LEVEL="DEBUG"

Environment Variable Mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``tfmate_APP_NAME`` → ``app_name``
- ``tfmate_APP_VERSION`` → ``app_version``
- ``tfmate_DEFAULT_OUTPUT_FORMAT`` → ``default_output_format``
- ``tfmate_ENABLE_COLORS`` → ``enable_colors``
- ``tfmate_QUIET_MODE`` → ``quiet_mode``
- ``tfmate_AWS_DEFAULT_REGION`` → ``aws_default_region``
- ``tfmate_AWS_DEFAULT_PROFILE`` → ``aws_default_profile``
- ``tfmate_TERRAFORM_TIMEOUT`` → ``terraform_timeout``
- ``tfmate_TERRAFORM_MAX_RETRIES`` → ``terraform_max_retries``
- ``tfmate_LOG_LEVEL`` → ``log_level``
- ``tfmate_LOG_FILE`` → ``log_file``

Command-Line Options
--------------------

Global Options
~~~~~~~~~~~~~~

All commands support these global options:

.. code-block:: bash

    # Enable verbose output
    tfmate --verbose command

    # Suppress all output except errors
    tfmate --quiet command

    # Specify custom configuration file
    tfmate --config-file /path/to/config.toml command

    # Choose output format
    tfmate --output json command
    tfmate --output table command
    tfmate --output text command

Option Reference
~~~~~~~~~~~~~~~~

**--verbose, -v**
    Enable verbose output with detailed logging.

    Example:
    .. code-block:: bash

        tfmate --verbose analyze config

**--quiet, -q**
    Suppress all output except errors.

    Example:
    .. code-block:: bash

        tfmate --quiet aws services

**--config-file**
    Specify a custom configuration file path.

    Example:
    .. code-block:: bash

        tfmate --config-file ./custom-config.toml analyze config

**--output**
    Choose output format: ``json``, ``table``, or ``text``.

    Default: ``table``

    Example:
    .. code-block:: bash

        tfmate --output json aws services

Configuration Examples
----------------------

Basic Setup
~~~~~~~~~~~

For basic usage with defaults:

.. code-block:: toml

    # ~/.config/tfmate/config.toml
    # No configuration file needed - defaults work for most cases

Development Environment
~~~~~~~~~~~~~~~~~~~~~~~

For development and testing:

.. code-block:: toml

    # ~/.config/tfmate/config.toml
    [tfmate]
    default_output_format = "json"
    enable_colors = true
    log_level = "DEBUG"
    terraform_timeout = 10
    terraform_max_retries = 1

Production Environment
~~~~~~~~~~~~~~~~~~~~~~

For production systems:

.. code-block:: toml

    # /etc/tfmate/config.toml
    [tfmate]
    default_output_format = "table"
    enable_colors = false
    log_level = "WARNING"
    terraform_timeout = 60
    terraform_max_retries = 5
    log_file = "/var/log/tfmate.log"

AWS-Specific Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~

For AWS-focused workflows:

.. code-block:: toml

    # ~/.config/tfmate/config.toml
    [tfmate]
    aws_default_region = "us-west-2"
    aws_default_profile = "production"
    terraform_timeout = 45
    terraform_max_retries = 3

Scripting Configuration
~~~~~~~~~~~~~~~~~~~~~~~

For automation and scripting:

.. code-block:: toml

    # ~/.config/tfmate/config.toml
    [tfmate]
    default_output_format = "json"
    enable_colors = false
    quiet_mode = true
    log_level = "ERROR"

Network-Specific Configuration
------------------------------

Slow Networks
~~~~~~~~~~~~~

For slow or unreliable networks:

.. code-block:: toml

    # ~/.config/tfmate/config.toml
    [tfmate]
    terraform_timeout = 120
    terraform_max_retries = 5
    log_level = "INFO"

Security Considerations
-----------------------

Configuration File Security
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Protect your configuration files:

.. code-block:: bash

    # Set proper permissions for user configuration
    chmod 600 ~/.config/tfmate/config.toml

    # For system-wide configuration
    chmod 640 /etc/tfmate/config.toml
    chown root:root /etc/tfmate/config.toml

Environment Variable Security
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Secure environment variable usage:

.. code-block:: bash

    # Set variables for current session only
    export tfmate_AWS_DEFAULT_REGION="us-west-2"

    # Clear sensitive variables when done
    unset tfmate_AWS_DEFAULT_REGION
    unset tfmate_AWS_DEFAULT_PROFILE

Troubleshooting Configuration
-----------------------------

Configuration Debugging
~~~~~~~~~~~~~~~~~~~~~~~

Check which configuration is being used:

.. code-block:: python

    from tfmate.settings import Settings

    # Load and display configuration
    settings = Settings()
    print(f"Output format: {settings.default_output_format}")
    print(f"Timeout: {settings.terraform_timeout}")
    print(f"AWS region: {settings.aws_default_region}")

Common Issues
~~~~~~~~~~~~~

**Configuration Not Loaded**
    - Check file permissions
    - Verify file format (TOML syntax)
    - Ensure file is in correct location
    - Check for syntax errors in TOML file

**Configuration Not Valid**
    - Verify TOML syntax is correct
    - Check that setting names match expected values
    - Ensure boolean values are ``true``/``false``, not ``True``/``False``

**Environment Variables Not Recognized**
    - Check variable names (must start with ``tfmate_``)
    - Restart terminal session
    - Verify variable values

**Command-Line Options Override**
    - Command-line options take highest priority
    - Check for conflicting options
    - Use ``--help`` to see current options

Configuration Validation
------------------------

Validation Rules
~~~~~~~~~~~~~~~~

The library validates configuration:

- **default_output_format**: Must be one of ``table``, ``json``, or ``text``
- **terraform_timeout**: Must be a positive integer
- **terraform_max_retries**: Must be a non-negative integer
- **log_level**: Must be one of ``DEBUG``, ``INFO``, ``WARNING``, or ``ERROR``
- **enable_colors**: Must be a boolean value
- **quiet_mode**: Must be a boolean value

Error Messages
~~~~~~~~~~~~~~

Common validation errors:

.. code-block:: bash

    # Invalid output format
    Error: Invalid default_output_format value

    # Invalid timeout
    Error: terraform_timeout must be a positive integer

    # Invalid log level
    Error: log_level must be one of DEBUG, INFO, WARNING, ERROR

Best Practices
--------------

Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~

1. **Use configuration files for defaults**

   - Set common settings in ``~/.config/tfmate/config.toml``
   - Use environment variables for overrides
   - Use command-line options for one-time changes

2. **Separate environments**

   - Use different config files for different environments
   - Use environment variables for sensitive data
   - Document configuration requirements

3. **Version control**

   - Don't commit sensitive configuration
   - Use templates for configuration files
   - Document configuration changes

4. **Security**

   - Protect configuration files with proper permissions
   - Use environment variables for credentials
   - Clear sensitive environment variables

5. **Testing**

   - Test timeout settings for your environment
   - Verify output formats work for your use case
   - Test logging configuration

Configuration Templates
-----------------------

Basic Template
~~~~~~~~~~~~~~

.. code-block:: toml

    # config.toml.template
    # Application settings
    [tfmate]
    app_name = "tfmate"
    app_version = "0.1.0"

    # Output settings
    default_output_format = "table"
    enable_colors = true
    quiet_mode = false

    # AWS settings
    aws_default_region = "us-west-2"
    aws_default_profile = "default"

    # Terraform settings
    terraform_timeout = 30
    terraform_max_retries = 3

    # Logging settings
    log_level = "INFO"
    log_file = null

Production Template
~~~~~~~~~~~~~~~~~~~

.. code-block:: toml

    # production.toml
    # Application settings
    [tfmate]
    app_name = "tfmate"
    app_version = "0.1.0"

    # Output settings
    default_output_format = "table"
    enable_colors = false
    quiet_mode = false

    # AWS settings
    aws_default_region = "us-west-2"
    aws_default_profile = "production"

    # Terraform settings
    terraform_timeout = 60
    terraform_max_retries = 5

    # Logging settings
    log_level = "WARNING"
    log_file = "/var/log/tfmate.log"

Development Template
~~~~~~~~~~~~~~~~~~~~

.. code-block:: toml

    # development.toml
    # Application settings
    [tfmate]
    app_name = "tfmate"
    app_version = "0.1.0"

    # Output settings
    default_output_format = "json"
    enable_colors = true
    quiet_mode = false

    # AWS settings
    aws_default_region = "us-east-1"
    aws_default_profile = "dev"

    # Terraform settings
    terraform_timeout = 10
    terraform_max_retries = 1

    # Logging settings
    log_level = "DEBUG"
    log_file = null

Scripting Template
~~~~~~~~~~~~~~~~~~

.. code-block:: toml

    # scripting.toml
    # Application settings
    [tfmate]
    app_name = "tfmate"
    app_version = "0.1.0"

    # Output settings
    default_output_format = "json"
    enable_colors = false
    quiet_mode = true

    # AWS settings
    aws_default_region = "us-west-2"
    aws_default_profile = "automation"

    # Terraform settings
    terraform_timeout = 45
    terraform_max_retries = 3

    # Logging settings
    log_level = "ERROR"
    log_file = null