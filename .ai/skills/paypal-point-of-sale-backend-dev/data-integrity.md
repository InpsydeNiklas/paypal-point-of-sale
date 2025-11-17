# Data Integrity Guidelines

## Table of Contents

- [Preventing Accidental Data Loss](#preventing-accidental-data-loss)
    - [Validation Before Deletion](#validation-before-deletion)
    - [Consider Race Conditions](#consider-race-conditions)
    - [Session-Based Operations](#session-based-operations)
- [When in Doubt, Ask](#when-in-doubt-ask)
- [Security Checklist for Data Operations](#security-checklist-for-data-operations)
- [Common Pitfalls to Avoid](#common-pitfalls-to-avoid)

## Preventing Accidental Data Loss

Always verify entity state before deletion/modification to prevent accidental data loss.

### Validation Before Deletion

Always verify the state of the entity before deleting or modifying it.

#### Example: Deleting a product sync record

```php
// GOOD: Verify sync status before deletion
public function delete_sync_record( int $product_id ) {
    $sync_record = $this->get_sync_record( $product_id );

    if ( ! $sync_record ) {
        return false;
    }

    // Verify it's actually pending or failed (not completed)
    if ( ! in_array( $sync_record->get_status(), array( 'pending', 'failed' ), true ) ) {
        throw new \Exception( 'Cannot delete completed sync record' );
    }

    return $this->data_store->delete( $sync_record->get_id() );
}

// BAD: No verification
public function delete_sync_record( int $product_id ) {
    $sync_record = $this->get_sync_record( $product_id );
    return $this->data_store->delete( $sync_record->get_id() );  // Could delete any record!
}
```

### Consider Race Conditions

Think about whether race conditions could occur that might affect the wrong data.

#### Example: User-specific data deletion

```php
// GOOD: Verify ownership before deletion
public function delete_user_auth_token( int $token_id, int $user_id ) {
    $token = $this->get_auth_token( $token_id );

    if ( ! $token ) {
        return false;
    }

    // Prevent race condition: verify token belongs to this user
    if ( (int) $token->get_user_id() !== $user_id ) {
        throw new \Exception( 'Cannot delete token belonging to another user' );
    }

    return $this->data_store->delete( $token_id );
}

// BAD: Race condition possible
public function delete_user_auth_token( int $token_id, int $user_id ) {
    // Another user could have taken this token in the meantime
    return $this->data_store->delete( $token_id );
}
```

### Session-Based Operations

For session-based operations, verify session ownership.

#### Example: Clearing sync queue data

```php
// GOOD: Verify session ownership
public function clear_sync_queue( int $session_id ) {
    $current_session_id = WC()->session->get_customer_id();

    if ( $session_id !== $current_session_id ) {
        throw new \Exception( 'Cannot clear sync queue for another session' );
    }

    $this->queue->clear();
}
```

## When in Doubt, Ask

If unsure about data operations, ask for clarification about:

- Required state/ownership verifications
- Soft delete vs hard delete requirements
- Protected states that prevent deletion
- PayPal POS API sync implications

## Security Checklist for Data Operations

Before implementing code that modifies or deletes data:

- [ ] Verify entity exists
- [ ] Verify entity state (status, type, etc.)
- [ ] Verify ownership (user_id, session_id, etc.)
- [ ] Check for race conditions
- [ ] Consider using soft delete (trash) instead of hard delete
- [ ] Add appropriate error handling
- [ ] Log sensitive operations for audit trail
- [ ] Add capability checks (`current_user_can()`)
- [ ] Consider PayPal POS sync state before deletion

## Common Pitfalls to Avoid

### 1. Trusting User Input

```php
// BAD
$product_id = $_POST['product_id'];
$product->delete_sync_record();

// GOOD
$product_id = absint( $_POST['product_id'] );
if ( ! current_user_can( 'manage_woocommerce', $product_id ) ) {
    wp_die( 'Unauthorized' );
}
// ... additional validation ...
```

### 2. Batch Operations Without Verification

```php
// BAD: Deletes all without verification
foreach ( $product_ids as $product_id ) {
    $this->delete_sync_record( $product_id );
}

// GOOD: Verify each item
foreach ( $product_ids as $product_id ) {
    $sync_record = $this->get_sync_record( $product_id );
    if ( $sync_record && $sync_record->get_status() === 'failed' ) {
        $this->delete_sync_record( $product_id );
    }
}
```

### 3. Ignoring Return Values

```php
// BAD: Doesn't check if operation succeeded
$this->sync_product_to_pos( $product_id );
wp_send_json_success();

// GOOD: Check result
if ( $this->sync_product_to_pos( $product_id ) ) {
    wp_send_json_success();
} else {
    wp_send_json_error( 'Failed to sync product to PayPal POS' );
}
```