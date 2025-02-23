# Payment Method

## Payment Method Resource

The `Payment Method` resource represents the payment methods associated with a customer `Account`. There are two parts to the state associated with this resource, a first generic set of attributes kept by Kill Bill core subsystem, and another set of attributes kept at the (payment) plugin level.

* The core Kill Bill attributes shown below mostly track the associated payment plugin that is used to interact with the payment gateway.
* The plugin attributes are typically the details about such customer payment method: In the case of a credit card for instance, the plugin would keep track of things like `name`, `address`, `last4`, and `token`. Not only are such attributes dependent on the payment method, but they are also dependent on the third party payment gateway, and on the tokenization model, which is why they are kept by the plugin (internal tables), and not by the Kill Bill core payment subsystem.


Kill Bill also supports a more advanced use case for payment routing, where the choice of the payment gateway is decided at run time
based on custom business rules. Additional information can be found in our [Payment Manual](http://docs.killbill.io/latest/userguide_payment.html).


The Kill Bill attributes are the following:

| Name | Type | Generated by | Description |
| ---- | -----| -------- | ------------ |
| **paymentMethodId** | string | system | UUID for this payment method |
| **externalKey** | string | user | Optional external key provided by the client |
| **accountId** | string | system | UUID for the associated account |
| **isDefault** | boolean | user | Indicates whether this is the default payment method |
| **pluginName** | string | user | Name of the associated plugin. 
| **pluginInfo** | string | user | Plugin specific information, as required. |

All payment operations associated with this payment method will be delegated to the plugin specified by **pluginName**.


## Payment Methods

Basic operations to retrieve, list, search and delete payment methods.

Note that the creation of a payment method needs to be done at the Account level using the [Add a payment method](https://killbill.github.io/slate/#account-add-a-payment-method) endpoint. The `pluginName` attribute identifies the payment plugin that will be used by the system when this payment method is used for a payment. 

### Retrieve a payment method by id

Retrieve information on a payment method, given its payment method ID.

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}`

> Example Request:

```shell
curl \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Accept: application/json' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("3c449da6-7ec4-4c74-813f-f5055739a0b9");
Boolean includedDeleted = false; // Will not include deleted
Boolean withPluginInfo = true; // Will include plugin info
Map<String, String> NULL_PLUGIN_PROPERTIES = null;

PaymentMethod paymentMethodJson = paymentMethodApi.getPaymentMethod(paymentMethodId, 
                                                                    includedDeleted, 
                                                                    withPluginInfo, 
                                                                    NULL_PLUGIN_PROPERTIES, 
                                                                    AuditLevel.NONE, 
                                                                    requestOptions);
```

```ruby
payment_method_id = "6a0bf13e-d57f-4f79-84bd-3690135f1923"
with_plugin_info = false

KillBillClient::Model::PaymentMethod.find_by_id(payment_method_id, 
                                                with_plugin_info, 
                                                options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
payment_method_id = '0052cddd-0f61-4f68-b653-ca49b5d7f915'

paymentMethodApi.get_payment_method(payment_method_id, api_key, api_secret)
```

> Example Response:

```json
{
    "paymentMethodId":"916619a4-02bb-4d3d-b3da-2584ac897b19",
    "externalKey":"coolPaymentMethod",
    "accountId":"84c7e0d4-a5ed-405f-a655-3ed16ae19997",
    "isDefault":false,
    "pluginName":"__EXTERNAL_PAYMENT__",
    "pluginInfo":null,
    "auditLogs":[]
}
```


**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- |
| **includedDeleted** | boolean | no | false | If true, include deleted payment methods |
| **withPluginInfo** | boolean | no | false | If true, include plugin info |
| **audit** | string | no | "NONE" | Level of audit information to return |

If **withPluginInfo** is set to true, attributes for the active plugin are returned. These are plugin-dependent (see discussion above).
Audit information options are "NONE", "MINIMAL" (only inserts), or "FULL".


**Response**

If successful, returns a status code of 200 and a payment method resource object.

### Retrieve a payment method by external key

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/paymentMethods`

> Example Request:

```shell
curl \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Accept: application/json' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19?externalKey=coolPaymentMethod' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

String externalKey = "foo";
Boolean includedDeleted = false; // Will not include deleted
Boolean withPluginInfo = false; // Will not reflect plugin info
Map<String, String> NULL_PLUGIN_PROPERTIES = null;

PaymentMethod paymentMethod = paymentMethodApi.getPaymentMethodByKey(externalKey, 
                                                                     includedDeleted,
                                                                     withPluginInfo,
                                                                     NULL_PLUGIN_PROPERTIES, 
                                                                     AuditLevel.NONE,
                                                                     requestOptions);
```

```ruby
payment_method_ek = "sample_external_key"
included_deleted = false
with_plugin_info = false
audit = 'NONE'

KillBillClient::Model::PaymentMethod.find_by_external_key(payment_method_ek,
                                                          included_deleted,
                                                          with_plugin_info,
                                                          audit,
                                                          options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
external_key = 'sample_external_key'

paymentMethodApi.get_payment_method_by_key(external_key, api_key, api_secret)
```

> Example Response:

```json
{
    "paymentMethodId":"916619a4-02bb-4d3d-b3da-2584ac897b19",
    "externalKey":"coolPaymentMethod",
    "accountId":"84c7e0d4-a5ed-405f-a655-3ed16ae19997",
    "isDefault":false,
    "pluginName":"__EXTERNAL_PAYMENT__",
    "pluginInfo":null,
    "auditLogs":[]
}
```

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- |
| **externalKey** | string | yes | none | External key |
| **includedDeleted** | boolean | no | false | If true, include deleted payment methods |
| **withPluginInfo** | boolean | no | false | If true, include plugin info |
| **audit** | string | no | "NONE" | Level of audit information to return |

If **withPluginInfo** is set to true, attributes for the active plugin are returned. These are plugin-dependent (see discussion above).
Audit information options are "NONE", "MINIMAL" (only inserts), or "FULL".


**Response**

If successful, returns a status code of 200 and a payment method resource object.

### Delete a payment method

This API deletes a payment method. The default payment method may not be deleted, depending on the query parameters.

**HTTP Request** 

`DELETE http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'X-Killbill-CreatedBy: demo' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19' 	
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("3c449da6-7ec4-4c74-813f-f5055739a0b9");
Boolean deleteDefaultPmWithAutoPayOff = true; // Will delete default payment method with auto pay off
Boolean forceDefaultPmDeletion = true; // Will force default payment method deletion
Map<String, String> NULL_PLUGIN_PROPERTIES = null;

paymentMethodApi.deletePaymentMethod(paymentMethodId, 
                                     deleteDefaultPmWithAutoPayOff, 
                                     forceDefaultPmDeletion, 
                                     NULL_PLUGIN_PROPERTIES, 
                                     requestOptions);
```

```ruby
payment_method_id = "4307ac7c-04a7-41e1-9cb0-8a4d4420104c"
set_auto_pay_off = false
force_default_deletion = false
KillBillClient::Model::PaymentMethod.destroy(payment_method_id,
                                             set_auto_pay_off,
                                             force_default_deletion,
                                             user,
                                             reason,
                                             comment,
                                             options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
payment_method_id = '0052cddd-0f61-4f68-b653-ca49b5d7f915'

paymentMethodApi.delete_payment_method(payment_method_id, 
                                       created_by, 
                                       api_key, 
                                       api_secret)
```

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- |
| **deleteDefaultPmWithAutoPayOff** | boolean | no | false | if true, delete default payment method only if AUTO_PAY_OFF is set |
| **forceDefaultPmDeletion** | boolean | no | false | if true, force default payment method deletion |

The query parameters determine the behavior if the payment method specified is the default method: If **forceDefaultPmDeletion** is true, the payment method will be deleted unconditionally. If **deleteDefaultPmWithAutoPayOff** is true, the payment method will also be deleted, and AUTO_PAY_OFF will be set (if not already). If neither parameter is true, the default payment method will not be deleted (the call will fail). 

**Response**

If successful, returns a status code of 204 and an empty body. If the payment method is the default and cannot be deleted, an error code of 500 and a suitable message will be returned.


## List and Search

These endpoints provide the ability to list and search for payment methods for a specific tenant

###  List payment methods

List all payment methods stored for the accounts maintained by this tenant

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/paymentMethods/pagination`

> Example Request:

```shell
curl \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Accept: application/json' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/pagination' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

Long offset = 0L;
Long limit = 1L;
String pluginName = null;
Boolean withPluginInfo = false;
Map<String, String> NULL_PLUGIN_PROPERTIES = null;


PaymentMethods allPaymentMethods = paymentMethodApi.getPaymentMethods(offset,
                                                                      limit,
                                                                      pluginName,
                                                                      withPluginInfo,
                                                                      NULL_PLUGIN_PROPERTIES,
                                                                      AuditLevel.NONE,
                                                                      requestOptions);
```

```ruby
offset = 0
limit = 100

payment_method.find_in_batches(offset,
                               limit,
                               options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()

paymentMethodApi.get_payment_methods(api_key, api_secret)
```
> Example Response:

```json
[
  {
    "paymentMethodId": "916619a4-02bb-4d3d-b3da-2584ac897b19",
    "externalKey": "coolPaymentMethod",
    "accountId": "84c7e0d4-a5ed-405f-a655-3ed16ae19997",
    "isDefault": false,
    "pluginName": "__EXTERNAL_PAYMENT__",
    "pluginInfo": null,
    "auditLogs": []
  },
  {
    "paymentMethodId": "dc89832d-18a3-42fd-b3be-cac074fddb36",
    "externalKey": "paypal",
    "accountId": "ca15adc4-1061-4e54-a9a0-15e773b3b154",
    "isDefault": false,
    "pluginName": "killbill-paypal-express",
    "pluginInfo": null,
    "auditLogs": []
  }
]
```

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- | 
| **offset** | long | no | 0 | Starting position in list |
| **limit** | long | no | 100 | Max number of items to return |
| **withPluginInfo** | boolean | no | false | If true, include plugin info |
| **pluginName** | string | no | all plugins | If present, list only payment methods that use this plugin |
| **pluginProperties** | array of strings | no | no properties | return these properties for this plugin |
| **audit** | string | no | "NONE" | Level of audit information to return |

If **withPluginInfo** is set to true, attributes for each payment method's plugin are returned. These are plugin-dependent (see discussion above). If **pluginName** is given, list only payment methods that use this plugin, and return only the attributes specified by **pluginProperties**.
Audit information options are "NONE", "MINIMAL" (only inserts), or "FULL".



**Response**

If successful, returns a status code of 200 and a list of payment method resource objects.

###  Search payment methods

This API searches all payment methods for a specified search string. The search string is given as a path parameter.

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/paymentMethods/search/{searchKey}`

> Example Request:

```shell
curl \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Accept: application/json' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/search/coolPaymentMethod' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

String searchKey = "4365";
Long offset = 0L;
Long limit = 100L;
String pluginName = null;
Boolean withPluginInfo = true;
Map<String, String> NULL_PLUGIN_PROPERTIES = null;

List<PaymentMethod> results = paymentMethodApi.searchPaymentMethods(searchKey, 
                                                                    offset, 
                                                                    limit, 
                                                                    pluginName, 
                                                                    withPluginInfo, 
                                                                    NULL_PLUGIN_PROPERTIES,  
                                                                    AuditLevel.NONE,  
                                                                    requestOptions);
```

```ruby
search_key = 'example'
offset = 0
limit = 100

payment_method.find_in_batches_by_search_key(search_key,
                                             offset,
                                             limit,
                                             options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
search_key = '__EXTERNAL_PAYMENT__'

paymentMethodApi.search_payment_methods(search_key, api_key, api_secret)
```

> Example Response:

```json
[
  {
    "paymentMethodId": "916619a4-02bb-4d3d-b3da-2584ac897b19",
    "externalKey": "coolPaymentMethod",
    "accountId": "84c7e0d4-a5ed-405f-a655-3ed16ae19997",
    "isDefault": false,
    "pluginName": "__EXTERNAL_PAYMENT__",
    "pluginInfo": null,
    "auditLogs": []
  }
]
```

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- | 
| **offset** | long | no | 0 | Starting position in list |
| **limit** | long | no | 100 | Max number of items to return |
| **withPluginInfo** | boolean | no | false | If true, include plugin info |
| **pluginName** | string | no | all plugins | If present, list only payment methods that use this plugin |
| **pluginProperties** | array of strings | no | no properties | return these properties for this plugin |
| **audit** | string | no | "NONE" | Level of audit information to return |

If **withPluginInfo** is set to true, attributes for each payment method's plugin are returned. These are plugin-dependent (see discussion above). If **pluginName** is given, list only payment methods that use this plugin, and return only the attributes specified by **pluginProperties**.
Audit information options are "NONE", "MINIMAL" (only inserts), or "FULL".

**Response**

If successful, returns a status code of 200 and a list of payment method resource objects that match the search key.



## Custom Fields

`Custom fields` are `{key, value}` attributes that can be attached to any customer resources. For more on custom fields see [Custom Fields](#custom-fields). These endpoints manage custom fields associated with `PaymentMethod` objects.

### Add custom fields to a payment method

Adds one or more custom fields to a payment method object. Existing custom fields are not disturbed.

**HTTP Request** 

`POST http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}/customFields`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Content-Type: application/json' \
    -H 'X-Killbill-CreatedBy: demo' \
    -d '[{ 
            "objectId": "916619a4-02bb-4d3d-b3da-2584ac897b19",
            "objectType": "PAYMENT_METHOD",
            "name": "Test Custom Field",
            "value": "test_value"
    }]' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19/customFields' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("3c449da6-7ec4-4c74-813f-f5055739a0b9");

final List<AuditLog> EMPTY_AUDIT_LOGS = Collections.emptyList();

CustomFields customFields = new CustomFields();
customFields.add(new CustomField(null, 
                                 paymentMethodId, 
                                 ObjectType.PAYMENT_METHOD, 
                                 "Test Custom Field", 
                                 "test_value", 
                                 EMPTY_AUDIT_LOGS));

paymentMethodApi.createPaymentMethodCustomFields(paymentMethodId, 
                                                 customFields, 
                                                 requestOptions);
```

```ruby
custom_field = KillBillClient::Model::CustomFieldAttributes.new
custom_field.object_type = 'PAYMENT_METHOD'
custom_field.name = 'Test Custom Field'
custom_field.value = 'test_value'

payment_method.add_custom_field(custom_field, 
                                user,
                                reason,
                                comment,
                                options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
payment_method_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'
body = CustomField(name='Test Custom Field', value='test_value')

paymentMethodApi.create_payment_method_custom_fields(payment_method_id,
                                                     [body],
                                                     created_by,
                                                     api_key,
                                                     api_secret)
```

**Request Body**

A list of objects giving the name and value of the custom field, or fields, to be added. For example:

[
  {
    "name": "CF1",
    "value": "123"
  }
]


**Query Parameters**

None.

**Response**

If successful, returns a 201 status code. In addition, a `Location` header is returned giving the URL to retrieve the custom fields associated with the payment method.

###  Retrieve payment method custom fields

Returns any custom field objects associated with the specified payment method

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}/customFields`

> Example Request:

```shell
curl \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Accept: application/json' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19/customFields' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("3c449da6-7ec4-4c74-813f-f5055739a0b9");

List<CustomField> customFields = paymentMethodApi.getPaymentMethodCustomFields(paymentMethodId,
                                                                               AuditLevel.NONE,
                                                                               requestOptions);
```

```ruby
audit = 'NONE'

payment_method.custom_fields(audit, options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
payment_method_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'

paymentMethodApi.get_payment_method_custom_fields(payment_method_id, 
                                                  api_key, 
                                                  api_secret)
```

> Example Response:

```json
[
  {
    "customFieldId": "6d4c073b-fd89-4e39-9802-eba65f42492f",
    "objectId": "916619a4-02bb-4d3d-b3da-2584ac897b19",
    "objectType": "PAYMENT_METHOD",
    "name": "Test Custom Field",
    "value": "test_value",
    "auditLogs": []
  }
]
```


**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- | 
| **audit** | string | no | "NONE" | Level of audit information to return:"NONE", "MINIMAL", or "FULL" |


**Response**

If successful, returns a status code of 200 and a (possibly empty) list of custom field objects.

###  Modify custom fields for payment method

Modifies the value of one or more existing custom fields associated with a payment object

**HTTP Request** 

`PUT http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}/customFields`

> Example Request:

```shell
curl -v \
    -X PUT \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Content-Type: application/json' \
    -H 'X-Killbill-CreatedBy: demo' \
    -d '[{ 
            "customFieldId": "6d4c073b-fd89-4e39-9802-eba65f42492f",
            "value": "NewValue"
    }]' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19/customFields' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("3c449da6-7ec4-4c74-813f-f5055739a0b9");

UUID customFieldsId = UUID.fromString("9913e0f6-b5ef-498b-ac47-60e1626eba8f");

CustomField customFieldModified = new CustomField();
customFieldModified.setCustomFieldId(customFieldsId);
customFieldModified.setValue("NewValue");
CustomFields customFields = new CustomFields();
customFields.add(customFieldModified);
paymentMethodApi.modifyPaymentMethodCustomFields(paymentMethodId, 
                                                 customFields, 
                                                 requestOptions);

```

```ruby
custom_field.custom_field_id = '7fb3dde7-0911-4477-99e3-69d142509bb9'
custom_field.name = 'Test Modify'
custom_field.value = 'test_modify_value'

payment_method.modify_custom_field(custom_field,                                                                                            
                                   user, 
                                   reason,
                                   comment, 
                                   options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
payment_method_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'
custom_field_id = '7fb3dde7-0911-4477-99e3-69d142509bb9'
body = CustomField(custom_field_id=custom_field_id, 
                   name='Test Custom Field', 
                   value='test_value')

paymentMethodApi.modify_payment_method_custom_fields(payment_method_id, 
                                                     [body], 
                                                     created_by, 
                                                     api_key, 
                                                     api_secret)
```

**Request Body**

A list of objects specifying the id and the new value for the custom fields to be modified. For example:

[ { "customFieldId": "6d4c073b-fd89-4e39-9802-eba65f42492f", "value": "123" } ]

Although the `fieldName` and `objectType` can be specified in the request body, these cannot be modified, only the field value can be modified.

**Query Parameters**

None.

**Response**

If successful, a status code of 204 and an empty body.

###  Remove custom fields from payment method

Delete one or more custom fields from a payment method

**HTTP Request** 

`DELETE http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}/customFields`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'X-Killbill-CreatedBy: demo' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/77e23878-8b9d-403b-bf31-93003e125712/customFields?customField=439ed0f8-9b37-4688-bace-e2595b1d3801' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("3c449da6-7ec4-4c74-813f-f5055739a0b9");
UUID customFieldsId = UUID.fromString("9913e0f6-b5ef-498b-ac47-60e1626eba8f");
List<UUID> customFieldsList = List.of(customFieldsId);
paymentMethodApi.deletePaymentMethodCustomFields(paymentMethodId, 
                                                 customFieldsList, 
                                                 requestOptions);
```

```ruby
custom_field_id = custom_field.id

payment_method.remove_custom_field(custom_field_id,                                                                                            
                                   user, 
                                   reason,
                                   comment, 
                                   options)
```

```python
paymentMethodApi = killbill.api.PaymentMethodApi()
payment_method_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'

paymentMethodApi.delete_payment_method_custom_fields(payment_method_id,
                                                     created_by,
                                                     api_key, 
                                                     api_secret)
```

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- | 
| **customField** | string | yes | none | Custom field object ID that should be deleted. Multiple custom fields can be deleted by specifying a separate **customField** parameter corresponding to each field. |

**Response**

If successful, returns a status code of 204 and an empty body.


## Audit Logs

This endpoint enables access to payment method audit logs. For more on audit logs see the Audit and History section under Using Kill Bill APIs.

### Retrieve payment method audit logs with history by id

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/paymentMethods/{paymentMethodId}/auditLogsWithHistory`

> Example Request:

```shell
curl \
    -u admin:password \
    -H 'X-Killbill-ApiKey: bob' \
    -H 'X-Killbill-ApiSecret: lazar' \
    -H 'Accept: application/json' \
    'http://127.0.0.1:8080/1.0/kb/paymentMethods/916619a4-02bb-4d3d-b3da-2584ac897b19/auditLogsWithHistory' 
```

```java
import org.killbill.billing.client.api.gen.PaymentMethodApi;
protected PaymentMethodApi paymentMethodApi;

UUID paymentMethodId = UUID.fromString("e9d95f16-a426-46d0-b76b-90814792fb36");

List<AuditLog> result = paymentMethodApi.getPaymentMethodAuditLogsWithHistory(paymentMethodId, requestOptions);
```

```python
accountApi = killbill.api.AccountApi()
account_id = 'c62d5f6d-0b57-444d-bf9b-dd23e781fbda'
account_email_id = 'bb390282-6757-4f4f-8dd5-456abd9f30b2'

accountApi.get_account_email_audit_logs_with_history(account_id,
                                                     account_email_id,
                                                     api_key,
                                                     api_secret)
```

```ruby
account_email_id = 'a4627e89-a73b-4167-a7ba-92a2881eb3c4'

account.email_audit_logs_with_history(account_email_id, options)
```

> Example Response:

```json
[
  {
    "changeType": "INSERT",
    "changeDate": "2018-07-19T14:56:07.000Z",
    "objectType": "PAYMENT_METHOD",
    "objectId": "916619a4-02bb-4d3d-b3da-2584ac897b19",
    "changedBy": "admin",
    "reasonCode": null,
    "comments": null,
    "userToken": "f77892e9-32bd-4d59-8039-5e12798b53fe",
    "history": 
    {
      "id": null,
      "createdDate": "2018-07-19T14:56:07.000Z",
      "updatedDate": "2018-07-19T14:56:07.000Z",
      "recordId": 10,
      "accountRecordId": 35,
      "tenantRecordId": 1,
      "externalKey": "unknown",
      "accountId": "84c7e0d4-a5ed-405f-a655-3ed16ae19997",
      "pluginName": "__EXTERNAL_PAYMENT__",
      "isActive": true,
      "active": true,
      "tableName": "PAYMENT_METHODS",
      "historyTableName": "PAYMENT_METHOD_HISTORY"
    }
  }
]
```

**Query Parameters**

None.

**Returns**
    
If successful, returns a status code of 200 and a list of payment method audit logs with history.

