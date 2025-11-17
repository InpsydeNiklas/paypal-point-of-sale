---
name: paypal-pos-code-review
description: Review PayPal Point of Sale code changes for coding standards compliance. Use when reviewing code locally, performing automated PR reviews, or checking code quality.
---

# PayPal Point of Sale Code Review

Review code changes against PayPal Point of Sale coding standards and conventions.

## Critical Violations to Flag

### Backend PHP Code

Consult the `paypal-pos-backend-dev` skill for detailed standards. Using these standards as guidance, flag these violations and other similar ones:

**Architecture & Structure:**

- ❌ **Standalone functions** - Must use class methods ([file-entities.md](../paypal-pos-backend-dev/file-entities.md))
- ❌ **Wrong module location** - Classes must be in appropriate module under `modules.local/` ([file-entities.md](../paypal-pos-backend-dev/file-entities.md))
- ❌ **Missing service definition** - New classes need entries in `services.php` ([dependency-injection.md](../paypal-pos-backend-dev/dependency-injection.md))
- ❌ **Wrong namespace** - Must use `Syde\PayPal\PointOfSale\{ModuleName}\` pattern ([file-entities.md](../paypal-pos-backend-dev/file-entities.md))
- ❌ **Direct instantiation** - Should use container for services, not `new` ([dependency-injection.md](../paypal-pos-backend-dev/dependency-injection.md))

**Naming & Conventions:**

- ❌ **camelCase naming** - Must use snake_case for methods/variables/hooks ([code-entities.md](../paypal-pos-backend-dev/code-entities.md))
- ❌ **Yoda condition violations** - Must follow WordPress Coding Standards ([coding-conventions.md](../paypal-pos-backend-dev/coding-conventions.md))
- ❌ **Wrong hook prefix** - Must use `paypal_pos_` not `zettle_` or `izettle_` ([hooks.md](../paypal-pos-backend-dev/hooks.md))

**Documentation:**

- ❌ **Missing `@since` annotations** - Required for public/protected methods and hooks ([code-entities.md](../paypal-pos-backend-dev/code-entities.md))
- ❌ **Missing docblocks** - Required for all hooks and methods ([code-entities.md](../paypal-pos-backend-dev/code-entities.md))
- ❌ **Verbose docblocks** - Keep concise, one line is ideal ([code-entities.md](../paypal-pos-backend-dev/code-entities.md))
- ❌ **Wrong version in @since** - Must match plugin VERSION constant ([code-entities.md](../paypal-pos-backend-dev/code-entities.md))

**Data Integrity:**

- ❌ **Missing validation** - Must verify state before deletion/modification ([data-integrity.md](../paypal-pos-backend-dev/data-integrity.md))
- ❌ **No ownership checks** - Must verify user/session ownership before operations ([data-integrity.md](../paypal-pos-backend-dev/data-integrity.md))
- ❌ **Ignoring return values** - Must check if operations succeeded ([data-integrity.md](../paypal-pos-backend-dev/data-integrity.md))

**Testing:**

- ❌ **Using `$instance` in tests** - Must use `$sut` variable name ([unit-tests.md](../paypal-pos-backend-dev/unit-tests.md))
- ❌ **Missing `@testdox`** - Required in test method docblocks ([unit-tests.md](../paypal-pos-backend-dev/unit-tests.md))
- ❌ **Test file naming** - Must follow `{ClassName}Test.php` pattern ([unit-tests.md](../paypal-pos-backend-dev/unit-tests.md))
- ❌ **Wrong test location** - Must be in `tests/Unit/{ModuleName}/` matching source structure ([unit-tests.md](../paypal-pos-backend-dev/unit-tests.md))
- ❌ **Not using Brain Monkey** - WordPress functions must be mocked with Brain Monkey ([unit-tests.md](../paypal-pos-backend-dev/unit-tests.md))
- ❌ **PHPUnit mocks instead of Mockery** - Project standard is Mockery ([unit-tests.md](../paypal-pos-backend-dev/unit-tests.md))

**Dependency Injection:**

- ❌ **Wrong DI pattern** - Must use constructor injection, not `init()` method ([dependency-injection.md](../paypal-pos-backend-dev/dependency-injection.md))
- ❌ **Missing from services.php** - New services must be registered ([dependency-injection.md](../paypal-pos-backend-dev/dependency-injection.md))
- ❌ **Service ID naming** - Should follow `{module}.{service_name}` pattern ([dependency-injection.md](../paypal-pos-backend-dev/dependency-injection.md))

### UI Text & Copy

Consult the `paypal-pos-copy-guidelines` skill. Flag:

- ❌ **Title Case in UI** - Must use sentence case ([sentence-case.md](../paypal-pos-copy-guidelines/sentence-case.md))
    - Wrong: "Save Changes", "Product Settings", "Sync Options"
    - Correct: "Save changes", "Product settings", "Sync options"
    - Exceptions: Proper nouns (PayPal), acronyms (API), brand names

## Review Approach

1. **Scan for critical violations** listed above
2. **Cite specific skill files** when flagging issues
3. **Provide correct examples** from the skill documentation
4. **Group related issues** for clarity
5. **Be constructive** - explain why the standard exists when relevant

## Output Format

For each violation found:

```text
❌ [Issue Type]: [Specific problem]
Location: [File path and line number]
Standard: [Link to relevant skill file]
Fix: [Brief explanation or example]
```

## Module-Specific Considerations

When reviewing code, consider the module's domain:

- **paypal-pos-auth**: OAuth flows, token management, security
- **paypal-pos-sync**: Product/inventory sync, data consistency
- **paypal-pos-webhooks**: Signature validation, event handling
- **paypal-pos-queue**: Background jobs, retry logic
- **paypal-pos-settings**: Admin UI, option storage
- **paypal-pos-onboarding**: Setup wizard, initial configuration

## Notes

- All detailed standards are in the `paypal-pos-backend-dev`, `paypal-pos-dev-cycle`, and `paypal-pos-copy-guidelines` skills
- Consult those skills for complete context and examples
- When in doubt, refer to the specific skill documentation linked above
- PayPal POS uses modular architecture - ensure code is in the correct module
- Project requires PHP 8.2+, WordPress 6.8+, WooCommerce 10.2+
