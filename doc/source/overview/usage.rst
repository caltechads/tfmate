Using the Command Line Interface
================================

The ``tfmate`` command-line interface provides supporting functionality for
working with Terraform code. This guide covers all available commands and options.

.. important::

    **Terraform Workspace Support**: tfmate automatically detects and supports Terraform workspaces. When analyzing Terraform configurations or retrieving version information, only the currently selected workspace will be checked. Workspaces are detected from the ``.terraform/environment`` file or the ``TF_WORKSPACE`` environment variable. If you need to check multiple workspaces, you must switch workspaces manually using ``terraform workspace select``.

Getting Help
------------

Basic Help
~~~~~~~~~~

.. code-block:: bash

    # Show main help
    tfmate --help

    # Show help for specific command groups
    tfmate analyze --help
    tfmate aws --help
    tfmate terraform --help
    tfmate settings --help

    # Show help for specific commands
    tfmate analyze config --help
    tfmate aws services --help
    tfmate terraform version --help

Command Structure
-----------------

The CLI follows a hierarchical command structure:

.. code-block:: bash

    tfmate [global-options] <command-group> <subcommand> [options]

Global Options
--------------

Common options available for all commands:

.. code-block:: bash

    # Enable verbose output
    tfmate --verbose command

    # Suppress all output except errors
    tfmate --quiet command

    # Specify custom configuration file
    tfmate --config-file /path/to/config.yaml command

    # Choose output format (json, table, text)
    tfmate --output json command
    tfmate --output table command
    tfmate --output text command

Analysis Commands
-----------------

The ``analyze`` command group provides tools for analyzing Terraform configurations.

Analyze Configuration
~~~~~~~~~~~~~~~~~~~~~

Analyze Terraform configuration files to understand the setup:

.. code-block:: bash

    # Analyze current directory
    tfmate analyze config

    # Analyze specific directory
    tfmate analyze config --directory /path/to/terraform

    # Show provider configurations
    tfmate analyze config --show-providers

    # Show backend configuration
    tfmate analyze config --show-backend

    # Use JSON output
    tfmate --output json analyze config

Example output:

.. code-block:: json

    {
      "directory": "/path/to/terraform",
      "terraform_block": true,
      "required_version": ">= 1.5.0",
      "backend_type": "s3",
      "provider_count": 3,
      "providers": {
        "aws": {
          "region": "us-west-2",
          "version": "~> 5.0"
        }
      },
      "backend_config": {
        "bucket": "my-terraform-state",
        "key": "prod/terraform.tfstate",
        "region": "us-west-2"
      }
    }

Example output with workspace:

.. code-block:: json

    {
      "directory": "/path/to/terraform",
      "terraform_block": true,
      "required_version": ">= 1.5.0",
      "backend_type": "s3",
      "provider_count": 3,
      "workspace": "qa",
      "workspace_state_path": "s3://my-terraform-state/:env/qa/prod/terraform.tfstate",
      "providers": {
        "aws": {
          "region": "us-west-2",
          "version": "~> 5.0"
        }
      },
      "backend_config": {
        "bucket": "my-terraform-state",
        "key": "prod/terraform.tfstate",
        "region": "us-west-2"
      }
    }

AWS Commands
------------

The ``aws`` command group provides tools for AWS service discovery and analysis.

List AWS Services
~~~~~~~~~~~~~~~~~

List all available AWS services from botocore definitions:

.. code-block:: bash

    # List all services
    tfmate aws services

    # Output only service names
    tfmate aws services --names-only

    # Filter services by name pattern
    tfmate aws services --filter-name "ec2*"

    # Sort by different fields
    tfmate aws services --sort-by name
    tfmate aws services --sort-by service_id
    tfmate aws services --sort-by api_version

    # Use JSON output
    tfmate --output json aws services

Example output:

.. code-block:: json

    [
      {
        "name": "ec2",
        "service_id": "Amazon EC2",
        "api_version": "2016-11-15",
        "endpoints": ["ec2"],
        "documentation_url": "https://docs.aws.amazon.com/ec2/"
      },
      {
        "name": "s3",
        "service_id": "Amazon S3",
        "api_version": "2006-03-01",
        "endpoints": ["s3"],
        "documentation_url": "https://docs.aws.amazon.com/s3/"
      }
    ]

Terraform Commands
------------------

The ``terraform`` command group provides tools for Terraform-specific operations.

Settings Commands
-----------------

The ``settings`` command group provides tools for viewing and managing tfmate configuration.

Show Settings
~~~~~~~~~~~~~

Display current tfmate configuration settings:

.. code-block:: bash

    # Show all settings in table format (default)
    tfmate settings

    # Show settings in JSON format
    tfmate --output json settings

    # Show settings in text format
    tfmate --output text settings

    # Show settings with verbose output
    tfmate --verbose settings

    # Show settings with custom configuration file
    tfmate --config-file /path/to/config.env settings

Example output:

.. code-block:: json

    {
      "app_name": "tfmate",
      "app_version": "0.1.0",
      "default_output_format": "table",
      "enable_colors": true,
      "quiet_mode": false,
      "aws_default_region": null,
      "aws_default_profile": null,
      "terraform_timeout": 30,
      "terraform_max_retries": 3,
      "log_level": "INFO",
      "log_file": null
    }

Get Terraform Version
~~~~~~~~~~~~~~~~~~~~~

Get Terraform version from state file using Terraform-configured credentials:

.. code-block:: bash

    # Get version from current directory
    tfmate terraform version

    # Get version from specific directory
    tfmate terraform version --directory /path/to/terraform

    # Use explicit state file (local only)
    tfmate terraform version --state-file /path/to/terraform.tfstate

    # Use JSON output
    tfmate --output json terraform version

.. important::

    **Terraform Workspaces**: If the Terraform directory uses workspaces, only the currently selected workspace will be checked. The workspace is automatically detected from the ``.terraform/environment`` file or the ``TF_WORKSPACE`` environment variable. When a workspace is detected, the command will display workspace information and use the workspace-specific state file path.

Example output:

.. code-block:: json

    {
      "terraform_version": "1.5.7",
      "backend_type": "s3",
      "state_location": "s3://my-terraform-state/prod/terraform.tfstate"
    }

Example output with workspace:

.. code-block:: json

    {
      "terraform_version": "1.5.7",
      "backend_type": "s3",
      "state_location": "s3://my-terraform-state/prod/terraform.tfstate",
      "workspace_info": "Currently in workspace 'qa'. Switch workspaces to check versions."
    }

Show Settings
~~~~~~~~~~~~~

    # Use JSON output
    tfmate --output json settings show

Example output:

.. code-block:: json

    {
      "app_name": "tfmate",
      "app_version": "0.1.0",
      "default_output_format": "table",
      "enable_colors": true,
      "quiet_mode": false,
      "aws_default_region": null,
      "aws_default_profile": null,
      "terraform_timeout": 30,
      "terraform_max_retries": 3,
      "log_level": "INFO",
      "log_file": null
    }

Output Formats
--------------

JSON Format
~~~~~~~~~~~

.. code-block:: bash

    # JSON output for scripting and automation
    tfmate --output json analyze config > config.json

    # JSON output for AWS services
    tfmate --output json aws services > services.json

    # JSON output for settings
    tfmate --output json settings show > settings.json

Table Format (Default)
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Table output for better readability
    tfmate analyze config

    # Table output for AWS services
    tfmate aws services

    # Table output for settings
    tfmate settings show

Text Format
~~~~~~~~~~~

.. code-block:: bash

    # Simple text output
    tfmate --output text analyze config

    # Text output for names-only lists
    tfmate --output text aws services --names-only

    # Text output for settings
    tfmate --output text settings show

Configuration
-------------

See :doc:`/overview/configuration` for details on how to configure ``tfmate``
for your specific Terraform environment.

Examples
--------

Basic Usage Examples
~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Analyze a Terraform configuration
    tfmate analyze config --directory ./infrastructure

    # List AWS services
    tfmate aws services

    # Get Terraform version from state
    tfmate terraform version

    # Show current settings
    tfmate settings show

Advanced Usage Examples
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Analyze with detailed provider and backend info
    tfmate analyze config --show-providers --show-backend

    # Filter AWS services for compute-related services
    tfmate aws services --filter-name "*ec2*" --sort-by name

    # Get version from specific state file
    tfmate terraform version --state-file ./terraform.tfstate

    # Show settings with custom config file
    tfmate --config-file ./custom.env settings show

    # Use verbose output for debugging
    tfmate --verbose analyze config

    # Check Terraform version in different workspaces
    terraform workspace select prod
    tfmate terraform version
    terraform workspace select staging
    tfmate terraform version

Scripting Examples
~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    #!/bin/bash
    # Terraform analysis script

    echo "Analyzing Terraform configuration..."

    # Analyze configuration
    echo "Configuration Analysis:"
    tfmate --output table analyze config

    # Get Terraform version
    echo "Terraform Version:"
    tfmate --output table terraform version

    # List relevant AWS services
    echo "AWS Services (Compute):"
    tfmate --output table aws services --filter-name "*ec2*"

    # Show current settings
    echo "Current Settings:"
    tfmate --output table settings show

    echo "Analysis complete."

Error Handling
--------------

Common Error Scenarios
~~~~~~~~~~~~~~~~~~~~~~

**Configuration Not Found**
    .. code-block:: bash

        # Error: No Terraform configuration found
        tfmate analyze config
        # Error: No .tf files found in directory

        # Solution: Ensure you're in a Terraform directory
        ls *.tf

**State File Not Found**
    .. code-block:: bash

        # Error: State file not found
        tfmate terraform version
        # Error: Could not read state file

        # Solution: Check backend configuration or run terraform init

**AWS Credentials Not Found**
    .. code-block:: bash

        # Error: AWS credentials not configured
        tfmate terraform version
        # Error: No AWS credentials found

        # Solution: Configure AWS credentials
        aws configure

**Permission Errors**
    .. code-block:: bash

        # Error: Permission denied
        tfmate analyze config
        # Error: Cannot read directory

        # Solution: Check file permissions
        ls -la

**Workspace State File Not Found**
    .. code-block:: bash

        # Error: Workspace state file not found
        tfmate terraform version
        # Error: State file not found in S3

        # Solution: Check workspace state file path or switch workspaces
        terraform workspace list
        terraform workspace select <workspace-name>

Troubleshooting
---------------

Debugging Commands
~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Enable verbose output for debugging
    tfmate --verbose analyze config

    # Check if Terraform files exist
    find . -name "*.tf" -type f

    # Check AWS credentials
    aws sts get-caller-identity

    # Test state file access
    terraform show

Common Issues
~~~~~~~~~~~~~

**No Terraform Files Found**
    - Ensure you're in a directory with `.tf` files
    - Check that the directory path is correct
    - Verify file permissions

**State File Access Issues**
    - Run `terraform init` to initialize the backend
    - Check AWS credentials and permissions
    - Verify S3 bucket and key configuration

**AWS Service List Issues**
    - Ensure botocore is properly installed
    - Check internet connectivity for service definitions
    - Verify Python environment

**Output Format Issues**
    - Use `--output json` for machine-readable output
    - Use `--output table` for human-readable output
    - Use `--output text` for simple text output

**Workspace Issues**
    - Check current workspace with `terraform workspace show`
    - List available workspaces with `terraform workspace list`
    - Switch workspaces with `terraform workspace select <name>`
    - Verify workspace state file exists in the backend

Best Practices
--------------

Output Format Selection
~~~~~~~~~~~~~~~~~~~~~~~

Choose appropriate output formats:

.. code-block:: bash

    # Use JSON for scripting and automation
    tfmate --output json analyze config > config.json

    # Use table for human reading
    tfmate --output table analyze config

    # Use text for simple lists
    tfmate --output text aws services --names-only

Configuration Management
~~~~~~~~~~~~~~~~~~~~~~~~

Use configuration files when necessary:

.. code-block:: bash

    # Use custom configuration file
    tfmate --config-file ./tfmate.yaml analyze config

    # Set environment variables
    export tfmate_CONFIG_FILE=./tfmate.yaml
    tfmate analyze config

Directory Organization
~~~~~~~~~~~~~~~~~~~~~~

Organize your Terraform projects effectively:

.. code-block:: bash

    # Analyze specific environments
    tfmate analyze config --directory ./environments/prod
    tfmate analyze config --directory ./environments/staging

    # Compare configurations
    tfmate --output json analyze config --directory ./environments/prod > prod.json
    tfmate --output json analyze config --directory ./environments/staging > staging.json