---
title: Marketplace Product Offer feature integration
last_updated: Mar 29, 2021
description: This document describes the process how to integrate the Marketplace Product Offer feature into a Spryker project.
template: feature-integration-guide-template
---

This document describes how to integrate the Marketplace Product Offer into a Spryker project.

## Install feature core

Follow the steps below to install the Marketplace Product Offer feature core.

### Prerequisites

To start feature integration, integrate the required features:

| NAME | VERSION | INTEGRATION GUIDE |
| --------------- | ------- | -------|
| Spryker Core         | 202001.0   | [Spryker Core feature integration](https://documentation.spryker.com/docs/spryker-core-feature-integration) |
| Marketplace Merchant | dev-master | [Marketplace Merchant feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/marketplace-merchant-feature-integration.html) |
| Product              | 202001.0   | [Product feature integration](https://github.com/spryker-feature/product) |

###  1) Install the required modules using Composer

Install the required modules:

```bash
composer require spryker-feature/marketplace-product-offer --update-with-dependencies
```

{% info_block warningBox "Verification" %}

Make sure that the following modules were installed:

| MODULE | EXPECTED DIRECTORY |
| ------------------ | ---------------------- |
| MerchantProductOffer           | spryker/merchant-product-offer             |
| MerchantProductOfferDataImport | spryker/merchant-product-offer-data-import |
| MerchantProductOfferGui        | spryker/merchant-product-offer-gui         |
| MerchantProductOfferSearch     | spryker/merchant-product-offer-search      |
| MerchantProductOfferStorage    | spryker/merchant-product-offer-storage     |
| ProductOffer                   | spryker/product-offer                      |
| ProductOfferGui                | spryker/product-offer-gui                  |
| ProductOfferSales              | spryker/product-offer-sales                |
| ProductOfferValidity           | spryker/product-offer-validity             |
| ProductOfferValidityGui        | spryker/product-offer-validity-gui         |
| ProductOfferValidityDataImport | spryker/product-offer-validity-data-import |

{% endinfo_block %}

### 2) Set up database schema

Adjust the schema definition so that entity changes will trigger events:

**src/Pyz/Zed/ProductOffer/Persistence/Propel/Schema/spy_product_offer.schema.xml**

```xml
<?xml version="1.0"?>
<database xmlns="spryker:schema-01"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          name="zed"
          xsi:schemaLocation="spryker:schema-01 https://static.spryker.com/schema-01.xsd"
          namespace="Orm\Zed\ProductOffer\Persistence"
          package="src.Orm.Zed.ProductOffer.Persistence">

    <table name="spy_product_offer" phpName="SpyProductOffer">
        <behavior name="event">
            <parameter name="spy_product_offer_all" column="*"/>
            <parameter name="spy_product_offer_product_offer_reference" column="product_offer_reference" keep-additional="true"/>
            <parameter name="spy_product_offer_concrete_sku" column="concrete_sku" keep-additional="true"/>
        </behavior>
    </table>

    <table name="spy_product_offer_store" phpName="SpyProductOfferStore">
        <behavior name="event">
            <parameter name="spy_product_offer_store_all" column="*"/>
        </behavior>
    </table>
</database>
```

Apply database changes and to generate entity and transfer changes:

```bash
console transfer:generate
console propel:install
console transfer:generate
```

{% info_block warningBox "Verification" %}

Verify that the following changes have been implemented by checking your database:

| DATABASE ENTITY | TYPE | EVENT |
| -------------------------- | ----- | ------ |
| spy_product_offer                            | table  | created |
| spy_product_offer_store                      | table  | created |
| spy_product_concrete_product_offers_storage  | table  | created |
| spy_product_offer_storage                    | table  | created |
| spy_sales_order_item.product_offer_reference | column | created |
| spy_product_offer_validity                   | table  | created |


Make sure that the following changes were applied in transfer objects:

| TRANSFER  | TYPE  | EVENT | PATH  |
| --------------- | -------- | ------ | ---------------- |
| MerchantProductOfferCriteriaFilter | class     | created | src/Generated/Shared/Transfer/MerchantProductOfferCriteriaFilterTransfer |
| DataImporterConfiguration          | class     | created | src/Generated/Shared/Transfer/DataImporterConfigurationTransfer |
| ProductOffer                       | class     | created | src/Generated/Shared/Transfer/ProductOfferTransfer           |
| StringFacetMap                     | class     | created | src/Generated/Shared/Transfer/StringFacetMapTransfer         |
| ProductOffer.idProductConcrete     | attribute | created | src/Generated/Shared/Transfer/ProductOfferTransfer           |
| ProductOfferResponse.isSuccessful  | attribute | created | src/Generated/Shared/Transfer/ProductOfferResponseTransfer   |
| ProductOfferCriteriaFilter         | class     | created | src/Generated/Shared/Transfer/ProductOfferCriteriaFilterTransfer |
| Item.productOfferReference         | attribute | created | src/Generated/Shared/Transfer/ItemTransfer                   |
| ProductOfferValidity               | class     | created | src/Generated/Shared/Transfer/ProductOfferValidityTransfer   |
| ProductOffer.productOfferValidity  | attribute | created | src/Generated/Shared/Transfer/ProductOfferTransfer           |

{% endinfo_block %}

### 3) Add Zed translations

Generate a new translation cache for Zed:

```bash
console translator:generate-cache
```

### 4) Configure export to Redis and Elasticsearch

To configure export to Redis and Elasticsearch, take the following steps:

#### Set up event listeners

| PLUGIN | SPECIFICATION | PREREQUISITES | NAMESPACE |
| --------------- | ------------- | ----------- | ---------------- |
| MerchantProductOfferStorageEventSubscriber | Registers listeners responsible for publishing merchant product offers to storage. |           | Spryker\Zed\MerchantProductOfferStorage\Communication\Plugin\Event\Subscriber |
| MerchantProductOfferSearchEventSubscriber  | Registers listeners responsible for publishing merchant product offer search to storage. |           | Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\Event\Subscriber |
| MerchantSearchEventSubscriber              | Registers listeners responsible for publishing merchant search to storage. |           | Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\Event\Subscriber |

**src/Pyz/Zed/Event/EventDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Event;

use Spryker\Zed\Event\EventDependencyProvider as SprykerEventDependencyProvider;
use Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\Event\Subscriber\MerchantProductOfferSearchEventSubscriber;
use Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\Event\Subscriber\MerchantSearchEventSubscriber;
use Spryker\Zed\MerchantProductOfferStorage\Communication\Plugin\Event\Subscriber\MerchantProductOfferStorageEventSubscriber;

class EventDependencyProvider extends SprykerEventDependencyProvider
{
    public function getEventSubscriberCollection()
    {
        $eventSubscriberCollection = parent::getEventSubscriberCollection();
        $eventSubscriberCollection->add(new MerchantSearchEventSubscriber());
        $eventSubscriberCollection->add(new MerchantProductOfferSearchEventSubscriber());
        $eventSubscriberCollection->add(new MerchantProductOfferStorageEventSubscriber());

        return $eventSubscriberCollection;
    }
}
```

Register the synchronization queue and synchronization error queue:

**src/Pyz/Client/RabbitMq/RabbitMqConfig.php**

```php
<?php

namespace Pyz\Client\RabbitMq;

use Spryker\Client\RabbitMq\RabbitMqConfig as SprykerRabbitMqConfig;
use Spryker\Shared\MerchantProductOfferStorage\MerchantProductOfferStorageConfig;

class RabbitMqConfig extends SprykerRabbitMqConfig
{
    /**
     * @return array
     */
    protected function getSynchronizationQueueConfiguration(): array
    {
        return [
            MerchantProductOfferStorageConfig::MERCHANT_PRODUCT_OFFER_SYNC_STORAGE_QUEUE,
        ];
    }

}
```

#### Configure message processors

| PLUGIN | SPECIFICATION | PREREQUISITES | NAMESPACE |
| ----------------- | -------------- | -------- | ------------ |
| SynchronizationStorageQueueMessageProcessorPlugin | Configures all merchant product offers to sync with Redis storage and marks messages as failed in case of error. |           | Spryker\Zed\Synchronization\Communication\Plugin\Queue |

**src/Pyz/Zed/Queue/QueueDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Queue;

use Spryker\Shared\MerchantProductOfferStorage\MerchantProductOfferStorageConfig;
use Spryker\Zed\Kernel\Container;
use Spryker\Zed\Queue\QueueDependencyProvider as SprykerDependencyProvider;
use Spryker\Zed\Synchronization\Communication\Plugin\Queue\SynchronizationStorageQueueMessageProcessorPlugin;

class QueueDependencyProvider extends SprykerDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return \Spryker\Zed\Queue\Dependency\Plugin\QueueMessageProcessorPluginInterface[]
     */
    protected function getProcessorMessagePlugins(Container $container)
    {
        return [
            MerchantProductOfferStorageConfig::MERCHANT_PRODUCT_OFFER_SYNC_STORAGE_QUEUE => new SynchronizationStorageQueueMessageProcessorPlugin(),
        ];
    }
}
```

#### Set up, re-generate, and re-sync features

| PLUGIN | SPECIFICATION | PREREQUISITES | NAMESPACE |
| ----------------- | --------------- | ---------- | ---------------- |
| ProductConcreteProductOffersSynchronizationDataBulkRepositoryPlugin | Allows synchronizing the entire storage table content into Storage. |           | Spryker\Zed\MerchantProductOfferStorage\Communication\Plugin\Synchronization |
| ProductOfferSynchronizationDataBulkRepositoryPlugin                 | Allows synchronizing the entire storage table content into Storage. |           | Spryker\Zed\MerchantProductOfferStorage\Communication\Plugin\Synchronization |

**rc/Pyz/Zed/Synchronization/SynchronizationDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Synchronization;

use Spryker\Zed\MerchantProductOfferStorage\Communication\Plugin\Synchronization\ProductConcreteProductOffersSynchronizationDataBulkRepositoryPlugin;
use Spryker\Zed\MerchantProductOfferStorage\Communication\Plugin\Synchronization\ProductOfferSynchronizationDataBulkRepositoryPlugin;
use Spryker\Zed\Synchronization\SynchronizationDependencyProvider as SprykerSynchronizationDependencyProvider;

class SynchronizationDependencyProvider extends SprykerSynchronizationDependencyProvider
{
    /**
     * @return \Spryker\Zed\SynchronizationExtension\Dependency\Plugin\SynchronizationDataPluginInterface[]
     */
    protected function getSynchronizationDataPlugins(): array
    {
        return [
            new ProductConcreteProductOffersSynchronizationDataBulkRepositoryPlugin(),
            new ProductOfferSynchronizationDataBulkRepositoryPlugin(),
        ];
    }
}
```

Configure synchronization pool name:

**src/Pyz/Zed/MerchantProductOfferStorage/MerchantProductOfferStorageConfig.php**

```php
<?php

namespace Pyz\Zed\MerchantProductOfferStorage;

use Pyz\Zed\Synchronization\SynchronizationConfig;
use Spryker\Shared\Publisher\PublisherConfig;
use Spryker\Zed\MerchantProductOfferStorage\MerchantProductOfferStorageConfig as SprykerMerchantProductOfferStorageConfig;

class MerchantProductOfferStorageConfig extends SprykerMerchantProductOfferStorageConfig
{
    /**
     * @api
     *
     * @return string|null
     */
    public function getMerchantProductOfferSynchronizationPoolName(): ?string
    {
        return SynchronizationConfig::DEFAULT_SYNCHRONIZATION_POOL_NAME;
    }
}
```

{% info_block warningBox "Verification" %}

 Make sure that after setting up the event listeners, the following commands do the following:

   1. `console sync:data product_concrete_product_offers` exports data from `spy_product_concrete_product_offers_storage` table to Redis.
   2. `console sync:data product_offer` exports data from `spy_product_offer_storage` table to Redis.

Make sure that when the following entities get updated via the ORM, the corresponding Redis keys have the correct values.

| TARGET ENTITY | EXAMPLE EXPECTED DATA IDENTIFIER | EXAMPLE EXPECTED DATA FRAGMENT |
| ---------- | ------------------------- | ------------------ |
| ProductOffer  | kv:product_offer:offer2     | {“id_product_offer”:1,“id_merchant”:6,“product_offer_reference”:“offer1",“merchant_sku”:“GS952M00H-Q11"} |
| ProductOffer  | kv:product_concrete_product_offers:093_24495843 | [“offer3”,“offer4"]   |

{% endinfo_block %}

### 5) Import data

Prepare your data according to your requirements using the demo data:

<details><summary>data/import/common/common/marketplace/merchant_product_offer.csv</summary>

```csv
product_offer_reference,concrete_sku,merchant_reference,merchant_sku,is_active,approval_status
offer1,093_24495843,MER000001,GS952M00H-Q11,1,approved
offer2,090_24495844,MER000002,,1,approved
offer3,091_25873091,MER000001,M9122A0AQ-C11,1,approved
offer4,091_25873091,MER000002,M9122A0AQ-C11,1,approved
offer5,092_24495842,MER000001,TH344E01G-Q11,0,approved
offer6,092_24495842,MER000002,OB054P005-Q11,1,approved
offer7,193_32124735,MER000001,,1,approved
offer8,001_25904006,MER000002,,1,approved
offer9,002_25904004,MER000002,,0,approved
offer10,003_26138343,MER000002,,0,waiting_for_approval
offer11,004_30663302,MER000002,,1,waiting_for_approval
offer12,005_30663301,MER000002,,1,approved
offer13,006_30692993,MER000002,,1,approved
offer14,007_30691822,MER000002,,1,approved
offer15,008_30692992,MER000002,,1,approved
offer16,009_30692991,MER000002,,1,approved
offer17,010_30692994,MER000002,,1,approved
offer18,011_30775359,MER000002,,1,approved
offer19,012_25904598,MER000002,,1,approved
offer20,013_25904584,MER000002,,1,approved
offer21,014_25919241,MER000002,,1,approved
offer22,015_25904009,MER000002,,1,approved
offer23,016_21748907,MER000002,,1,approved
offer24,017_21748906,MER000002,,1,approved
offer25,018_21081477,MER000002,,1,approved
offer26,019_21081473,MER000002,,1,approved
offer27,020_21081478,MER000002,,1,approved
offer28,021_21081475,MER000002,,1,approved
offer29,022_21994751,MER000002,,1,approved
offer30,023_21758366,MER000002,,1,approved
offer31,024_21987578,MER000002,,1,approved
offer32,025_21764665,MER000002,,1,approved
offer33,026_21748904,MER000002,,1,approved
offer34,027_26976107,MER000002,,1,approved
offer35,028_26976108,MER000002,,1,approved
offer36,029_26976109,MER000002,,1,approved
offer37,030_30021698,MER000002,,1,approved
offer38,031_30021637,MER000002,,1,approved
offer39,032_32125551,MER000002,,1,approved
offer40,033_32125568,MER000002,,1,approved
offer41,034_32125390,MER000002,,1,approved
offer42,035_17360369,MER000002,,1,approved
offer43,036_17360368,MER000002,,1,approved
offer44,037_25904011,MER000002,,1,approved
offer45,038_25905593,MER000002,,1,approved
offer46,039_25904010,MER000002,,1,approved
offer47,040_25904665,MER000002,,1,approved
offer48,041_25904691,MER000002,,1,approved
offer49,001_25904006,MER000005,,1,approved
offer50,002_25904004,MER000005,,1,approved
offer51,003_26138343,MER000005,,0,approved
offer52,004_30663302,MER000005,,1,approved
offer53,005_30663301,MER000005,,0,approved
offer54,006_30692993,MER000005,,1,approved
offer55,007_30691822,MER000005,,1,waiting_for_approval
offer56,008_30692992,MER000005,,1,waiting_for_approval
offer57,009_30692991,MER000005,,1,approved
offer58,010_30692994,MER000005,,1,approved
offer59,011_30775359,MER000005,,1,approved
offer60,012_25904598,MER000005,,1,approved
offer61,013_25904584,MER000005,,1,approved
offer62,014_25919241,MER000005,,1,approved
offer63,015_25904009,MER000005,,1,approved
offer64,016_21748907,MER000005,,1,approved
offer65,017_21748906,MER000005,,1,approved
offer66,018_21081477,MER000005,,1,approved
offer67,019_21081473,MER000005,,1,approved
offer68,020_21081478,MER000005,,1,approved
offer69,021_21081475,MER000005,,1,approved
offer70,022_21994751,MER000005,,1,approved
offer71,023_21758366,MER000005,,1,approved
offer72,024_21987578,MER000005,,1,approved
offer73,025_21764665,MER000005,,1,approved
offer74,026_21748904,MER000005,,1,approved
offer75,027_26976107,MER000005,,1,approved
offer76,028_26976108,MER000005,,1,approved
offer77,029_26976109,MER000005,,1,approved
offer78,030_30021698,MER000005,,1,approved
offer79,031_30021637,MER000005,,1,approved
offer80,032_32125551,MER000005,,1,approved
offer81,033_32125568,MER000005,,1,approved
offer82,034_32125390,MER000005,,1,approved
offer83,035_17360369,MER000005,,1,approved
offer84,036_17360368,MER000005,,1,approved
offer85,037_25904011,MER000005,,1,approved
offer86,038_25905593,MER000005,,1,approved
offer87,039_25904010,MER000005,,1,approved
offer88,040_25904665,MER000005,,1,approved
offer89,041_25904691,MER000005,,1,approved
offer90,016_21748907,MER000006,,1,approved
offer91,017_21748906,MER000006,,1,waiting_for_approval
offer92,018_21081477,MER000006,,1,approved
offer93,019_21081473,MER000006,,0,approved
offer94,020_21081478,MER000006,,1,approved
offer95,021_21081475,MER000006,,1,approved
offer96,022_21994751,MER000006,,1,approved
offer97,023_21758366,MER000006,,1,approved
offer98,024_21987578,MER000006,,1,approved
offer99,025_21764665,MER000006,,1,approved
offer100,026_21748904,MER000006,,0,approved
offer101,027_26976107,MER000006,,1,approved
offer102,028_26976108,MER000006,,1,approved
offer103,029_26976109,MER000006,,1,waiting_for_approval
offer169,076_24394207,MER000006,,1,approved
offer170,077_24584210,MER000006,,1,approved
offer171,078_24602396,MER000006,,1,approved
offer172,079_24394211,MER000006,,1,approved
offer173,080_24394206,MER000006,,1,approved
offer348,193_32124735,MER000006,,1,approved
offer349,194_25904145,MER000006,,1,approved
offer350,195_25904159,MER000006,,1,approved
offer351,196_23120327,MER000006,,1,approved
offer352,197_21421718,MER000006,,1,approved
offer353,198_19692589,MER000006,,1,approved
offer354,199_7016823,MER000006,,1,approved
offer355,199_24788780,MER000006,,1,approved
offer356,200_5787536,MER000006,,1,approved
offer357,201_11217755,MER000006,,1,approved
offer358,202_5782479,MER000006,,1,approved
offer359,203_15619960,MER000006,,1,approved
offer360,204_29851280,MER000006,,1,approved
offer402,101_29727910,MER000004,,1,approved
offer403,102_30727008,MER000005,,0,approved
offer404,102_30727008,MER000005,,1,denied
offer405,102_30727008,MER000005,,1,waiting_for_approval
offer410,104_30727010,MER000005,,1,approved
offer411,113_29885591,MER000005,,1,approved
offer412,113_29885591,MER000002,,1,approved
offer413,118_29804739,MER000005,,1,approved
offer414,118_29804739,MER000002,,1,approved
offer415,112_312526171,MER000005,,1,approved
offer416,112_306918001,MER000002,,1,approved
offer417,112_312526191,MER000005,,1,approved
offer418,112_312526172,MER000002,,1,approved
```

</details>

| COLUMN | REQUIRED? | DATA TYPE | DATA EXAMPLE | DATA EXPLANATION |
| -------------- | ----------- | -------- | --------- | ------------------ |
| product_offer_reference | &check;      | string    | offer1        | Product offer reference that will be referenced to this merchant. |
| concrete_sku            | &check;        | string    | 093_24495843  | Concrete product SKU this product offer is attached to.      |
| merchant_reference      | &check;        | string    | `MER000002`   | Merchant identifier.                                         |
| merchant_sku            |        | string    | GS952M00H-Q11 | merchant internal SKU for the product offer.                 |
| is_active               |        | boolean   | 1             | Product offer status, defaults to 1.                          |
| approval_status         |        | string    | approved      | Approval status (Waiting for Approval – Approved – Denied). Denied and Waiting for Approval statuses mean that the offer is not visible on PDP regardless of Product Offer → Active = true.This can be configured (along with the transition between statuses in ProductOfferConfig). If not supplied, ProductOfferConfig → getDefaultStatus is applied. |

<details><summary>data/import/common/common/marketplace/merchant_product_offer_store.csv</summary>

```csv
product_offer_reference,store_name
offer1,DE
offer2,DE
offer3,DE
offer4,DE
offer6,DE
offer7,DE
offer8,DE
offer9,DE
offer10,DE
offer11,DE
offer12,DE
offer13,DE
offer14,DE
offer15,DE
offer16,DE
offer17,DE
offer18,DE
offer19,DE
offer20,DE
offer21,DE
offer22,DE
offer23,DE
offer24,DE
offer25,DE
offer26,DE
offer27,DE
offer28,DE
offer29,DE
offer30,DE
offer31,DE
offer32,DE
offer33,DE
offer34,DE
offer35,DE
offer36,DE
offer37,DE
offer38,DE
offer39,DE
offer40,DE
offer41,DE
offer42,DE
offer43,DE
offer44,DE
offer45,DE
offer46,DE
offer47,DE
offer48,DE
offer49,DE
offer50,DE
offer51,DE
offer52,DE
offer53,DE
offer54,DE
offer55,DE
offer56,DE
offer57,DE
offer58,DE
offer59,DE
offer60,DE
offer61,DE
offer62,DE
offer63,DE
offer64,DE
offer65,DE
offer66,DE
offer67,DE
offer68,DE
offer69,DE
offer70,DE
offer71,DE
offer72,DE
offer73,DE
offer74,DE
offer75,DE
offer76,DE
offer77,DE
offer78,DE
offer79,DE
offer80,DE
offer81,DE
offer82,DE
offer83,DE
offer84,DE
offer85,DE
offer86,DE
offer87,DE
offer88,DE
offer89,DE
offer90,DE
offer91,DE
offer92,DE
offer93,DE
offer94,DE
offer95,DE
offer96,DE
offer97,DE
offer98,DE
offer99,DE
offer100,DE
offer101,DE
offer102,DE
offer103,DE
offer169,DE
offer170,DE
offer171,DE
offer172,DE
offer173,DE
offer348,DE
offer349,DE
offer350,DE
offer351,DE
offer352,DE
offer353,DE
offer354,DE
offer355,DE
offer356,DE
offer357,DE
offer358,DE
offer359,DE
offer360,DE
offer402,DE
offer403,DE
offer404,DE
offer405,DE
offer410,DE
offer411,DE
offer412,DE
offer413,DE
offer414,DE
offer415,DE
offer416,DE
offer417,DE
offer418,DE
offer2,US
offer4,US
offer6,US
offer7,US
offer8,US
offer9,US
offer10,US
offer11,US
offer12,US
offer13,US
offer14,US
offer15,US
offer16,US
offer17,US
offer18,US
offer19,US
offer20,US
offer21,US
offer22,US
offer23,US
offer24,US
offer25,US
offer26,US
offer27,US
offer28,US
offer29,US
offer30,US
offer31,US
offer32,US
offer33,US
offer34,US
offer35,US
offer36,US
offer37,US
offer38,US
offer39,US
offer40,US
offer41,US
offer42,US
offer43,US
offer44,US
offer45,US
offer46,US
offer47,US
offer48,US
offer49,US
offer50,US
offer51,US
offer52,US
offer53,US
offer54,US
offer55,US
offer56,US
offer57,US
offer58,US
offer59,US
offer60,US
offer61,US
offer62,US
offer63,US
offer64,US
offer65,US
offer66,US
offer67,US
offer68,US
offer69,US
offer70,US
offer71,US
offer72,US
offer73,US
offer74,US
offer75,US
offer76,US
offer77,US
offer78,US
offer79,US
offer80,US
offer81,US
offer82,US
offer83,US
offer84,US
offer85,US
offer86,US
offer87,US
offer88,US
offer89,US
offer90,US
offer91,US
offer92,US
offer93,US
offer94,US
offer95,US
offer96,US
offer97,US
offer98,US
offer99,US
offer100,US
offer101,US
offer102,US
offer103,US
offer169,US
offer170,US
offer171,US
offer172,US
offer173,US
offer348,US
offer349,US
offer350,US
offer351,US
offer352,US
offer353,US
offer354,US
offer355,US
offer356,US
offer357,US
offer358,US
offer359,US
offer360,US
offer1,AT
offer2,AT
offer3,AT
offer4,AT
offer6,AT
offer7,AT
offer8,AT
offer9,AT
offer10,AT
offer11,AT
offer12,AT
offer13,AT
offer14,AT
offer15,AT
offer16,AT
offer17,AT
offer18,AT
offer19,AT
offer20,AT
offer21,AT
offer22,AT
offer23,AT
offer24,AT
offer25,AT
offer26,AT
offer27,AT
offer28,AT
offer29,AT
offer30,AT
offer31,AT
offer32,AT
offer33,AT
offer34,AT
offer35,AT
offer36,AT
offer37,AT
offer38,AT
offer39,AT
offer40,AT
offer41,AT
offer42,AT
offer43,AT
offer44,AT
offer45,AT
offer46,AT
offer47,AT
offer48,AT
offer49,AT
offer50,AT
offer51,AT
offer52,AT
offer53,AT
offer54,AT
offer55,AT
offer56,AT
offer57,AT
offer58,AT
offer59,AT
offer60,AT
offer61,AT
offer62,AT
offer63,AT
offer64,AT
offer65,AT
offer66,AT
offer67,AT
offer68,AT
offer69,AT
offer70,AT
offer71,AT
offer72,AT
offer73,AT
offer74,AT
offer75,AT
offer76,AT
offer77,AT
offer78,AT
offer79,AT
offer80,AT
offer81,AT
offer82,AT
offer83,AT
offer84,AT
offer85,AT
offer86,AT
offer87,AT
offer88,AT
offer89,AT
offer90,AT
offer91,AT
offer92,AT
offer93,AT
offer94,AT
offer95,AT
offer96,AT
offer97,AT
offer98,AT
offer99,AT
offer100,AT
offer101,AT
offer102,AT
offer103,AT
offer169,AT
offer170,AT
offer171,AT
offer172,AT
offer173,AT
offer348,AT
offer349,AT
offer350,AT
offer351,AT
offer352,AT
offer353,AT
offer354,AT
offer355,AT
offer356,AT
offer357,AT
offer358,AT
offer359,AT
offer360,AT
offer402,AT
offer403,AT
offer404,AT
offer405,AT
offer410,AT
offer411,AT
offer412,AT
offer413,AT
offer414,AT
offer415,AT
offer416,AT
offer417,AT
offer418,AT

```

| COLUMN | REQUIRED? | DATA TYPE | DATA EXAMPLE | DATA EXPLANATION |
| ---------------- | ------------ | ------- | ------------- | ------- |
| product_offer_reference | &check;     | string    | roan-gmbh-und-co-k-g | Product Offer reference, unique identifier per Offer. |
| store_name              | &check;    | string    | DE  | The name of the store.  |

**data/import/common/common/marketplace/product_offer_validity.csv**

```csv
product_offer_reference,valid_from,valid_to
offer1,,2020-01-20 00:00:00.000000
offer2,,2020-01-20 00:00:00.000000
offer3,,2020-01-20 00:00:00.000000
offer4,,2020-01-20 00:00:00.000000
offer5,2030-01-01 00:00:00.000000,
offer6,2030-01-01 00:00:00.000000,
offer7,2030-01-01 00:00:00.000000,
offer8,2030-01-01 00:00:00.000000,
offer9,2020-07-01 00:00:00.000000,2025-12-01 00:00:00.000000
offer10,2020-07-01 00:00:00.000000,2025-12-01 00:00:00.000000
offer49,,2020-01-20 00:00:00.000000
offer50,,2020-01-20 00:00:00.000000
offer51,2030-01-01 00:00:00.000000,
offer52,2030-01-01 00:00:00.000000,
offer53,2020-07-01 00:00:00.000000,2025-12-01 00:00:00.000000
offer54,2020-07-01 00:00:00.000000,2025-12-01 00:00:00.000000
offer90,,2020-01-20 00:00:00.000000
offer91,,2020-01-20 00:00:00.000000
offer92,2030-01-01 00:00:00.000000,
offer93,2030-01-01 00:00:00.000000,
offer94,2020-07-01 00:00:00.000000,2025-12-01 00:00:00.000000
offer95,2020-07-01 00:00:00.000000,2025-12-01 00:00:00.000000
```

</details>

| COLUMN | REQUIRED? | DATA TYPE | DATA EXAMPLE | DATA EXPLANATION |
| ------------ | ----------- | ------ | ----------- | ---------------- |
| product_offer_reference | &check; | string    | offer1       | Unique product offer identifier.             |
| valid_from              |  | String  | 2020-01-01   | Date since which the product offer is valid. |
| valid_to                |  | String | 2020-01-01   | Date till which the product offer is valid.  |

Register the following plugins to enable data import:

| PLUGIN | SPECIFICATION | PREREQUISITES | NAMESPACE |
| ------------------ | ---------------------- | ---------- | ---------------------- |
| MerchantProductOfferDataImportPlugin      | Imports merchant product offer data.          |           | Spryker\Zed\MerchantProductOfferDataImport\Communication\Plugin\DataImport |
| MerchantProductOfferStoreDataImportPlugin | Imports the product offer to store relation data. |           | Spryker\Zed\MerchantProductOfferDataImport\Communication\Plugin\DataImport |
| ProductOfferValidityDataImportPlugin      | Imports product offer validity data.             |           | Spryker\Zed\ProductOfferValidityDataImport\Communication\DataImport     |

**src/Pyz/Zed/DataImport/DataImportDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\DataImport;

use Spryker\Zed\DataImport\DataImportDependencyProvider as SprykerDataImportDependencyProvider;

use Spryker\Zed\MerchantProductOfferDataImport\Communication\Plugin\DataImport\MerchantProductOfferDataImportPlugin;
use Spryker\Zed\MerchantProductOfferDataImport\Communication\Plugin\DataImport\MerchantProductOfferStoreDataImportPlugin;
use Spryker\Zed\ProductOfferValidityDataImport\Communication\DataImport\ProductOfferValidityDataImportPlugin;


class DataImportDependencyProvider extends SprykerDataImportDependencyProvider
{
    protected function getDataImporterPlugins(): array
    {
        return [
            new MerchantProductOfferDataImportPlugin(),
            new MerchantProductOfferStoreDataImportPlugin(),
            new ProductOfferValidityDataImportPlugin(),
        ];
    }
}
```

Import data:

```bash
console data:import merchant-product-offer
console data:import merchant-product-offer-store
console data:import product-offer-validity
```

{% info_block warningBox "Verification" %}

Make sure that the product offer data is attached to Merchants in `spy_product_offer`.

Make sure that the product offer data is attached to Stores in `spy_product_offer_store`.

Make sure that the product offer validity data is correctly imported in `spy_product_offer_validity`.

{% endinfo_block %}

### 6) Set up behavior

Enable the following behaviors by registering the plugins:

| PLUGIN | DESCRIPTION | PREREQUISITES | NAMESPACE |
| ---------------- | --------------- | -------------- | ---------------- |
| MerchantProductOfferTableExpanderPlugin              | Expands the  `ProductOfferGui` product table with merchant data.     |                            | Spryker\Zed\MerchantProductOfferGui\Communication\Plugin |
| MerchantProductOfferViewSectionPlugin                | Adds a new merchant section to the `ProductOfferGui` view.     |                            | Spryker\Zed\MerchantProductOfferGui\Communication\Plugin\ProductOfferGui     |
| ProductOfferValidityProductOfferViewSectionPlugin    | Adds a new validity section to the `ProductOfferGui` view.      |                            | Spryker\Zed\ProductOfferValidityGui\Communication\Plugin\ProductOfferGui |
| MerchantProductOfferListActionViewDataExpanderPlugin | Expands product offer view data with merchant data when showing it in the  `ProductOfferGui` module. |                            | Spryker\Zed\MerchantGui\Communication\Plugin\ProductOffer    |
| MerchantReferenceQueryExpanderPlugin                 | Adds filter by the merchant reference to the search query.           |                            | Spryker\Client\MerchantProductOfferSearch\Plugin\Search             |
| MerchantNameSearchConfigExpanderPlugin               | Expands facet configuration with the merchant name filter.       |                            | Spryker\Client\MerchantProductOfferSearch\Plugin\Search             |
| MerchantProductPageDataExpanderPlugin                | Expands the provided `ProductAbstractPageSearch` transfer object's data by merchant names. |                            | Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch |
| MerchantProductPageDataLoaderPlugin                  | Expands the `ProductPageLoadTransfer` object with merchant data.   |                            | Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch |
| MerchantNamesProductAbstractMapExpanderPlugin        | Adds merchant names to product abstract search data.         |                            | Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch |
| MerchantReferencesProductAbstractsMapExpanderPlugin  | Adds merchant references to product abstract search data.    |                            | Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch |
| DefaultProductOfferReferenceStrategyPlugin           | Sets the default selected product offer in PDP for a concrete product. It selects the first product offer in the list. | ProductViewProductOfferExpanderPlugin | Spryker\Client\MerchantProductOfferStorage\Plugin\MerchantProductOfferStorage |
| ProductOfferReferenceStrategyPlugin                  | Sets selected oroduct offer in `ProductConcreteTransfer` if one is already selected on PDP. | ProductViewProductOfferExpanderPlugin | Spryker\Client\MerchantProductOfferStorage\Plugin\MerchantProductOfferStorage |
| ProductViewProductOfferExpanderPlugin | Adds product offer data to `ProductViewTransfer` when a retrieving product. |                            | Spryker\Client\MerchantProductOfferStorage\Plugin\ProductStorage |
| ProductOfferValidityProductOfferPostCreatePlugin     | Creates product offer validity dates after the product offer is created. |                            | Spryker\Zed\ProductOfferValidity\Communication\Plugin\ProductOffer |
| ProductOfferValidityProductOfferPostUpdatePlugin     | Updates product offer validity dates after the product offer is updated. |                            | Spryker\Zed\ProductOfferValidity\Communication\Plugin\ProductOffer |
| ProductOfferValidityProductOfferExpanderPlugin       | Expands product offer data with validity dates when the product offer is fetched. |                            | Spryker\Zed\ProductOfferValidity\Communication\Plugin\ProductOffer |
| ProductOfferValidityConsole                          | Updates product offers to have the `isActive` flag to be `false` where their validity date is not current anymore. |                            | Spryker\Zed\ProductOfferValidity\Communication\Console       |

**src/Pyz/Client/Catalog/CatalogDependencyProvider.php**

```php
<?php

namespace Pyz\Client\Catalog;

use Spryker\Client\Catalog\CatalogDependencyProvider as SprykerCatalogDependencyProvider;
use Spryker\Client\MerchantProductOfferSearch\Plugin\Search\MerchantReferenceQueryExpanderPlugin;

class CatalogDependencyProvider extends SprykerCatalogDependencyProvider
{
    /**
     * @return \Spryker\Client\Search\Dependency\Plugin\QueryExpanderPluginInterface[]|\Spryker\Client\SearchExtension\Dependency\Plugin\QueryExpanderPluginInterface[]
     */
    protected function createCatalogSearchQueryExpanderPlugins()
    {
        return [
            new MerchantReferenceQueryExpanderPlugin(),
        ];
    }

    /**
     * @return \Spryker\Client\Search\Dependency\Plugin\QueryExpanderPluginInterface[]|\Spryker\Client\SearchExtension\Dependency\Plugin\QueryExpanderPluginInterface[]
     */
    protected function createSuggestionQueryExpanderPlugins()
    {
        return [
            new MerchantReferenceQueryExpanderPlugin(),
        ];
    }
}
```

<details>
<summary markdown='span'>src/Pyz/Client/Search/SearchDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Client\Search;

use Spryker\Client\Kernel\Container;
use Spryker\Client\MerchantProductOfferSearch\Plugin\Search\MerchantNameSearchConfigExpanderPlugin;
use Spryker\Client\Search\SearchDependencyProvider as SprykerSearchDependencyProvider;

class SearchDependencyProvider extends SprykerSearchDependencyProvider
{
    /**
     * @param \Spryker\Client\Kernel\Container $container
     *
     * @return \Spryker\Client\Search\Dependency\Plugin\SearchConfigExpanderPluginInterface[]
     */
    protected function createSearchConfigExpanderPlugins(Container $container)
    {
        $searchConfigExpanderPlugins = parent::createSearchConfigExpanderPlugins($container);

        $searchConfigExpanderPlugins[] = new MerchantNameSearchConfigExpanderPlugin();

        return $searchConfigExpanderPlugins;
    }
}
```

**src/Pyz/Client/SearchElasticsearch/SearchElasticsearchDependencyProvider.php**

```php
<?php

namespace Pyz\Client\SearchElasticsearch;

use Spryker\Client\Kernel\Container;
use Spryker\Client\MerchantProductOfferSearch\Plugin\Search\MerchantNameSearchConfigExpanderPlugin;
use Spryker\Client\SearchElasticsearch\SearchElasticsearchDependencyProvider as SprykerSearchElasticsearchDependencyProvider;

class SearchElasticsearchDependencyProvider extends SprykerSearchElasticsearchDependencyProvider
{
    /**
     * @param \Spryker\Client\Kernel\Container $container
     *
     * @return \Spryker\Client\SearchExtension\Dependency\Plugin\SearchConfigExpanderPluginInterface[]
     */
    protected function getSearchConfigExpanderPlugins(Container $container): array
    {
        return [
            new MerchantNameSearchConfigExpanderPlugin(),
        ];
    }
}
```

</details>

<details>
<summary markdown='span'>src/Pyz/Zed/ProductOfferGui/ProductOfferGuiDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\ProductOfferGui;

use Spryker\Zed\MerchantGui\Communication\Plugin\ProductOffer\MerchantProductOfferListActionViewDataExpanderPlugin;
use Spryker\Zed\MerchantProductOfferGui\Communication\Plugin\MerchantProductOfferTableExpanderPlugin;
use Spryker\Zed\MerchantProductOfferGui\Communication\Plugin\ProductOfferGui\MerchantProductOfferViewSectionPlugin;
use Spryker\Zed\ProductOfferGui\ProductOfferGuiDependencyProvider as SprykerProductOfferGuiDependencyProvider;
use Spryker\Zed\ProductOfferValidityGui\Communication\Plugin\ProductOfferGui\ProductOfferValidityProductOfferViewSectionPlugin;

class ProductOfferGuiDependencyProvider extends SprykerProductOfferGuiDependencyProvider
{
    /**
     * @return \Spryker\Zed\ProductOfferGuiExtension\Dependency\Plugin\ProductOfferListActionViewDataExpanderPluginInterface[]
     */
    protected function getProductOfferListActionViewDataExpanderPlugins(): array
    {
        return [
            new MerchantProductOfferListActionViewDataExpanderPlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\ProductOfferGuiExtension\Dependency\Plugin\ProductOfferTableExpanderPluginInterface[]
     */
    protected function getProductOfferTableExpanderPlugins(): array
    {
        return [
            new MerchantProductOfferTableExpanderPlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\ProductOfferGuiExtension\Dependency\Plugin\ProductOfferViewSectionPluginInterface[]
     */
    public function getProductOfferViewSectionPlugins(): array
    {
        return [
            new MerchantProductOfferViewSectionPlugin(),
            new ProductOfferValidityProductOfferViewSectionPlugin(),
        ];
    }
}
```
</details>

<details>
<summary markdown='span'>src/Pyz/Zed/ProductPageSearch/ProductPageSearchDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\ProductPageSearch;

use Spryker\Shared\MerchantProductOfferSearch\MerchantProductOfferSearchConfig;
use Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch\MerchantProductPageDataExpanderPlugin;
use Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch\MerchantProductPageDataLoaderPlugin;
use Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch\MerchantNamesProductAbstractMapExpanderPlugin;
use Spryker\Zed\MerchantProductOfferSearch\Communication\Plugin\ProductPageSearch\MerchantReferencesProductAbstractsMapExpanderPlugin;
use Spryker\Zed\ProductPageSearch\ProductPageSearchDependencyProvider as SprykerProductPageSearchDependencyProvider;

class ProductPageSearchDependencyProvider extends SprykerProductPageSearchDependencyProvider
{
    /**
     * @return \Spryker\Zed\ProductPageSearch\Dependency\Plugin\ProductPageDataExpanderInterface[]
     */
    protected function getDataExpanderPlugins()
    {
        $dataExpanderPlugins = [];

        $dataExpanderPlugins[MerchantProductOfferSearchConfig::PLUGIN_PRODUCT_MERCHANT_DATA] = new MerchantProductPageDataExpanderPlugin();

        return $dataExpanderPlugins;
    }

    /**
     * @return \Spryker\Zed\ProductPageSearchExtension\Dependency\Plugin\ProductPageDataLoaderPluginInterface[]
     */
    protected function getDataLoaderPlugins()
    {
        return [
            new MerchantProductPageDataLoaderPlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\ProductPageSearchExtension\Dependency\Plugin\ProductAbstractMapExpanderPluginInterface[]
     */
    protected function getProductAbstractMapExpanderPlugins(): array
    {
        return [
            new MerchantNamesProductAbstractMapExpanderPlugin(),
            new MerchantReferencesProductAbstractsMapExpanderPlugin(),
        ];
    }
}
```

**src/Pyz/Client/MerchantProductOfferStorage/MerchantProductOfferStorageDependencyProvider.php**

```php
<?php

namespace Pyz\Client\MerchantProductOfferStorage;

use Spryker\Client\MerchantProductOfferStorage\MerchantProductOfferStorageDependencyProvider as SprykerMerchantProductOfferStorageDependencyProvider;
use Spryker\Client\MerchantProductOfferStorage\Plugin\MerchantProductOfferStorage\DefaultProductOfferReferenceStrategyPlugin;
use Spryker\Client\MerchantProductOfferStorage\Plugin\MerchantProductOfferStorage\ProductOfferReferenceStrategyPlugin;

class MerchantProductOfferStorageDependencyProvider extends SprykerMerchantProductOfferStorageDependencyProvider
{
    /**
     * @return \Spryker\Client\MerchantProductOfferStorageExtension\Dependency\Plugin\ProductOfferReferenceStrategyPluginInterface[]
     */
    protected function getProductOfferReferenceStrategyPlugins(): array
    {
        return [
            new ProductOfferReferenceStrategyPlugin(),
            new DefaultProductOfferReferenceStrategyPlugin(),
        ];
    }
}
```

</details>

**src/Pyz/Client/ProductStorage/ProductStorageDependencyProvider.php**

```php
<?php

namespace Pyz\Client\ProductStorage;

use Spryker\Client\MerchantProductOfferStorage\Plugin\ProductStorage\ProductViewProductOfferExpanderPlugin;
use Spryker\Client\ProductStorage\ProductStorageDependencyProvider as SprykerProductStorageDependencyProvider;

class ProductStorageDependencyProvider extends SprykerProductStorageDependencyProvider
{
    /**
     * @return \Spryker\Client\ProductStorage\Dependency\Plugin\ProductViewExpanderPluginInterface[]
     */
    protected function getProductViewExpanderPlugins()
    {
        return [
            new ProductViewProductOfferExpanderPlugin(),
        ];
    }
}
```

<details>
<summary markdown='span'>src/Pyz/Zed/ProductOffer/ProductOfferDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\ProductOffer;

use Spryker\Zed\ProductOffer\ProductOfferDependencyProvider as SprykerProductOfferDependencyProvider;
use Spryker\Zed\ProductOfferValidity\Communication\Plugin\ProductOffer\ProductOfferValidityProductOfferExpanderPlugin;
use Spryker\Zed\ProductOfferValidity\Communication\Plugin\ProductOffer\ProductOfferValidityProductOfferPostCreatePlugin;
use Spryker\Zed\ProductOfferValidity\Communication\Plugin\ProductOffer\ProductOfferValidityProductOfferPostUpdatePlugin;

class ProductOfferDependencyProvider extends SprykerProductOfferDependencyProvider
{
    /**
     * @return \Spryker\Zed\ProductOfferExtension\Dependency\Plugin\ProductOfferPostCreatePluginInterface[]
     */
    protected function getProductOfferPostCreatePlugins(): array
    {
        return [
            new ProductOfferValidityProductOfferPostCreatePlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\ProductOfferExtension\Dependency\Plugin\ProductOfferPostUpdatePluginInterface[]
     */
    protected function getProductOfferPostUpdatePlugins(): array
    {
        return [
            new ProductOfferValidityProductOfferPostUpdatePlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\ProductOfferExtension\Dependency\Plugin\ProductOfferExpanderPluginInterface[]
     */
    protected function getProductOfferExpanderPlugins(): array
    {
        return [
            new ProductOfferValidityProductOfferExpanderPlugin(),
        ];
    }
}
```

</details>

**src/Pyz/Zed/Console/ConsoleDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Console;

use Spryker\Zed\Console\ConsoleDependencyProvider as SprykerConsoleDependencyProvider;
use Spryker\Zed\ProductOfferValidity\Communication\Console\ProductOfferValidityConsole;

class ConsoleDependencyProvider extends SprykerConsoleDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return \Symfony\Component\Console\Command\Command[]
     */
    protected function getConsoleCommands(Container $container)
    {
        $commands = [
            new ProductOfferValidityConsole(),
        ];

        return $commands;
    }
}
```

{% info_block warningBox "Verification" %}

Make sure that a default product offer is given when retrieving product concrete data.

Make sure that validity data is saved when saving a product offer.

Make sure Merchant and Product Offer Validity sections exist on the product offer edit page in `ProductOfferGui`.

Make sure the Merchant column is in the Product Offers list in `ProductOfferGui`.

Make sure the console command invalidates expired product offers and reactivates product offers that are within their validity dates.

Make sure that when a merchant gets updated or published, or when a product offer gets published, created, or updated, the corresponding product abstracts get updated in the catalog search pages.
It means the following:
1. If a merchant gets deactivated, `ProductAbstract`s that were on the catalog search only because they had a product offer from that merchant get removed.
2. If a product offer gets created, and the `ProductAbstract` related to it was not available on catalog search, it would be available now.

{% endinfo_block %}

### 7) Configure navigation
Add product offers section to marketplace section of `navigation.xml`:

**config/Zed/navigation.xml**

```xml
<?xml version="1.0"?>
<config>
    <marketplace>
        <pages>
            <product-offer-gui>
                <label>Offers</label>
                <title>Offers</title>
                <bundle>product-offer-gui</bundle>
                <controller>list</controller>
                <action>index</action>
            </product-offer-gui>
        </pages>
    </marketplace>
</config>
```

Execute the following command:
```bash
console navigation:build-cache
```

{% info_block warningBox "Verification" %}

Make sure that, in the navigation menu of the Back Office, you can see the **Marketplace->Offers** menu item.

{% endinfo_block %}


## Install feature front end

Follow the steps below to install the Marketplace Product Offer feature front end.

### Prerequisites

To start feature integration, integrate the following features:

| NAME | VERSION | INTEGRATION GUIDE |
| ---------- | ----- | --------------|
| Spryker Core | 202001.0 | [Spryker Core feature integration](https://documentation.spryker.com/docs/spryker-core-feature-integration)  |

### 1) Install the required modules using Composer

If installed before, not needed.

{% info_block warningBox "Verification" %}

Verify that the following modules were installed:

| MODULE | EXPECTED DIRECTORY |
| ---------- | -------------- |
| MerchantProductOfferWidget | spryker-shop/merchant-product-offer-widget |

{% endinfo_block %}

### 2) Add Yves Translations

Append glossary according to your configuration:

**src/data/import/glossary.csv**

```csv
merchant_product_offer.view_seller,View Seller,en_US
merchant_product_offer.view_seller,Händler ansehen,de_DE
merchant_product_offer.sold_by,Sold by,en_US
merchant_product_offer.sold_by,Verkauft durch,de_DE
merchant.sold_by,Sold by,en_US
merchant.sold_by,Verkauft durch,de_DE
```

Import data:

```bash
console data:import glossary
```

{% info_block warningBox "Verification" %}

Make sure that the configured data is added to the `spy_glossary` table in the database.

{% endinfo_block %}

### 3) Set up widgets

Register the following plugins to enable widgets:

| PLUGIN | DESCRIPTION | PREREQUISITES | NAMESPACE |
| -------------- | --------------- | ------ | ---------------- |
| MerchantProductOfferWidget       | Shows the list of the offers with their prices for a concrete product. |           | SprykerShop\Yves\MerchantProductOfferWidget\Widget |
| ProductOfferSoldByMerchantWidget | Shows merchant data for an offer given a cart item.          |           | SprykerShop\Yves\MerchantProductOfferWidget\Widget |

**src/Pyz/Yves/ShopApplication/ShopApplicationDependencyProvider.php**

```php
<?php

namespace Pyz\Yves\ShopApplication;

use SprykerShop\Yves\MerchantProductOfferWidget\Widget\MerchantProductOfferWidget;
use SprykerShop\Yves\MerchantWidget\Widget\SoldByMerchantWidget;
use SprykerShop\Yves\ShopApplication\ShopApplicationDependencyProvider as SprykerShopApplicationDependencyProvider;

class ShopApplicationDependencyProvider extends SprykerShopApplicationDependencyProvider
{
    /**
     * @return string[]
     */
    protected function getGlobalWidgets(): array
    {
        return [
            MerchantProductOfferWidget::class,
            SoldByMerchantWidget::class,
        ];
    }
}
```

Enable Javascript and CSS changes:

```bash
console frontend:yves:build
```

{% info_block warningBox "Verification" %}

Make sure that the following widgets were registered:

| MODULE | TEST |
| ----------------- | ----------------- |
| MerchantProductOfferWidget       | Go to a product concrete detail page that has offers, and you will see the default offer is selected, and the widget is displayed. |
| SoldByMerchantWidget | Go through the checkout process with an offer, and you will see the sold by text and merchant data throughout the checkout process. |

{% endinfo_block %}

## Related features

| FEATURE | REQUIRED FOR THE CURRENT FEATURE | INTEGRATION GUIDE |
| -------------- | -------------------------------- | ----------------- |
| Marketplace Product Offer API | | [Marketplace Product Offer feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/glue/marketplace-product-offer-feature-integration.html) |
| Marketplace Product Offer + Cart | | [Marketplace Product Offer + Cart feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/marketplace-product-offer-cart-feature-integration.html) |