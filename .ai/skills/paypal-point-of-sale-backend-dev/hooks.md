# Working with Hooks

## Hook Callback Naming Convention

Name hook callback methods: `handle_{hook_name}` with `@internal` annotation.

**Examples:**

```php
/**
 * Handle the init hook.
 *
 * @internal
 */
public function handle_init() {
    // Initialize components
}

/**
 * Handle the woocommerce_product_options_inventory_product_data hook.
 *
 * @internal
 */
public function handle_woocommerce_product_options_inventory_product_data() {
    // Add custom inventory fields
}
```

## Hook Docblocks

If you modify a line that fires a hook without a docblock:

1. Add docblock with description and `@param` tags
2. Use `git log -S "hook_name"` to find when it was introduced
3. Add `@since` annotation with that version

```php
/**
 * Fires after a product has been synced to PayPal POS.
 *
 * @param int $product_id The synced product ID.
 * @param array $sync_data The synchronization data.
 *
 * @since 1.5.0
 */
do_action( 'paypal_pos_product_synced', $product_id, $sync_data );
```

## Hook Documentation Requirements

All hooks must have docblocks that include:

- Description of when the hook fires
- `@param` tags for each parameter passed to the hook
- `@since` annotation with the version number (last line, with blank line before)
    - For new hooks: Use the version from the main plugin file `VERSION` constant
    - For existing hooks: Use `git log -S "hook_name"` to find when it was introduced

**Action hook example:**

```php
/**
 * Fires after inventory has been synced.
 *
 * @param int   $product_id The product ID.
 * @param array $inventory  The inventory data synced.
 *
 * @since 2.0.0
 */
do_action( 'paypal_pos_inventory_synced', $product_id, $inventory );
```

**Filter hook example:**

```php
/**
 * Filters the product data before syncing to PayPal POS.
 *
 * @param array      $product_data The product data to sync.
 * @param WC_Product $product      The product object.
 *
 * @since 2.0.0
 */
$product_data = apply_filters( 'paypal_pos_sync_product_data', $product_data, $product );
```

## Plugin-Specific Hook Naming

PayPal POS plugin hooks should follow this naming pattern:

- Prefix: `paypal_pos_` (not `zettle_` or `izettle_`)
- Domain-specific: Include the feature area (e.g., `sync`, `webhook`, `auth`)
- Action: Describe what happened or will happen

**Examples:**

```php
// Good - clear, prefixed, domain-specific
do_action( 'paypal_pos_sync_completed', $product_id );
do_action( 'paypal_pos_webhook_received', $webhook_data );
do_action( 'paypal_pos_auth_token_refreshed', $token );

// Avoid - missing prefix or unclear purpose
do_action( 'product_synced', $product_id );
do_action( 'webhook', $data );
```