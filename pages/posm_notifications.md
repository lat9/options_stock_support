# Notifications Issued by Products' Options' Stock Manager

## Notification Changes

One of the major changes in POSM v4.4.0 is the migration of its notifications to use a `NOTIFY_` prefix  otherwise, any observers you've developed that use the [camelCased event-name syntax](https://docs.zen-cart.com/dev/code/notifiers/#camelcased-event-name) will fail to get that message in zc158!  The *legacy* notifications are still present in POSM v4.4.0 to support any back-testing needed to your custom observers as you convert them to use the now-current forms of notification.

***Note***: All *legacy* notifications will be removed in a future version of POSM.

## Storefront Notifications Issued

### From `/includes/classes/ajax/zcAjaxOptionsStockDependencies.php`

#### `NOTIFY_AJAX_POSM_DEPENDENCIES_SELECT_CLAUSE`

Indicates that POSM's AJAX processing is preparing to gather information about the current product's managed variants from the database, giving an observer the chance to add to the fields selected from the `products_options_stock` (pos) and `products_options_stock_attributes` (posa) tables.

##### Parameters

`$p2`: A reference to the `$select_clause` variable, initialized to `SELECT posa.options_id, posa.options_values_id, pos.products_quantity AS quantity, pos.pos_name_id AS oos_msg_id, pos.pos_model AS model, pos.pos_date AS oos_date`. An observer can add to this collection of fields.

#### `NOTIFY_AJAX_POSM_DEPENDENCIES_EXTRA_INFO`

Indicates that POSM's AJAX processing has successfully gathered the product's fields identified in the previous notification, enabling an observer to record some 'extra_info' into the option-values returned by the AJAX handler.

##### Parameters

`$p1`: An associative array of values representing the `$db` fields returned by the just-issued database query.

`$p2`: A reference to the `$extra_info` variable, initialized to an empty string. An observer can append its information here and the value is returned by the AJAX request as the 'extra_info' element of the `options_value` array.

#### `NOTIFY_AJAX_POSM_DEPENDENCIES_EXTENSION_INFO`

Indicates that POSM's AJAX processing has completed its processing and gives an observer a chance to identify any extension-related jQuery functions to be called upon return by POSM's dependent-attributes jQuery.

##### Parameters

`$p1`: An associative array containing the `$option_values` to be returned to the jQuery processing.

`$p2`: A reference to the `$extra_functions` variable, initialized to `(bool)false`. If a POSM extension has additional jQuery to process the attributes, its observer sets this value to an array of associative arrays keyed on the to-be-run jQuery function name with the array element's value being the parameters expected by said jQuery function.

### From `/includes/classes/observers/class.products_options_stock_observer.php`

#### `PRODUCTS_OPTIONS_STOCK_OBSERVER_INSTANTIATED` (legacy)

Indicates that the observer has been instantiated.  Starting with POSM v4.2.0, the issuance of the notification also indicates that POSM is enabled.  For previous versions of POSM, a watching observer needs to check the class' `enabled` flag for that indication.

##### Parameters

None.

##### Replaced by

 `NOTIFY_PRODUCTS_OPTIONS_STOCK_OBSERVER_INSTANTIATED`, using the same parameters

#### `POSM_ORDER_STOCK_DECREMENT_BEGIN` (legacy)

Issued by the observer during order-creation, once per unique product in the order, based on its receipt of the `NOTIFY_ORDER_PROCESSING_STOCK_DECREMENT_BEGIN` notification from the `order` class.  This gives a watching observer the opportunity to either bypass the stock decrement for ***all*** products or just for the current product (so long as the product is currently stock-managed by POSM).

The processing performed at this point enables POSM and/or a POSM-extension to modify the base `order` class processing of stock-decrementation.

##### Parameters

`$p1`: A read-only associative array containing the order's `product` array for the current product and its `stock` information.  The `stock` value is `(bool)false` if the product is not POSM-managed, otherwise the value contains a `$db` return that has selected all fields from the product variant's `products_options_stock` table entry.

`$p2`: A reference to the `$bypass_stock_decrement` variable, initialized to `(bool)false`.  If an observer sets this value to `(bool)true`, the stock-decrement for the product is not performed regardless of whether or not the product is POSM-managed.

`$p3`: A reference to the `$decrement_options_stock` variable, initialized to `(bool)true`.  If an observer sets this value to `(bool)false` and the product is POSM-managed, then the stock-decrement for the product is not performed.

##### Replaced by

 `NOTIFY_POSM_ORDER_STOCK_DECREMENT_BEGIN`, using the same parameters.

#### `POSM_ORDER_ADDED_PRODUCT_LINE_ITEM` (legacy)

Issued by the observer during order-creation, once per unique product in the order, based on its receipt of the `NOTIFY_ORDER_DURING_CREATE_ADDED_PRODUCT_LINE_ITEM` notification from the `order` class.  This gives a watching observer the opportunity to either bypass the stock decrement for ***all*** products or just for the current product (so long as the product is currently stock-managed by POSM). 

The processing performed at this point provides POSM an/or a POSM-extension a means to affect the product variant's stock recording in the database.

##### Parameters

`$p1`: A read-only associative array containing the  `product` (set to the order's `$sql_data_array` that just created an `orders_products` record for the current product) and the product's `stock` information.  The `stock` value is `(bool)false` if the product is not POSM-managed, otherwise the value contains a `$db` return that has selected all fields from the product variant's `products_options_stock` table entry.

`$p2`: A reference to the `$decrement_managed_stock` variable, initialized to `(bool)true`.  If an observer sets this value to `(bool)true`, the stock-decrement for the product is not performed regardless of whether or not the product is POSM-managed.

`$p3`: A reference to the `$decrement_options_stock` variable, initialized to `(bool)true`.  If an observer sets this value to `(bool)false` and the product is POSM-managed, then the stock-decrement for the product is not performed.

POSM's observer processes the returned values as one of four (4) possibilities ... in the order identified below:

1. If any observer has set `$decrement_managed_stock` (`$p2`) to a value other than `(bool)true`, POSM's observer totally bypasses its stock-handling.
2. If the overall product is not POSM-managed, the observer ensures that the product's recorded quantity does not fall below 0.
3. If the product variant is not POSM-managed or any observer has set `$decrement_options_stock` (`$p3`) to a value other than `(bool)true`, the observer sets the base product's quantity to be the sum of all the product's managed-variants' quantities.
4. Otherwise, the product-variant is POSM-managed and its stock is to be handled.  The variant's quantity is decremented and the base product's quantity is set to the sum of all the product's managed-variants' quantities.  If configured and indicated, a variant-specific low-stock e-mail is sent.

##### Replaced by

 `NOTIFY_POSM_ORDER_ADDED_PRODUCT_LINE_ITEM`, using the same parameters.

#### `POSM_GET_IN_STOCK_MESSAGE` (legacy)

Issued by the observer's `get_in_stock_message` method at various points in a product's rendering, conditionally adding an in-/out-of-stock message to the product's name.  This gives a watching observer the opportunity to further control the stock message's display.

##### Parameters

`$p1`: A read-only associative array containing the `prid` (containing the product's `uprid`) and the product's `pos_record`.  The `pos_record` value is `(bool)false` if the product is not POSM-managed, otherwise the value contains a `$db` return that has selected all fields from the product variant's `products_options_stock` table entry.

`$p2`: A reference to the `$message_override` variable, initialized to `(bool)false`. 

`$p3`: A reference to the `$use_in_stock_message` variable, initialized to `(bool)true`. 

`$p4`: A reference to the `$show_mixed_stock_messages` variable, initialized to `(bool)true`. 

Watching observers can control whether:

1. The overall in-/out-of-stock message handling should be bypassed, using the in- or out-of-stock message as  indicated.  The observer, in this case, sets the `$message_override `(`$p2`) to` (bool)true` and the `$use_in_stock_message` (`$p3`)  to identify whether the in-stock (`true`) or out-of-stock (otherwise) messaging should be used.
2. No mixed-stock messages should be displayed.  If the ordered quantity of a managed product is mixed, i.e. some  in-stock and some out, and the observer sets `$show_mixed_stock_messages` (`$p4`) to something _other than_ `(bool)tru`e  then the message for the product shows the out-of-stock message **only**.

##### Replaced by

 `NOTIFY_POSM_GET_IN_STOCK_MESSAGE`, using the same parameters.

### From `/includes/templates/YOUR_TEMPLATE/jscript/jscript_posm_dependencies`

### `.php`

#### `NOTIFY_POSM_DEPENDENCIES_ENABLE_OVERRIDE`

Indicates that POSM's jQuery processing for its dependent-attributes feature is preparing to include its `<script>`, giving an observer the opportunity to either opt-out by setting the reference to the `$posm_dependent_attrs_enable` variable to a value other than `(bool)true` or to force the handling to continue by setting the value to specifically `(bool)true`.

##### Parameters

`$p2`: A reference to the `$posm_dependent_attrs_enable` variable, initialized to `(bool)true` if POSM's `POSM_DEPENDENT_ATTRS_ENABLE` configuration is set to 'true'; `(bool)false` otherwise.

## Admin Notifications Issued

### From `/includes/classes/observers/class.products_options_stock_admin_observer.php`

#### `POSM_EO_ADD_PRODUCT_BYPASS` (legacy)

Indicates that the observer has received an `EDIT_ORDERS_ADD_PRODUCT` notification from the ***Edit Orders*** plugin.

##### Parameters

`$p1`: A read-only associative array containing the information that ***Edit Order***s supplied in the first parameter of its notification.

`$p2`: A reference to the `$bypass_add` variable, initialized to `(bool)false`.  If an observer sets this value to `(bool)true`, the POSM observer's processing of the product-addition is bypassed.

##### Replaced by

 `NOTIFY_POSM_EO_ADD_PRODUCT_BYPASS`, using the same parameters

#### `POSM_EO_PRODUCT_ADD_STOCK_UPDATE` (legacy)

Issued by the observer during its processing of an `EDIT_ORDERS_ADD_PRODUCT` notification from the ***Edit Orders*** plugin, so long as no observer has requested that the base processing be bypassed (see the notification above).

##### Parameters

`$p1`: A read-only associative array containing EO's `product` array for the current product and the product's `stock` information.  The `stock` value is `(bool)false` if the product is not POSM-managed, otherwise the value contains a `$db` return that has selected all fields from the product variant's `products_options_stock` table entry.

`$p2`: A reference to the `$bypass_managed_stock_update` variable, initialized to `(bool)false`.  If an observer sets this value to `(bool)true`, the stock-decrement for the base product/product-variant is not performed regardless of whether or not the product is POSM-managed.

##### Replaced by

 `NOTIFY_POSM_EO_PRODUCT_ADD_STOCK_UPDATE`, using the same parameters.

#### `POSM_EO_REMOVE_PRODUCT_QUANTITY_BYPASS` (legacy)

Issued by the observer during the removal of a product from an order, either requested by the Zen Cart admin's `orders.php` or via the ***Edit Orders*** plugin's processing.

##### Parameters

`$p1`: A read-only value identifying the `$orders_products_id` of the ordered product being removed from the order.

`$p2`: A reference to the `$bypass_quantity_updates` variable, initialized to `(bool)false`.  If an observer sets this value to any other value, neither the product variant's quantity nor the base product's overall quantity are updated.

##### Replaced by

 `NOTIFY_POSM_EO_REMOVE_PRODUCT_QUANTITY_BYPASS`, using the same parameters.

#### `POSM_GET_IN_STOCK_MESSAGE_BYPASS` (legacy)

Issued by the observer's `getInStockMessage` method at various points in a product's rendering, conditionally adding an in-/out-of-stock message to the product's name.  This gives a watching observer the opportunity to further control the stock message's display.

##### Parameters

`$p1`: A read-only associative array containing

- `op_info`.  A `$db` return from a query for the `products_id` and `products_quantity` of the current ordered product, i.e. this reflects the quantity ordered.
- `prod_info`. A `$db` return from a query for the `products_type` and `products_quantity` of the associated product, i.e. this reflects the currently stocked overall quantity of the product.
- `pos_info`.  Either `(bool)false`, if the product is not POSM-managed, or a `$db` return from a query for the current products `pos_id`, `products_quantity`, `pos_date` and `pos_name_id` associated with the managed-variant of the product.  This quantity reflects the variant's currently stocked quantity.

`$p2`: A reference to the `$message_text_override` variable, initialized to an empty string (`''`). 

`$p3`: A reference to the `$force_out_of_stock_message` variable, initialized to `(bool)false`. 

Watching observers can control whether:

1. A specific stock-related message is appended to the product's name by setting the `$message_text_override` value to a non-empty string.
2. Force, for POSM-managed products, POSM's out-of-stock message for out-of-stock products, i.e. no 'mixed-stock' messages will be used.

##### Replaced by

 `NOTIFY_POSM_GET_IN_STOCK_MESSAGE_BYPASS`, using the same parameters.

### From `/includes/functions/products_options_stock_admin_functions.php`

#### `POSM_MAIN_OPTION_INSERTED` (legacy)

Issued by the `insert_stock_option` function to indicate that a managed-variant for a product has been inserted into the database.

##### Parameters

`$p1`: A read-only associative array containing

- `pID`.  The product's id associated with the variant.
- `options_values_array`. An associative array, containing key/value pairs to identify each `options_id` and `options_values_id` associated with the variant.
- `quantity`.  The initial quantity value applied to the variant.
- `pos_id`.  The `pos_id` from the `products_options_stock` table that is associated with the variant.

##### Replaced by

 `NOTIFY_POSM_MAIN_OPTION_INSERTED`, using the same parameters.

### From `/products_options_stock.php`

#### `POSM_UPDATE_PREPARE_INPUTS` (legacy)

Issued at the beginning of any `update` processing, gives observers a chance to record their posted values and ensure that all inputs have been retrieved.

##### Parameters

`$p1`: The number of 'base' `quantity` inputs received from the form's posting.  An observer, for non-checkbox type inputs, must ensure that the count of each posted input array received matches this count.

`$p2`: A reference to the `$missing_inputs_flag` variable, initialized to a `(bool)false`.  Setting this flag to `(bool)true` will result in the update failing to complete with a POSM-set error message.

##### Replaced by

 `NOTIFY_POSM_UPDATE_PREPARE_INPUTS`, with additional parameters; see below.

#### `POSM_UPDATE_PREPARE_INPUTS2` (legacy)

Issued at the beginning of any `update` processing, gives observers a chance to record their posted values and ensure that all inputs have been retrieved.

##### Parameters

`$p1`: The number of 'base' `quantity` inputs received from the form's posting.  An observer, for non-checkbox type inputs,  must ensure that the count of each posted input array received matches this count.

`$p2`: A reference to the `$missing_inputs_flag` variable, initialized to a `(bool)false`.   Setting this flag to `(bool)true` will result in the update failing to complete.

`$p3`: A reference to the `$error` variable, initialized to a `(bool)false`.   Setting this flag to `(bool)true` will result in the update failing to complete; the observer is expected to have set a message in the `messageStack` identifying the source of the error.

`$p4`: A reference to the `$onload` variable, initialized to an empty string (`''`).   The variable contains script-related actions to be performed on the page body's load.

##### Replaced by

 `NOTIFY_POSM_UPDATE_PREPARE_INPUTS`, using the same parameters.

#### `POSM_UPDATE_DATABASE_RECORD` (legacy)

Issued just prior to updating a record in the `products_options_stock` table, giving observers the chance to include their fields in the update.  Observers watching this notification needed to declare `$update_posm_record_sql` as a global to make their additions.

##### Parameters

`$p1`: The `$pos_id` of the record being updated.

##### Replaced by

 `NOTIFY_POSM_UPDATE_DATABASE_RECORD`, using *different* parameters; see below.

#### `POSM_UPDATE_DATABASE_RECORD2` (legacy)

Issued just prior to updating a record in the `products_options_stock` table, giving observers the chance to include their fields in the update.

##### Parameters

`$p1`: The `$pos_id` of the record being updated.

`$p2`: A reference to the `$update_posm_record_sql` variable.

##### Replaced by

 `NOTIFY_POSM_UPDATE_DATABASE_RECORD`, using *different* parameters.

#### `NOTIFY_POSM_UPDATE_DATABASE_RECORD`

Issued just prior to updating a record in the `products_options_stock` table, giving observers the chance to include their fields in the update.  This version of the notification also enables observers to indicate that an error was detected in one of their custom fields, resulting in the database update being bypassed.

##### Parameters

`$p1`: The `$pos_id` of the record being updated.

`$p2`: A reference to the `$update_posm_record_sql` variable.

`$p3`: A reference to the `$error` variable, initialized to a `(bool)false`.   Setting this flag to `(bool)true` will result in the update failing to complete; the observer is expected to have set a message in the `messageStack` identifying the source of the error.

`$p4`: A reference to the `$onload` variable, initialized to an empty string (`''`).   The variable contains script-related actions to be performed on the page body's load.

#### `POSM_REMOVE_OPTIONS` (legacy)

Issued when a request is received to remove one or more managed-options from a product, enabling an observer to perform any custom actions based on the base product's current managed-status.

##### Parameters

`$p1`: The `products_id` of the product from which one or more managed options were removed.

##### Replaced by

 `NOTIFY_POSM_REMOVE_OPTIONS`, using the same parameters.

#### `POSM_INSERT_ADDITIONAL_CSS` (legacy)

Issued during the rendering of in-line CSS for the page's header; an observer was expected to directly echo any CSS additions.

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_INSERT_HEAD`, using different parameters and, possibly, methods; see below.

#### `POSM_END_OF_HTML_HEAD` (legacy)

Issued just prior to the `</head>` for the page; an observer was expected to directly echo any additional `<script>` tags required.

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_INSERT_HEAD`, using different parameters and, possibly, methods; see below.

#### `NOTIFY_POSM_INSERT_HEAD`

Issued just prior to the `</head>` for the page; an observer can identify additional CSS styles to be inserted into a `<style>` tag or javascript (**not jQuery**) to be inserted into a `<script>` tag.

1.  An alternative to the use of this notification is to include add-on specific CSS/JS content via separate files present in the admin's `/includes/css` and   `/includes/javascript` sub-directories, respectively.  Files' names to be loaded for this page should start with `products_options_stock_` for the files to be auto-loaded.
2. Since this javascript is loaded *prior to* the load of the jQuery base, any JS content must be **pure** javascript.  Any jQuery content required by the POSM extension for this page should be placed in the `/includes/javascript` sub-directory, as identified above.

##### Parameters

`$p2`: A reference to the `$css_content` string, set initially to an empty string.  If, on return, the value is not empty, the string is rendered in a `<style>` tag.

`$p3`: A reference to the `$js_content` string, set initially to an empty string.  If, on return, the value is not empty, the string is rendered in a `<script>` tag.

`$p4`: A reference to the `$onload` string.  The observer should append its onload actions to the pre-existing string.

#### `POSM_ADD_PRODUCT_OPTION` (legacy)

Issued during the creation of the array of the current product's managed variants (used for lower table's display), once for each option-combination.  Observers watching this notification needed to declare `$pos_product_options` and `$next_option` as a global to make their additions.

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_ADD_PRODUCT_OPTION`, using different parameters; see below.

#### `POSM_ADD_PRODUCT_OPTION2` (legacy)

Issued during the creation of the array of the current product's managed variants (used for lower table's display), once for each option-combination.  

##### Parameters

`$p1`:  An associative array of all `products_options_stock` table fields associated with the current option.  The `pos_id` element of the array defines the current element of the `$pos_product_options` array being processed.

`$p2`: A reference to the `$pos_product_options` array.  An observer is expected to add fields in the current `pos_id` element containing values in the `info_array` supplied to observers of the `NOTIFY_POSM_LOWER_CONTENT_INSERT_B4_QTY` and `NOTIFY_POSM_LOWER_CONTENT_INSERT_AFTER_QTY` notifications described below.

##### Replaced by

 `NOTIFY_POSM_ADD_PRODUCT_OPTION`, using the same parameters.

#### `POSM_START_HTML_OUTPUT` (legacy)

Issued prior to the in-page rendering, requesting that observers identify the number of columns of data that they will be adding to the upper and lower tables' display.

##### Parameters

`$p2`: Initialized to the count of columns that the POSM manager uses.  Each observer is expected to update this count by their respective number of added columns.  Upon return from this notification, the constant `STATIC_FIELD_COUNT` is defined with the returned value.

##### Replaced by

 `NOTIFY_POSM_START_HTML_OUTPUT`, using the same parameters.

#### `POSM_UPPER_HEADING_INSERT` (legacy)

Issued prior to the rendering of the table-heading for the upper form's ***Quantity*** input column.  Observers watching this notification were expected to directly echo `<td></td>` pairs to identify their additional column headings ... one column (or a collective `colspan`) for each additional column that they've registered in the `POSM_START_HTML_OUTPUT` notification!

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_UPPER_HEADING_INSERT`, using different parameters; see below.

#### `NOTIFY_POSM_UPPER_HEADING_INSERT`

Issued prior to the rendering of the data for the upper form's ***Quantity*** heading column.  Watching observers are expected to fill this array with additional heading elements to encompass the additional columns that were registered in the `POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p2`: A reference to the `$additional_content` array (initialized as an empty array). 

The `$additional_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column heading, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column-headings:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

#### `POSM_UPPER_CONTENT_INSERT` (legacy)

Issued prior to the rendering of the table-heading for the upper form's ***Quantity*** input column.  Observers watching this notification were expected to directly echo `<td></td>` pairs to identify their additional column data ... one column (or a collective `colspan`) for each additional column that they've registered in the `POSM_START_HTML_OUTPUT` notification!

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_UPPER_CONTENT_INSERT`, using different parameters; see below.

#### `NOTIFY_POSM_UPPER_CONTENT_INSERT`

Issued prior to the rendering of the element for the upper form's ***Quantity*** input column.  Watching observers are expected to fill this array with additional data to encompass the additional columns that were registered in the `NOTIFY_POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p2`: A reference to the `$additional_content` array (initialized as an empty array). 

The `$additional_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column elements:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

#### `POSM_SET_INSTRUCTIONS` (legacy)

Issued after the rendering of the tool's instructions.  Watching observers were expected to directly echo their HTML content.

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_SET_INSTRUCTIONS`, using different parameters; see below.

#### `NOTIFY_POSM_SET_INSTRUCTIONS`

Issued after the rendering of the tool's instructions, enabling a POSM extension to include its instructions on the screen.

##### Parameters

`$p2`: A reference to the `$additional_instructions` array (initialized as an empty array). 

The `$additional_instructions` array is a numerically-indexed array, with each array element containing string to be output as a single, full-width, table row.

#### `POSM_LOWER_HEADING_INSERT` (legacy)

Issued *prior to* the rendering of the table-heading for the lower form's ***Quantity*** input column.  Observers watching this notification were expected to directly echo `<td></td>` pairs to identify their additional column headings ... one column (or a collective `colspan`) for each additional column that they've registered in the `POSM_START_HTML_OUTPUT` notification!

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_LOWER_HEADING_INSERT_B4_QTY`, using different parameters; see below.

#### `POSM_LOWER_HEADING_INSERT2` (legacy)

Issued *prior to* the rendering of the table-heading for the lower form's ***Quantity*** input column.  Observers could 'split' their added columns to be either before (using this notification) or after (using the `POSM_LOWER_HEADING_INSERT3` notification) the base tool's ***Quantity*** column.

##### Parameters

`$p2`: A reference to the `$extra_headings` array (initialized as `(bool)false`). 

The `$extra_headings` value, if not `false`,  is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column heading, even if it's blank!
- `class`.  This *optional* entry identifies additional HTML `class` values to apply to the heading content.

##### Replaced by

 `NOTIFY_POSM_LOWER_HEADING_INSERT_B4_QTY`, using different parameters; see below.

#### `NOTIFY_POSM_LOWER_HEADING_INSERT_B4_QTY`

Issued *prior to* the rendering of the lower-form's ***Quantity*** column heading.   Observers can 'split' their added columns to be either before (using this notification) or after (using the `NOTIFY_POSM_LOWER_HEADING_INSERT_AFTER_QTY` notification) the base tool's ***Quantity*** column.  Between their response to the two notifications, watching observers are expected to fill this array with additional data elements to encompass the additional columns that were registered in the `NOTIFY_POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p2`: A reference to the `$extra_headings` array (initialized as an empty array). 

The `$extra_headings` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.
- `class`.  This *optional* entry identifies any additional extension-specific class(es) to be applied to the heading.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column-headings:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

#### `POSM_LOWER_HEADING_INSERT3` (legacy)

Issued *after* the rendering of the table-heading for the lower form's ***Quantity*** input column.   Observers could 'split' their added columns to be either after (using this notification) or before (using the `POSM_LOWER_HEADING_INSERT2` notification) the base tool's ***Quantity*** column.

##### Parameters

`$p2`: A reference to the `$extra_headings` variable (initialized as `(bool)false`). 

The `$extra_headings` variable, if not `false`, is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.

##### Replaced by

 `NOTIFY_POSM_LOWER_HEADING_INSERT_AFTER_QTY`, using different parameters; see below.

#### `NOTIFY_POSM_LOWER_HEADING_INSERT_AFTER_QTY`

Issued *after* the rendering of the lower form's ***Quantity*** column heading.  Observers can 'split' their added columns to be either after (using this notification) or before (using the `POSM_LOWER_HEADING_INSERT_B4_QTY` notification) the base tool's ***Quantity*** column.  Between their response to the two notifications, watching observers are expected to fill this array with additional data elements to encompass the additional columns that were registered in the `NOTIFY_POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p2`: A reference to the `$extra_headings` array (initialized as an empty array). 

The `$extra_headings` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.
- `class`.  This *optional* entry identifies any additional extension-specific class to be applied to the input.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column-headings:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

#### `POSM_SET_UPDATE_BUTTON_PARMS` (legacy)

Issued prior to rendering the lower form's ***Update*** button, enabling observers to identify additional HTML parameters to apply to form's ***Update*** buttons.  Watching observers were expected to declare the `$posm_update_button_parms` variable as `global` to make their changes.

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_SET_UPDATE_BUTTON_PARAMETERS`, using different parameters; see below.

#### `NOTIFY_POSM_SET_UPDATE_BUTTON_PARAMETERS`

Issued prior to rendering the lower form's ***Update*** button, enabling observers to identify additional HTML parameters to apply to form's ***Update*** buttons.

##### Parameters

`$p2`: A reference to the `$posm_update_button_parms` variable (initialized as an empty string), to which observers can append their required HTML parameters.

#### `POSM_LOWER_CONTENT_INSERT` (legacy)

Issued *prior to* the rendering of the lower form's ***Quantity*** input column.  Observers watching this notification were expected to directly echo `<td></td>` pairs to identify their additional column data ... one column (or a collective `colspan`) for each additional column that they've registered in the `POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p1`: An associative array identifying the `pos_id` associated with the current row as well as an `info_array` for the row, containing the data previously gathered via one of the `POSM_ADD_PRODUCT_OPTION`, `POSM_ADD_PRODUCT_OPTION2` or `NOTIFY_POSM_ADD_PRODUCT_OPTION` notifications. 

##### Replaced by

 `NOTIFY_POSM_LOWER_CONTENT_INSERT_B4_QTY`, using different parameters; see below.

#### `POSM_LOWER_CONTENT_INSERT2` (legacy)

Issued *prior to* the rendering of the lower form's ***Quantity*** input column.  Observers could 'split' their added columns to be either before (using this notification) or after (using the `POSM_LOWER_CONTENT_INSERT3` notification) the base tool's ***Quantity*** column.

##### Parameters

`$p1`: An associative array identifying the `pos_id` associated with the current row as well as an `info_array` for the row, containing the data previously gathered via one of the `POSM_ADD_PRODUCT_OPTION`, `POSM_ADD_PRODUCT_OPTION2` or `NOTIFY_POSM_ADD_PRODUCT_OPTION` notifications. 

`$p2`: A reference to the `$lower_content` array (initialized as `(bool)false`). 

The `$lower_content` value, if not `false`,  is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column heading, even if it's blank!
- `class`.  This *optional* entry identifies additional HTML `class` values to apply to the heading content.

##### Replaced by

 `NOTIFY_POSM_LOWER_HEADING_INSERT_B4_QTY`, using different parameters; see below.

#### `NOTIFY_POSM_LOWER_CONTENT_INSERT_B4_QTY`

Issued *prior to* the rendering of the lower form's ***Quantity*** input column.  Observers can 'split' their added columns to be either before (using this notification) or after (using the `NOTIFY_POSM_LOWER_CONTENT_INSERT_AFTER_QTY` notification) the base tool's ***Quantity*** column.  Between their response to the two notifications, watching observers are expected to fill this array with additional data elements to encompass the additional columns that were registered in the `NOTIFY_POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p1`: An associative array identifying the `pos_id` associated with the current row, the current tool's `action`, as well as the `info_array` for the row, containing the data previously gathered via the `NOTIFY_POSM_ADD_PRODUCT_OPTION` notification. 

`$p2`: A reference to the `$lower_content` array (initialized as an empty array). 

The `$lower_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column heading, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column elements:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

#### `POSM_LOWER_CONTENT_INSERT3` (legacy)

Issued *after* the rendering of the lower form's ***Quantity*** input column.   Observers could 'split' their added columns to be either after (using this notification) or before (using the `POSM_LOWER_CONTENT_INSERT2` notification) the base tool's ***Quantity*** column.

##### Parameters

`$p1`: An associative array identifying the `pos_id` associated with the current row as well as an `info_array` for the row, containing the data previously gathered via one of the `POSM_ADD_PRODUCT_OPTION`, `POSM_ADD_PRODUCT_OPTION2` or `NOTIFY_POSM_ADD_PRODUCT_OPTION` notifications. 

`$p2`: A reference to the `$lower_content` variable (initialized as `(bool)false`). 

The `$lower_content` variable, if not `false`, is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.

##### Replaced by

 `NOTIFY_POSM_LOWER_HEADING_INSERT_AFTER_QTY`, using different parameters; see below.

#### `NOTIFY_POSM_LOWER_CONTENT_INSERT_AFTER_QTY`

Issued *after* the rendering of the lower form's ***Quantity*** input column.   Observers can 'split' their added columns to be either after (using this notification) or before (using the `NOTIFY_POSM_LOWER_CONTENT_INSERT_B4_QTY` notification) the base tool's ***Quantity*** column.  Between their response to the two notifications, watching observers are expected to fill this array with additional data elements to encompass the additional columns that were registered in the `NOTIFY_POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p1`: An associative array identifying the `pos_id` associated with the current row, the current tool's `action`, as well as the `info_array` for the row, containing the data previously gathered via the `NOTIFY_POSM_ADD_PRODUCT_OPTION` notification. 

`$p2`: A reference to the `$lower_content` array (initialized as an empty array). 

The `$lower_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.
- `class`.  This *optional* entry identifies any additional extension-specific class to be applied to the input.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column-headings:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

### From `/products_options_stock_view_all.php`

#### `POSM_VIEW_ALL_UPDATE_INIT` (legacy)

Issued at the beginning of any `update` processing, gives observers a chance to record their posted values and ensure that all inputs have been retrieved.

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_UPDATE_INIT`, with the same parameters.

#### `POSM_VIEW_ALL_UPDATE` (legacy)

Issued just prior to updating a record in the `products_options_stock` table, giving observers the chance to include their fields in the update.  Observers watching this notification needed to declare `$error` and `$onload` as a global if they needed to disallow a database update based on their error-checking.

##### Parameters

`$p1`: The `$pos_id` of the record being updated.

`$p2`: A reference to the `$posm_sql_data` array with which the `products_options_stock` table record is to be updated.

`$p3`: A reference to the `$where_str` to be used when performing the update.

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_UPDATE`, using *different* parameters; see below.

#### `NOTIFY_POSM_VIEW_ALL_UPDATE`

Issued just prior to updating a record in the `products_options_stock` table, giving observers the chance to include their fields in the update.  This version of the notification also enables observers to indicate that an error was detected in one of their custom fields, resulting in the database update being bypassed.

##### Parameters

`$p1`: The `$pos_id` of the record being updated.

`$p2`: A reference to the `$posm_sql_data` array with which the `products_options_stock` table record is to be updated.

`$p3`: A reference to the `$where_str` to be used when performing the update.

`$p4`: A reference to the `$error` variable, initialized to a `(bool)false`.   Setting this flag to `(bool)true` will result in the update failing to complete; the observer is expected to have set a message in the `messageStack` identifying the source of the error.

`$p5`: A reference to the `$onload` variable, initialized to an empty string (`''`).   The variable contains script-related actions to be performed on the page body's load.

#### `NOTIFY_POSM_VIEW_ALL_UPDATE_PRODUCT`

Issued during database update processing, once for each base product whose managed options were updated.  Added for POSM v4.4.0.

##### Parameters

`$p1`: The `products_id` of the product from which one or more managed options were updated.

#### `POSM_VIEW_ALL_INSERT_HEAD` (legacy)

Issued just after the rendering of in-line CSS for the page's header; an observer was expected to directly echo any CSS additions.

##### Parameters

`$p2`: A reference to the `$onload` string, identifying any onload processing to be performed for the page.

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_INSERT_HEAD`, using different parameters and, possibly, methods; see below.

#### `NOTIFY_POSM_VIEW_ALL_INSERT_HEAD`

Issued just after the rendering of in-line CSS for the page's header; an observer can identify additional CSS styles to be inserted into a `<style>` tag or javascript (**not jQuery**) to be inserted into a `<script>` tag.

1.  An alternative to the use of this notification is to include add-on specific CSS/JS content via separate files present in the admin's `/includes/css` and   `/includes/javascript` sub-directories, respectively.  Files' names to be loaded for this page should start with `products_options_stock_view_all_`for the files to be auto-loaded.
2. Since this javascript is loaded *prior to* the load of the jQuery base, any JS content must be **pure** javascript.  Any jQuery content required by the POSM extension for this page should be placed in the `/includes/javascript` sub-directory, as identified above.

##### Parameters

`$p2`: A reference to the `$onload` string.  The observer should append its onload actions to the pre-existing string.

`$p3`: A reference to the `$css_content` string, set initially to an empty string.  If, on return, the value is not empty, the string is rendered in a `<style>` tag.

`$p4`: A reference to the `$js_content` string, set initially to an empty string.  If, on return, the value is not empty, the string is rendered in a `<script>` tag.

#### `POSM_VIEW_ALL_START_BODY` (legacy)

Issued prior to the in-page rendering, requesting that observers identify the number of columns of data that they will be adding to the upper and lower tables' display.

##### Parameters

`$p2`: Initialized to the count of columns that the POSM ***View All*** tool supplies.  Each observer is expected to update this count by their respective number of added columns.  Upon return from this notification, the constant `STATIC_FIELD_COUNT` is defined with the returned value.

`$p3`: A reference to the `$instructions2` string, initially set to the language constant `TEXT_POS_INSTRUCTIONS2`.  Observers should append their usage instructions to the input string; the resultant value is echoed between a `<p></p>` tag-pair.

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_START_BODY`, using the same parameters.

#### `POSM_VIEW_ALL_TABLE_HEADING` (legacy)

Issued prior to the rendering of the form's ***Quantity*** column heading.  Observers watching this notification were expected to directly echo `<td></td>` pairs to identify their additional column headings ... one column (or a collective `colspan`) for each additional column that they've registered in the `NOTIFY_POSM_VIEW_ALL_START_BODY` notification!

##### Parameters

None.

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_TABLE_HEADING`, using different parameters; see below.

#### `NOTIFY_POSM_VIEW_ALL_TABLE_HEADING`

Issued prior to the rendering of the form's ***Quantity*** column heading.  Watching observers are expected to fill this array with additional heading elements to encompass the additional columns that were registered in the `NOTIFY_POSM_VIEW_ALL_START_BODY` notification!

##### Parameters

`$p2`: A reference to the `$additional_content` array (initialized as an empty array). 

The `$additional_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column heading, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the heading content.

For example, an observer that has indicated that it supplies 2 additional columns should add two column-headings:

```php
$p2[] = ['text' => 'Column 1', 'align' => 'center'];
$p2[] = ['text' => 'Column 2'];
```

#### `POSM_VIEW_ALL_PRODUCTS_NAME` (legacy)

Issued prior to the rendering of each form-row that contains the current product's name (and link).   Observers could add strings (or form elements) to be prepended to that name and link.

##### Parameters

`$p1`: Identifies the `products_id` of the current product.

 `$p2`: A reference to the `$products_name_extra_info` string (initialized as an empty string).

 `$p3`: A reference to the `$products_name_additional_columns` string (initialized as `'<td colspan="' . (STATIC_FIELD_COUNT - 1) . '">&nbsp;</td>'`).

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_PRODUCTS_NAME`, using different parameters; see below.

#### `NOTIFY_POSM_VIEW_ALL_PRODUCTS_NAME`

Issued prior to the rendering of each form-row that contains a main product's name (and link).   Observers could add strings (or form elements) to be prepended to that name and link and (optionally) insert additional heading-columns into that row.

##### Parameters

`$p1`: Identifies the `products_id` of the current main product.

 `$p2`: A reference to the `$products_name_extra_info` string (initialized as an empty string).

 `$p3`: A reference to the `$additional_content` array (initialized as an empty array).

The `$additional_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column data, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `colspan="3"`.

For example, an observer that has indicated that it supplies 2 additional columns could either add two empty column elements:

```php
$p2[] = ['text' => '&nbsp;'];
$p2[] = ['text' => '&nbsp;'];
```

or the observer could simply identify a single column spanning 2 columns:

```php
$p2[] = ['text' => '&nbsp;', 'params' => 'colspan="2"']
```

#### `POSM_VIEW_ALL_INSERT_DATA` (legacy)

Issued *prior to* the rendering of the form's ***Quantity*** input column for the current product variant.  Observers watching this notification were expected to directly echo `<td></td>` pairs to identify their additional column data ... one column (or a collective `colspan`) for each additional column that they've registered in the `POSM_START_HTML_OUTPUT` notification!

##### Parameters

`$p1`: An associative array containing all `products_options_stock` database fields and values for the row's current `pos_id`. 

##### Replaced by

 `NOTIFY_POSM_VIEW_ALL_INSERT_DATA`, using different parameters; see below.

#### `NOTIFY_POSM_VIEW_ALL_INSERT_DATA`

Issued *prior to* the rendering of the form's ***Quantity*** input column for the current product variant.  Watching observers are expected to fill this array with additional form elements to encompass the additional columns that were registered in the `NOTIFY_POSM_VIEW_ALL_START_BODY` notification!

##### Parameters

`$p1`: An associative array containing all `products_options_stock` database fields and values for the row's current `pos_id`. 

`$p2`: A reference to the `$additional_content` array (initialized as an empty array). 

The `$additional_content` array is a numerically-indexed array, with each array element containing an associative array containing:

- `text`.  This **required** entry identifies the text to be associated with the column heading, even if it's blank!
- `align`.  This *optional* entry identifies the alignment (one of `center`, `right` or `left`) to apply to the content.
- `params`.  This *optional* entry identifies any additional parameters to apply to the column, e.g. `onclick="runme();"`.
- `class`.  This *optional* entry identifies any additional extension-specific class to be applied to the column.

For example, an observer that has indicated that it supplies 2 additional columns adds two empty column elements:

```php
$pos_id = $p1['pos_id'];
$p2[] = [
    'text' => zen_draw_input_field("pos_var1[$pos_id]", $var1),
    'align' => 'center',
];
$p2[] = [
    'text' => zen_draw_input_field("pos_var2[$pos_id]", $var2),
    'align' => 'center',
];
```

