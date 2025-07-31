Frequently Asked Questions
==========================

This section answers common questions about tfmate and provides solutions to frequently encountered issues.

General Questions
-----------------

What is tfmate?
^^^^^^^^^^^^^^^

tfmate is a Python command-line tool designed for Terraform module maintenance and analysis. It supports Terraform 1.5+ and provides capabilities for:

- Analyzing Terraform configurations
- Accessing state files from various backends (local, S3, HTTP, Terraform Enterprise)
- Listing AWS services and their metadata
- Extracting Terraform version information from state files

What Terraform versions does tfmate support?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tfmate is designed specifically for Terraform 1.5 and later versions. It requires state files to be in version 4 format, which is the standard for Terraform 1.5+.

What backends does tfmate support?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tfmate supports the following Terraform backends:

- **Local**: Standard local state files
- **S3**: AWS S3 backend with profile and assume role support
- **HTTP**: HTTP backend with basic authentication
- **Remote**: Terraform Enterprise backend

Installation Issues
-------------------

How do I install tfmate?
^^^^^^^^^^^^^^^^^^^^^^^^

See the :doc:`installation` guide for detailed installation instructions. The recommended methods are:

- Using ``uv tool``: ``uv tool install tfmate``
- Using ``pipx``: ``pipx install tfmate``
- Using ``pip``: ``pip install tfmate``
- From source: Clone the repository and run ``uv sync``

I get a "command not found" error after installation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This usually means the installation directory is not in your PATH. Try:

1. Restart your terminal session
2. Check if the installation directory is in your PATH
3. For pipx installations, ensure pipx is in your PATH
4. For uv tool installations, ensure uv is properly configured

Usage Questions
---------------

How do I list AWS services?
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``aws services`` command:

.. code-block:: bash

    # List all services
    tfmate aws services

    # List only service names (JSON format)
    tfmate aws services --names-only

    # Filter services by pattern
    tfmate aws services --filter "ec*"

    # Sort by API version
    tfmate aws services --sort-by api_version

How do I get the Terraform version from a state file?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``terraform version`` command:

.. code-block:: bash

    # Get version from current directory
    tfmate terraform version

    # Get version from specific directory
    tfmate terraform version --directory /path/to/terraform

    # Get version from explicit state file
    tfmate terraform version --state-file terraform.tfstate

How do I analyze a Terraform configuration?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``analyze config`` command:

.. code-block:: bash

    # Basic analysis
    tfmate analyze config

    # Show provider configurations
    tfmate analyze config --show-providers

    # Show backend configuration
    tfmate analyze config --show-backend

    # Analyze specific directory
    tfmate analyze config --directory /path/to/terraform

Backend and Credential Issues
-----------------------------

How does tfmate handle AWS credentials?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tfmate automatically detects credentials from:

1. S3 backend configuration (profile, region, assume_role)
2. AWS provider configuration (fallback)
3. Environment variables and AWS config files

The tool follows the same credential resolution order as the AWS CLI and Terraform.

I get "Access denied" errors when accessing S3 state
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This usually indicates credential or permission issues:

1. **Check AWS credentials**: Ensure your AWS credentials are properly configured.  If you use SSO, ensure you are logged in and have done ``aws sso login`` from the command line.
2. **Verify permissions**: The credentials need S3 read access to the state bucket
3. **Check bucket policy**: Ensure the bucket allows access from your account/role
4. **Verify region**: Make sure you're using the correct AWS region
5. **Check assume role**: If using assume role, verify the role ARN and permissions

Common solutions:

.. code-block:: bash

    # Check your AWS credentials
    aws sts get-caller-identity

    # Test S3 access directly
    aws s3 ls s3://your-terraform-state-bucket/

    # Use verbose mode for more details
    tfmate --verbose terraform version

I get "State file not found" errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This can happen for several reasons:

1. **Wrong bucket/key**: Check that the S3 bucket and key are correct
2. **State file doesn't exist**: The state file may not have been created yet
3. **Wrong backend configuration**: Verify the backend configuration in your Terraform files

Solutions:

.. code-block:: bash

    # Check if the state file exists in S3
    aws s3 ls s3://your-bucket/your-key

    # Run terraform init to initialize the backend
    terraform init

    # Use verbose mode to see backend detection
    tfmate --verbose analyze config

How do I use tfmate with Terraform Enterprise?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For Terraform Enterprise backends, tfmate needs:

1. **Organization name**: Specified in the backend configuration
2. **Workspace name**: Specified in the backend configuration
3. **TFE token**: Set as an environment variable or in the backend configuration

Example configuration:

.. code-block:: hcl

    terraform {
      backend "remote" {
        hostname = "app.terraform.io"
        organization = "my-org"
        workspaces {
          name = "my-workspace"
        }
      }
    }

Set your TFE token:

.. code-block:: bash

    export TF_TOKEN_app_terraform_io="your-tfe-token"
    tfmate terraform version

Output and Formatting Issues
----------------------------

How do I change the output format?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``--output`` option:

.. code-block:: bash

    # JSON output
    tfmate --output json aws services

    # Table output (default)
    tfmate --output table aws services

    # Text output
    tfmate --output text aws services

The output format applies to all commands in the session.

How do I enable verbose output?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``--verbose`` option:

.. code-block:: bash

    # Enable verbose output
    tfmate --verbose aws services

    # Verbose output with specific command
    tfmate --verbose terraform version

Verbose output shows additional details about:
- Backend detection
- Credential resolution
- API calls and responses
- Error details

How do I suppress output except errors?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``--quiet`` option:

.. code-block:: bash

    # Suppress all output except errors
    tfmate --quiet aws services

This is useful in scripts where you only want to see error messages.

Configuration Issues
--------------------

How do I use a custom configuration file?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the ``--config-file`` option:

.. code-block:: bash

    # Use custom configuration file
    tfmate --config-file /path/to/config.toml aws services

The configuration file should be in TOML format. See the :doc:`configuration` guide for details.

What configuration options are available?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tfmate supports configuration for:

- AWS credentials and regions
- Output formatting preferences
- Logging levels
- Timeout settings

See the :doc:`configuration` guide for a complete list of options.

Troubleshooting
---------------

The tool is slow when accessing S3 state
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This can happen due to:

1. **Network latency**: S3 access depends on network speed
2. **Large state files**: Very large state files take longer to download
3. **Cold S3 requests**: First access to S3 objects may be slower

Solutions:

.. code-block:: bash

    # Use verbose mode to see timing information
    tfmate --verbose terraform version

    # Check state file size
    aws s3 ls s3://your-bucket/your-key --human-readable

I get "Unsupported state version" errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This means the state file is from a Terraform version older than 1.5. tfmate only supports state version 4, which is used by Terraform 1.5+.

Solutions:

1. **Upgrade Terraform**: Update to Terraform 1.5 or later
2. **Migrate state**: Run ``terraform state migrate`` to upgrade the state format
3. **Use older tool**: If you must use older Terraform, use a different tool

The tool crashes with "Invalid JSON" errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This usually indicates a corrupted state file or invalid response from the backend.

Solutions:

.. code-block:: bash

    # Check if the state file is valid JSON
    cat terraform.tfstate | python -m json.tool

    # Try pulling a fresh state file
    terraform state pull > terraform.tfstate

    # Use verbose mode for more error details
    tfmate --verbose terraform version

I get "Module not found" errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This usually means the Terraform configuration references modules that aren't available.

Solutions:

1. **Initialize Terraform**: Run ``terraform init`` to download modules
2. **Check module sources**: Verify module sources are accessible
3. **Check working directory**: Ensure you're in the correct Terraform directory

Performance and Limitations
---------------------------

What are the performance characteristics?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **AWS services listing**: Very fast (reads local botocore files)
- **Local state access**: Very fast (reads local files)
- **S3 state access**: Depends on network speed and state file size
- **HTTP state access**: Depends on server response time
- **TFE state access**: Depends on TFE API response time

Are there any limitations?
^^^^^^^^^^^^^^^^^^^^^^^^^^

- **Terraform version**: Only supports Terraform 1.5+
- **State format**: Only supports state version 4
- **Backend types**: Limited to local, S3, HTTP, and TFE backends
- **Authentication**: Basic authentication for HTTP, token-based for TFE
- **AWS services**: Only lists services available in botocore

Can I use tfmate in CI/CD pipelines?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Yes, tfmate is designed to work in CI/CD environments:

.. code-block:: yaml

    # Example GitHub Actions step
    - name: Get Terraform version
      run: |
        tfmate terraform version --output json > terraform_version.json

    # Example with error handling
    - name: Analyze Terraform config
      run: |
        if ! tfmate analyze config --output json > config_analysis.json; then
          echo "Terraform configuration analysis failed"
          exit 1
        fi

Getting Help
------------

Where can I get more help?
^^^^^^^^^^^^^^^^^^^^^^^^^^

1. **Documentation**: Check the other sections of this documentation
2. **Command help**: Use ``tfmate --help`` or ``tfmate <command> --help``
3. **Verbose mode**: Use ``--verbose`` for detailed error information
4. **GitHub issues**: Report bugs or request features on the project repository

How do I report a bug?
^^^^^^^^^^^^^^^^^^^^^^

When reporting a bug, please include:

1. **Command used**: The exact command that failed
2. **Error message**: The complete error output
3. **Environment**: OS, Python version, tfmate version
4. **Terraform version**: The Terraform version being used
5. **Backend type**: Local, S3, HTTP, or TFE
6. **Verbose output**: Use ``--verbose`` and include the output

Example bug report:

.. code-block:: text

    Command: tfmate terraform version --directory /path/to/terraform
    Error: State file error at s3://bucket/key: Access denied
    OS: macOS 14.0
    Python: 3.11.9
    tfmate: 0.1.0
    Terraform: 1.5.0
    Backend: S3

    Verbose output:
    [Include verbose output here]