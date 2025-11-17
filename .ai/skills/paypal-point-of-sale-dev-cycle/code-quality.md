# Code Quality Commands

## Table of Contents

- [Overview](#overview)
- [PHP Linting](#php-linting)
- [JavaScript Linting](#javascript-linting)
- [Markdown Linting](#markdown-linting)
- [Important Linting Guidelines](#important-linting-guidelines)
- [Example Workflow](#example-workflow)
- [Understanding Linting Output](#understanding-linting-output)
- [Pre-Commit Checklist](#pre-commit-checklist)
- [Integration with Development Cycle](#integration-with-development-cycle)
- [Additional Linting Tools](#additional-linting-tools)
- [Troubleshooting](#troubleshooting)
- [Notes](#notes)

## Overview

When making changes to the PayPal Point of Sale codebase, run these commands to ensure code quality and adherence to coding standards.

For detailed PHP linting patterns and common issues, see [php-linting-patterns.md](php-linting-patterns.md).

For markdown linting rules and workflow, see [markdown-linting.md](markdown-linting.md).

## PHP Linting

### Check for PHP Linting Issues

```bash
# Check changed files for linting issues
vendor/bin/phpcs --standard=Inpsyde path/to/changed/file.php

# Check entire module
vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/
```

Checks files for Inpsyde Coding Standards violations (which include WordPress Coding Standards).

### Fix PHP Code Style Issues

```bash
# Automatically fix PHP code style issues
vendor/bin/phpcbf --standard=Inpsyde path/to/file.php

# Fix entire module
vendor/bin/phpcbf --standard=Inpsyde modules.local/paypal-pos-sync/
```

This command:

- Automatically fixes code style violations where possible
- Applies Inpsyde/WordPress Coding Standards formatting
- Modifies files in place
- Should be run before committing

### Advanced PHP Linting

If you need more control, you can use phpcs and phpcbf with additional options:

```bash
# Check specific file or directory
vendor/bin/phpcs --standard=Inpsyde path/to/file.php

# Show all violations (including warnings)
vendor/bin/phpcs -s --standard=Inpsyde path/to/file.php

# Fix specific file
vendor/bin/phpcbf --standard=Inpsyde path/to/file.php

# Check with specific severity
vendor/bin/phpcs --standard=Inpsyde --severity=5 path/to/file.php
```

## JavaScript Linting

### Check for JS Linting Issues

If the project includes JavaScript/TypeScript:

```bash
# Run JS linting (if configured)
npm run lint:js

# Or using yarn
yarn lint:js
```

This command:

- Checks JavaScript/TypeScript files for style issues
- Identifies code quality and potential issues
- Does not modify files

## Markdown Linting

Always lint markdown files after making changes. See [markdown-linting.md](markdown-linting.md) for complete details.

**Quick commands:**

```bash
# Auto-fix most issues
markdownlint --fix path/to/file.md

# Check for remaining errors
markdownlint path/to/file.md
```

## Important Linting Guidelines

### Only Fix Code in Your Branch

**Important:** Only fix linting errors for code that has been added or modified in the branch you are working on.

Do not fix linting errors in unrelated code unless specifically asked to do so.

**Why?**

- Keeps pull requests focused on the actual changes
- Avoids merge conflicts with other branches
- Makes code review easier
- Maintains clear git history

### Example Workflow

```bash
# 1. Make your code changes
# ... edit files ...

# 2. Check what you've changed
git status
git diff

# 3. Run linting on your changes
vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# 4. Fix issues automatically
vendor/bin/phpcbf --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# 5. Review the fixes
git diff

# 6. If you have markdown changes
markdownlint --fix README.md

# 7. Commit your changes
git add .
git commit -m "Your commit message"
```

## Understanding Linting Output

### PHP CodeSniffer Output

```text
FILE: /path/to/file.php
----------------------------------------------------------------------
FOUND 2 ERRORS AFFECTING 2 LINES
----------------------------------------------------------------------
 12 | ERROR | [x] Expected 1 space after opening parenthesis;
    |       |     0 found
 25 | ERROR | [ ] Variable "$productID" is not in valid snake_case
    |       |     format
----------------------------------------------------------------------
```

**Legend:**

- `[x]` - Can be fixed automatically with phpcbf
- `[ ]` - Requires manual fixing

### Common PHP Issues

1. **Spacing issues** - Usually auto-fixable

   ```php
   // Wrong
   if($condition){

   // Right
   if ( $condition ) {
   ```

2. **Naming conventions** - Requires manual fix

   ```php
   // Wrong
   $productID

   // Right
   $product_id
   ```

3. **Yoda conditions** - Requires manual fix

   ```php
   // Wrong
   if ( $value === 'active' )

   // Right
   if ( 'active' === $value )
   ```

## Pre-Commit Checklist

Before committing your changes:

- [ ] Run `vendor/bin/phpcs --standard=Inpsyde` on changed files
- [ ] Run `vendor/bin/phpcbf --standard=Inpsyde` if issues found
- [ ] Run `markdownlint --fix` on any markdown files changed
- [ ] Review all automatic fixes with `git diff`
- [ ] Address any remaining issues that can't be auto-fixed
- [ ] Run tests to ensure fixes didn't break functionality

## Integration with Development Cycle

Code quality checks fit into the overall development workflow:

1. Make code changes
2. Run relevant tests (see running-tests.md)
3. **Run linting checks** ← You are here
4. **Fix code quality issues** ← You are here
5. Commit changes only after tests pass and linting is clean

## Additional Linting Tools

### Running Composer Scripts

Check `composer.json` for available scripts:

```bash
# See all available scripts
composer run-script --list

# Common scripts may include:
composer lint              # Run all linters
composer lint:fix          # Fix auto-fixable issues
composer test              # Run tests
```

### Static Analysis

The project may use Psalm or PHPStan for static analysis:

```bash
# Run Psalm (if configured)
vendor/bin/psalm

# Run PHPStan (if configured)
vendor/bin/phpstan analyse
```

## Troubleshooting

### Linting Command Not Found

**Problem:** Command fails with "command not found"

**Solution:** Install dependencies:

```bash
composer install
```

### Too Many Issues Reported

**Problem:** Linting reports issues in files you didn't change

**Solution:** Only lint the specific files you modified:

```bash
# Good - only checks your changes
vendor/bin/phpcs --standard=Inpsyde path/to/your/file.php

# Avoid - checks entire codebase
vendor/bin/phpcs --standard=Inpsyde .
```

### Conflicts After Auto-Fix

**Problem:** Git conflicts after running phpcbf

**Solution:**

1. Review the automatic fixes: `git diff`
2. If fixes are incorrect, revert: `git checkout -- path/to/file.php`
3. Address the issues manually instead

### Standard Not Found

**Problem:** "ERROR: Referenced sniff 'Inpsyde' does not exist"

**Solution:** Ensure Inpsyde coding standards are installed:

```bash
composer install
```

## Notes

- Code quality tools help maintain consistency across the codebase
- Automatic fixes save time but should always be reviewed
- Some issues require manual intervention and understanding of the context
- Linting is required before committing to ensure code quality standards
- The project uses Inpsyde PHP Coding Standards which include WordPress standards
