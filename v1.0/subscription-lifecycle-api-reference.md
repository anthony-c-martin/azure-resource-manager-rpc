# Subscription Lifecycle API Reference
- [Creating or Updating a subscription](#creating-or-updating-a-subscription) <br/>
  - [Request](#request) <br/>
  - [Response](#response) <br/>
  - [Subscription States](#subscription-states) <br/>

## Creating or Updating a Subscription

Creates or updates a subscription for this particular resource provider. It includes changes in the state of the subscription which may trigger other actions (setup or teardown).

This API uses the &quot;system&quot; version of 2.0 because it can be triggered by commerce and not necessarily by a user request.

### Request

| Method | Request URI |
| --- | --- |
| PUT | https://&lt;registered-resource-provider-endpoint&gt;/subscriptions/{subscriptionId}?api-version=2.0 |

**Arguments**

| Argument | Description |
| --- | --- |
| subscriptionId | The subscriptionId for the Azure user. |

**Request Body**
```json
{
	"state": "Registered | Unregistered | Warned | Suspended | Deleted",
	"registrationDate": "Tue, 15 Nov 1994 08:12:31 GMT",
	"properties": {
		"tenantId": "ac430efe-1866-4124-9ed9-ee67f9cb75db",
		"locationPlacementId": "Internal_2014-09-01",
		"quotaId": "Default_2014-09-01",
		"registeredFeatures": [
			{
				"name": "<featureName>",
				"state": "Registered"
			}
		],
		"availabilityZones": {
			"location": "<locationName>",
			"zoneMappings": [
				{
					"logicalZone": "1",
					"physicalZone": "2"
				},
				{
					"logicalZone": "2",
					"physicalZone": "1"
				}
			]
		},
		"subscriptionSpendingLimit": "True",
		"subscriptionAccountOwner": "account@company.com",
		"managedByTenants": [
			{
				"tenantId": "<managedByTenantId>"
			}
		],
		"additionalProperties": {
			"<key1>": "<JToken1>",
			"<keyN>": "<JTokenN>"
		}
	}
}
```

| **Element name** | Description |
| --- | --- |
| **state** | Required.One of &quot;Registered&quot;, &quot;Unregistered&quot;, &quot;Warned&quot;, &quot;Suspended&quot; , or &quot;Deleted&quot;; used to indicate the current state of the subscription. The resource provider should always take the latest state. Transition among any states is valid (for example - it is possible to receive Suspended / Warned before Registered). Details on these states could be found below. |
|**registrationDate**| Required. Date the subscription was registered. |
| **properties** | Required.Property bag contains other name/value pairs that can be used for telemetry and logging. The resource provider should handle unexpected property key/value pairs without issue, as we will introduce new metadata without updating the contract version. The value inside the properties envelope may be a complex type / object / token itself. |
| **properties.tenantId** | Optional.The AAD directory/tenant to which the subscription belongs. |
| **properties.locationPlacementId** | Optional.The placement requirement for the subscription based on its country of origin / offer type / offer category / etc. This is used in geo-fencing of certain regions or regulatory boundaries (e.g. Australia ring-fencing). |
| **properties.quotaId** | Optional.The quota requirement for the subscription based on the offer type / category (e.g. free vs. pay-as-you-go). This can be used to inform quota information for the subscription (e.g. max # of resource groups or max # of virtual machines. |
| **properties.registeredFeatures** | Optional.All AFEC features that the subscriptions has been registered under RP namespace and platform namespace (Microsoft.Resources).  Null or an empty array would mean that there are no registered features for the given subscription. |
| **properties.availabilityZones** | Optional.Physical to logical zone mapping. This mapping will be per location (region). |
| **properties.subscriptionSpendingLimit** | Boolean for subscription spending limit. |
| **Properties.subscriptionAccountOwner** | String for Account owner. |
| **properties.managedByTenants** | Optional.All tenants managing the subscription. Null or empty means that there are no managing tenants. |
| **properties.additionalProperties** | Required. AdditionalProperty bag, which is a dictionary of JTokens. More details can be found below. |


#### AdditionalProperties
Additional Properties property would be a dictionary of JTokens. Please note that in the future the dictionary may be appended with additional JTokens. Also, the JTokens may also be modified to have additional data . Resource Providers should develop code that can handle modifications, and should not strongly parse this dictionary. 

```json
{
    "properties": {
        "additionalProperties": {
            "billingProperties": {
                "costCategory": "FR | FG | FS | FX | FB | None",
                "channelType": "Internal | FieldLed | CustomerLed | PartnerLed | None",
                "billingType": "Legacy| Modern",
                "paymentType": "Paid |Free| Entitlement| SponsoredPlus | Sponsored | Benefit | None",
                "workloadType": "Production | DevTest | None",
                "tier": "Standard | Premium | Priority | Unknown ",
                "billingAccount": {
                    "id": "/providers/Microsoft.Billing/billingAccounts/54731783"
                },

                "additionalStateInformation": {
                    "releaseNonDataRetentionResource": {
                    "value": True | False, 
                    "effectiveDate": "DateTime" 
                },
                    "blockNewResourceCreation": {
                     "value": True | False, 
                     "effectiveDate": "DateTime" 
                }
            }
            }
            "resourceProviderProperties": {
                "resourceProviderNamespace": "<Provider Namespace>"
            }
        }
    }
}
```
| **Element name** | Description |
| --- | --- |
| **additionalproperties.billingproperties**| Commerce object identifying billing properties associated with the subscription |
| **billingproperties.costCategory**| String. Cost categorizations used for internal subscriptions. |
| **billingproperties.channelType**| String. Indicates the sales motion that this subscription type belongs to. This can be changed if a subscription moves from one channel type to another (e.g., CustomerLed to PartnerLed)  |
| **billingproperties.billingType**| String. Indicates the commerce stack that this account is on - modern or legacy |
| **billingproperties.paymentType**| String. Differentiates how customer is paying for the subscription. This can change if customer changes from free to paid, etc.|
| **billingproperties.workloadType**| String. Indicates the importance of this subscription. DevTest subscriptions get lower SLA compared to Production ones. This property can be changed later if needed as well. |
| **billingproperties.tier**| String. Customer segment |
| **billingproperties.billingAccount**| Object. Billing account of the customer that holds customer payment instrument, address etc. The subscription is always linked to a billing account. |
| **billingproperties.billingAccount.Id**| String. Unique Id identifying the commerce object for the billing account. |
| **billingproperties.additionalStateInformation**| Commerce object identifying additional state information associated with the subscription. |
| **billingproperties.releaseNonDataRetentionResource.value**| Boolean, When true, indicates that the Non data retention resources can be released. This is a high confidence signal that would be set to true when the subscription is already in disabled state. To learn about Azure Data Retention policy after subscription has been disabled see [What happens after subscription cancellation?](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/cancel-azure-subscription#what-happens-after-subscription-cancellation). |
| **billingproperties.releaseNonDataRetentionResource.effectiveDate**| DateTime, Indicates the time when the above property got set as true |
| **billingproperties.blockNewResourceCreation.value**| Boolean, When true, indicates that new resource creation should be blocked. Existing resources functionality should not be impacted. This signal should be looked at when subscription state is Active. The service should be able to recover if the flag is reset to false. |
| **billingproperties.blockNewResourceCreation.effectiveDate**| DateTime, Indicates the time when the above property got set as true |
| **additionalproperties.resourceProviderProperties**| Required. Object identifying additional Resource Provider properties. |
| **resourceProviderProperties.resourceProviderNamespace**| Required. Resource Provider Namespace e.g.: Microsoft.Contonso. |


### Response

The response includes an HTTP status code, a set of response headers, and a response body.

**Status Code**

The resource provider should return 200 (OK) to indicate that the operation completed successfully. 202 (Accepted) can be returned to indicate that the operation will [complete asynchronously](async-api-reference.md#asynchronous-operations). Any status other than 200 (OK), 201 (Created), or 204 (No Content) will cause ARM to retry subject to their retry/backoff handling.

The location header is not followed as part of the notification; instead, the notification will be retried with a delay. It is expected that subsequent updates that are a no-op will complete synchronously.

This operation is expected to be idempotent, so the resource provider should return a 200 status code if the notification can be honored correctly. As an example, if the RP only sees an &quot;unregistered&quot; subscription state for a subscription it has no record of, it should return a 200.

**Response Headers**

Headers common to all responses.

**Response Body**

If a 200, the response body will contain the original request that was PUT per the Azure REST guidelines.

### Subscription States

**Allowed Actions when 'Warned' or 'Suspended'**

When configured correctly inside a given RP manifest, ARM will handle filtering actions (example: not allowing PUT/POST/PATCH) and returning a 409 when a given action is not allowed for a current state. This filtering does not need to be managed directly by the RP.

| SubscriptionState | Description |
|-------------| ----------------|
| **Registered** | The subscription was entitled to use your &quot;ResourceProviderNamespace&quot;.   Azure will use this subscription in future communications. You may also do any initial state setup as a result of this notification type.  When a subscription is &quot;fixed&quot; / restored from being suspended, it will return to the &quot;Registered&quot; state.  All management APIs must function (PUT/PATCH/DELETE/POST/GET), all resources must run normally; Bill normally.|
| **Warned** | The subscription has been warned (generally due to forthcoming suspension resulting from fraud or non-payment). Resources must be offline but in running (or quickly recoverable state).  Do **not** deallocate resources.   GET/DELETE management APIs must function; PUT/PATCH/POST must not.  **Don't emit any usage. Any emitted usage will be ignored.**|
| **Suspended** | The subscription has been suspended (generally due to fraud or non-payment) and the Resource Provider should stop the subscription from generating any additional usage. Pay-for-use resource should have access rights revoked when the subscription is disabled.   In such cases the Resource Provider should also mark the Resource State as &quot;Suspended.&quot;  We recommend that you treat this as a soft-delete: GET/DELETE management APIs must continue to function, yet PUT/PATCH/POST must not. **Don't emit any usage. Any emitted usage will be ignored.**|
| **Deleted** | The customer has cancelled their Windows Azure subscription and its content \*must\* be cleaned up by the resource provider.The resource provider does \*not\* receive a DELETE call for each resource – this is expected to be handled internally by the RP as a &quot;cascade&quot; deletion. Because this subscription lifecycle handling bypasses ARM's linked notification handling, a given RP will also be responsible for this internal deletion of any associated extension resources.|
| **Unregistered** | Either the customer has not yet chosen to use the resource provider, or the customer has decided to stop using the Resource Provider. Only GETs are permitted. In the case of formerly registered subscriptions, all existing tracked resources would already have been deleted by the customer explicitly. RP extension resources would require an internal &quot;cascade&quot; delete cleanup similar to the **Deleted** case above.|

