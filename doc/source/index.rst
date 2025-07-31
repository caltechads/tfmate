======
tfmate
======

.. toctree::
   :maxdepth: 2
   :caption: Getting Started
   :hidden:

   overview/installation
   overview/quickstart

.. toctree::
   :maxdepth: 2
   :caption: User Guide
   :hidden:

   overview/usage
   overview/configuration
   overview/faq

.. toctree::
   :maxdepth: 2
   :caption: Development
   :hidden:

   runbook/contributing
   runbook/guidelines
   runbook/testing

Current version is |release|.

``tfmate`` is a Python command-line tool for Terraform module maintenance and analysis.
It supports Terraform 1.5+ and provides capabilities for analyzing Terraform configurations,
accessing state files from various backends, and listing AWS services.

Features
--------

tfmate provides the following key features:

- **Terraform Configuration Analysis**: Parse and analyze Terraform configuration files
- **State File Access**: Access state files from local, S3, HTTP, and Terraform Enterprise backends
- **AWS Services Listing**: List and filter AWS services with metadata
- **Credential Management**: Automatic detection and management of AWS credentials
- **Rich CLI Interface**: Beautiful command-line interface with progress indicators
- **Multiple Output Formats**: Support for JSON, table, and text output formats
- **Comprehensive Error Handling**: User-friendly error messages with actionable suggestions

Getting Started
---------------

To get started with tfmate:

1. **Installation**: Follow the :doc:`/overview/installation` guide
2. **Quick Start**: See the :doc:`/overview/quickstart` guide for basic usage
3. **Usage Guide**: Learn about commands and options in :doc:`/overview/usage`
4. **Configuration**: Learn about configuration options in :doc:`/overview/configuration`
5. **FAQ**: Check the :doc:`/overview/faq` section for common questions and troubleshooting

For developers, see the :doc:`/runbook/contributing` and :doc:`/runbook/testing` guides.

Supported Backends
------------------

tfmate supports the following Terraform backends:

- **Local**: Standard local state files
- **S3**: AWS S3 backend with profile and assume role support
- **HTTP**: HTTP backend with basic authentication
- **Remote**: Terraform Enterprise backend

Requirements
------------

- Python 3.10 or later
- Terraform 1.5 or later (for state file access)
- AWS credentials (for S3 backend access)
- Terraform Enterprise token (for TFE backend access)

