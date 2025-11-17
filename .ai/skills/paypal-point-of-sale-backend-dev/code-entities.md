# Creating Code Entities (Methods, Variables, Parameters)

## Table of Contents

- [Naming Conventions](#naming-conventions)
- [Method Visibility](#method-visibility)
- [Static Methods](#static-methods)
- [Docblock Requirements](#docblock-requirements)
    - [Public, Protected Methods, and Hooks](#public-protected-methods-and-hooks)
    - [Private Methods and Internal Callbacks](#private-methods-and-internal-callbacks)
- [Hook Docblocks](#hook-docblocks)

## Naming Conventions

Use snake_case for methods, variables, and hooks (not camelCase or PascalCase).

**Examples:**

```php
// Correct
public function calculate_order_total() { }
private $order_items;
```

## Method Visibility

New class methods should be `private` by default.

**Use `protected`** only if it's clear the method will be used in derived classes.

**Use `public`** only if the method will be called from outside the class.

**Examples:**

```php
class OrderProcessor {
    // Default: private for internal helpers
    private function validate_items( array $items ) { }

    // Protected: for use in child classes
    protected function get_tax_rate( string $country ) { }

    // Public: external API
    public function process_order( int $order_id ) { }
}
```

## Static Methods

Pure methods (output depends only on inputs, no external dependencies) must be `static`.

**Examples of pure methods (should be static):**

```php
// Mathematical calculations
public static function calculate_percentage( float $amount, float $percent ) {
    return $amount * ( $percent / 100 );
}

// String manipulations
public static function format_product_sku( string $sku ) {
    return strtoupper( trim( $sku ) );
}

// Data transformations
public static function normalize_address( array $address ) {
    return array_map( 'trim', $address );
}
```

**Examples of non-pure methods (should NOT be static):**

```php
// Depends on database
public function get_order_total( int $order_id ) { }

// Depends on system time
public function is_order_recent( int $order_id ) { }

// Uses object state
public function calculate_with_tax( float $amount ) {
    return $amount * $this->tax_rate;
}
```

**Exception:** Non-pure methods should not be `static` unless there's a specific architectural reason (e.g., singleton pattern, factory methods).

## Docblock Requirements

Add concise docblocks to all hooks and methods. One line is ideal.

### Public, Protected Methods, and Hooks

Must include a `@since` annotation with the next plugin version number.

The `@since` annotation must be:

- The last line in the docblock
- Preceded by a blank comment line
- Use the version from the main plugin file constant `VERSION`
  (e.g., if the plugin shows `2.0.0`, use `@since 2.0.0`)

**Good - Concise:**

```php
/**
 * Process the order and update status.
 *
 * @param int $order_id The order ID.
 * @return bool True if successful.
 *
 * @since 2.0.0
 */
public function process_order( int $order_id ) { }

/**
 * Fires after an order is synced to PayPal POS.
 *
 * @param int $order_id The order ID.
 *
 * @since 2.0.0
 */
do_action( 'paypal_pos_order_synced', $order_id );
```

**Avoid - Over-explained:**

```php
/**
 * This method processes the order by validating the order data,
 * checking inventory levels, syncing with PayPal POS, and then updating
 * the order status to reflect the successful processing.
 *
 * @param int $order_id The unique identifier for the order that needs to be processed.
 * @return bool Returns true if the order was processed successfully, false otherwise.
 *
 * @since 2.0.0
 */
```

For hooks, aim for a single descriptive line whenever possible.

### Private Methods and Internal Callbacks

Do NOT require a `@since` annotation if they are:

- Private methods
- Internal callbacks (marked with `@internal`)

**Example:**

```php
/**
 * Internal helper to validate order items.
 *
 * @param array $items The items to validate.
 * @return bool
 */
private function validate_items( array $items ) { }
```

## Hook Docblocks

For information about documenting hooks (including adding docblocks to existing hooks), see [hooks.md](hooks.md).