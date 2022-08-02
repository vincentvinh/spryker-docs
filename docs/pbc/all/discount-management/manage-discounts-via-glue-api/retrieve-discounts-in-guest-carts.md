---
title: Retrieve discounts in guest carts
description: Retrieve details about cart rules and vouchers in guest carts
last_updated: July 25, 2022
template: glue-api-storefront-guide-template

---

This document describes how to retrieve cart rules, vouchers, and promotional items in guest carts. To learn about all the management options of guest carts, see [Managing guest carts of registered users](/docs/scos/dev/glue-api-guides/{{site.version}}/managing-carts/guest-carts/managing-guest-carts.html).

## Installation

For detailed information on the modules that provide the API functionality and related installation instructions, see:

* [Glue API: Cart feature integration](/docs/scos/dev/feature-integration-guides/{{site.version}}/glue-api/glue-api-cart-feature-integration.html)
* [Glue API: Promotions & Discounts feature integration](/docs/scos/dev/feature-integration-guides/{{site.version}}/glue-api/glue-api-promotions-and-discounts-feature-integration.html)

## Retrieve a guest cart

To retrieve a guest cart, send the request:

***
`GET` **/guest-carts**
***

{% info_block infoBox "Guest cart ID" %}


Guest users have one guest cart by default. If you already have a guest cart, you can optionally specify its ID when adding items. To do that, use the following endpoint. The information in this section is valid for both of the endpoints.

`GET` **/guest-carts/*{% raw %}{{{% endraw %}guestCartId{% raw %}}}{% endraw %}***

| PATH PARAMETER | DESCRIPTION |
| --- | --- |
| ***{% raw %}{{{% endraw %}guestCartId{% raw %}}}{% endraw %}*** | Unique identifier of the guest cart. To get it, [retrieve a guest cart](#retrieve-a-guest-cart). |

{% endinfo_block %}

{% info_block warningBox "Note" %}

When retrieving the cart with `guestCartId`, the response includes a single object, and when retrieving the resource without specifying it, you get an array containing a single object.

{% endinfo_block %}

### Request

| HEADER KEY | HEADER VALUE EXAMPLE | REQUIRED | DESCRIPTION |
| --- | --- | --- | --- |
| X-Anonymous-Customer-Unique-Id | 164b-5708-8530 | &check; | Guest user's unique identifier. For security purposes, we recommend passing a hyphenated alphanumeric value, but you can pass any. If you are sending automated requests, you can configure your API client to generate this value.|

| PATH PARAMETER | DESCRIPTION | Possible values |
| --- | --- | --- |
| include | Adds resource relationships to the request. | <ul><li>cart-rules</li><li>promotional-items</li><li>vouchers</li> |

| REQUEST | USAGE |
| --- | --- |
| `GET https://glue.mysprykershop.com/guest-carts?include=cart-rules` | Retrieve a guest cart with information about the cart rules. |
| `GET https://glue.mysprykershop.com/guest-carts?include=vouchers` | Retrieve a guest cart with information about vouchers. |


### Response


<details>
<summary markdown='span'>Response sample: retrieve a guest cart with cart rules included</summary>

```json
{
    "data": [
        {
            "type": "guest-carts",
            "id": "f8782b6c-848d-595e-b3f7-57374f1ff6d7",
            "attributes": {
                "priceMode": "GROSS_MODE",
                "currency": "EUR",
                "store": "DE",
                "name": "Shopping cart",
                "isDefault": true,
                "totals": {
                    "expenseTotal": 0,
                    "discountTotal": 10689,
                    "taxTotal": 15360,
                    "subtotal": 106892,
                    "grandTotal": 96203,
                    "priceToPay": 96203
                },
                "discounts": [
                    {
                        "displayName": "10% Discount for all orders above",
                        "amount": 10689,
                        "code": null
                    }
                ],
                "thresholds": []
            },
            "links": {
                "self": "https://glue.mysprykershop.com/guest-carts/f8782b6c-848d-595e-b3f7-57374f1ff6d7"
            },
            "relationships": {
                "cart-rules": {
                    "data": [
                        {
                            "type": "cart-rules",
                            "id": "1"
                        }
                    ]
                }
            }
        }
    ],
    "links": {
        "self": "https://glue.mysprykershop.com/cart-codes?include=cart-rules"
    },
    "included": [
        {
            "type": "cart-rules",
            "id": "1",
            "attributes": {
                "amount": 10689,
                "code": null,
                "discountType": "cart_rule",
                "displayName": "10% Discount for all orders above",
                "isExclusive": false,
                "expirationDateTime": "2020-12-31 00:00:00.000000",
                "discountPromotionAbstractSku": null,
                "discountPromotionQuantity": null
            },
            "links": {
                "self": "https://glue.mysprykershop.com/cart-rules/1"
            }
        }
    ]
}
```
</details>


<details>
<summary markdown='span'>Response sample: retrieve a guest cart with a cart rule and a discount voucher</summary>

```json
{
    "data": {
        "type": "guest-carts",
        "id": "1ce91011-8d60-59ef-9fe0-4493ef3628b2",
        "attributes": {...},
        "links": {...},
        "relationships": {
            "vouchers": {
                "data": [
                    {
                        "type": "vouchers",
                        "id": "mydiscount-yu8je"
                    }
                ]
            },
            "cart-rules": {
                "data": [
                    {
                        "type": "cart-rules",
                        "id": "1"
                    }
                ]
            }
        }
    },
    "included": [
        {
            "type": "vouchers",
            "id": "mydiscount-yu8je",
            "attributes": {
                "amount": 49898,
                "code": "mydiscount-yu8je",
                "discountType": "voucher",
                "displayName": "My Discount",
                "isExclusive": false,
                "expirationDateTime": "2020-02-29 00:00:00.000000",
                "discountPromotionAbstractSku": null,
                "discountPromotionQuantity": null
            },
            "links": {
                "self": "http://glue.mysprykershop.com/vouchers/mydiscount-yu8je"
            }
        },
        {
            "type": "cart-rules",
            "id": "1",
            "attributes": {
                "amount": 19959,
                "code": null,
                "discountType": "cart_rule",
                "displayName": "10% Discount for all orders above",
                "isExclusive": false,
                "expirationDateTime": "2020-12-31 00:00:00.000000",
                "discountPromotionAbstractSku": null,
                "discountPromotionQuantity": null
            },
            "links": {
                "self": "http://glue.mysprykershop.com/cart-rules/1"
            }
        }
    ]
}
```
</details>

<a name="guest-cart-response-attributes"></a>

{% include pbc/all/glue-api-guides/manage-guest-carts-response-attributes.md %} <!-- To edit, see /_includes/pbc/all/glue-api-guides/manage-guest-carts-response-attributes.md -->


| INCLUDED RESOURCE | ATTRIBUTE | TYPE | DESCRIPTION |
| --- | --- | --- | --- |
| vouchers, cart-rules | displayName | String | Discount name displayed on the Storefront. |
| vouchers, cart-rules | amount | Integer | Amount of the provided discount. |
| vouchers, cart-rules | code | String | Discount code. |
| vouchers, cart-rules | discountType | String | Discount type. |
| vouchers, cart-rules  | isExclusive | Boolean | Discount exclusivity. |
| vouchers, cart-rules | expirationDateTime | DateTimeUtc | Date and time on which the discount expires. |
| vouchers, cart-rules | discountPromotionAbstractSku | String | SKU of the products to which the discount applies. If the discount can be applied to any product, the value is `null`. |
| vouchers, cart-rules | discountPromotionQuantity | Integer | Specifies the amount of the product required to be able to apply the discount. If the minimum number is `0`, the value is `null`. |

## Possible errors

| CODE | REASON |
| --- | --- |
| 101 | Cart with given uuid not found. |
| 104 | Cart uuid is missing. |
| 109 | Anonymous customer unique id is empty. |

To view generic errors that originate from the Glue Application, see [Reference information: GlueApplication errors](/docs/scos/dev/glue-api-guides/{{site.version}}/reference-information-glueapplication-errors.html).