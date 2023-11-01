# Common API Details 

## Table of Contents ##
- [Proxy Request Header Modifications](#proxy-request-header-modifications) </br>
- [Client Request Headers](#client-request-headers) </br>
- [Request Query Parameters](#request-query-parameters) </br>
- [Case Insensitivity for Requests](#case-insensitivity-for-requests) </br>
- [Client Request Timeout](#client-request-timeout) </br>
- [Request Throttling](#request-throttling) </br>
- [Common API Response Details](#common-api-response-details) </br>

## Proxy Request Header Modifications ##

The resource provider proxy will preserve all the client requests headers, with the exception of modifications per the details below. The headers below are reserved and cannot be set by clients.


| Header                     | Description |  1st or 3rd Party |
| :----------------------------| :------------------------| :-----------------------------------|
| referer | Always added. Set to the full URI that the client connected to (which will be different than the RP URI, since it will have the public hostname instead of the RP hostname). This value can be used in generating FQDN for Location headers or other requests since RPs should not reference their endpoint name. |  1st and 3rd party |
| authorization | Always removed/changed. The authorization used by the client to the proxy will be different than the authorization used to communicate from the proxy to the resource provider. | 1st and 3rd party |
| x-ms-client-ip-address | Always added . Set to the client IP address used in the request; this is required since the resource provider will not have access to the client IP. | 1st and 3rd party |
| x-ms-client-principal-name | Always added. Set to the principal name / UPN of the client JWT making the request. | 1st party only |
| x-ms-client-principal-id | Added when available. Set to the principal Id of the client JWT making the request. Service principal will not have the principal Id. | 1st party only |
| x-ms-client-tenant-id | Always added. Set to the tenant ID of the client JWT making the request. | 1st party only |
| x-ms-home-tenant-id | Added for requests at or below subscription scopes. Set to the tenant ID of the subscription. This will be different from `x-ms-client-tenant-id` in cross-tenant scenarios. | 1st party only |
| x-ms-client-audience | Always added. Set to the audience of the client JWT making the request. | 1st party only |
| x-ms-client-issuer | Always added. Set to the issuer of the client JWT making the request. | 1st party only |
| x-ms-client-object-id | Always added. Set to the object Id of the client JWT making the request. Not all users have object Id. For CSP (reseller) scenarios for example, object Id is not available. | 1st party only |
| x-ms-client-app-id | Always added. Set to the app Id of the client JWT making the request. | 1st party only |
| x-ms-client-app-id-acr | Always added. Set to the app Id acr claim of the client JWT making the request. This is the application authentication context class reference claim which indicates how the client was authenticated. | 1st party only |
| x-ms-client-authorization-source | Always added. Specifies the authorization source of the token. It&#39;s value can be NotSpecified, Legacy, RoleBased, Bypassed, Direct and Management. | 1st party only |
| x-ms-client-identity-provider | Always added. Set to the identity provider of the client JWT. |1st party only |
| x-ms-client-wids | Always added. Set to the wids of the client JWT. These identify the admins of the tenant which issued the JWT. | 1st party only |
| x-ms-client-authentication-methods | Always added. Set to the authentication method references of client JWT. | 1st party only|
| x-ms-management-group-ancestors | Always added. Set to the management groups that subscription might belong to. If there are multiple ancestors they will be comma separated. Example: `d27e3b8a-3d55-44b7-b2ba-1b3ef9227527, NonProduction` | 1st party only|
| x-ms-arm-resource-routing-location | Added when extension resources or child proxy resources are routed regionally. Set to the endpoint location the request is routed to in the region's lower-case normalized form (i.e. `eastus2`). The endpoint location will be decided by the location of the closest tracked ancestor. If there is no tracked ancestor, the header will not be set. If the request is routed to the global endpoint because there is no matching regional endpoint, it will be set to an empty string. | 1st and 3rd party |

## Client Request Headers ##

Any non-reserved headers provided by the client will pass as-is to the resource provider. All requests to resource providers may include the following standard headers and **must** be supported:

| Header | Description |
| --- | --- |
| Content-Type | Set to application/json. This header is not sent in requests that don&#39;t have any content, such as all GET calls. |
| Accept-Language | Specifies the preferred language for the response; all RPs should use this header when generating error messages or client facing text. |
| x-ms-client-request-id | Caller-specified value identifying the request, in the form of a GUID with no decoration such as curly braces (e.g. `client-request-id: 9C4D50EE-2D56-4CD3-8152-34347DC9F2B0`). If the caller provides this header – the resource provider **must** log this with their traces to facilitate tracing a single request. If specified, this will be included in response information as a way to map the request if "x-ms-return-client-request-id"; is specified as "true". Because this header can be client-generated, it should not be assumed to be unique by the RP implementation. |
| x-ms-correlation-request-id | Optional. Caller-specified value identifying a set of related operations that the request belongs to, in the form of a GUID. If the caller does not specify this header, ARM will randomly generate a unique GUID. Used for tracing the correlation Id of the request; the resource provider **must** log this so that end-to-end requests can be correlated across Azure. Because this header can be client-generated or re-used for multiple requests, it should not be assumed to be unique by the RP implementation. |
| x-ms-return-client-request-id | Optional. True or false and indicates if a client-request-id should be included in the response. Default is false. |

## Request Query Parameters ##

ARM will proxy request parameters (e.g. $filter; $expand; $skipToken; etc.) as-is to the Resource Providers. It will not delete, modify or add any query parameters before relaying the request. The only exceptions are the following query parameters which are not supported on requests into ARM and will cause the request to be rejected with a 400 status code:

- `sub`
- `subId`
- `subscription`
- `subscriptionId`

## Case Insensitivity for Requests ##

When satisfying incoming requests, it is assumed that the following values are stored / indexed / compared in a case **insensitive** way:

- Resource group name
- Resource name
- Other names of entities in the URL, even if they are not resources.

## Client Request Timeout ##

Requests proxied to the resource provider are made with a client timeout of 60 seconds. If requests take more than 60 seconds please consider using asynchronous request/response pattern.

The resource provider must respond within that time interval or the client will receive a 504 (timeout) error code and will not see the response from the RP.

## Request Throttling ##

ARM provides subscription level throttling. More details on these limits can be found [here](https://azure.microsoft.com/en-us/documentation/articles/azure-subscription-service-limits/#overview).

## Common API Response Details ##

### Response Headers ###

All responses from resource providers should include the following headers:

| Header | Description |
| --- | --- |
| `Content-Type` | Set to application/json. This header is not required in responses that don&#39;t have any content. |
| `Date` | The date that the request was processed, in RFC 1123 format. |
| `x-ms-request-id` | A unique identifier for the current operation, service generated. All the resource providers \*must\* return this value in the response headers to facilitate debugging. |
| `x-ms-failurecause` | An ___optional___ header used to provide additional failure attribution on error responses. See [below](#x-ms-failurecause-header) for additional information. |

All long running operations response details are described below.

### Error Response Content ###

If the resource provider needs to return an error to any operation, it should return the appropriate HTTP error code and a message body as can be seen below. The message should be localized per the Accept-Language header specified in the original request such that it could be directly be exposed to users.

The resource providers must return the `code` and `message` fields; and should also follow the recommended schema for the "ErrorResponse" Type from the Common Types definition in the Azure Rest API Specification [repository](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/common-types/resource-management/v2/types.json). This format is inherited from [OData v4.0 schema](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Toc372793091) for error responses.

**Response Body**
```json
{
  "error": {
    "code": "",
    "message": "",
    "target": "",
    "additionalInfo": [],
    "details": [
      {
        "code": "",
        "target": "",
        "message": "",
        "additionalInfo": [
          {
            "type": "PolicyViolation",
            "info": {
              "policySetDefinitionDisplayName": "Secure the environment",
              "policySetDefinitionId": "/subscriptions/00000-00000-0000-000/providers/Microsoft.Authorization/policySetDefinitions/TestPolicySet",
              "policyDefinitionDisplayName": "Allowed locations",
              "policyDefinitionId": "/subscriptions/00000-00000-0000-000/providers/Microsoft.Authorization/policyDefinitions/TestPolicy1",
              "policyAssignmentDisplayName": "Allow Central US and WEU only",
              "policyAsssignmentId": "/subscriptions/00000-00000-0000-000/providers/Microsoft.Authorization/policyAssignments/TestAssignment1"
            }
          },
          {
            "type": "SomeOtherViolation",
            "info": {
              "innerException": "innerException Details"
            }
          }
        ]
      }
    ]
  }
}
    
```
| Element name | Type | Description |
| --- | --- | --- |
| `message` | string (required) | Describes the error in detail and provides debugging information. If Accept-Language is set in the request, it must be localized to that language. |
| `code` | string (required) | Unlocalized string which can be used to programmatically identify the error. The code should be Pascal-cased, and should serve to uniquely identify a particular class of error, for example &quot;BadArgument&quot;. |
| `target` | string (optional) | The target of the particular error (for example, the name of the property in error). |
| `details` | array (optional) | An array of additional nested error response info objects, as described by this contract. |
| `additionalInfo` | array (optional) | An array of objects with "type" (string), and "info" (object) properties. The schema of "info" is service-specific and dependent on the "type" string. |


### `x-ms-failurecause` header
Resource Providers may include the `x-ms-failurecause` HTTP header in their responses as a means of providing contextualized
failure attribution for internal telemetry and monitoring purposes. This capability is opt-in and is supported on all request
types (including both `Location` and `Azure-AsyncOperation` asynchronous operation protocols).

> The purpose of this header is to address scenarios which result in HTTP 5xx (Server Failure) or async operation failures as a
> consequence of the customer's input rather than a problem with the Resource Provider in question. In general, the expectation
> is that customer induced failures are reported with HTTP 4xx failures codes and that async operations are rejected during the
> validation phase, however the `x-ms-failurecause` header provides a backstop solution where it is not technically feasible to
> align with this guidance.

The `x-ms-failurecause` header is expected to be provided in the HTTP response headers sent by a Resource Provider in response
to a request made by ARM. It is expected to appear as follows:

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
Date: Thu, 01 Dec 2023 16:00:00 GMT
x-ms-request-id: 00000000-0000-0000-0000-00000000000
x-ms-failurecause: <responsibleParty>:<failureCategory>

{ "error": { "code": "...", "message": "..." } }
```

The value of the `x-ms-failurecause` header is a tuple consisting of the `responsibleParty` and the (optional) `failureCategory`.
This is string encoded using a single colon (`:`) delimiter between components, resulting in values like `client`, `service`, 
`client:nic-in-use`, `service:rnm`. *Note that if the `failureCategory` is not provided, the delimiter is omitted.*

The `responsibleParty` component MUST be one of `client` (caused by the customer) or `service` (caused by the Resource Provider).
The value of `responsibleParty` is used to determine whether a given failure contributes towards the Resource Provider's error
rate, with `client` failures being excluded from service-level success rate calculations. **If no value is provided, or an unrecognized `responsibleParty` is specified, this will be set to `service`.**

In addition to specifying the `responsibleParty`, Resource Providers may optionally provide a `failureCategory` value. This
value is exposed in reporting to enable service teams to classify their failure modes and provide improved visibility into
this within their centralized SLI telemetry. **The values used for the `failureCategory` field SHOULD be low cardinality and 
should not exceed 100-characters in length.**

> As a practical example, the Microsoft.Storage resource provider offers a proxy API for accessing storage account contents.
> This API is required to exactly match the semantics of the underlying storage account APIs, and this includes using *HTTP 503
> Service Unavailable* to represent client rate limiting. By setting the `x-ms-failurecause` header to `client:rate-limiting`
> it is possible for Microsoft.Storage to remove a significant source of signal error in their centralized SLIs and improve the
> accuracy of their integrated monitoring and reporting.

### Max Request Body Size ###

The maximum size of a request body that ARM will accept is 4 MB.

Any request with a body larger than 4 MB will not be sent to the resource provider, and a **413 Payload Too Large** will be returned to the client.

### Max Response Size ###

In all the calls that ARM makes to the resource provider, the maximum size of a response that ARM will accept from the resource providers is 8 MB **after compression**. Any response greater than 8 MB in size should use pagination instead.

In addition, ARM cannot support any single resource response that exceeds 20MB in size, which is the hard limit currently set. When this happens, the payload will be dropped and a **502 Bad Gateway** error will be returned to the client. In general, APIs exposed by the resource provider should be designed to transmit relatively little data in keeping with the management nature of the API.

### Transfer-Encoding ###

Chunked transfer encoding is supported and can be used for larger payloads.

### Redirecting the Client ###

The Resource Provider may return a 307 response code to the customer if they want to expose a direct URL / host (with no proxy) to the user. As an example – when downloading a large file, the RP may return a 307 to a URL on storage to download that file.

The Azure Resource Manager will **not** follow any redirects and will instead proxy them directly back to the client.
