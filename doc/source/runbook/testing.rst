Testing Guide
=============

This guide covers how to run tests for tfmate, how to add new tests, and testing best practices.

Running Tests
-------------

Basic Test Execution
^^^^^^^^^^^^^^^^^^^^

To run all tests:

.. code-block:: bash

    # Run all tests
    pytest

    # Run with verbose output
    pytest -v

    # Run with coverage
    pytest --cov=tfmate

    # Run specific test file
    pytest tests/test_cli.py

    # Run specific test class
    pytest tests/test_models.py::TestAWSService

    # Run specific test method
    pytest tests/test_models.py::TestAWSService::test_valid_service

Test Categories
^^^^^^^^^^^^^^^

The test suite includes several categories:

- **Unit tests**: Test individual functions and classes
- **Integration tests**: Test with real Terraform projects
- **CLI tests**: Test command-line interface functionality
- **Model tests**: Test Pydantic model validation
- **Service tests**: Test business logic services

Running Integration Tests
^^^^^^^^^^^^^^^^^^^^^^^^^

Integration tests require a real Terraform project. Set the environment variable:

.. code-block:: bash

    # Set path to a real Terraform project
    export TFTEST_PROJECT_PATH=/path/to/terraform/project
    export TFTEST_WORKSPACE_PROJECT_PATH=/path/to/terraform/project-with-workspaces

    # Run integration tests
    pytest tests/test_integration.py -v

    # Run all tests including integration
    pytest -v

- If ``TFTEST_PROJECT_PATH`` is not set, the non-workspace integration tests will be skipped.
- If ``TFTEST_WORKSPACE_PROJECT_PATH`` is not set, the workspace integration tests will be skipped.

Test Configuration
^^^^^^^^^^^^^^^^^^

The test configuration is defined in ``pyproject.toml``:

.. code-block:: toml

    [tool.pytest.ini_options]
    testpaths = ["tests"]
    python_files = ["test_*.py"]
    python_classes = ["Test*"]
    python_functions = ["test_*"]
    addopts = [
        "--strict-markers",
        "--strict-config",
        "--cov=tfmate",
        "--cov-report=term-missing",
        "--cov-report=html",
    ]

Coverage Reports
^^^^^^^^^^^^^^^^

Generate coverage reports:

.. code-block:: bash

    # Generate HTML coverage report
    pytest --cov=tfmate --cov-report=html

    # View coverage report
    open htmlcov/index.html

    # Generate XML coverage report (for CI)
    pytest --cov=tfmate --cov-report=xml

Adding New Tests
----------------

Test Structure
^^^^^^^^^^^^^^

Tests are organized by module:

.. code-block:: text

    tests/
    ├── test_cli.py          # CLI command tests
    ├── test_models.py       # Pydantic model tests
    ├── test_services.py     # Business logic tests
    ├── test_state_access.py # State file access tests
    ├── test_integration.py  # Integration tests
    └── fixtures/            # Test fixtures and data
        ├── terraform/       # Terraform test files
        └── aws/            # AWS test data

Test Naming Conventions
^^^^^^^^^^^^^^^^^^^^^^^

- Test files: ``test_<module>.py``
- Test classes: ``Test<ClassName>``
- Test methods: ``test_<description>``

Example:

.. code-block:: python

    class TestAWSService:
        """Test AWS service model validation."""

        def test_valid_service(self):
            """Test creating a valid AWS service."""
            service = AWSService(
                name="ecs",
                service_id="Amazon Elastic Container Service",
                api_version="2014-11-13",
                endpoints=["ecs"]
            )
            assert service.name == "ecs"

        def test_invalid_service_name(self):
            """Test invalid service name raises error."""
            with pytest.raises(ValidationError):
                AWSService(
                    name="invalid@service",
                    service_id="Test Service",
                    api_version="2014-11-13"
                )

Unit Test Guidelines
^^^^^^^^^^^^^^^^^^^^

1. **Test one thing at a time**: Each test should verify a single behavior
2. **Use descriptive names**: Test names should clearly describe what is being tested
3. **Arrange-Act-Assert**: Structure tests with setup, execution, and verification
4. **Use fixtures**: Reuse common test data and setup
5. **Mock external dependencies**: Don't rely on external services in unit tests

Example unit test:

.. code-block:: python

    def test_read_local_state_valid(tmp_path):
        """Test reading valid local state file."""
        # Arrange
        state_file = tmp_path / "terraform.tfstate"
        state_content = {
            "version": 4,
            "terraform_version": "1.5.0",
            "serial": 1,
            "lineage": "12345678-1234-1234-1234-123456789012",
            "outputs": {"test": {"value": "test"}},
            "resources": [],
        }
        state_file.write_text(json.dumps(state_content))

        # Act
        result = read_local_state(state_file)

        # Assert
        assert result["version"] == 4
        assert result["terraform_version"] == "1.5.0"
        assert result["outputs"]["test"]["value"] == "test"

CLI Test Guidelines
^^^^^^^^^^^^^^^^^^^

Use Click's testing utilities for CLI tests:

.. code-block:: python

    from click.testing import CliRunner

    def test_aws_services_names_only():
        """Test aws services command with names-only option."""
        runner = CliRunner()
        result = runner.invoke(cli, ['aws', 'services', '--names-only'])

        assert result.exit_code == 0
        assert 'accessanalyzer' in result.output
        assert 'account' in result.output

Mocking Guidelines
^^^^^^^^^^^^^^^^^^

Use mocking for external dependencies:

.. code-block:: python

    from unittest.mock import Mock, patch

    def test_read_s3_state_valid():
        """Test reading valid S3 state file."""
        # Mock AWS session and S3 client
        mock_response = Mock()
        mock_response.__getitem__ = Mock(return_value=Mock())
        mock_response.__getitem__.return_value.read.return_value = json.dumps({
            "version": 4,
            "terraform_version": "1.5.0"
        }).encode("utf-8")

        mock_s3 = Mock()
        mock_s3.get_object.return_value = mock_response

        mock_session = Mock()
        mock_session.client.return_value = mock_s3

        with patch("tfmate.services.state_access.s3.CredentialManager") as mock_manager:
            mock_manager.return_value.create_aws_session.return_value = mock_session

            result = read_s3_state(config, credentials)
            assert result["version"] == 4

Integration Test Guidelines
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Terraform oriented integration tests should:

1. **Use real Terraform projects**: Test with actual Terraform configurations
2. **Be flexible**: Don't make assumptions about specific resource counts or values
3. **Test functionality**: Focus on whether the tool can successfully parse and access data
4. **Handle missing prerequisites**: Skip gracefully when requirements aren't met

Example integration test:

.. code-block:: python

    @pytest.mark.integration
    def test_real_terraform_project():
        """Test with a real Terraform project."""
        project_path = os.getenv('TFTEST_PROJECT_PATH')
        if not project_path:
            pytest.skip("TFTEST_PROJECT_PATH environment variable not set")

        project_dir = Path(project_path)
        if not project_dir.exists():
            pytest.skip(f"Project path does not exist: {project_path}")

        # Test that we can parse the configuration
        parser = TerraformParser()
        config = parser.parse_directory(project_dir)
        assert config is not None

        # Test backend detection
        detector = StateDetector()
        backend = detector.detect_state_location(config)
        assert backend.type in {'local', 's3', 'http', 'remote'}

Test Fixtures
-------------

Using Fixtures
^^^^^^^^^^^^^^

Create reusable test data with fixtures:

.. code-block:: python

    @pytest.fixture
    def sample_terraform_config():
        """Sample Terraform configuration for testing."""
        return {
            "terraform": [{
                "required_version": ">= 1.5.0",
                "backend": [{
                    "s3": [{
                        "bucket": "my-terraform-state",
                        "key": "prod/terraform.tfstate",
                        "region": "us-west-2"
                    }]
                }]
            }],
            "provider": [{
                "aws": [{
                    "region": "us-west-2"
                }]
            }]
        }

    def test_parse_terraform_config(sample_terraform_config):
        """Test parsing Terraform configuration."""
        # Use the fixture
        config = parse_config(sample_terraform_config)
        assert config.required_version == ">= 1.5.0"

Creating Fixtures
^^^^^^^^^^^^^^^^^

Define fixtures in test files or in ``conftest.py``:

.. code-block:: python

    # In conftest.py
    @pytest.fixture
    def mock_aws_session():
        """Mock AWS session for testing."""
        session = Mock()
        session.client.return_value = Mock()
        return session

    # In test file
    def test_aws_functionality(mock_aws_session):
        """Test AWS functionality with mocked session."""
        with patch("boto3.Session", return_value=mock_aws_session):
            # Test code here
            pass

Test Data
^^^^^^^^^

Store test data in the fixtures directory:

.. code-block:: text

    tests/fixtures/
    ├── terraform/
    │   ├── local_backend/
    │   │   ├── main.tf
    │   │   └── terraform.tfstate
    │   ├── s3_backend/
    │   │   └── main.tf
    │   └── http_backend/
    │       └── main.tf
    └── aws/
        └── mock_services.json

Testing Best Practices
----------------------

Error Testing
^^^^^^^^^^^^^

Always test error conditions:

.. code-block:: python

    def test_invalid_input_raises_error():
        """Test that invalid input raises appropriate error."""
        with pytest.raises(ValidationError) as exc_info:
            AWSService(name="", service_id="test")

        assert "String should have at least 1 character" in str(exc_info.value)

    def test_file_not_found_raises_error(tmp_path):
        """Test that missing file raises appropriate error."""
        missing_file = tmp_path / "nonexistent.tfstate"

        with pytest.raises(StateFileError) as exc_info:
            read_local_state(missing_file)

        assert "State file not found" in str(exc_info.value)

Performance Testing
^^^^^^^^^^^^^^^^^^^

Test performance for critical operations:

.. code-block:: python

    def test_aws_services_performance(benchmark):
        """Test AWS services listing performance."""
        def list_services():
            session = botocore.session.get_session()
            return session.get_available_services()

        result = benchmark(list_services)
        assert len(result) > 0

Edge Cases
^^^^^^^^^^

Test edge cases and boundary conditions:

.. code-block:: python

    def test_empty_state_file(tmp_path):
        """Test handling of empty state file."""
        state_file = tmp_path / "empty.tfstate"
        state_file.write_text("{}")

        with pytest.raises(StateFileError):
            read_local_state(state_file)

    def test_malformed_json(tmp_path):
        """Test handling of malformed JSON."""
        state_file = tmp_path / "malformed.tfstate"
        state_file.write_text("{ invalid json")

        with pytest.raises(StateFileError):
            read_local_state(state_file)

Testing CLI Commands
--------------------

CLI Testing Patterns
^^^^^^^^^^^^^^^^^^^^

Test CLI commands with various options:

.. code-block:: python

    def test_cli_help():
        """Test CLI help output."""
        runner = CliRunner()
        result = runner.invoke(cli, ['--help'])

        assert result.exit_code == 0
        assert "Terraform maintenance tool" in result.output

    def test_aws_services_with_options():
        """Test aws services command with various options."""
        runner = CliRunner()

        # Test with filter
        result = runner.invoke(cli, ['aws', 'services', '--filter', 'ec*'])
        assert result.exit_code == 0

        # Test with sort
        result = runner.invoke(cli, ['aws', 'services', '--sort-by', 'api_version'])
        assert result.exit_code == 0

        # Test with verbose
        result = runner.invoke(cli, ['--verbose', 'aws', 'services'])
        assert result.exit_code == 0

Testing Error Handling
^^^^^^^^^^^^^^^^^^^^^^

Test CLI error handling:

.. code-block:: python

    def test_nonexistent_directory():
        """Test CLI with nonexistent directory."""
        runner = CliRunner()
        result = runner.invoke(cli, ['analyze', 'config', '--directory', '/nonexistent'])

        assert result.exit_code == 1
        assert "Error" in result.output

    def test_invalid_output_format():
        """Test CLI with invalid output format."""
        runner = CliRunner()
        result = runner.invoke(cli, ['--output', 'invalid', 'aws', 'services'])

        assert result.exit_code == 2  # Click error code for invalid choice

Testing with Real Data
----------------------

Integration Testing Setup
^^^^^^^^^^^^^^^^^^^^^^^^^

For integration testing, you need a real Terraform project:

1. **Create a test project**: Set up a simple Terraform project with various backends
2. **Set environment variable**: ``export TFTEST_PROJECT_PATH=/path/to/project``
3. **Run integration tests**: ``pytest tests/test_integration.py -v``

Example test project structure:

.. code-block:: text

    test-project/
    ├── main.tf              # Terraform configuration
    ├── variables.tf         # Variable definitions
    ├── outputs.tf           # Output definitions
    ├── terraform.tfstate    # State file (if using local backend)
    └── .terraform/          # Terraform working directory

Testing Different Backends
^^^^^^^^^^^^^^^^^^^^^^^^^^

Test with different backend configurations:

.. code-block:: python

    @pytest.mark.integration
    def test_s3_backend_integration():
        """Test S3 backend integration."""
        project_path = os.getenv('TFTEST_PROJECT_PATH')
        if not project_path:
            pytest.skip("TFTEST_PROJECT_PATH not set")

        # Test S3 backend detection and access
        # This requires AWS credentials to be configured

    @pytest.mark.integration
    def test_http_backend_integration():
        """Test HTTP backend integration."""
        project_path = os.getenv('TFTEST_PROJECT_PATH')
        if not project_path:
            pytest.skip("TFTEST_PROJECT_PATH not set")

        # Test HTTP backend detection and access
        # This requires HTTP server to be running

Continuous Integration
----------------------

CI Configuration
^^^^^^^^^^^^^^^^

Configure CI to run tests automatically:

.. code-block:: yaml

    # .github/workflows/test.yml
    name: Tests

    on: [push, pull_request]

    jobs:
      test:
        runs-on: ubuntu-latest
        strategy:
          matrix:
            python-version: [3.10, 3.11, 3.12]

        steps:
        - uses: actions/checkout@v3
        - name: Set up Python ${{ matrix.python-version }}
          uses: actions/setup-python@v4
          with:
            python-version: ${{ matrix.python-version }}

        - name: Install dependencies
          run: |
            python -m pip install --upgrade pip
            pip install -e ".[test]"

        - name: Run tests
          run: |
            pytest --cov=tfmate --cov-report=xml

        - name: Upload coverage
          uses: codecov/codecov-action@v3
          with:
            file: ./coverage.xml

Test Commands for CI
^^^^^^^^^^^^^^^^^^^^

Common test commands for CI environments:

.. code-block:: bash

    # Install test dependencies
    pip install -e ".[test]"

    # Run tests with coverage
    pytest --cov=tfmate --cov-report=xml

    # Run tests in parallel
    pytest -n auto

    # Run only unit tests (skip integration)
    pytest -m "not integration"

    # Run with specific Python version
    python -m pytest

Debugging Tests
---------------

Debugging Failed Tests
^^^^^^^^^^^^^^^^^^^^^^

Use pytest debugging features:

.. code-block:: bash

    # Run with maximum verbosity
    pytest -vvv

    # Stop on first failure
    pytest -x

    # Show local variables on failure
    pytest -l

    # Run with debugger
    pytest --pdb

    # Run specific failing test
    pytest tests/test_specific.py::test_failing_method -vvv

Common Test Issues
^^^^^^^^^^^^^^^^^^

1. **Mock setup issues**: Ensure mocks are properly configured
2. **Path issues**: Use ``tmp_path`` fixture for file operations
3. **Environment issues**: Check environment variables and dependencies
4. **Timing issues**: Use appropriate timeouts for external calls

Example debugging session:

.. code-block:: python

    def test_debug_example(tmp_path):
        """Example of debugging a test."""
        # Add debug prints
        print(f"Working directory: {tmp_path}")

        # Use pdb for interactive debugging
        import pdb; pdb.set_trace()

        # Test code here
        pass

Test Maintenance
----------------

Keeping Tests Updated
^^^^^^^^^^^^^^^^^^^^^

1. **Update tests when code changes**: Ensure tests reflect current behavior
2. **Review test coverage**: Maintain good coverage of critical paths
3. **Remove obsolete tests**: Delete tests for removed functionality
4. **Update fixtures**: Keep test data current

Test Documentation
^^^^^^^^^^^^^^^^^^

Document complex tests:

.. code-block:: python

    def test_complex_integration_scenario():
        """
        Test complex integration scenario with multiple backends.

        This test verifies that tfmate can handle:
        1. S3 backend with assume role
        2. HTTP backend with authentication
        3. TFE backend with workspace lookup

        Prerequisites:
        - TFTEST_PROJECT_PATH environment variable set
        - AWS credentials configured
        - TFE token available
        """
        # Test implementation
        pass

Running Tests in Development
----------------------------

Development Workflow
^^^^^^^^^^^^^^^^^^^^

1. **Write tests first**: Follow TDD principles
2. **Run tests frequently**: Test as you develop
3. **Use watch mode**: Automatically run tests on file changes
4. **Check coverage**: Ensure new code is tested

Quick Test Commands
^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    # Run tests for current module
    pytest tests/test_current_module.py -v

    # Run tests matching pattern
    pytest -k "test_aws" -v

    # Run tests with coverage for current changes
    pytest --cov=tfmate --cov-report=term-missing

    # Run tests in watch mode (requires pytest-watch)
    ptw

    # Run specific test with debugging
    pytest tests/test_file.py::test_method -vvv --pdb