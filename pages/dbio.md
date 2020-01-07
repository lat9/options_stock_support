# Database I/O Manager Reports

Starting with v2.2.0 of _POSM_, there's now an integration with the [Database I/O Manager (DbIo)](https://vinosdefrutastropicales.com/index.php?main_page=product_info&cPath=2_3&products_id=68). This enables you to perform exports and imports of your store's _POSM_ information using an Excel® or Open-Office `.csv` file, making it easier to manage your products.

**Notes:**

-   You'll need to ensure that you've previously installed the _Database I/O Manager_ … v1.1.0 or later. Earlier versions did not include the support required by the _POSM_ handlers.
-   These handlers _fully support_ the additional database fields required by the [Products' Options' Stock Manager — Price/Weight Extension](https://vinosdefrutastropicales.com/index.php?main_page=product_info&cPath=2_7&products_id=60)!

The sections below contain detailed descriptions of the "handlers" provided and some usage tips.

-  [DbIoOptionsStockFullHandler](#dbiooptionsstockfullhandler)
-  [DbIoOptionsStockUpdateHandler](#dbiooptionsstockupdatehandler)
-  [DbIoOptionsStockNamesHandler](#dbiooptionsstocknameshandler)


## DbIoOptionsStockFullHandler

This handler enables you to export and, more importantly, import configurations for your store's products' option-combinations! The handler can perform the following operations in support of your store's _POSM_ configuration:

*   Export
*   Import
    *   Update a record
    *   Add a record, with possible "side-effects":
        1.  Option-values added to an _existing_ option
        2.  Attributes added to _existing_ products
    *   Remove a record

I suggest first exporting your existing _POSM_ configuration; that process will output a `.csv` that contains the field-defining headers that you'll need to _import_ changes. In addition to that header information, the file will contain one line for each product-option-combination currently configured.

The import's processing, on a record-by-record basis, attempts to identify the associated product using the `v_products_id` or, if no match is found for the product's ID, its `v_products_model`. When a match-by-model is attempted, the model number _must be unique_ within your products; an imported record going through match-by-model with a _non-unique_ model will be disallowed.

Once an import-record's product has been identified, the handler next validates the `v_products_options_combination` value. This value identifies the option/option-value pairs for the current record's import; the field is formatted as `option_name1**~**option_value_name1[**^**option_name2**~**option_value_name2 …]`, where the option-names and option-value-names are specified _in the store's **default** language_. Processing continues in the sequence described in the following table, stopping whenever the record has been determined to be _not importable_.

| Condition | Action Taken |
| No option found in the store's database matching _option_name<sub>n</sub>_ | The record is _not importable_. |
| No option-value found in the store's database matching _option_value_name<sub>n</sub>_ | If a **_unique_** _option_name<sub>n</sub>_ has been located and no other issues are found with the record, _option_value_name<sub>n</sub>_ will be created and associated with _option_name<sub>n</sub>_ … for **all** languages supported by the store. Otherwise, the record is _not importable_. |
| No attribute is found for the current product for the combination _option_name<sub>n</sub>~option_value_name<sub>n</sub>_ | The option-combination is added (with default values) as an attribute for the current product. |
|All previous checks have passed. | The product's option-combination is either added to, updated in or removed from the _POSM_ configuration in the database. |

## DbIoOptionsStockUpdateHandler

</noscript>

This handler enables you to export and update configurations for your store's products' option-combinations! The handler can perform the following operations in support of your store's _POSM_ configuration:

*   Export
*   Import, update an existing record

I suggest first exporting your existing _POSM_ configuration; that process will output a `.csv` that contains the field-defining headers that you'll need to _import_ changes. In addition to that header information, the file will contain one line for each product-option-combination currently configured.

The import's processing, on a record-by-record basis, attempts to identify the associated product using the `v_products_id` or, if no match is found for the product's ID, its `v_products_model`. When a match-by-model is attempted, the model number _must be unique_ within your products; an imported record going through match-by-model with a non-unique model will result in that record's import being disallowed.

Once an import-record's product has been identified, the handler next validates the `v_products_options_combination` value. This value identifies the option/option-value pairs for the current record's import; the field is formatted as `option_name1**~**option_value_name1[**^**option_name2**~**option_value_name2 …]`, where the option-names and option-value-names are specified _in the store's **default** language_. Processing continues in the sequence described in the following table, stopping whenever the record has been determined to be _not importable_.

| Condition | Action Taken |
| No option found in the store's database matching _option_name<sub>n</sub>_ | The record is _not importable_. |
| No option-value found in the store's database matching _option_value_name<sub>n</sub>_ | The record is _not importable_. |
| No attribute is found for the current product for the combination _option_name<sub>n</sub>~option_value_name<sub>n</sub>_ | The record is _not importable_; use the `DbIoOptionsStockFullHandler` instead. |
| The product's option-combination is not currently _POSM_-managed. | The record is _not importable_; use the `DbIoOptionsStockFullHandler` instead. |
| All previous checks have passed. | The product's option-combination is updated in the _POSM_ configuration in the database. |

## DbIoOptionsStockNamesHandler

This simple handler provides the interface to export and import the configurations that you've made to the `products_options_stock_names` table. As with the other handlers, I suggest performing an "Export" of the table's current contents prior to any "Import" so that you've got the field names/columns auto-created.
