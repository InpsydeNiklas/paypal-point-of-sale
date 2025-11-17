---
name: paypal-pos-dev-cycle
description: Run tests, linting, and quality checks for PayPal Point of Sale development. Use when running tests, fixing code style, or following the development workflow.
---

# PayPal Point of Sale Development Cycle

This skill provides guidance for the PayPal Point of Sale development workflow, including running tests, code quality checks, and troubleshooting.

## Instructions

Follow these guidelines for PayPal Point of Sale development workflow:

1. **Running tests**: See [running-tests.md](running-tests.md) for PHP test commands, test environment setup, and troubleshooting
2. **Code quality**: See [code-quality.md](code-quality.md) for linting and code style fixes
3. **PHP linting patterns**: See [php-linting-patterns.md](php-linting-patterns.md) for common PHP linting issues and fixes
4. **Markdown linting**: See [markdown-linting.md](markdown-linting.md) for markdown file linting and formatting

## Development Workflow

The standard development workflow:

1. Make code changes in appropriate module
2. Run relevant tests: `vendor/bin/phpunit --filter YourTestClass`
3. Run linting: `vendor/bin/phpcs --standard=Inpsyde path/to/changed/file.php`
4. Fix any issues: `vendor/bin/phpcbf --standard=Inpsyde path/to/changed/file.php`
5. Commit changes only after tests pass and linting is clean

## Key Principles

- Always run tests after making changes to verify functionality
- Use specific test filters to run relevant tests during development
- Fix linting errors solely for code in your current branch
- Test failures provide detailed output showing expected vs actual values
- Use Brain Monkey for WordPress function mocking in tests
- Use Mockery for general object mocking
- Each module should have its own test directory structure

## Module-Based Development

When working on a specific feature:

1. **Identify the module**: Determine which module your changes belong to (sync, webhooks, auth, etc.)
2. **Make changes**: Edit files in `modules.local/{module-name}/src/`
3. **Update services**: Add service definitions in `modules.local/{module-name}/services.php` if needed
4. **Write tests**: Create tests in `tests/Unit/{ModuleName}/`
5. **Run module tests**: `vendor/bin/phpunit tests/Unit/{ModuleName}/`
6. **Lint module code**: `vendor/bin/phpcs --standard=Inpsyde modules.local/{module-name}/`

## Testing with DDEV

If using DDEV for local development:

```bash
# Run tests inside DDEV
ddev exec vendor/bin/phpunit

# Run linting inside DDEV
ddev exec vendor/bin/phpcs --standard=Inpsyde path/to/file.php

# Fix linting issues inside DDEV
ddev exec vendor/bin/phpcbf --standard=Inpsyde path/to/file.php
```

## Requirements

- PHP >= 8.2
- WordPress >= 6.8
- WooCommerce >= 10.2
- Composer dependencies installed
- DDEV (optional, for local development)
