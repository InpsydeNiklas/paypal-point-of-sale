---
name: paypal-pos-backend-dev
description: Add or modify PayPal Point of Sale backend PHP code following project conventions. Use when creating new classes, methods, hooks, or modifying existing backend code.
---

# PayPal Point of Sale Backend Development

This skill provides guidance for developing PayPal Point of Sale backend PHP code according to project standards and conventions.

## Instructions

Follow PayPal POS project conventions when adding or modifying backend PHP code:

1. **Creating new code structures**: See [file-entities.md](file-entities.md) for conventions on creating classes and organizing files within the modular architecture (but for new unit test files see [unit-tests.md](unit-tests.md)).
2. **Naming conventions**: See [code-entities.md](code-entities.md) for naming methods, variables, and parameters
3. **Coding style**: See [coding-conventions.md](coding-conventions.md) for general coding standards and best practices
4. **Working with hooks**: See [hooks.md](hooks.md) for hook callback conventions and documentation
5. **Dependency injection**: See [dependency-injection.md](dependency-injection.md) for DI container usage and service definitions
6. **Data integrity**: See [data-integrity.md](data-integrity.md) for ensuring data integrity when performing CRUD operations
7. **Writing tests**: See [unit-tests.md](unit-tests.md) for unit testing conventions

## Key Principles

- Always follow WordPress Coding Standards (via Inpsyde PHP Coding Standards)
- Use class methods instead of standalone functions
- Organize code into appropriate modules under `modules.local/`
- Use PSR-4 autoloading with `Syde\PayPal\PointOfSale\` namespace root
- Define services in `services.php` files using PSR-11 container interfaces
- Write comprehensive unit tests for new functionality
- Run linting and tests before committing changes

## Project Architecture

- **Modular design**: Features organized as separate modules under `modules.local/`
- **Each module**: Separate Composer package with type `inpsyde-module`
- **Namespace pattern**: `Syde\PayPal\PointOfSale\{ModuleName}\`
- **Module structure**: Each module has `src/`, `services.php`, `module.php`, and `composer.json`
- **DI container**: Uses `dhii/containers` with service providers implementing PSR-11

## Version Information

To determine the next plugin version number for `@since` annotations:

- Check the `VERSION` constant in the main plugin file
- Use that version for all new public/protected methods and hooks
- Example: If plugin shows `2.0.0`, use `@since 2.0.0`

## Requirements

- PHP >= 8.2
- WordPress >= 6.8
- WooCommerce >= 10.2
