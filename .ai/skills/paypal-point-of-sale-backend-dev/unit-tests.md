# Unit Testing Conventions

## Table of Contents

- [Test File Naming and Location](#test-file-naming-and-location)
- [System Under Test Variable](#system-under-test-variable)
- [Test Method Documentation](#test-method-documentation)
- [Comments in Tests](#comments-in-tests)
- [Test Configuration](#test-configuration)
- [Test Base Classes](#test-base-classes)
- [Mocking with Brain Monkey](#mocking-with-brain-monkey)
- [Example: Testing a Module Service](#example-testing-a-module-service)
- [General Testing Best Practices](#general-testing-best-practices)

## Test File Naming and Location

| Source | Test | Pattern |
|--------|------|---------|
| Module classes in `src/` | `tests/Unit/{ModuleName}/{path}/{ClassName}Test.php` | Append `Test` (no hyphen) |
| Core plugin classes | `tests/Unit/{path}/{ClassName}Test.php` | Append `Test` (no hyphen) |

Test class: Same name as source class + `Test` suffix, extends appropriate base test case.

**Examples:**

```
Source: modules.local/paypal-pos-sync/src/ProductSyncHandler.php
Test:   tests/Unit/Sync/ProductSyncHandlerTest.php

Source: modules.local/paypal-pos-webhooks/src/Validator/SignatureValidator.php
Test:   tests/Unit/Webhooks/Validator/SignatureValidatorTest.php

Source: src/Container/WpOptionContainer.php
Test:   tests/Unit/Container/WpOptionContainerTest.php
```

## System Under Test Variable

Use `$sut` with docblock "The System Under Test."

```php
/**
 * The System Under Test.
 *
 * @var ProductSyncHandler
 */
private $sut;
```

## Test Method Documentation

When adding or modifying a unit test method, the part of the docblock that describes the test must be prepended with `@testdox`. End the comment with `.` for compliance with linting rules.

**Example:**

```php
/**
 * @testdox Should return true when product is valid.
 */
public function test_returns_true_for_valid_product() {
    // ...
}

/**
 * @testdox Should throw exception when product ID is negative.
 */
public function test_throws_exception_for_negative_product_id() {
    // ...
}
```

## Comments in Tests

**Avoid over-commenting tests.** Test names and assertion messages should explain intent.

**Good - Self-explanatory:**

```php
/**
 * @testdox Should sync product when status is pending.
 */
public function test_syncs_product_when_status_pending() {
    $product = $this->create_pending_product();

    $result = $this->sut->sync( $product );

    $this->assertTrue( $result, 'Pending products should be synced' );
}
```

**Avoid - Over-commented:**

```php
/**
 * @testdox Should sync product when status is pending.
 */
public function test_syncs_product_when_status_pending() {
    // Create a pending product
    $product = $this->create_pending_product();

    // Call the method we're testing
    $result = $this->sut->sync( $product );

    // Verify the result is true
    $this->assertTrue( $result, 'Pending products should be synced' );
}
```

**Avoid - Arrange/Act/Assert comments:**

```php
// Don't add these structural comments
// Arrange
$product = $this->create_pending_product();

// Act
$result = $this->sut->sync( $product );

// Assert
$this->assertTrue( $result );
```

Use blank lines for visual separation instead. The test structure should be self-evident.

**When comments ARE useful in tests:**

- Explaining complex test setup: `// Simulate race condition by...`
- Documenting known issues: `// Workaround for PayPal API bug #12345`
- Clarifying business rules: `// PayPal POS requires 24h hold for inventory sync`

## Test Configuration

Test configuration file: `phpunit.xml`

PHPUnit version: 8.0 or 9.0

## Test Base Classes

The project provides helper test base classes:

### MonkeryTestCase

For basic unit tests using Mockery:

```php
use Syde\PayPal\PointOfSale\Tests\PHPUnit\Helper\MonkeryTestCase;

class ProductSyncHandlerTest extends MonkeryTestCase {
    // ...
}
```

### BrainMonkeyWpTestCase

For tests requiring WordPress function mocking via Brain Monkey:

```php
use Syde\PayPal\PointOfSale\Tests\PHPUnit\Helper\BrainMonkeyWpTestCase;

class WpOptionContainerTest extends BrainMonkeyWpTestCase {
    protected function setUp(): void {
        parent::setUp();
        \Brain\Monkey\setUp();
    }

    protected function tearDown(): void {
        \Brain\Monkey\tearDown();
        parent::tearDown();
    }
}
```

### ModuleContainerAwareTestCase

For tests requiring the full DI container with all modules:

```php
use Syde\PayPal\PointOfSale\Tests\PHPUnit\Helper\ModuleContainerAwareTestCase;

class IntegrationTest extends ModuleContainerAwareTestCase {
    public function test_service_is_registered() {
        $service = self::$container->get( 'sync.product_handler' );
        $this->assertInstanceOf( ProductSyncHandler::class, $service );
    }
}
```

## Mocking with Brain Monkey

Brain Monkey allows mocking WordPress functions in unit tests:

```php
use Brain\Monkey\Functions;

public function test_gets_product_meta() {
    Functions\expect( 'get_post_meta' )
        ->once()
        ->with( 123, '_sku', true )
        ->andReturn( 'PROD-123' );

    $result = $this->sut->get_product_sku( 123 );

    $this->assertSame( 'PROD-123', $result );
}
```

**Mocking actions and filters:**

```php
use Brain\Monkey\Actions;
use Brain\Monkey\Filters;

public function test_fires_sync_completed_action() {
    Actions\expectDone( 'paypal_pos_sync_completed' )
        ->once()
        ->with( 123 );

    $this->sut->complete_sync( 123 );
}

public function test_applies_product_data_filter() {
    Filters\expectApplied( 'paypal_pos_sync_product_data' )
        ->once()
        ->with( \Mockery::type( 'array' ), \Mockery::type( 'WC_Product' ) )
        ->andReturn( array( 'sku' => 'FILTERED-SKU' ) );

    $result = $this->sut->prepare_product_data( $product );

    $this->assertSame( 'FILTERED-SKU', $result['sku'] );
}
```

## Example: Testing a Module Service

```php
<?php

declare(strict_types=1);

namespace Syde\PayPal\PointOfSale\Tests\Unit\Sync;

use Mockery;
use Syde\PayPal\PointOfSale\Logging\Logger;
use Syde\PayPal\PointOfSale\Sync\ProductSyncHandler;
use Syde\PayPal\PointOfSale\Sync\Repository\SyncRecordRepository;
use Syde\PayPal\PointOfSale\Tests\PHPUnit\Helper\MonkeryTestCase;

class ProductSyncHandlerTest extends MonkeryTestCase {
    /**
     * The System Under Test.
     *
     * @var ProductSyncHandler
     */
    private $sut;

    /**
     * @var Logger|Mockery\MockInterface
     */
    private $logger;

    /**
     * @var SyncRecordRepository|Mockery\MockInterface
     */
    private $repository;

    protected function setUp(): void {
        parent::setUp();
        
        $this->logger = Mockery::mock( Logger::class );
        $this->repository = Mockery::mock( SyncRecordRepository::class );
        
        $this->sut = new ProductSyncHandler(
            $this->logger,
            $this->repository
        );
    }

    /**
     * @testdox Should return true when product syncs successfully.
     */
    public function test_returns_true_for_successful_sync() {
        $product_id = 123;
        
        $this->repository
            ->shouldReceive( 'sync_product' )
            ->once()
            ->with( $product_id )
            ->andReturn( true );

        $this->logger
            ->shouldReceive( 'log' )
            ->once()
            ->with( Mockery::containsSubString( 'Syncing product 123' ) );

        $result = $this->sut->sync( $product_id );

        $this->assertTrue( $result );
    }

    /**
     * @testdox Should throw exception when product ID is invalid.
     */
    public function test_throws_exception_for_invalid_product_id() {
        $this->expectException( \InvalidArgumentException::class );
        $this->expectExceptionMessage( 'Invalid product ID' );

        $this->sut->sync( -1 );
    }

    /**
     * @testdox Should log error when sync fails.
     */
    public function test_logs_error_when_sync_fails() {
        $product_id = 123;
        
        $this->repository
            ->shouldReceive( 'sync_product' )
            ->once()
            ->with( $product_id )
            ->andReturn( false );

        $this->logger
            ->shouldReceive( 'error' )
            ->once()
            ->with( Mockery::containsSubString( 'Failed to sync product 123' ) );

        $result = $this->sut->sync( $product_id );

        $this->assertFalse( $result );
    }
}
```

## General Testing Best Practices

1. **Always run tests after making changes** to verify functionality
2. **Use specific test filters** during development (see running-tests.md in the paypal-pos-dev-cycle skill)
3. **Write descriptive test names** that explain what is being tested
4. **Use data providers** for testing multiple scenarios with the same logic
5. **Include helpful assertion messages** for debugging when tests fail
6. **Test both success and failure cases**
7. **Mock external dependencies** (WordPress functions, database, API calls, etc.)
8. **Use Brain Monkey for WordPress functions** - never call WordPress functions directly in unit tests
9. **Keep tests focused** - one concept per test method
10. **Use type hints** for mock objects in docblocks for better IDE support

## Mocking Best Practices

### Use Mockery for Dependencies

```php
// Good - using Mockery
$logger = Mockery::mock( Logger::class );
$logger->shouldReceive( 'log' )->once();

// Avoid - using PHPUnit mocks (Mockery is project standard)
$logger = $this->createMock( Logger::class );
```

### Mock at the Right Level

```php
// Good - mock the direct dependency
$this->repository
    ->shouldReceive( 'get_product' )
    ->andReturn( $product );

// Avoid - mocking too deep
Functions\expect( 'wpdb::get_row' )
    ->andReturn( $row );
```

### Verify Mock Expectations

```php
public function test_calls_repository() {
    $this->repository
        ->shouldReceive( 'save' )
        ->once()  // Verifies the method is called exactly once
        ->with( Mockery::type( 'array' ) )
        ->andReturn( true );

    $this->sut->save_data( array( 'key' => 'value' ) );

    // Mockery automatically verifies expectations in tearDown
}
```