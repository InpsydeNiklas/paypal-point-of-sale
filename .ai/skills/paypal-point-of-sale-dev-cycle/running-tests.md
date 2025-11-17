# Running Tests

## Table of Contents

- [PHP Unit Tests](#php-unit-tests)
    - [Basic Test Commands](#basic-test-commands)
    - [Examples](#examples)
- [Common Test Commands](#common-test-commands)
- [Test Environment](#test-environment)
- [Troubleshooting Tests](#troubleshooting-tests)
- [Interpreting Test Output](#interpreting-test-output)
- [Best Practices](#best-practices)
- [Test Configuration](#test-configuration)
- [Notes](#notes)

## PHP Unit Tests

To run PHP unit tests in the PayPal Point of Sale plugin directory, use the following commands:

### Basic Test Commands

```bash
# Run all PHP unit tests
vendor/bin/phpunit

# Run specific test class
vendor/bin/phpunit --filter ProductSyncHandlerTest

# Run specific test method
vendor/bin/phpunit --filter ProductSyncHandlerTest::test_syncs_product_successfully

# Run tests with verbose output
vendor/bin/phpunit --verbose --filter ProductSyncHandlerTest
```

### Examples

```bash
# Run sync module tests
vendor/bin/phpunit tests/Unit/Sync/

# Run specific test method
vendor/bin/phpunit --filter ProductSyncHandlerTest::test_returns_true_for_valid_product

# Run all webhook tests
vendor/bin/phpunit tests/Unit/Webhooks/
```

## Common Test Commands

### Run Tests for a Directory

```bash
# Run all tests in a specific module
vendor/bin/phpunit tests/Unit/Sync/

# Run all tests in a subdirectory
vendor/bin/phpunit tests/Unit/Sync/Repository/
```

### Run Tests Matching a Pattern

```bash
# Run all Sync tests
vendor/bin/phpunit --filter "Sync.*Test"

# Run all tests with "Product" in the name
vendor/bin/phpunit --filter "Product"

# Run all tests with "webhook" in the name
vendor/bin/phpunit --filter "webhook"
```

### Stop on First Failure

```bash
# Useful during development to quickly identify issues
vendor/bin/phpunit --stop-on-failure
```

### Get Test Coverage

```bash
# Get coverage report (if configured with Xdebug)
vendor/bin/phpunit --coverage-text

# Generate HTML coverage report
vendor/bin/phpunit --coverage-html coverage/
```

## Test Environment

### How It Works

Tests run using PHPUnit 8.0 or 9.0 with Brain Monkey for WordPress function mocking and Mockery for general mocking.

### Environment Setup

The test environment uses the configuration in `phpunit.xml`:

```bash
# No special setup needed - just run tests
vendor/bin/phpunit

# If dependencies are missing
composer install
```

## Troubleshooting Tests

| Problem | Solution |
|---------|----------|
| "Class not found" errors | Run `composer install` |
| Brain Monkey errors | Ensure `brain/monkey` is installed: `composer require --dev brain/monkey` |
| Mockery errors | Check `mockery/mockery` is installed |
| Namespace errors | Verify PSR-4 autoloading in `composer.json` |
| Module not found | Run `composer install` to install module dependencies |

## Interpreting Test Output

### Successful Test Run

```text
PHPUnit 9.6.24

..................................................  50 / 100 ( 50%)
..................................................  100 / 100 (100%)

Time: 00:02.345, Memory: 24.00 MB

OK (100 tests, 250 assertions)
```

### Failed Test

```text
PHPUnit 9.6.24

.....F.................................................  50 / 100 ( 50%)
..................................................  100 / 100 (100%)

Time: 00:02.345, Memory: 24.00 MB

There was 1 failure:

1) ProductSyncHandlerTest::test_syncs_product_successfully
Pending products should be synced
Failed asserting that false is true.

/path/to/test/file.php:123

FAILURES!
Tests: 100, Assertions: 250, Failures: 1.
```

### Understanding Failures

Test failures provide:

- **Which test failed:** Test class and method name
- **Test data:** Data set used (if using data providers)
- **Expected vs actual:** What was expected and what was received
- **Location:** File and line number where assertion failed

## Best Practices

### During Development

1. **Run specific tests** for the code you're changing:

   ```bash
   vendor/bin/phpunit --filter ProductSyncHandlerTest
   ```

2. **Use verbose mode** when debugging:

   ```bash
   vendor/bin/phpunit --verbose --filter ProductSyncHandlerTest
   ```

3. **Stop on first failure** to focus on one issue at a time:

   ```bash
   vendor/bin/phpunit --stop-on-failure --filter ProductSyncHandlerTest
   ```

### Before Committing

1. **Run all affected tests:**

   ```bash
   vendor/bin/phpunit tests/Unit/Sync/
   ```

2. **Ensure all tests pass** before committing

3. **Check code quality** (see code-quality.md)

### Module-Specific Testing

When working on a specific module, run tests for that module:

```bash
# Test sync module
vendor/bin/phpunit tests/Unit/Sync/

# Test webhooks module
vendor/bin/phpunit tests/Unit/Webhooks/

# Test auth module
vendor/bin/phpunit tests/Unit/Auth/
```

## Test Configuration

Test configuration file: `phpunit.xml`

This file contains:

- Test suite definitions
- Bootstrap files
- Coverage settings
- Logging configuration

**Example phpunit.xml structure:**

```xml
<?xml version="1.0"?>
<phpunit bootstrap="tests/bootstrap.php">
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

## Notes

- The test environment uses PHPUnit 8.0 or 9.0
- Brain Monkey is used for WordPress function mocking
- Mockery is used for general object mocking
- Tests are organized by module in `tests/Unit/{ModuleName}/`
- Always run tests before committing changes
- Use `--filter` to run specific tests during development
- Module tests should be independent and not rely on other modules
- Use the appropriate base test class (`MonkeryTestCase`, `BrainMonkeyWpTestCase`, or `ModuleContainerAwareTestCase`)
