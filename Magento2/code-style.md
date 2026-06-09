# Code Style — Magento 2 / Adobe Commerce (Authoritative)

This document defines mandatory coding rules.
Deviation from these rules is a failure.

All examples below are canonical and must be followed exactly.

## Architecture
- Use Magento 2 Service Contracts.
- Always define an interface first.
- Concrete implementations must strictly implement their interfaces.
- Use constructor dependency injection only.
- Do not use service locators, static helpers, or global state.
- Keep classes small and single-responsibility.

### Admin UI Separation
- All adminhtml UI related code (UI components, blocks, controllers, phtml templates, adminhtml layouts, etc.) MUST be placed in a separate module.
- Such module MUST use the same base name with the postfix `AdminUi`.
    - Example: `EPuzzle_CustomerPrice` → `EPuzzle_CustomerPriceAdminUi`
- Business logic MUST NOT be duplicated in AdminUi modules.
- AdminUi modules may only orchestrate and call existing services.
- The Magento module name MUST end with `AdminUi` (module.xml / registration.php).

## PHP Rules
- Always use `declare(strict_types=1);`
- All method arguments and return values must be explicitly typed.
- Avoid nullable types unless strictly required by existing contracts.
- Methods must be short and intention-revealing.
- No fluent APIs unless already used in the codebase.
- Maximum line length is 120.
- Only one enter (new line) symbol at the end of class files.

## JavaScript Rules

### Architectural Principles
- **Encapsulation**: Components must be self-contained. Communication must occur strictly via events, `customerData` observables, or declarative `imports`/`listens` wiring in the component configuration.
- **Minimalism**: Code must be simple and readable by design. Comments are considered exceptions; code should be self-explanatory through naming and structure.
- **English Language**: Any necessary comments must be written exclusively in English.
- **No Global Scope**: Do not attach custom objects to the `window` or `document` scope.
- **Logic Separation**: Business logic must reside in PHP Services, not in JS. JS components are strictly for UI orchestration.

### Implementation Standards
- **AMD Modules**: All JS files must be wrapped in `define([...], function (...) { 'use strict'; ... });`.
- **Initialization**: Always call `this._super()` within `initialize()`. Event listeners must be attached using `.bind(this)` to maintain context.
- **Naming**:
  - Public methods: `camelCase`.
  - Event handlers: Must end with the `Handle` suffix (e.g., `updateCartHandle`).
- **Data Binding**: Favor `defaults` and `tracks` for reactive properties. Direct DOM manipulation is forbidden; use Knockout templates and bindings.

## Naming Conventions
- Interfaces: `SomethingInterface`
- Implementations: `Something`
- Methods: `getX()`, `setX()`
- No abbreviations.
- Names must reflect responsibility precisely.

## Data Handling
- Use `getData()` / `setData()` patterns consistently.
- Do not access internal data arrays directly.

### DTO Rules
- Magento Service Contract Data Interfaces ARE allowed and expected.
- Custom DTO classes (separate data carriers) are FORBIDDEN unless an identical pattern already exists in the codebase.

## Dependency Injection
- All dependencies must be injected via constructor.
- DI wiring must be defined in `di.xml` (preferences, virtualTypes, plugins, arguments).
- Use other Magento `etc/*` XML files only for their intended purpose (webapi.xml, acl.xml, events.xml, etc.).
- No runtime resolution, no optional dependencies, no fallback logic.

## XML (di.xml)
- Keep XML minimal and explicit.
- Preferences must map interfaces to implementations exactly.
- Do not add unused or speculative configuration.
- XML must stay in sync with PHP classes.

## Rule of Similarity
- Before implementing anything, find the closest existing example.
- Copy structure, naming, visibility, and style from that example.
- If no close example exists, stop and ask one clarifying question.

## Absolute Rule
If a requested change violates these rules:
- DO NOT implement the change.
- Stop and explain exactly which rule is violated and why.

## Forbidden Patterns
- Introducing new architectural layers.
- Refactoring unrelated code.
- Renaming existing public APIs.
- Adding abstractions "for future use".
- Explaining instead of implementing.

## Examples
Examples below are canonical; copy structure, docblocks, and naming exactly.

### Example 1 - Service Contracts
#### Interface
```php
<?php

declare(strict_types=1);

namespace EPuzzle\CustomerPrice\Api\Data;

/**
 * The customer price entity
 */
interface CustomerPriceInterface
{
    /**
     * Get customer price ID
     *
     * @return int
     */
    public function getItemId(): int;

    /**
     * Set customer price ID
     *
     * @param int $value
     * @return void
     */
    public function setItemId(int $value): void;

    /**
     * Get product ID
     *
     * @return int
     */
    public function getProductId(): int;

    /**
     * Set product ID
     *
     * @param int $value
     * @return void
     */
    public function setProductId(int $value): void;

    /**
     * Get customer ID
     *
     * @return int
     */
    public function getCustomerId(): int;

    /**
     * Set customer ID
     *
     * @param int $value
     * @return void
     */
    public function setCustomerId(int $value): void;

    /**
     * Get price
     *
     * @return float
     */
    public function getPrice(): float;

    /**
     * Set price
     *
     * @param float $value
     * @return void
     */
    public function setPrice(float $value): void;

    /**
     * Get quantity
     *
     * @return float
     */
    public function getQty(): float;

    /**
     * Set quantity
     *
     * @param float $value
     * @return void
     */
    public function setQty(float $value): void;

    /**
     * Get website ID
     *
     * @return int
     */
    public function getWebsiteId(): int;

    /**
     * Set website ID
     *
     * @param int $value
     * @return void
     */
    public function setWebsiteId(int $value): void;

    /**
     * Get created at
     *
     * @return string
     */
    public function getCreatedAt(): string;

    /**
     * Set created at
     *
     * @param string $value
     * @return void
     */
    public function setCreatedAt(string $value): void;

    /**
     * Get updated at
     *
     * @return string
     */
    public function getUpdatedAt(): string;

    /**
     * Set updated at
     *
     * @param string $value
     * @return void
     */
    public function setUpdatedAt(string $value): void;
}

```

#### Implementation
```php
<?php

declare(strict_types=1);

namespace EPuzzle\CustomerPrice\Model;

use EPuzzle\CustomerPrice\Api\Data\CustomerPriceInterface;
use Magento\Framework\Model\AbstractModel;

/**
 * The model of the customer price entity
 * @SuppressWarnings(PHPMD.CamelCasePropertyName)
 * @SuppressWarnings(PHPMD.CamelCaseMethodName)
 */
class CustomerPrice extends AbstractModel implements CustomerPriceInterface
{
    /**
     * @var string
     */
    protected $_cacheTag = 'epuzzle_customer_price';
    /**
     * @var string
     */
    protected $_eventPrefix = 'epuzzle_customer_price';

    /**
     * @inheritDoc
     */
    protected function _construct()
    {
        $this->_init(ResourceModel\CustomerPrice::class);
        $this->setIdFieldName(ResourceModel\CustomerPrice::PK);
    }

    /**
     * @inheritDoc
     */
    public function getItemId(): int
    {
        return (int)$this->getData(ResourceModel\CustomerPrice::PK);
    }

    /**
     * @inheritDoc
     */
    public function setItemId(int $value): void
    {
        $this->setData(ResourceModel\CustomerPrice::PK, $value);
    }

    /**
     * @inheritDoc
     */
    public function getProductId(): int
    {
        return (int)$this->getData('product_id');
    }

    /**
     * @inheritDoc
     */
    public function setProductId(int $value): void
    {
        $this->setData('product_id', $value);
    }

    /**
     * @inheritDoc
     */
    public function getCustomerId(): int
    {
        return (int)$this->getData('customer_id');
    }

    /**
     * @inheritDoc
     */
    public function setCustomerId(int $value): void
    {
        $this->setData('customer_id', $value);
    }

    /**
     * @inheritDoc
     */
    public function getPrice(): float
    {
        return (float)$this->getData('price');
    }

    /**
     * @inheritDoc
     */
    public function setPrice(float $value): void
    {
        $this->setData('price', $value);
    }

    /**
     * @inheritDoc
     */
    public function getQty(): float
    {
        return (float)$this->getData('qty');
    }

    /**
     * @inheritDoc
     */
    public function setQty(float $value): void
    {
        $this->setData('qty', $value);
    }

    /**
     * @inheritDoc
     */
    public function getWebsiteId(): int
    {
        return (int)$this->getData('website_id');
    }

    /**
     * @inheritDoc
     */
    public function setWebsiteId(int $value): void
    {
        $this->setData('website_id', $value);
    }

    /**
     * @inheritDoc
     */
    public function getCreatedAt(): string
    {
        return (string)$this->getData('created_at');
    }

    /**
     * @inheritDoc
     */
    public function setCreatedAt(string $value): void
    {
        $this->setData('created_at', $value);
    }

    /**
     * @inheritDoc
     */
    public function getUpdatedAt(): string
    {
        return (string)$this->getData('updated_at');
    }

    /**
     * @inheritDoc
     */
    public function setUpdatedAt(string $value): void
    {
        $this->setData('updated_at', $value);
    }
}

```
### Example 2 - XML files

#### di.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="EPuzzle\CustomerPrice\Api\Data\CustomerPriceInterface" type="EPuzzle\CustomerPrice\Model\CustomerPrice"/>
    <preference for="EPuzzle\CustomerPrice\Api\Data\CustomerPriceSearchResultsInterface" type="EPuzzle\CustomerPrice\Model\CustomerPriceSearchResults"/>
    <preference for="EPuzzle\CustomerPrice\Api\CustomerPriceRepositoryInterface" type="EPuzzle\CustomerPrice\Model\CustomerPriceRepository"/>
    <preference for="EPuzzle\CustomerPrice\Model\Customer\CustomerProviderInterface" type="EPuzzle\CustomerPrice\Model\Customer\CustomerProvider"/>

    <type name="EPuzzle\CustomerPrice\Model\CustomerPriceRepository">
        <arguments>
            <argument name="getById" xsi:type="object">EPuzzle\CustomerPrice\Model\CustomerPrice\GetById\Proxy</argument>
            <argument name="save" xsi:type="object">EPuzzle\CustomerPrice\Model\CustomerPrice\Save\Proxy</argument>
            <argument name="deleteById" xsi:type="object">EPuzzle\CustomerPrice\Model\CustomerPrice\DeleteById\Proxy</argument>
            <argument name="delete" xsi:type="object">EPuzzle\CustomerPrice\Model\CustomerPrice\Delete\Proxy</argument>
            <argument name="getList" xsi:type="object">EPuzzle\CustomerPrice\Model\CustomerPrice\GetList\Proxy</argument>
        </arguments>
    </type>

    <!-- Price Models and Collectors -->
    <virtualType name="Magento\Catalog\Pricing\Price\Pool">
        <arguments>
            <argument name="prices" xsi:type="array">
                <item name="epuzzle-customer-price" xsi:type="string">EPuzzle\CustomerPrice\Pricing\Price\CustomerPrice</item>
            </argument>
        </arguments>
    </virtualType>
    <type name="Magento\Catalog\Model\ResourceModel\Product\LinkedProductSelectBuilderInterface">
        <plugin name="epuzzle-customer-price-linked-product-customer-price"
                type="EPuzzle\CustomerPrice\Plugin\Model\ResourceModel\Product\LinkedProductSelectBuilder\LinkedProductCustomerPrice"
                sortOrder="10"/>
    </type>
    <type name="EPuzzle\CustomerPrice\Model\ConfigProvider">
        <arguments>
            <argument name="collectors" xsi:type="array">
                <item name="default" xsi:type="object">EPuzzle\CustomerPrice\Pricing\Price\CustomerPrice\PriceCollector</item>
            </argument>
        </arguments>
    </type>
    <!-- /Price Models and Collectors -->

    <!-- Indexer -->
    <virtualType name="additionalFieldsProviderForElasticsearch">
        <arguments>
            <argument name="fieldsProviders" xsi:type="array">
                <item name="epuzzleCustomerPriceFields" xsi:type="object">EPuzzle\CustomerPrice\Model\Adapter\BatchDataMapper\CustomerPriceFieldsProvider</item>
            </argument>
        </arguments>
    </virtualType>
    <type name="Magento\Elasticsearch\Model\Adapter\FieldMapper\Product\CompositeFieldProvider">
        <arguments>
            <argument name="providers" xsi:type="array">
                <item name="epuzzleCustomerPrice" xsi:type="object">EPuzzle\CustomerPrice\Model\Adapter\FieldMapper\Product\FieldProvider\CustomerPriceField</item>
            </argument>
        </arguments>
    </type>
    <type name="Magento\Elasticsearch\Model\Adapter\FieldMapperInterface">
        <plugin name="epuzzle-customer-price-update-price-field-to-customer-price"
                type="EPuzzle\CustomerPrice\Plugin\Model\Adapter\FieldMapper\FieldMapperResolver\UpdatePriceFieldToCustomerPriceField"
                sortOrder="10"/>
    </type>
    <!-- /Indexer -->

    <!-- Cache -->
    <type name="EPuzzle\CustomerPrice\Model\Command\FlushCacheByTags">
        <arguments>
            <argument name="cacheList" xsi:type="array">
                <item name="block_html" xsi:type="const">Magento\Framework\App\Cache\Type\Block::TYPE_IDENTIFIER</item>
                <item name="collections" xsi:type="const">Magento\Framework\App\Cache\Type\Collection::TYPE_IDENTIFIER</item>
            </argument>
        </arguments>
    </type>
    <type name="EPuzzle\CustomerPrice\Model\ResourceModel\CustomerPrice">
        <plugin name="epuzzle-customer-price-reindex-after-save-customer-price"
                type="EPuzzle\CustomerPrice\Plugin\Model\ResourceModel\CustomerPrice\CustomerPrice\ReindexAndFlushCache"
                sortOrder="10"/>
    </type>
    <type name="Magento\Framework\Pricing\Render\PriceBox">
        <plugin name="epuzzle-customer-price-aad-cache-tags-for-price-box"
                type="EPuzzle\CustomerPrice\Plugin\Pricing\Render\PriceBox\AddCacheTagsToPriceBox"
                sortOrder="10"/>
    </type>
    <type name="Magento\CatalogSearch\Model\Indexer\Mview\Action">
        <plugin name="epuzzle-customer-price-flush-cache-after-reindex"
                type="EPuzzle\CustomerPrice\Plugin\Model\Indexer\Mview\Action\FlushCacheAfterReindex"
                sortOrder="10"/>
    </type>
    <!-- /Cache -->
</config>
```
#### webapi.xml

```xml
<?xml version="1.0"?>
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    <route url="/V1/epuzzle-customer-price/:itemId" method="GET">
        <service class="EPuzzle\CustomerPrice\Api\CustomerPriceRepositoryInterface" method="get"/>
        <resources>
            <resource ref="EPuzzle_CustomerPrice::webapi"/>
        </resources>
    </route>
    <route url="/V1/epuzzle-customer-price" method="POST">
        <service class="EPuzzle\CustomerPrice\Api\CustomerPriceRepositoryInterface" method="save"/>
        <resources>
            <resource ref="EPuzzle_CustomerPrice::webapi"/>
        </resources>
    </route>
    <route url="/V1/epuzzle-customer-price/:itemId" method="PUT">
        <service class="EPuzzle\CustomerPrice\Api\CustomerPriceRepositoryInterface" method="save"/>
        <resources>
            <resource ref="EPuzzle_CustomerPrice::webapi"/>
        </resources>
    </route>
    <route url="/V1/epuzzle-customer-price/:itemId" method="DELETE">
        <service class="EPuzzle\CustomerPrice\Api\CustomerPriceRepositoryInterface" method="deleteById"/>
        <resources>
            <resource ref="EPuzzle_CustomerPrice::webapi"/>
        </resources>
    </route>
    <route url="/V1/epuzzle-customer-price" method="GET">
        <service class="EPuzzle\CustomerPrice\Api\CustomerPriceRepositoryInterface" method="getList"/>
        <resources>
            <resource ref="EPuzzle_CustomerPrice::webapi"/>
        </resources>
    </route>
</routes>
```
#### acl.xml

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/acl.xsd">
    <acl>
        <resources>
            <resource id="Magento_Backend::admin">
                <resource id="EPuzzle_Base::base">
                    <resource id="EPuzzle_CustomerPrice::base" title="Customer Prices" sortOrder="100">
                        <resource id="EPuzzle_CustomerPrice::webapi" title="Webapi" sortOrder="10"/>
                    </resource>
                </resource>
            </resource>
        </resources>
    </acl>
</config>
```
### Example 3 - system files
#### composer.json

```json
{
    "name": "epuzzle/magento2-customer-price",
    "description": "This module provides prices for customers individually. If you want to add product prices for customers you can use this module.",
    "config": {
        "sort-packages": true
    },
    "version": "2.0.1",
    "require": {
        "php": "~8.1.0||~8.2.0||~8.3.0||~8.4.0",
        "magento/framework": "*",
        "magento/module-catalog": "*",
        "epuzzle/magento2-module-base": "*"
    },
    "suggest": {},
    "type": "magento2-module",
    "license": [
        "MIT"
    ],
    "autoload": {
        "files": [
            "registration.php"
        ],
        "psr-4": {
            "EPuzzle\\CustomerPrice\\": ""
        }
    }
}
```

#### registration.php

```php
<?php

declare(strict_types=1);

use Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,
    'EPuzzle_CustomerPrice',
    __DIR__
);

```

#### Canonical JS Component Example
```javascript
define([
    'uiComponent',
    'Magento_Customer/js/customer-data'
], function (Component, customerData) {
    'use strict';
  
    return Component.extend({
        defaults: {
            tracks: {
                cartData: true
            },
            cartData: {}
        },

        /**
         * @inheritDoc
         */
        initialize() {
            this._super();
            this.initializeSubscriptions();
      
            return this;
        },

        /**
         * Initialize the list of subscriptions
         *
         * @returns {void}
         */
        initializeSubscriptions() {
            customerData.get('cart').subscribe(this.updateCartHandle.bind(this));
        },

        /**
         * Updates the cart handle
         *
         * @param {Object} cart
         * @returns {void}
         */
        updateCartHandle(cart) {
            this.cartData = cart;
        }
    });
});
```
