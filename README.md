Welcome to the support area for the Products' Options' Stock Manager plugin from Vinos de Frutas Tropicales, available here: http://vinosdefrutastropicales.com/index.php?main_page=product_info&cPath=2_7&products_id=46.

# Updating Products Options Stock Manager to v4.0.0

2018 brought us Zen Cart v1.5.6 and _POSM_ v4.0.0 provides that integration.  If you've purchased an earlier version of _POSM_, this document identifies the changes you'll need to make in your update to v4.0.0.

## Background
Previous versions of _POSM_ provided multiple versions (`YOUR_TEMPLATE` and `YOUR_RC_CLONE`) of three template files in the storefront:
1. `tpl_account_history_info_default.php`
1. `tpl_checkout_confirmation_default.php`
1. `tpl_shopping_cart_default.php`

Those updates insert the _POSM_ in-/out-of-stock indications for customer guidance.  Unfortunately, each of those modules have been updated for Zen Cart 1.5.6.  As such, _POSM_ v4.0.0 and later takes a different approach to insert those stock-indications and future-proof its changes.

Similarly, previous versions of _POSM_ included modifications to the **core** admin `invoice.php` and `packingslip.php` modules.  Those changes, too, are now provided via Zen Cart `notifier/observer` methods to reduce the number of core-file overwrites required.

## File-Specific Changes

These changes should be made to _pre-existing_ copies of the modules identified in the following sections.

### Admin Changes

For `POSM` v4.0.0 and later, the stock indicators for the `invoice` and `packingslip` and the icon in the admin's **Categories->Categories/Products listing** are now introduced by the plugin's admin-level observer.

#### /admin/invoice.php

This module no longer has any `POSM` required changes, so each of the 3 previous additions should be removed.

1. Find the following section:

```php
<?php //-bof-options_stock-lat9  *** 1 of 3 *** ?>
<style type="text/css">
<!--
.odd { background-color: white; }
.attrComments { max-width: 97%; }
-->
</style>
<script type="text/javascript" src="includes/menu.js"></script>
<script type="text/javascript"><!--
<?php //-eof-options_stock-lat9  *** 1 of 3 ?>
```

... and remove the `POSM`-specific change:

```php
<script type="text/javascript" src="includes/menu.js"></script>
<script type="text/javascript"><!--
```

2. Find the following section:

```php
    for ($i = 0, $n = sizeof($order->products), $evenodd = 'even'; $i < $n; $i++) {  //-options_stock-lat9  *** 2 of 3 ***
```

... and remove the `POSM`-specific change:

```php
    for ($i = 0, $n = sizeof($order->products); $i < $n; $i++) {
```

3. Find the following section:

```php
//-bof-options_stock-lat9  *** 3 of 3 ***
      echo '      <tr class="dataTableRow ' . $evenodd . '">' . "\n" .
           '        <td class="dataTableContent" valign="top" align="right">' . $order->products[$i]['qty'] . '&nbsp;x</td>' . "\n" .
           '        <td class="dataTableContent" valign="top" align="left">' . pos_extract_stock_type ($order->products[$i]['name']);
//-eof-options_stock-lat9  *** 3 of 3 ***
```

... and remove the `POSM`-specific change:

```php
      echo '      <tr class="dataTableRow">' . "\n" .
           '        <td class="dataTableContent" valign="top" align="right">' . $order->products[$i]['qty'] . '&nbsp;x</td>' . "\n" .
           '        <td class="dataTableContent" valign="top" align="left">' . $order->products[$i]['name'];
```

#### /admin/packingslip.php

This module no longer has any `POSM` required changes, so the previous changes should be removed.

Find the following section:

```php
//-bof-options_stock-lat9  *** 1 of 1 ***
      echo '      <tr class="dataTableRow">' . "\n" .
           '        <td class="dataTableContent" valign="top" align="right">' . $order->products[$i]['qty'] . '&nbsp;x</td>' . "\n" .
           '        <td class="dataTableContent" valign="top" style="max-width: 97%;">' . pos_extract_stock_type ($order->products[$i]['name']) . '<span class="smallText">';
      if (isset($order->products[$i]['attributes']) && (sizeof($order->products[$i]['attributes']) > 0)) {
        for ($j=0, $k=sizeof($order->products[$i]['attributes']); $j<$k; $j++) {
          echo '<br>&nbsp;<i> - ' . $order->products[$i]['attributes'][$j]['option'] . ': ' . nl2br(zen_output_string_protected($order->products[$i]['attributes'][$j]['value']));
          echo '</i>';
        }
      }

      echo '        </span></td>' . "\n" .
           '        <td class="dataTableContent" valign="top">' . $order->products[$i]['model'] . '</td>' . "\n" .
           '      </tr>' . "\n";
    }
//-eof-options_stock-lat9  *** 1 of 1 ***
```

... and remove the `POSM`-specific change:

```php
      echo '      <tr class="dataTableRow">' . "\n" .
           '        <td class="dataTableContent" valign="top" align="right">' . $order->products[$i]['qty'] . '&nbsp;x</td>' . "\n" .
           '        <td class="dataTableContent" valign="top">' . $order->products[$i]['name'];

      if (isset($order->products[$i]['attributes']) && (sizeof($order->products[$i]['attributes']) > 0)) {
        for ($j=0, $k=sizeof($order->products[$i]['attributes']); $j<$k; $j++) {
          echo '<br><nobr><small>&nbsp;<i> - ' . $order->products[$i]['attributes'][$j]['option'] . ': ' . nl2br(zen_output_string_protected($order->products[$i]['attributes'][$j]['value']));
          echo '</i></small></nobr>';
        }
      }

      echo '        </td>' . "\n" .
           '        <td class="dataTableContent" valign="top">' . $order->products[$i]['model'] . '</td>' . "\n" .
           '      </tr>' . "\n";
    }
```

#### /admin/includes/modules/category_product_listing.php

**Note**: For zc156 and later, this module has been moved to `/admin/category_product_listing.php`.

Find the following section:

```php
<?php
//-bof-products_options_stock-lat9  *** 1 of 1 ***
?>
                <td class="dataTableContent" width="80" align="left">
<?php
      if (is_pos_product ($products->fields['products_id'])) {
        echo '<a href="' . zen_href_link (FILENAME_PRODUCTS_OPTIONS_STOCK, 'pID=' . $products->fields['products_id']) . '">' . zen_image (DIR_WS_IMAGES . 'icon_blue_on.gif', POS_ALT_PRODUCT_HAS_OPTIONS_STOCK) . '</a>&nbsp;&nbsp;';
        
      } else {
        echo zen_image (DIR_WS_IMAGES . 'pixel_trans.gif', '', 16, 16) . '&nbsp;&nbsp;';
        
      }
//-eof-products_options_stock-lat9  *** 1 of 1 ***
```

and replace it with:

```php
<?php
//-bof-products_options_stock-lat9  *** 1 of 1 ***
?>
                <td class="dataTableContent" width="80" align="left">
<?php
                      $additional_icons = '';
                      $zco_notifier->notify('NOTIFY_ADMIN_PROD_LISTING_ADD_ICON', $product, $additional_icons);
                      echo $additional_icons;
//-eof-products_options_stock-lat9  *** 1 of 1 ***
?>
```

### Storefront Changes

For `POSM` v4.0.0 and later, the stock indications are now added by `header_php_xxx.php` files for the specific page.

#### tpl_account_history_info_default.php

Find the following section:

```php
//-bof-20140327-lat9-products_options_stock  *** 1 of 1 ***
    $product_timeframe = '';
    if ($posObserver->enabled) {
      if (preg_match ('/(.*)\[(.*)\]$/', $order->products[$i]['name'], $matches)) {
        $order->products[$i]['name'] = $matches[1];
        if ($posObserver->show_stock_messages) {
          $msg = $matches[2];
          $extra_msg = '';
          if ($msg == PRODUCTS_OPTIONS_STOCK_IN_STOCK) {
            $extra_class = 'in-stock';
            
          } elseif (strpos ($msg, ', ') === false) {
            $extra_class = 'no-stock';
          
          } else {
            $extra_class = 'in-stock';
            $message_parts = explode (', ', $msg);
            $msg = $message_parts[0];
            $extra_msg = ' ' . sprintf (PRODUCTS_OPTIONS_STOCK_STOCK_HTML, 'no-stock', $message_parts[1]);
            
          }
          $msg = sprintf (PRODUCTS_OPTIONS_STOCK_STOCK_HTML, $extra_class, $msg) . $extra_msg;
          $product_timeframe = '<br />' . sprintf (PRODUCTS_OPTIONS_STOCK_WRAPPER, $msg);
          
        }
      }
    }
    echo  $order->products[$i]['name'] . $product_timeframe;
//-eof-20140327-lat9-products_options_stock  *** 1 of 1 ***
```

... and remove the `POSM`-specific changes:

```php
    echo  $order->products[$i]['name'];
```

#### tpl_checkout_confirmation_default.php

Find the following section:

```php
<?php
      } // end loop
      echo '</ul>';
    } // endif attribute-info
    
    echo '<br />' . $posStockMessage[$i];  //-lat9-20140314-Add products' options' stock message  *** 1 of 1 ***
?>
```

... and remove the `POSM`-specific change:

```php
<?php
      } // end loop
      echo '</ul>';
    } // endif attribute-info
?>
```

#### tpl_shopping_cart_default.php

Find the following section:

```php
<?php
  echo $product['attributeHiddenField'];
  if (isset($product['attributes']) && is_array($product['attributes'])) {
  echo '<div class="cartAttribsList">';
  echo '<ul>';
    reset($product['attributes']);
    foreach ($product['attributes'] as $option => $value) {
?>

<li><?php echo $value['products_options_name'] . TEXT_OPTION_DIVIDER . nl2br($value['products_options_values_name']); ?></li>

<?php
    }
  echo '</ul>';
  echo '</div>';
  }
  
  echo $product['posStockMessage'];  //-lat9-20140314-Output products' options' stock message  *** 1 of 1 ***
?>
```

... and remove the `POSM`-specific change:

```php
<?php
  echo $product['attributeHiddenField'];
  if (isset($product['attributes']) && is_array($product['attributes'])) {
  echo '<div class="cartAttribsList">';
  echo '<ul>';
    reset($product['attributes']);
    foreach ($product['attributes'] as $option => $value) {
?>

<li><?php echo $value['products_options_name'] . TEXT_OPTION_DIVIDER . nl2br($value['products_options_values_name']); ?></li>

<?php
    }
  echo '</ul>';
  echo '</div>';
  }
?>
```

