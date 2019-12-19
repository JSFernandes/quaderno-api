# Receipts

A receipt is a detailed list of goods shipped or services rendered, with an account of all costs. It differs from invoices in a few key ways (mostly in that it does not require customer information in a legal sense). Read more [here](http://support.quaderno.io/article/37-what-is-a-sales-receipt).

## Create a receipt

> `POST /receipts.json`

```shell
# body.json
{
  "payment_method":"credit_card",
  "payment_processor":"stripe",
  "payment_processor_id":"ch_19yUdh2eZvKYlo2CkFVBOZG7",
  "contact_id":"5059bdbf2f412e0901000024",
  "contact_name":"STARK",
  "po_number":"",
  "currency":"USD",
  "tag_list":"playboy, businessman",
  "items_attributes":[
    {
      "description":"Whiskey",
      "quantity":"1.0",
      "unit_price":"20.0",
      "discount_rate":"0.0",
      "reference":"item_code_X"
    }
  ],
  "custom_metadata":{
    "a_custom_key":"a custom value"
  }
}

curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X POST \
     --data-binary @body.json \
     'https://ACCOUNT_NAME.quadernoapp.com/api/receipts.json'
```

```php?start_inline=1
$receipt = new QuadernoReceipt(array(
                                 'contact_id' => '5059bdbf2f412e0901000024',
                                 'payment_method' => 'credit_card',
                                 'payment_processor' => 'stripe',
                                 'payment_processor_id' => 'ch_19yUdh2eZvKYlo2CkFVBOZG7',
                                 'po_number' => '',
                                 'currency' => 'USD',
                                 'tag_list' => array('playboy', 'businessman'),
                                 'custom_metadata' => array('a_custom_key' => 'a custom value')));
$item = new QuadernoDocumentItem(array(
                               'description' => 'Pizza bagels',
                               'unit_price' => 9.99,
                               'quantity' => 20));
$contact = QuadernoContact::find('5059bdbf2f412e0901000024');

$receipt->addItem($item);
$receipt->addContact($contact);

$receipt->save(); // Returns true (success) or false (error)
```

```ruby
contact = Quaderno::Contact.find('50603e722f412e0435000024') #=> Quaderno::Contact

params = {
  contact_id: contact.id,
  po_number: '',
  payment_method: 'credit_card',
  payment_processor: 'stripe',
  payment_processor_id: 'ch_19yUdh2eZvKYlo2CkFVBOZG7',
  currency: 'USD',
  tag_list: ['playboy', 'businessman'],
  items_attributes: [
    {
      description: 'Whiskey',
      quantity: '1.0',
      unit_price: '20.0',
      discount_rate: '0.0'
    }
  ],
  custom_metadata: {
    a_custom_key: 'a custom value'
  }
}
receipt = Quaderno::Receipt.create(params) #=> Quaderno::Receipt
```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let params : [String: Any] = [
    "payment_method":"credit_card",
    "payment_processor": "stripe",
    "payment_processor_id": "ch_19yUdh2eZvKYlo2CkFVBOZG7",
    "contact_id":"5059bdbf2f412e0901000024",
    "contact_name":"STARK",
    "po_number":"",
    "currency":"USD",
    "tag_list":"playboy, businessman"
]

let createReceipt = Receipt.create(params)
client.request(createReceipt) { response in
    // response will contain the result of the request.
}
```

`POST`ing to `/receipts.json` will create a new receipt from the parameters passed.

This will return `201 Created` and the current JSON representation of the receipt if the creation was a success, along with the location of the new receipt in the `url` field.

### Attributes

Attribute       | Mandatory                                  | Type/Description
----------------|--------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
number          | no                                         | String(20 chars) Available for invoices, credit_notes, receipts and estimates. Automatic numbering is used by default. Validates uniqueness
issue_date      | no                                         | String(255 chars) Available for all except for `recurring`. Defaults to the current date. Format YYYY-MM-DD
po_number       | no                                         | String(255 chars)
due_date        | no                                         | String(255 chars) Available for invoices, expenses and credit notes. Format `YYYY-MM-DD`
currency        | no                                         | String(3 chars) Defaults to the account currency. En formato ISO 4217
tag_list        | no                                         | String(255 chars). Multiple tags should be separated by commas
payment_details | no                                         | Text
notes           | no                                         | Text
contact_id      | (Mandatory if `contact` is not present)    | ID
contact         | (Mandatory if `contact_id` is not present) | Hash with a contact data for creation
street_line_1   | no                                         | String(255 chars). Available for updates
street_line_2   | no                                         | String(255 chars)
city            | no                                         | String(255 chars). Available for updates
region          | no                                         | String(255 chars). Available for updates
postal_code     | no                                         | String(255 chars). Available for updates
items_attributes| **yes**                                    | Array of document items (check available attributes for document items below). No more than 200 items are allowed in a request. To add more use subsequent update requests. Maximum items per document are limited up to 1000 items.
payment_method  | **yes**                                    | One of the following: `credit_card`, `cash`, `wire_transfer`, `direct_debit`, `check`, `promissory_note`, `iou`, `paypal` or `other`
payment_processor  | no                                      | Payment processor that you used to process the payment.
payment_processor_id  | no                                   | The `id` that the payment processor assigned to the payment.
custom_metadata | no                                         | Key-value data. You can have up to 20 keys, with key names up to 40 characters long and values up to 500 characters long.

<aside class="notice">
If you pass a `contact` JSON object instead of a `contact_id`, and the email or first and last name combination does not match any of your existing contacts, a new one will be created, otherwise a new receipt will be created for the existing contact. Only a `contact` object OR a `contact_id` property should be passed in the same call.<br /><br />

<p>Please note that you can pass <strong>either</strong> contact_id or contact, but if you pass both then results may not be what you expect.</p>

<p>Allowed contact fields are the same as when creating a contact via the dedicated <a href="#contacts">Contacts API</a>.</p>
</aside>

<aside class="warning">
You can't create a receipt for a contact with a tax ID. If the contact has a tax ID a regular (full) invoice should be created.
</aside>

### Document Item Attributes

Attribute     | Mandatory                                | Type/Description
--------------|------------------------------------------|----------------------------------------------------------------------------------------------------------------------
id            | no                                       | ID. Available only for updates
description   | **yes**                                  | String(255 chars)
quantity      | no                                       | Decimal. Defaults to 1.0
unit_price    | **yes**                                  | Decimal
total_amount  | Mandatory if `unit_price` is not present | Decimal
discount_rate | no                                       | Decimal
tax_1_name    | Mandatory if `tax_1_rate` is present     | String(255 chars)
tax_1_rate    | Mandatory if `tax_1_name` is present     | Decimal between -100.00 and 100.00 (not included)
tax_1_country | no                                       | String(2 chars). Defaults to the contact's country
tax_1_region  | no                                       | String(255 chars). Recommendable to set for Canadian or United States taxes.
tax_1_county  | no                                       | String(2 chars). Recommendable to set for Canadian or United States taxes.
tax_1_transaction_type | no                              | String. Accepts `eservice`, `ebook` or `standard`. Defaults to your account "default tax type".
tax_2_name    | Mandatory if `tax_2_rate` is present     | String(255 chars)
tax_2_rate    | Mandatory if `tax_2_name` is present     | Decimal between -100.00 and 100.00 (not included)
tax_2_country | no                                       | String(2 chars). Defaults to the contact's country
tax_2_region  | no                                       | String(255 chars). Recommendable to set for Canadian or United States taxes.
tax_2_county  | no                                       | String(2 chars). Recommendable to set for Canadian or United States taxes.
tax_2_transaction_type | no                              | String. Accepts `eservice`, `ebook` or `standard`. Defaults to your account "default tax type".
reference     | no                                       | String(255 chars) Code (`code`) of an existing item. **If present none of the mandatory attributes are mandatory (as you are referencing an item that already exists)**
_destroy      | no                                       | Set it to 1 if you want to remove the document item selected by ID. Available only for updates

### Receipt States

Possible receipt states are:

- `paid`: Payment received (default state).
- `refunded`: Payment refunded and a credit note has been issued. 
- `invoiced`: The receipt has beend converted into a full invoice (*cannot be undone*).

## Retrieve: Get and filter all receipts

> `GET /receipts.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/receipts.json'
```

```ruby
Quaderno::Receipt.all() #=> Array
```

```php?start_inline=1
$receipts = QuadernoReceipt::find(); // Returns an array of QuadernoReceipt
```

```swift
let client = Quaderno.Client(/* ... */)

let listReceipts = Receipt.list(pageNum)
client.request(listReceipts) { response in
  // response will contain the result of the request.
}
```

```json
[
  {
    "id":"507693322f412e0e2e00000f",
    "number":"0000047",
    "issue_date":"2012-10-11",
    "created_at":"1325376000",
    "contact":{
      "id":"5073f9c22f412e02d0000032",
      "full_name":"Garfield"
    },
    "country":"US",
    "street_line_1":"Street 23",
    "street_line_2":"",
    "city":"New York",
    "region":"New York",
    "postal_code":"10203",
    "po_number":null,
    "currency":"USD",
    "exchange_rate":"0.680309",
    "items":[
      {
        "id":"48151623429",
        "description":"lasagna",
        "quantity":"25.0",
        "unit_price_cents":"375",
        "discount_rate":"0.0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "reference":"Awesome!",
        "subtotal_cents":"9375",
        "discount_cents":"0",
        "gross_amount_cents":"9375"
      }
    ],
    "subtotal_cents":"9375",
    "discount_cents":"0",
    "taxes":[],
    "total_cents":"9375",
    "payments":[
      {
        "id":"50aca7d92f412eda5200002c",
        "date":"2012-11-21",
        "payment_method":"credit_card",
        "payment_processor":"stripe",
        "payment_processor_id":"ch_19yUdh2eZvKYlo2CkFVBOZG7",
        "amount_cents":"9375",
      },
    ],
    "notes":"",
    "state":"paid",
    "tag_list":["lasagna", "cat"],
    "secure_id":"7hef1rs7p3rm4l1nk",
    "permalink":"https://quadernoapp.com/receipt/7hef1rs7p3rm4l1nk",
    "pdf":"https://quadernoapp.com/receipt/7hef1rs7p3rm4l1nk.pdf",
    "url":"https://ACCOUNT_NAME.quadernoapp.com/api/receipts/507693322f412e0e2e00000f.json",
    "custom_metadata":{}
  },

  {
    "id":"507693322f412e0e2e0000da",
    "number":"0000047",
    "issue_date":"2012-10-11",
    "created_at":"1325376020",
    "contact":{
      "id":"5073f9c22f412e02d00004cf",
      "full_name":"Teenage Mutant Ninja Turtles"
    },
    "country":"US",
    "street_line_1":"Melrose Ave, Sewer #3",
    "street_line_2":"",
    "city":"New York",
    "region":"New York",
    "postal_code":"",
    "po_number":null,
    "currency":"USD",
    "exchange_rate":"0.680309",
    "items":[
      {
        "id":"481516234291",
        "description":"pizza",
        "quantity":"1.0",
        "unit_price_cents":"6000",
        "discount_rate":"0",
        "tax_1_name":"",
        "tax_1_rate":"",
        "tax_2_name":"",
        "tax_2_rate":"",
        "reference":"Even the bad ones taste good!",
        "subtotal_cents":"6000",
        "discount_cents":"0",
        "gross_amount_cents":"6000"
      }
    ],
    "subtotal_cents":"6000",
    "discount_cents":"0",
    "taxes":[],
    "total_cents":"6000",
    "payments":[
      {
        "id":"50aca7d92f412eda5200002ssdc",
        "date":"2012-11-21",
        "payment_method":"credit_card",
        "payment_processor":"stripe",
        "payment_processor_id":"ch_19yUdh2eZvKYlo2CkFVBOZG7",
        "amount_cents":"6000",
      },
    ],
    "notes":"",
    "state":"outstanding",
    "tag_list":["pizza", "turtles"],
    "secure_id":"7hes3c0ndp3rm4l1nk",
    "permalink":"https://ACCOUNT_NAME.quadernoapp.com/receipt/7hes3c0ndp3rm4l1nk",
    "pdf":"https://ACCOUNT_NAME.quadernoapp.com/receipt/7hes3c0ndp3rm4l1nk.pdf",
    "url":"https://ACCOUNT_NAME.quadernoapp.com/api/receipts/507693322f412e0e2e0000da.json",
    "custom_metadata":{}
  }
]
```

`GET`ting from `/receipts.json` will return all the user's receipts.

You can filter the results in a few ways:

- By `number`, `contact_name` or `po_number` by passing the `q` parameter in the url like `?q=KEYWORD`.
- By date range, passing the `date` parameter in the url like `?date=DATE1,DATE2`. Date range allows these formats: `2019-01-01,2019-12-31` or `2019/01/01,2019/12/31`.
- By state, passing the `state` parameter like `?state=STATE`.
- By contact, passing the contact ID in the `contact` parameter, like `?contact=3231`.

## Retrieve: Get a single receipt

> `GET /receipts/RECEIPT_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/receipts/RECEIPT_ID.json'
```

```ruby
Quaderno::Receipt.find(RECEIPT_ID) #=> Quaderno::Receipt
```

```php?start_inline=1
$receipt = QuadernoReceipt::find('RECEIPT_ID'); // Returns a QuadernoReceipt
```

```swift
let client = Quaderno.Client(/* ... */)

let readReceipt = Receipt.read(RECEIPT_ID)
client.request(readReceipt) { response in
  // response will contain the result of the request.
}
```

```json
{
  "id":13638223434,
  "number":"0000001",
  "issue_date":"2016-05-05",
  "created_at":1462453039,
  "contact":{
    "id":424230933,
    "full_name":"Toxic Crusader"
  },
  "country":"US",
  "street_line_1":null,
  "street_line_2":null,
  "city":null,
  "region":null,
  "postal_code":null,
  "po_number":null,
  "currency":"EUR",
  "items":[{
    "id":2621550,
    "description":"Mop",
    "quantity":"1.0",
    "unit_price":"30.0",
    "discount_rate":"0.0",
    "tax_1_name":"",
    "tax_1_rate":null,
    "tax_2_name":"",
    "tax_2_rate":null,
    "reference":null,
    "subtotal_cents":"3000",
    "discount_cents":"0",
    "gross_amount_cents":"3000"
  }],
  "subtotal_cents":"3000",
  "discount_cents":"0",
  "taxes":[],
  "total_cents":"3000",
  "payments":[{
    "id":972444,
    "date":"2016-05-05",
    "payment_method":"credit_card",
    "payment_processor":"stripe",
    "payment_processor_id":"ch_19yUdh2eZvKYlo2CkFVBOZG7",
    "amount_cents":"3000"
  }],
  "notes":null,
  "state":"paid",
  "tag_list":[],
  "secure_id":"f6a78e399aa6369e3b0329da78cc24534bc1934d",
  "permalink":
  "http://ACCOUNT_NAME.quadernoapp.com/receipt/f6a78e399aa6369e3b0329da78cc24534bc1934dzzRot",
  "pdf":"https://ACCOUNT_NAME.quadernoapp.com/receipt/f6a78e399aa6369e3b0329da78cc24534bc1934dzzRot.pdf",
  "url":"https://ACCOUNT_NAME.quadernoapp.com/api/receipts/13638223434.json",
  "custom_metadata":{}
}
```

`GET`ting from `/receipts/RECEIPT_ID.json` will return that specific receipt.

If you have connected Quaderno and Stripe, you can also `GET /stripe/charges/STRIPE_CHARGE_ID.json` to get the Quaderno receipt for a Stripe charge.

## Update a receipt

> `PUT /receipts/RECEIPT_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X PUT \
     -d '{"notes":"You better pay this time, Tony.", "custom_metadata":{"memo":"I think he is not paying again."}}' \
     'https://ACCOUNT_NAME.quadernoapp.com/api/receipts/RECEIPT_ID.json'
```

```ruby
params = {
    notes: 'You better pay this time, Tony.',
    custom_metadata: {
        memo: 'I think he is not paying again.'
    }
}
Quaderno::Receipt.update(RECEIPT_ID, params) #=> Quaderno::Receipt
```

```php?start_inline=1
$receipt->notes = 'You better pay this time, Tony.';
$receipt->custom_metadata = array('memo' => 'I think he is not paying again.');
$receipt->save();
```

````swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let params : [String: Any] = [
    "notes": "You better pay this time, Tony."
]

let updateReceipt = Receipt.update(RECEIPT_ID, params)
client.request(updateReceipt) { response in
    // response will contain the result of the request.
}
```

`PUT`ting to `/receipts/RECEIPT_ID.json` will update the receipt with the passed parameters.

This will return `200 OK` along with the current JSON representation of the receipt if successful.

## Delete a receipt

> `DELETE /receipts/RECEIPT_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X DELETE 'https://ACCOUNT_NAME.quadernoapp.com/api/receipts/RECEIPT_ID.json'
```

```ruby
Quaderno::Receipt.delete(RECEIPT_ID) #=> Boolean
```

```php?start_inline=1
$receipt->delete();
```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let deleteReceipt = Receipt.delete(INVOICE_ID)
client.request(deleteReceipt) { response in
    // response will contain the result of the request.
}
```

`DELETE`ing to `/receipt/RECEIPT_ID.json` will delete the specified receipt and returns `204 No Content` if successful.

## Deliver (Send) a receipt

```shell
curl -u YOUR_API_KEY: \
     -X GET \
     'https://ACCOUNT_NAME.quadernoapp.com/api/invoices/INVOICE_ID/deliver.json'
```

```ruby
receipt = Quaderno::Receipt.find(RECEIPT_ID)
receipt.deliver
```

```php?start_inline=1
$receipt->deliver(); // Return true (success) or false (error)
```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let deliverReceipt = Receipt.deliver(INVOICE_ID)
client.request(deliverReceipt) { response in
  // response will contain the result of the request.
}
```

`GET`ting `/receipts/RECEIPT_ID/deliver.json` will send the receipt to the assigned contact email. This will return `200 OK` if successful, along with a JSON representation of the receipt.

If the destination contact does not have an email address you will receive a `422 Unprocessable Entity` error.
