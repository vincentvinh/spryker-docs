---
title: Creating product lists
description: Use the procedure to create a product list by assigning products and selecting the category in the Back Office.
last_updated: Aug 11, 2021
template: back-office-user-guide-template
originalLink: https://documentation.spryker.com/2021080/docs/creating-a-product-list
originalArticleId: e7682701-b572-4934-9422-2a95d31610a1
redirect_from:
  - /2021080/docs/creating-a-product-list
  - /2021080/docs/en/creating-a-product-list
  - /docs/creating-a-product-list
  - /docs/en/creating-a-product-list
  - /docs/scos/user/back-office-user-guides/202200.0/catalog/product-lists/creating-product-lists.html
---

This doc describes how to create product lists. Product lists are used to allow or deny companies access to products.

## Prerequisites

* If you want to assign categories to the product list, [create the categories](/docs/scos/user/back-office-user-guides/{{page.version}}/catalog/category/create-categories.html).
* If you want to assign or import products for the product list, [create the products](/docs/scos/user/back-office-user-guides/{{page.version}}/catalog/products/manage-concrete-products/adding-product-alternatives.html).

## Create a product list

1. Go to **Catalog&nbsp;<span aria-label="and then">></span> Product Lists**.
2. On the **Overview of Product Lists** page, click **Create a Product List**.
3. On the **Create a Product List** page, enter a **TITLE**.
4. Select a **TYPE**.
5. Click **Save**.
    The page refreshes with a success message displayed.
6. Add products to the list using any of the following methods:
    * [Assign categories to the product list](#assign-categories-to-the-product-list)
    * [Assign products to the product list](#assign-products-to-the-product-list)
    * [Import products for the product list](#import-products-for-the-product-list)

### Reference information: Create product lists

| ATTRIBUTE | DESCRIPTION |
|-|-|
| TITLE | Name that you will use for identifying the list in the Back Office. |
| TYPE | Defines whether a company will be able to see the products in the list. |

## Assign categories to the product list

1. Click the **Assign Categories** tab.
2. Enter and select one or more **CATEGORIES**.
    The products from the selected categories will be added to the product list.
3. Click **Save**.
    The page refreshes with a success message displayed.

## Assign products to the product list

1. Click the **Assign Products** tab.
2. On the **Select Products to assign** subtab, select the products you want to assign.
3. Select **Save**.
    The page refreshes with the success message displayed. The assigned products are displayed in the **Products in this list** subtab.

**Tips and tricks**
</br> When assigning a lot of products at a time, it might be useful to double-check your selection in the **Products to be assigned** tab.

## Import products for the product lists

1. Click the **Assign Products** tab.
2. Click **Choose File**.
3. Select the product list file to be uploaded. The file should contain the `product_list_key` and `concrete_sku` columns.
4. Click **Save**.
    The page refreshes with the success message displayed. The assigned products are displayed in the **Products in this list** subtab.    


## Next steps

* Allow or deny a company access to the product list by creating a new merchant relationship.
* 

[Edit product lists](/docs/scos/user/back-office-user-guides/{{page.version}}/catalog/product-lists/edit-product-lists.html).