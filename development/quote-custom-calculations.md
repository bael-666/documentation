# Custom calculations for Quote totals

## Problem

You have added custom fields to quote items or/and quote entity types. You want Total Amount field and other totals to be calculated considering new custom fields.

## Resolution

### Server-side calculation

You need to create custom repository for Quote entity type.

Create a new file:

`custom/Espo/Custom/Repositories/Quote.php`

```php
<?php

namespace Espo\Custom\Repositories;

class Quote extends \Espo\Modules\Sales\Repositories\Quote
{
    protected function calculateItems(\Espo\ORM\Entity $entity, array $options = array())
    {
        parent::calculateItems($entity, $options);

        $itemList = $entity->get('itemList');

        $amount = 0.0;		
        foreach ($itemList as $item) {
            $amount +=  $item->quantity * $item->unitPrice * $item->factor;
        }		
        $entity->set('amount', $amount);
    }
}

```

### Client-side calculation

In Quote's clientDefs you need to specify custom calculation handler:

File: `custom/Espo/Custom/Resources/metadata/clientDefs/Quote.json`

```json
{
    "calculationHandler": "custom:quote-calculation-handler"
}
```

Create a new file:

`client/custom/src/quote-calculation-handler.js`

```js
Espo.define('custom:quote-calculation-handler', ['sales:quote-calculation-handler'], function (Dep) {

    return Dep.extend({
	
        // Define custom calculations here.
        // Use client/modules/sales/quote-calculation-handler.js as an example.

    });
});

```

It's also possible to make certain product fields copied to a quote item on product selection. The logic is determined in selectProduct method.

