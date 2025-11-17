# PHP Linting Patterns and Common Issues

## Table of Contents

- [Critical Rule: Lint Only Specific Files](#critical-rule-lint-only-specific-files)
- [Common PHP Linting Issues & Fixes](#common-php-linting-issues--fixes)
- [Translators Comment Placement](#translators-comment-placement)
- [PSR-12 File Header Order](#psr-12-file-header-order)
- [Mock Classes with Intentional Violations](#mock-classes-with-intentional-violations)
- [Multi-line Condition Alignment](#multi-line-condition-alignment)
- [Unused Closure Parameters](#unused-closure-parameters)
- [Array and Operator Alignment](#array-and-operator-alignment)
- [Indentation Rules](#indentation-rules)
- [Workflow for Fixing PHP Linting Issues](#workflow-for-fixing-php-linting-issues)
- [Quick Command Reference](#quick-command-reference)

## Critical Rule: Lint Only Specific Files

**NEVER run linting on the entire codebase.** Always lint specific files, changed files or specific modules only.

```bash
# ✅ CORRECT: Lint specific file
vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# ✅ CORRECT: Lint specific module
vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/

# ✅ CORRECT: Fix specific file
vendor/bin/phpcbf --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# ❌ WRONG: Lints entire codebase
vendor/bin/phpcs --standard=Inpsyde .
vendor/bin/phpcbf --standard=Inpsyde .
```

## Common PHP Linting Issues & Fixes

### Quick Reference Table

| Issue | Wrong | Correct |
|-------|-------|---------|
| **Translators comment** | Before return | Before function call |
| **File docblock (PSR-12)** | After `declare()` | Before `declare()` |
| **Indentation** | Spaces | Tabs only |
| **Array alignment** | Inconsistent | Align `=>` with context |
| **Equals alignment** | Inconsistent | Match surrounding style |

## Translators Comment Placement

Translators comments must be placed **immediately before the translation function call**, not before the return statement.

### Wrong - Comment Before Return

```php
/* translators: %s: Product name. */
return sprintf(
    esc_html__( '%s is not synced.', 'paypal-pos' ),
    'Product'
);
```

### Correct - Comment Before Translation Function

```php
return sprintf(
    /* translators: %s: Product name. */
    esc_html__( '%s is not synced.', 'paypal-pos' ),
    'Product'
);
```

### Multiple Parameters

```php
return sprintf(
    /* translators: 1: Product name, 2: Error message. */
    esc_html__( '%1$s failed to sync: %2$s.', 'paypal-pos' ),
    $product_name,
    $error_message
);
```

## PSR-12 File Header Order

File docblocks must come **before** the `declare()` statement, not after.

### Wrong - Docblock After declare()

```php
<?php
declare( strict_types=1 );

/**
 * File docblock
 *
 * @package Syde\PayPal\PointOfSale
 */
```

### Correct - Docblock Before declare()

```php
<?php
/**
 * File docblock
 *
 * @package Syde\PayPal\PointOfSale
 */

declare( strict_types=1 );
```

## Mock Classes with Intentional Violations

When creating mock classes that must match external class names, use phpcs:disable comments:

```php
if ( ! class_exists( 'WC_Product' ) ) {
    /**
     * Mock class for testing.
     *
     * phpcs:disable Squiz.Classes.ClassFileName.NoMatch
     * phpcs:disable Suin.Classes.PSR4.IncorrectClassName
     * phpcs:disable Squiz.Classes.ValidClassName.NotCamelCaps
     */
    class WC_Product {
        /**
         * Mock implementation.
         */
        public function get_id() {
            return 123;
        }
    }
}
```

## Multi-line Condition Alignment

Use tabs for continuation lines in multi-line conditions:

```php
// Correct - tabs for continuation
if ( $product->is_synced() &&
    $product->has_valid_sku() ) {
    // code
}

// Also correct - align with opening parenthesis
if ( $product->is_synced() &&
     $product->has_valid_sku() ) {
    // code
}
```

## Unused Closure Parameters

When creating closures with parameters required by signature but unused, use `unset()` to avoid PHPCS errors:

### The Problem

```php
// ❌ WRONG - PHPCS error: Generic.CodeAnalysis.UnusedFunctionParameter
'callback' => function ( string $product_id ) {
    return array( 'success' => true );
},
```

### The Solution

```php
// ✅ CORRECT - unset unused parameters
'callback' => function ( string $product_id ) {
    unset( $product_id ); // Avoid parameter not used PHPCS errors.
    return array( 'success' => true );
},
```

### Multiple Unused Parameters

```php
'callback' => function ( $arg1, $arg2, $arg3 ) {
    unset( $arg1, $arg2 ); // Avoid parameter not used PHPCS errors.
    return $arg3;
},
```

### Common Scenarios

- Mock method callbacks in PHPUnit tests
- Array/filter callbacks where signature is fixed
- Interface implementations with unused parameters

## Array and Operator Alignment

### Array Arrow Alignment

Align `=>` arrows consistently within each array context:

```php
// Correct - aligned arrows
$options = array(
    'sync_enabled'   => true,
    'webhook_url'    => 'https://example.com/webhook',
    'api_key'        => 'key_123',
);

// Also correct - no alignment for short arrays
$small = array(
    'id' => 123,
    'name' => 'Test',
);
```

### Assignment Operator Alignment

Match the surrounding code style:

```php
// When surrounding code aligns, align:
$sync_enabled     = true;
$webhook_url      = 'https://example.com/webhook';
$api_key          = 'key_123';

// When surrounding code doesn't align, don't align:
$sync_enabled = true;
$webhook_url = 'https://example.com/webhook';
$api_key = 'key_123';
```

## Indentation Rules

**Always use tabs, never spaces, for indentation.**

```php
// ✅ Correct - tabs for indentation
public function sync_product( int $product_id ): bool {
→   $product = wc_get_product( $product_id );
→
→   if ( ! $product ) {
→   →   return false;
→   }
→
→   return true;
}

// ❌ Wrong - spaces for indentation
public function sync_product( int $product_id ): bool {
    $product = wc_get_product( $product_id );

    if ( ! $product ) {
        return false;
    }

    return true;
}
```

## Workflow for Fixing PHP Linting Issues

1. **Run linting on changed files:**

   ```bash
   vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php
   ```

2. **Auto-fix what you can:**

   ```bash
   vendor/bin/phpcbf --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php
   ```

3. **Review remaining errors** - Common issues that require manual fixing:
   - Translators comment placement
   - File docblock order (PSR-12)
   - Unused closure parameters (add `unset()`)
   - Namespace and use statement order

4. **Address remaining issues manually**

5. **Verify the output is clean:**

   ```bash
   vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php
   ```

6. **Commit**

## Quick Command Reference

```bash
# Check specific file
vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# Fix specific file
vendor/bin/phpcbf --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# Check entire module
vendor/bin/phpcs --standard=Inpsyde modules.local/paypal-pos-sync/

# Check with error details
vendor/bin/phpcs -s --standard=Inpsyde modules.local/paypal-pos-sync/src/ProductSyncHandler.php

# Check with specific severity
vendor/bin/phpcs --standard=Inpsyde --severity=5 modules.local/paypal-pos-sync/
```
