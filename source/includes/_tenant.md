# Tenant

## Tenant Resource

Kill Bill has been designed from the ground up as a multi-tenant system, that is, one where multiple unrelated deployments can be hosted on the same physical system. Each deployment has its own separate configuration, catalog, and plugins, and of course its data set is kept entirely separate from the others. RBAC control allows different users/admin/apps to access zero, one or multiple tenants. This [blog](https://killbill.io/blog/subscription-service-using-kill-bill/) illustrates some interesting use cases. The `Tenant` resource allows the management of such tenants.

The attributes of the `Tenant` resource object are the following:

| Name | Type | Generated by | Description |
| ---- | -----| -------- | ------------ |
| **tenantId** | string | system | UUID for this tenant |
| **externalKey** | string | system or user | Optional external key provided by the client |
| **apiKey** | string | user | The API key associated with the tenant |
| **apiSecret** | string | user | The API secret associated with the tenant. This value is hashed and cannot be retrieved. |
| **auditLogs** | array | system | Array of audit log records for this tenant |

Note that the tenant itself has very few attributes, as most attributes belong to the individual accounts and related resources.

## Tenant

These endpoints manage information that is maintained at the tenant level. Unless otherwise stated, the tenant is identified by its API key in the header.

### Create a tenant

This API creates a new tenant.

**Note:** If you create a tenant using this API, it will *not* be recognized immediately by KAUI because KAUI will be unable to retrieve the `apiSecret`. To fix this, you should "create" the same tenant separately after logging into KAUI. This will *not* create another tenant, but it will synchronize KAUI with the one already created.

**HTTP Request**

`POST http://127.0.0.1:8080/1.0/kb/tenants`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    -d '{ "apiKey": "bob", "apiSecret": "lazar"}' \
    "http://127.0.0.1:8080/1.0/kb/tenants"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

UUID tenantId = null;
String externalKey = null;
String apiKey = "bob";
String apiSecret = "lazar";
List<AuditLog> EMPTY_AUDIT_LOGS = Collections.emptyList();

Tenant body = new Tenant(tenantId,
                         externalKey,
                         apiKey,
                         apiSecret,
                         EMPTY_AUDIT_LOGS);

tenantApi.createTenant(body, requestOptions);
```

```ruby
tenant = KillBillClient::Model::Tenant.new
tenant.external_key = "demo_external_key"
tenant.api_key = "demo_api_key"
tenant.api_secret = "demo_api_secret"

use_global_defalut = true
user = "demo"
reason = nil
comment = nil

tenant.create(use_global_defalut,
              user,
              reason,
              comment,
              options)
```

```python
tenantApi = killbill.api.TenantApi()

body = Tenant(api_key='demo_api_key', api_secret='demo_api_secret')

tenantApi.create_tenant(body, created_by='demo')
```

**Request Body**

A JSON string representing a Tenant resource. The only parameters required are `apiKey` and `apiSecret`. The `externalKey` is optional and will be set to null if not supplied. The `tenantId` is generated by the system.

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- |
| **useGlobalDefault** | boolean | false | false | If true, configure the tenant with a default catalog |

Setting the `useGlobalDefault` parameter to `true` can be used for test purposes. This will configure the tenant with a default catalog, and therefore make it easy to quickly start playing with the apis. Note that in order to then upload a new custom catalog, one would need to [invalidate the caches for this tenant](#admin-invalidates-caches-per-tenant-level).

**Response**

If successful, returns a status code of 201 and an empty body. In addition a `Location` parameter is returned in the header. This parameter gives the URL for the tenant, including the `tenantId`.

### Retrieve a tenant by id

Retrieves a tenant resource, given its ID.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/{tenantId}`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants/6907712e-e940-4033-8695-36894db128d3"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

UUID tenantId = UUID.fromString("6907712e-e940-4033-8695-36894db128d3");
tenantApi.getTenant(tenantId, requestOptions);
```

```ruby
tenant_id = "ab5981c2-de14-43d6-a35b-a2ed0b28c746"

KillBillClient::Model::Tenant.find_by_id(tenant_id, options)
```

```python
tenantApi = killbill.api.TenantApi()

tenant.get_tenant(tenant_id='3d90ec45-c640-4fd7-abde-798bc582513b')
```

> Example Response:

```json
{
  "tenantId": "6907712e-e940-4033-8695-36894db128d3",
  "externalKey": "1532546166-326384",
  "apiKey": "bob",
  "apiSecret": null,
  "auditLogs": []
}
```


**Query Parameters**

none

**Response**

If successful, returns a status code of 200 and a tenant resource object.  The `apiSecret` attribute is returned as `null`, since it cannot be retrieved.

### Retrieve a tenant by its API key

Retrieves a tenant resource, given its `apiKey`. The `apiKey` is passed as a query parameter.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants?apiKey=bob"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String apiKey = "bob";

tenantApi.getTenantByApiKey(apiKey, requestOptions);
```

```ruby
api_key = "demo_api_key"

KillBillClient::Model::Tenant.find_by_api_key(api_key, options)
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.get_tenant_by_api_key(api_key='bob')
```

> Example Response:

```json
{
  "tenantId": "6907712e-e940-4033-8695-36894db128d3",
  "externalKey": "1532546166-326384",
  "apiKey": "bob",
  "apiSecret": null,
  "auditLogs": []
}
```


**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- |
| **apiKey** | string | true | none | api key |

**Response**

If successful, returns a status code of 200 and a tenant resource object.  The `apiSecret` attribute is returned as `null`, since it cannot be retrieved.

## Push Notifications

Push notifications are a convenient way to get notified about events from the system.
One can register a callback, i.e, a valid URL that will be called whenever there is an event dispatched for this tenant.
Note that this can result in a large number of calls; every time there is a state change for one of the `Accounts`
in this tenant, the callback will be invoked.

In case of error, the system will retry the callback as defined by the system property `org.killbill.billing.server.notifications.retries`.

See push notification documentation [here](https://docs.killbill.io/latest/push_notifications.html) for further information.

### Register a push notification

Register a callback URL for this tenant for push notifications.The key name is PUSH_NOTIFICATION_CB. The API sets the value of this key, replacing any previous value.

**HTTP Request**

`POST http://127.0.0.1:8080/1.0/kb/tenants/registerNotificationCallback`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    'http://127.0.0.1:8080/1.0/kb/tenants/registerNotificationCallback?cb=http://demo/callmeback'
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String cb = "http://demo/callmeback";

TenantKeyValue result = tenantApi.registerPushNotificationCallback(cb, requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.register_push_notification_callback(created_by='demo', cb='http://demo/callmeback')
```

**Query Parameters**

| Name | Type | Required | Default | Description |
| ---- | -----| -------- | ------- | ----------- |
| **cb** | string | true | none | callback URL to register |

**Response**

If successful, returns a status code of 201 and an empty body. In addition, a `Location` item is returned in the header giving the URL for this callback.

### Retrieve a registered push notification

Gets the push notification registered for this tenant, if any.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/registerNotificationCallback`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://localhost:8080/1.0/kb/tenants/registerNotificationCallback"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

TenantKeyValue result = tenantApi.getPushNotificationCallbacks(requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.get_push_notification_callbacks()
```

> Example Response:

```json
{
  "key": "PUSH_NOTIFICATION_CB",
  "values": [
    "http://demo/callmeback"
  ]
}
```


**Query Parameters**

None.

**Response**

If successful, returns a status code of 200 and a body containing a key-value object as follows:

{
  "key": "PUSH_NOTIFICATION_CB",
  "values": list containing the callback URL, if any
}

### Delete a registered push notification

Deletes the registered callback URL, if any.

**HTTP Request**

`DELETE http://127.0.0.1:8080/1.0/kb/tenants/registerNotificationCallback`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    "http://127.0.0.1:8080/1.0/kb/tenants/registerNotificationCallback"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

tenantApi.deletePushNotificationCallbacks(requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.delete_push_notification_callbacks(created_by='demo')
```

**Query Parameters**

None.

**Response**

If successful, returns a status code of 204 and an empty body.

## Tenant Key-Value Pairs

These endpoints provide a mechanism to register and manage `{key, value}` pairs for a given tenant. This functionality
is used internally by the system to keep track of all the per-tenant configuration, including system properties, plugin configuration, and others discussed below. In addition, you can add *user* keys to keep track of additional information that may be desired. For example, some global setting that would be accessible for all plugins could be stored here.

### Add a per tenant user key/value

This API adds a key-value pair to the tenant database. If the key already exists its value is replaced. The key is given as a path parameter.

**HTTP Request**

`POST http://127.0.0.1:8080/1.0/kb/tenants/userKeyValue/{keyName}`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: text/plain" \
    -H "Accept: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    -d "demo_value" \
    "http://127.0.0.1:8080/1.0/kb/tenants/userKeyValue/demo_key"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String keyName = "demo_value";
String body = "demo_value";
TenantKeyValue result = tenantApi.insertUserKeyValue(keyName, body, requestOptions);
```

```ruby
user = "demo"
reason = nil
comment = nil

key_name = "demo_value"
key_value = "demo_value"

KillBillClient::Model::Tenant.upload_tenant_user_key_value(key_name,
                                                           key_value,
                                                           user,
                                                           reason,
                                                           comment,
                                                           options)
```

```python
tenantApi = killbill.api.TenantApi()

key_name = 'demo_value'
body = 'demo_value'

tenantApi.insert_user_key_value(key_name, body, created_by='demo')
```

**Request Body**

The body contains a single string representing the value.

**Query Parameters**

None.

**Response**

If successful, returns a status code of 201 and an empty body. In addition, a `Location` item is returned in the header giving the URL for this key-value pair.

### Retrieve a per tenant user key value

Retrieves the value for a specified key, if it exists, from the tenant database. The key name is given as a path parameter.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/userKeyValue/{keyName}`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants/userKeyValue/demo_value"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String keyName = "demo_value";

TenantKeyValue result = tenantApi.getUserKeyValue(keyName, requestOptions);
```

```ruby
key_name = "demo_value"

KillBillClient::Model::Tenant.get_tenant_user_key_value(key_name, options)
```

```python
tenantApi = killbill.api.TenantApi()

key_name = 'demo_value'

tenantApi.get_user_key_value(key_name)
```

> Example Response:

```json
{
  "key": "demo_value",
  "values": [
    "demo_value",
    "demo_value"
  ]
}
```


**Query Parameters**

None.

**Response**

If successful, returns a status code of 200 and a tenant key value object. The key value object includes the key name and a JSON array containing the value, if any, or a comma-separated list of values. For example:

{
  "key": "MYKEY",
  "values": [
    "value1",
    "value2"
  ]
}


If the key does not exist no error is signalled but the `values` list is empty.

### Delete a per tenant user key/value

Deletes a key and its value, if it exists, from the tenant database. The key is given as a path parameter.

**HTTP Request**

`DELETE http://127.0.0.1:8080/1.0/kb/tenants/userKeyValue/{keyName}`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    "http://127.0.0.1:8080/1.0/kb/tenants/userKeyValue/demo_value"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String keyName = "demo_value";

tenantApi.deleteUserKeyValue(keyName, requestOptions);
```

```ruby
user = "demo"
reason = nil
comment = nil
key_value = "demo_value"

KillBillClient::Model::Tenant.delete_tenant_user_key_value(key_value,
                                                           user,
                                                           reason,
                                                           comment,
                                                           options)
```

```python
tenantApi = killbill.api.TenantApi()

key_name = 'demo_value'

tenantApi.delete_user_key_value(key_name, created_by='demo')
```

**Query Parameters**

None.

**Returns**

If successful, returns a status code of 204 and an empty body. No error is signalled if the key does not exist.

### Retrieve per tenant keys and values based on a key prefix

This API enables searching for existing keys based on a prefix of the key name. For example, a search string of "MYK" would match keys such as `MYKEY1`, `MYKEY2`, etc. The search string is given as a path parameter.


**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig/{keyPrefix}/search`


> Example Request:


```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig/PER_TENANT/search"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String keyPrefix = "PER_TENANT";

TenantKeyValues result = tenantApi.getAllPluginConfiguration(keyPrefix, requestOptions);
```

```ruby
key_prefix = "PER_TENANT"

KillBillClient::Model::Tenant.search_tenant_config(key_prefix, options)
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.get_all_plugin_configuration(key_prefix='tenant_config')
```

> Example Response:

```json
{
  "key": "PER_TENANT_CONFIG",
  "values": [
    "{org.killbill.invoice.sanitySafetyBoundEnabled:false}"
  ]
}
```


**Query Parameters**

None.

**Response**

If successful, returns a status code of 200 and a tenant key value object containing the key and values for any keys that match the search string.




## System Properties Configuration

These endpoints allow setting of some system properties on a per-tenant basis. Please refer to our [configuartion guide](https://docs.killbill.io/latest/userguide_configuration.html) to see what can be configured in the system.
Some of the configuration can be overriden at the tenant level to allow for different behaviors.

Note that this is actually a special case of per-tenant key-value pairs; the key is "PER_TENANT_CONFIG" and the value is a comma-separated list of system properties with their values.

### Add a per tenant system properties configuration

This API is used to set the value of specific system properties, overriding the system-wide values.

For example, in order to disable the invoice safety bound mechanism on a per-tenant level, this API could be used to set the per-tenant system property `org.killbill.invoice.sanitySafetyBoundEnabled` to false.

The API sets the value of the "PER_TENANT_CONFIG" key, replacing any previous value.


**HTTP Request**

`POST http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig`


> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: text/plain" \
    -H "Accept: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    -d "{"org.killbill.invoice.sanitySafetyBoundEnabled":"false"}" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String body = "{\"org.killbill.invoice.sanitySafetyBoundEnabled\":\"false\"}";

TenantKeyValue result = tenantApi.uploadPerTenantConfiguration(body, requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()
body = '{"org.killbill.invoice.sanitySafetyBoundEnabled":"false"}'

tenantApi.upload_per_tenant_configuration(body, created_by='demo')
```

**Request Body**

A JSON string representing the per-tenant system property and its value.

**Query Parameters**

None.

**Response**

If successful, returns a status code of 201 and an empty body.

### Retrieve a per tenant system properties configuration

Retrieves the per-tenant system property settings, which are given as the value of the key `PER_TENANT_CONFIG`.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

TenantKeyValue result = tenantApi.getPerTenantConfiguration(requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.get_per_tenant_configuration()
```

> Example Response:

```json
{
  "key": "PER_TENANT_CONFIG",
  "values": [
    "{org.killbill.invoice.sanitySafetyBoundEnabled:false}"
  ]
}
```


**Query Parameters**

None.

**Response**

If successful, returns a status code of 200 and a tenant key value object for the key `PER_TENANT_CONFIG`.

### Delete a per tenant system properties configuration

Deletes any current per tenant system properties configuration parameters, which are given as the values of the `PER_TENANT_CONFIG` key.

**HTTP Request**

`DELETE http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPerTenantConfig"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

tenantApi.deletePerTenantConfiguration(requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

tenantApi.delete_per_tenant_configuration(created_by='demo')
```

**Query Parameters**

None.

**Response**

If successful, returns a status code of 204 and an empty body.


## Plugin Configuration

Plugins also support configuration on a per-tenant level. Please refer to our [plugin configuration manual](https://docs.killbill.io/latest/plugin_development.html#_plugin_configuration) for more details.

An example of the use of such per-tenant properties is to configure a payment plugin with different API keys,
one set of keys for each tenant. This allows for a true multi-tenant deployment where plugins have different configuration
based on the tenant in which they operate.

Upon adding or deleting a new per-tenant plugin configuration, the system will generate a `TENANT_CONFIG_CHANGE`/`TENANT_CONFIG_DELETION` event, which can be handled in the plugin to refresh its configuration. In multi-node scenarios, events will be dispatched on each node, that is, on each plugin instance so they end up with a consistent view. A lot of the logic to handle configuration update has been implemented in our plugin frameworks (see [ruby framework](https://github.com/killbill/killbill-plugin-framework-ruby) and [java framework](https://github.com/killbill/killbill-plugin-framework-java)).

As with the system properties configuration, this is actually a special case of per-tenant key-value pairs. The following endpoints provide the ability to configure plugins on a per-tenant level.

### Add a per tenant configuration for a plugin

Adds a per tenant key-value pair for the specified plugin. The plugin name is given as a path parameter. The key name is `PLUGIN_CONFIG_*plugin*` where *plugin* is the plugin name. The API sets the value of this key, replacing any previous value.

The value string uploaded is plugin dependent but typically consists of key/value properties,
or well formatted yml or a properties file.

**HTTP Request**

`POST http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginConfig/{pluginName}`


> Example Request:


```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: text/plain" \
    -H "Accept: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    -d @./config.properties \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginConfig/demo_plugin"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String pluginName = "PLUGIN_FOO";
String pluginConfig = "plugin configuration string";

TenantKeyValue result = tenantApi.uploadPluginConfiguration(pluginName, pluginConfig, requestOptions);
```

```ruby
plugin_name = "demo_plugin"
plugin_config = "tenant_config"
user = "demo"
reason = nil
comment = nil

KillBillClient::Model::Tenant.upload_tenant_plugin_config(plugin_name,
                                                          plugin_config,
                                                          user,
                                                          reason,
                                                          comment,
                                                          options)
```

```python
tenantApi = killbill.api.TenantApi()

plugin_name = 'demo_plugin'
body = 'tenant_config'

tenantApi.upload_plugin_configuration(plugin_name, body, created_by='demo')
```


**Request Body**

The request body can be specified as a JSON string consisting of the key-value pairs for the plugin configuration. Alternatively, the plugin configuration can be specified in a yml or properties file and its path can be specified.

**Query Parameters**

None.

**Response**

If successful, returns a status code of 201 and an empty body. A `Location` header is also returned giving the URL of the key-value pair.

### Retrieve a per tenant configuration for a plugin

Gets the per tenant configuration value string for a specified plugin. The plugin name is given as a path parameter.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginConfig/{pluginName}`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginConfig/demo_plugin"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

final String pluginName = "PLUGIN_FOO";

final TenantKeyValue result = tenantApi.getPluginConfiguration(pluginName, requestOptions);
```

```ruby
plugin_name = "demo_plugin"

KillBillClient::Model::Tenant.get_tenant_plugin_config(plugin_name, options)
```

```python
tenantApi = killbill.api.TenantApi()

plugin_name = 'demo_plugin'

tenantApi.get_plugin_configuration(plugin_name)
```

> Example Response:

```json
{
  "key": "PLUGIN_CONFIG_demo_plugin",
  "values": [
    "tenant_config"
  ]
}
```


**Query Parameters**

None.

**Response**

If successful, returns a status code of 200 and a tenant key value object for the key `PLUGIN_CONFIG_*plugin*`.

### Delete a per tenant configuration for a plugin

Deletes the per tenant plugin configuration value for the appropriate key. The plugin name is given as a path parameter.

**HTTP Request**

`DELETE http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginConfig/{pluginName}`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginConfig/demo_plugin"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

final String pluginName = "PLUGIN_FOO";

tenantApi.deletePluginConfiguration(pluginName, requestOptions);
```

```ruby
user = "demo"
reason = nil
comment = nil

plugin_name = "demo_plugin"

KillBillClient::Model::Tenant.delete_tenant_plugin_config(plugin_name,
                                                          user,
                                                          reason,
                                                          comment,
                                                          options)
```

```python
tenantApi = killbill.api.TenantApi()

plugin_name = 'demo_plugin'

tenantApi.delete_plugin_configuration(plugin_name, created_by='demo')
```

**Query Parameters**

None.

**Returns**

If successful, returns a status code of 204 and an empty body.

## Payment State Machines

This is a somewhat advanced use case to override the default internal payment state machine within Kill Bill. Please refer to our [payment manual](https://docs.killbill.io/latest/userguide_payment.html#_payment_states) for more details about payment states.

The endpoints below allow you to override such state machines on a per-tenant level.

### Add a per tenant payment state machine for a plugin

Adds a per tenant key-value pair for the specified plugin. The plugin name is given as a path parameter. The key name is `PLUGIN_PAYMENT_STATE_MACHINE_*plugin*` where *plugin* is the plugin name. The API sets the value of this key, replacing any previous value.

The state machine is defined in an XML file. The complete XML file becomes the value of the key.

**HTTP Request**

Let's say we want to overwrite the default Kill Bill payment state machine for the payment plugin `demo_plugin`, and assuming `SimplePaymentStates.xml`is a valid payment state machine XML file, then the HTTP Request would be:

`POST http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginPaymentStateMachineConfig/{pluginName}`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: text/plain" \
    -H "Accept: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
	-d '@SimplePaymentStates.xml' \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginPaymentStateMachineConfig/demo_plugin"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String pluginName = "noop";
String stateMachineConfig = getResourceBodyString("SimplePaymentStates.xml");

TenantKeyValue result = tenantApi.uploadPluginPaymentStateMachineConfig(pluginName,
                                                                        stateMachineConfig,
                                                                        requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

plugin_name = 'demo_plugin'
body = 'SimplePaymentStates.xml'

tenantApi.upload_plugin_payment_state_machine_config(plugin_name, body, created_by='demo')
```

**Query Parameters**

None.

**Response**

If successful, returns a status code of 201 and an empty body. In addition, a `Location` item is returned in the header giving the URL for the payment state machine.

### Retrieve a per tenant payment state machine for a plugin

Retrieves the value for the appropriate payment state machine key for the specified plugin. If present, this value should be the complete XML file that defines the state machine.

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginPaymentStateMachineConfig/{pluginName}`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginPaymentStateMachineConfig/demo_plugin"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String pluginName = "noop";

TenantKeyValue result = tenantApi.getPluginPaymentStateMachineConfig(pluginName, requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

plugin_name = 'demo_plugin'

tenantApi.get_plugin_payment_state_machine_config(plugin_name)
```

> Example Response:

```json
{
  "key": "PLUGIN_PAYMENT_STATE_MACHINE_demo_plugin",
  "values": [<?xml version="1.0" encoding="UTF-8"?>

    <stateMachineConfig xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                        xsi:noNamespaceSchemaLocation="StateMachineConfig.xsd">

        <stateMachines>
            <stateMachine name="BIG_BANG">
                <states>
                    <state name="BIG_BANG_INIT"/>
                </states>
                <transitions>
                    <transition>
                        <initialState>BIG_BANG_INIT</initialState>
                        <operation>OP_DUMMY</operation>
                        <operationResult>SUCCESS</operationResult>
                        <finalState>BIG_BANG_INIT</finalState>
                    </transition>
                </transitions>
                <operations>
                    <operation name="OP_DUMMY"/>
                </operations>
            </stateMachine>
            <stateMachine name="AUTHORIZE">
                <states>
                    <state name="AUTH_INIT"/>
                    <state name="AUTH_SUCCESS"/>
                    <state name="AUTH_FAILED"/>
                    <state name="AUTH_ERRORED"/>
                </states>
                <transitions>
                    <transition>
                        <initialState>AUTH_INIT</initialState>
                        <operation>OP_AUTHORIZE</operation>
                        <operationResult>SUCCESS</operationResult>
                        <finalState>AUTH_SUCCESS</finalState>
                    </transition>
                    <transition>
                        <initialState>AUTH_INIT</initialState>
                        <operation>OP_AUTHORIZE</operation>
                        <operationResult>FAILURE</operationResult>
                        <finalState>AUTH_FAILED</finalState>
                    </transition>
                    <transition>
                        <initialState>AUTH_INIT</initialState>
                        <operation>OP_AUTHORIZE</operation>
                        <operationResult>EXCEPTION</operationResult>
                        <finalState>AUTH_ERRORED</finalState>
                    </transition>
                </transitions>
                <operations>
                    <operation name="OP_AUTHORIZE"/>
                </operations>
            </stateMachine>
        </stateMachines>

        <linkStateMachines>
            <linkStateMachine>
                <initialStateMachine>BIG_BANG</initialStateMachine>
                <initialState>BIG_BANG_INIT</initialState>
                <finalStateMachine>AUTHORIZE</finalStateMachine>
                <finalState>AUTH_INIT</finalState>
            </linkStateMachine>
        </linkStateMachines>
    </stateMachineConfig>
    ]
}
```


**Query Parameters**

None.

**Response**

If successful, returns a status code of 200 and a key value object for the key `PLUGIN_PAYMENT_STATE_MACHINE_*plugin*`. The value of this key should be the complete XML file that defines the payment state machine.

### Delete a per tenant payment state machine for a plugin

Deletes the payment state machine for the specified plugin for this tenant. The plugin reverts to the default payment state machine.

**HTTP Request**

`DELETE http://127.0.0.1:8080/1.0/kb/tenants/uploadPluginPaymentStateMachineConfig/{pluginName}`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    -H "X-Killbill-Reason: demo" \
    -H "X-Killbill-Comment: demo" \
    "http://localhost:8080/1.0/kb/tenants/uploadPluginPaymentStateMachineConfig/demo_plugin"
```

```java
import org.killbill.billing.client.api.gen.TenantApi;
protected TenantApi tenantApi;

String pluginName = "noop";

tenantApi.deletePluginPaymentStateMachineConfig(pluginName, requestOptions);
```

```ruby
TODO
```

```python
tenantApi = killbill.api.TenantApi()

plugin_name = 'demo_plugin'

tenantApi.delete_plugin_payment_state_machine_config(plugin_name, created_by='demo')
```

**Query Parameters**

None.

**Returns**

If successful, returns a status code of 204 and an empty body.
