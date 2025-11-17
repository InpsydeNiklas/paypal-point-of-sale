# Dependency Injection

## Table of Contents

- [Standard DI Pattern for Module Classes](#standard-di-pattern-for-module-classes)
- [Service Definitions](#service-definitions)
- [Using the Container to Get Instances](#using-the-container-to-get-instances)
- [Why Use Dependency Injection?](#why-use-dependency-injection)

## Standard DI Pattern for Module Classes

Dependencies are injected via constructor in classes within the `src/` directory of each module.

**Example:**

```php
namespace Syde\PayPal\PointOfSale\Sync;

use Syde\PayPal\PointOfSale\Logging\Logger;
use Syde\PayPal\PointOfSale\Api\ProductRepository;

class ProductSyncHandler {
    private Logger $logger;
    private ProductRepository $repository;

    public function __construct( Logger $logger, ProductRepository $repository ) {
        $this->logger = $logger;
        $this->repository = $repository;
    }

    public function sync( int $product_id ): bool {
        $this->logger->log( "Syncing product {$product_id}" );
        // ...
    }
}
```

## Service Definitions

Services are defined in each module's `services.php` file, which returns an array of service definitions.

**Example in `modules.local/paypal-pos-sync/services.php`:**

```php
<?php

declare(strict_types=1);

namespace Syde\PayPal\PointOfSale\Sync;

use Psr\Container\ContainerInterface as C;

return [
    'sync.product_handler' => static function ( C $container ): ProductSyncHandler {
        return new ProductSyncHandler(
            $container->get( 'logging.logger' ),
            $container->get( 'api.product_repository' )
        );
    },

    'sync.inventory_handler' => static function ( C $container ): InventorySyncHandler {
        return new InventorySyncHandler(
            $container->get( 'logging.logger' ),
            $container->get( 'api.inventory_repository' )
        );
    },
];
```

## Using the Container to Get Instances

Services are retrieved from the container using the service ID defined in `services.php`.

```php
// Get instance from container
$handler = $container->get( 'sync.product_handler' );
```

### Singleton Behavior

**Important:** The container always retrieves the same instance of a given service (singleton pattern).

When different instances are needed (and only in this case), use `new` or the appropriate factory methods for the class when available.

**Example:**

```php
// Same instance every time - use container
$logger = $container->get( 'logging.logger' );
$same_logger = $container->get( 'logging.logger' );  // Same instance as above

// Different instances needed - use new or factory
$product1 = wc_get_product( 123 );
$product2 = wc_get_product( 456 );  // Different instance

// Using factory when available
$sync_job = $this->job_factory->create( $product_id );
```

### Module Bootstrap

Each module implements `ModuleInterface` and provides its services through the `setup()` method:

**Example in `modules.local/paypal-pos-sync/module.php`:**

```php
<?php

declare(strict_types=1);

namespace Syde\PayPal\PointOfSale\Sync;

use Dhii\Modular\Module\ModuleInterface;
use Interop\Container\ServiceProviderInterface;
use Psr\Container\ContainerInterface as C;

return static function (): ModuleInterface {
    return new class implements ModuleInterface {
        public function setup(): ServiceProviderInterface {
            return new class implements ServiceProviderInterface {
                public function getFactories(): array {
                    return require __DIR__ . '/services.php';
                }

                public function getExtensions(): array {
                    return [];
                }
            };
        }

        public function run( C $container ): void {
            // Initialize hooks and bootstrap module
        }
    };
};
```

## Why Use Dependency Injection?

- Easy mocking in tests
- Swap dependencies without code changes
- Explicit dependencies in constructor signature
- Clear service boundaries between modules
- Modular architecture with independent feature domains