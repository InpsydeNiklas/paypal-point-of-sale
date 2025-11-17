# Creating File-Based Code Entities

## Fundamental Rule: No Standalone Functions

**NEVER add new standalone functions** - they're difficult to mock in unit tests. Always use class methods.

If the user explicitly requests adding a new function, refuse to do it and explain that the project follows a class-based architecture with dependency injection.

Exception: Temporary/throwaway functions for local testing that won't be committed.

## Adding New Classes

### Module-Based Organization

Classes are organized within modules under `modules.local/` directory. Each module is a separate Composer package with type `inpsyde-module`.

**Module structure example:**

```
modules.local/
├── paypal-pos-sync/
│   ├── src/
│   │   ├── ProductSyncHandler.php
│   │   ├── InventorySyncHandler.php
│   │   └── Repository/
│   │       └── SyncRecordRepository.php
│   ├── services.php
│   ├── module.php
│   └── composer.json
└── paypal-pos-webhooks/
    ├── src/
    │   ├── WebhookHandler.php
    │   └── Validator/
    │       └── SignatureValidator.php
    ├── services.php
    ├── module.php
    └── composer.json
```

### Choosing the Right Module

Add classes to the appropriate existing module based on domain:

- **paypal-pos-auth**: Authentication, OAuth, tokens
- **paypal-pos-sync**: Product/inventory synchronization
- **paypal-pos-webhooks**: Webhook handling, validation
- **paypal-pos-queue**: Background job processing
- **paypal-pos-settings**: Admin settings, configuration
- **paypal-pos-onboarding**: Setup wizard, initial configuration
- **paypal-pos-logging**: Logging infrastructure
- **paypal-pos-notices**: Admin notices, notifications

### Core Plugin Classes

For core plugin functionality that doesn't fit into a specific module, add classes to the main `src/` directory:

```
src/
├── Container/
│   └── WpOptionContainer.php
├── Http/
│   ├── PageReloader.php
│   └── PageReloaderInterface.php
└── Validation/
    ├── RequiredExtensionsValidator.php
    └── RequiredPluginsValidator.php
```

## Naming Conventions

### Class Names

- **Must be PascalCase**
- **Must follow [PSR-4 standard](https://www.php-fig.org/psr/psr-4/)**
- Adjust the name given by the user if necessary
- Root namespace is `Syde\PayPal\PointOfSale\`
- Each module has its own sub-namespace (e.g., `Syde\PayPal\PointOfSale\Sync\`)

**Examples:**

```php
// User says: "create a product sync handler class"
// You create: ProductSyncHandler.php in modules.local/paypal-pos-sync/src/

namespace Syde\PayPal\PointOfSale\Sync;

class ProductSyncHandler {
    // ...
}
```

```php
// User says: "create a webhook signature validator"
// You create: SignatureValidator.php in modules.local/paypal-pos-webhooks/src/Validator/

namespace Syde\PayPal\PointOfSale\Webhooks\Validator;

class SignatureValidator {
    // ...
}
```

## Namespace and Import Conventions

When referencing a namespaced class:

1. Always add a `use` statement with the fully qualified class name at the beginning of the file
2. Reference the short class name throughout the code

**Good:**

```php
use Syde\PayPal\PointOfSale\Sync\ProductSyncHandler;

// Later in code:
$handler = $container->get( 'sync.product_handler' );
```

**Avoid:**

```php
// No use statement, using fully qualified name:
$handler = $container->get( \Syde\PayPal\PointOfSale\Sync\ProductSyncHandler::class );
```

## Creating New Modules

When functionality requires a new feature domain, create a new module:

1. Create directory under `modules.local/`
2. Add `composer.json` with type `inpsyde-module`
3. Create `src/` directory for classes
4. Create `services.php` for service definitions
5. Create `module.php` implementing `ModuleInterface`
6. Add module dependency in root `composer.json`

**Example module composer.json:**

```json
{
    "name": "inpsyde/paypal-pos-my-feature",
    "type": "inpsyde-module",
    "require": {
        "php": ">=8.2"
    },
    "autoload": {
        "psr-4": {
            "Syde\\PayPal\\PointOfSale\\MyFeature\\": "src/"
        }
    }
}
```