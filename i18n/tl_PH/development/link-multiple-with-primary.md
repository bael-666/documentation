# Ang Pag-uugnay ng Maramihang field sa Primarya

~Para sa EspoCRM 3.6.0 at higit pa.~

Minsan mayroong kang `hasMany` na kaugnayan at kailangan mong magkaroon ng kakayahan na pumili ng pangunahing rekord sa mga nauugnay. Bilang halimbawa, ang field ng `Contacts` Case entity.

>Kailangang gumawa ng `contacts` linkMultiple na field kasama ang isang primarya para sa ating custom entity na `Stock`. 
>

###Unang Hakbang
`custom/Espo/Custom/Resources/metadata/entityDefs/Stock.json`
```json
{
  "fields": {
     ...
	 "contacts": {
		"type": "linkMultiple",
		"view": "custom:views/stock/fields/contacts"
	 },
	 "contact": {
		"type": "link"
	 },
	 ...
  },
  "links": {
     ...
        "contact": {
            "type": "belongsTo",
            "entity": "Contact",
            "foreign": "stocksPrimary"
        },
        "contacts": {
            "type": "hasMany",
            "entity": "Contact",
            "foreign": "stocks",
            "layoutRelationshipsDisabled": true
        },
	 ...
  }
}
```

###Pangalawang Hakbang
`custom/Espo/Custom/Resources/metadata/entityDefs/Contact.json`
```json
{
  "links": {
     ...
        "stocksPrimary": {
            "type": "hasMany",
            "entity": "Stock",
            "foreign": "contact",
            "layoutRelationshipsDisabled": true
        },
        "stocks": {
            "type": "hasMany",
            "entity": "Stock",
            "foreign": "contacts"
        }
	 ...
  }
}
```

###Pangatlong Hakbang
`custom/Espo/Custom/Repositories/Stock.php`
```php
<?php

namespace Espo\Custom\Repositories;

class Stock extends \Espo\Core\ORM\Repositories\RDB
{
    public function afterSave(Entity $entity, array $options)
    {
        $result = parent::afterSave($entity, $options);
        $this->handleAfterSaveContacts($entity, $options);
        return $result;
    }
    protected function handleAfterSaveContacts(Entity $entity, array $options)
    {
        $contactIdChanged = $entity->has('contactId') && $entity->get('contactId') != $entity->getFetched('contactId');
        if ($contactIdChanged) {
            $contactId = $entity->get('contactId');
            if (empty($contactId)) {
                $this->unrelate($entity, 'contacts', $entity->getFetched('contactId'));
                return;
            }
        }
        if ($contactIdChanged) {
            $pdo = $this->getEntityManager()->getPDO();
            $sql = "
                SELECT id FROM contact_stock
                WHERE
                    contact_id = ".$pdo->quote($contactId)." AND
                    stock_id = ".$pdo->quote($entity->id)." AND
                    deleted = 0
            ";
            $sth = $pdo->prepare($sql);
            $sth->execute();
            if (!$sth->fetch()) {
                $this->relate($entity, 'contacts', $contactId);
            }
        }
    }
}
```

###Pang-apat na Hakbang
`client/custom/src/views/stock/fields/contacts.js`
```js
Espo.define('custom:views/stock/fields/contacts', 'views/fields/link-multiple-with-primary', function (Dep) {   
    return Dep.extend({
		primaryLink: 'contact'
    });
});
```

###Pang-limang Hakbang
Patakbuhin ang Rebuild

###Pang-anim na Hakbang
Isagawa ang SQL na query
```sql
UPDATE stock
JOIN contact_stock
ON contact_stock.stock_id = stock.id AND contact_stock.deleted = 0
SET stock.contact_id = contact_stock.contact_id
```

