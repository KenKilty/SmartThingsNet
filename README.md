# SmartThingsNet - C# library for the SmartThings API

# Overview

The SmartThings API supports [REST](https://en.wikipedia.org/wiki/Representational_state_transfer), resources are protected with [OAuth 2.0 Bearer Tokens](https://tools.ietf.org/html/rfc6750#section-2.1), and all responses are sent as [JSON](http://www.json.org/).

# Authentication

All SmartThings resources are protected with [OAuth 2.0 Bearer Tokens](https://tools.ietf.org/html/rfc6750#section-2.1) sent on the request as an `Authorization: Bearer <TOKEN>` header.

## Token types

Only personal access tokens are supported. These are used to interact with the API for non-SmartApp use cases. They can be created and managed on the [personal access tokens page](https://account.smartthings.com/tokens).

## OAuth2 scopes

Operations may be protected by one or more OAuth security schemes, which specify the required permissions.
Each scope for a given scheme is required.
If multiple schemes are specified (not common), you may use either scheme.

Personal access token scopes are associated with the specific permissions authorized when creating them.

Scopes are generally in the form `permission:entity-type:entity-id`.

**An `*` used for the `entity-id` specifies that the permission may be applied to all entities that the token type has access to, or may be replaced with a specific ID.**

For more information about authrization and permissions, please see the [Authorization and permissions guide](https://smartthings.developer.samsung.com/develop/guides/smartapps/auth-and-permissions.html).

# Errors

The SmartThings API uses conventional HTTP response codes to indicate the success or failure of a request.
In general, a `2XX` response code indicates success, a `4XX` response code indicates an error given the inputs for the request, and a `5XX` response code indicates a failure on the SmartThings platform.

API errors will contain a JSON response body with more information about the error:

```json
{
  \"requestId\": \"031fec1a-f19f-470a-a7da-710569082846\"
  \"error\": {
    \"code\": \"ConstraintViolationError\",
    \"message\": \"Validation errors occurred while process your request.\",
    \"details\": [
      { \"code\": \"PatternError\", \"target\": \"latitude\", \"message\": \"Invalid format.\" },
      { \"code\": \"SizeError\", \"target\": \"name\", \"message\": \"Too small.\" },
      { \"code\": \"SizeError\", \"target\": \"description\", \"message\": \"Too big.\" }
    ]
  }
}
```

## Error Response Body

The error response attributes are:

| Property | Type | Required | Description |
| - -- | - -- | - -- | - -- |
| requestId | String | No | A request identifier that can be used to correlate an error to additional logging on the SmartThings servers.
| error | Error | **Yes** | The Error object, documented below.

## Error Object

The Error object contains the following attributes:

| Property | Type | Required | Description |
| - -- | - -- | - -- | - -- |
| code | String | **Yes** | A SmartThings-defined error code that serves as a more specific indicator of the error than the HTTP error code specified in the response. See [SmartThings Error Codes](#section/Errors/SmartThings-Error-Codes) for more information.
| message | String | **Yes** | A description of the error, intended to aid developers in debugging of error responses.
| target | String | No | The target of the particular error. For example, it could be the name of the property that caused the error.
| details | Error[] | No | An array of Error objects that typically represent distinct, related errors that occurred during the request. As an optional attribute, this may be null or an empty array.

## Standard HTTP Error Codes

The following table lists the most common HTTP error response:

| Code | Name | Description |
| - -- | - -- | - -- |
| 400 | Bad Request | The client has issued an invalid request. This is commonly used to specify validation errors in a request payload.
| 401 | Unauthorized | Authorization for the API is required, but the request has not been authenticated.
| 403 | Forbidden | The request has been authenticated but does not have appropriate permissions, or a requested resource is not found.
| 404 | Not Found | Specifies the requested path does not exist.
| 406 | Not Acceptable | The client has requested a MIME type via the Accept header for a value not supported by the server.
| 415 | Unsupported Media Type | The client has defined a contentType header that is not supported by the server.
| 422 | Unprocessable Entity | The client has made a valid request, but the server cannot process it. This is often used for APIs for which certain limits have been exceeded.
| 429 | Too Many Requests | The client has exceeded the number of requests allowed for a given time window.
| 500 | Internal Server Error | An unexpected error on the SmartThings servers has occurred. These errors should be rare.
| 501 | Not Implemented | The client request was valid and understood by the server, but the requested feature has yet to be implemented. These errors should be rare.

## SmartThings Error Codes

SmartThings specifies several standard custom error codes.
These provide more information than the standard HTTP error response codes.
The following table lists the standard SmartThings error codes and their description:

| Code | Typical HTTP Status Codes | Description |
| - -- | - -- | - -- |
| PatternError | 400, 422 | The client has provided input that does not match the expected pattern.
| ConstraintViolationError | 422 | The client has provided input that has violated one or more constraints.
| NotNullError | 422 | The client has provided a null input for a field that is required to be non-null.
| NullError | 422 | The client has provided an input for a field that is required to be null.
| NotEmptyError | 422 | The client has provided an empty input for a field that is required to be non-empty.
| SizeError | 400, 422 | The client has provided a value that does not meet size restrictions.
| Unexpected Error | 500 | A non-recoverable error condition has occurred. Indicates a problem occurred on the SmartThings server that is no fault of the client.
| UnprocessableEntityError | 422 | The client has sent a malformed request body.
| TooManyRequestError | 429 | The client issued too many requests too quickly.
| LimitError | 422 | The client has exceeded certain limits an API enforces.
| UnsupportedOperationError | 400, 422 | The client has issued a request to a feature that currently isn't supported by the SmartThings platform. These should be rare.

## Custom Error Codes

An API may define its own error codes where appropriate.
These custom error codes are documented as part of that specific API's documentation.

# Warnings
The SmartThings API issues warning messages via standard HTTP Warning headers. These messages do not represent a request failure, but provide additional information that the requester might want to act upon.
For instance a warning will be issued if you are using an old API version.

# API Versions

The SmartThings API supports both path and header-based versioning.
The following are equivalent:

- https://api.smartthings.com/v1/locations
- https://api.smartthings.com/locations with header `Accept: application/vnd.smartthings+json;v=1`

Currently, only version 1 is available.

# Paging

Operations that return a list of objects return a paginated response.
The `_links` object contains the items returned, and links to the next and previous result page, if applicable.

```json
{
  \"items\": [
    {
      \"locationId\": \"6b3d1909-1e1c-43ec-adc2-5f941de4fbf9\",
      \"name\": \"Home\"
    },
    {
      \"locationId\": \"6b3d1909-1e1c-43ec-adc2-5f94d6g4fbf9\",
      \"name\": \"Work\"
    }
    ....
  ],
  \"_links\": {
    \"next\": {
      \"href\": \"https://api.smartthings.com/v1/locations?page=3\"
    },
    \"previous\": {
      \"href\": \"https://api.smartthings.com/v1/locations?page=1\"
    }
  }
}
```

# Localization

Some SmartThings API's support localization.
Specific information regarding localization endpoints are documented in the API itself.
However, the following should apply to all endpoints that support localization.

## Fallback Patterns

When making a request with the `Accept-Language` header, this fallback pattern is observed.
* Response will be translated with exact locale tag.
* If a translation does not exist for the requested language and region, the translation for the language will be returned.
* If a translation does not exist for the language, English (en) will be returned.
* Finally, an untranslated response will be returned in the absense of the above translations.

## Accept-Language Header
The format of the `Accept-Language` header follows what is defined in [RFC 7231, section 5.3.5](https://tools.ietf.org/html/rfc7231#section-5.3.5)

## Content-Language
The `Content-Language` header should be set on the response from the server to indicate which translation was given back to the client.
The absense of the header indicates that the server did not recieve a request with the `Accept-Language` header set.


This C# SDK is automatically generated by the [OpenAPI Generator](https://openapi-generator.tech) project (then modified to work):

- API version: 1.0-PREVIEW
- SDK version: 1.0.0
- Build package: SmartThingsNet.codegen.languages.CSharpNetCoreClientCodegen

<a name="frameworks-supported"></a>
## Frameworks supported
- .NET Core >=1.0
- .NET Framework >=4.6
- Mono/Xamarin >=vNext

<a name="dependencies"></a>
## Dependencies

- [RestSharp](https://www.nuget.org/packages/RestSharp) - 106.10.1 or later
- [Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/) - 12.0.1 or later
- [JsonSubTypes](https://www.nuget.org/packages/JsonSubTypes/) - 1.5.2 or later
- [System.ComponentModel.Annotations](https://www.nuget.org/packages/System.ComponentModel.Annotations) - 4.5.0 or later

The DLLs included in the package may not be the latest version. We recommend using [NuGet](https://docs.nuget.org/consume/installing-nuget) to obtain the latest version of the packages:
```
Install-Package RestSharp
Install-Package Newtonsoft.Json
Install-Package JsonSubTypes
Install-Package System.ComponentModel.Annotations
```

NOTE: RestSharp versions greater than 105.1.0 have a bug which causes file uploads to fail. See [RestSharp#742](https://github.com/restsharp/RestSharp/issues/742)

<a name="installation"></a>
## Installation
Generate the DLL using your preferred tool (e.g. `dotnet build`)

Then include the DLL (under the `bin` folder) in the C# project, and use the namespaces:
```csharp
using SmartThingsNet.Api;
using SmartThingsNet.Client;
using SmartThingsNet.Model;
```
<a name="getting-started"></a>
## Getting Started

```csharp
using System.Collections.Generic;
using System.Diagnostics;
using SmartThingsNet.Api;
using SmartThingsNet.Client;
using SmartThingsNet.Model;

namespace Example
{
    public class Example
    {
        public static void Main()
        {

            Configuration config = new Configuration();
            config.BasePath = "https://api.smartthings.com/v1";
            // Configure OAuth2 access token for authorization: Bearer
            config.AccessToken = "YOUR_ACCESS_TOKEN";

            var apiInstance = new AppsApi(config);
            var createOrUpdateAppRequest = new CreateAppRequest(); // CreateAppRequest | 
            var signatureType = signatureType_example;  // string | The Signature Type of the application. For WEBHOOK_SMART_APP only.  (optional) 
            var requireConfirmation = true;  // bool? | Override default configuration to use either PING or CONFIRMATION lifecycle. For WEBHOOK_SMART_APP only.  (optional) 

            try
            {
                // Create an app.
                CreateAppResponse result = apiInstance.CreateApp(createOrUpdateAppRequest, signatureType, requireConfirmation);
                Debug.WriteLine(result);
            }
            catch (ApiException e)
            {
                Debug.Print("Exception when calling AppsApi.CreateApp: " + e.Message );
                Debug.Print("Status Code: "+ e.ErrorCode);
                Debug.Print(e.StackTrace);
            }

        }
    }
}
```

<a name="documentation-for-api-endpoints"></a>
## Documentation for API Endpoints

All URIs are relative to *https://api.smartthings.com/v1*

Class | Method | HTTP request | Description
------------ | ------------- | ------------- | -------------
*AppsApi* | [**CreateApp**](docs/AppsApi.md#createapp) | **POST** /apps | Create an app.
*AppsApi* | [**DeleteApp**](docs/AppsApi.md#deleteapp) | **DELETE** /apps/{appNameOrId} | Delete an app.
*AppsApi* | [**GenerateAppOauth**](docs/AppsApi.md#generateappoauth) | **POST** /apps/{appNameOrId}/oauth/generate | Generate an app's oauth client/secret.
*AppsApi* | [**GetApp**](docs/AppsApi.md#getapp) | **GET** /apps/{appNameOrId} | Get an app.
*AppsApi* | [**GetAppOauth**](docs/AppsApi.md#getappoauth) | **GET** /apps/{appNameOrId}/oauth | Get an app's oauth settings.
*AppsApi* | [**GetAppSettings**](docs/AppsApi.md#getappsettings) | **GET** /apps/{appNameOrId}/settings | Get settings.
*AppsApi* | [**ListApps**](docs/AppsApi.md#listapps) | **GET** /apps | List apps.
*AppsApi* | [**Register**](docs/AppsApi.md#register) | **PUT** /apps/{appNameOrId}/register | Sends a confirmation request to App.
*AppsApi* | [**UpdateApp**](docs/AppsApi.md#updateapp) | **PUT** /apps/{appNameOrId} | Update an app.
*AppsApi* | [**UpdateAppOauth**](docs/AppsApi.md#updateappoauth) | **PUT** /apps/{appNameOrId}/oauth | Update an app's oauth settings.
*AppsApi* | [**UpdateAppSettings**](docs/AppsApi.md#updateappsettings) | **PUT** /apps/{appNameOrId}/settings | Update settings.
*AppsApi* | [**UpdateSignatureType**](docs/AppsApi.md#updatesignaturetype) | **PUT** /apps/{appNameOrId}/signature-type | Update an app's signature type.
*DeviceprofilesApi* | [**CreateDeviceProfile**](docs/DeviceprofilesApi.md#createdeviceprofile) | **POST** /deviceprofiles | Create a device profile
*DeviceprofilesApi* | [**DeleteDeviceProfile**](docs/DeviceprofilesApi.md#deletedeviceprofile) | **DELETE** /deviceprofiles/{deviceProfileId} | Delete a device profile
*DeviceprofilesApi* | [**GetDeviceProfile**](docs/DeviceprofilesApi.md#getdeviceprofile) | **GET** /deviceprofiles/{deviceProfileId} | GET a device profile
*DeviceprofilesApi* | [**ListDeviceProfiles**](docs/DeviceprofilesApi.md#listdeviceprofiles) | **GET** /deviceprofiles | List all device profiles for the authenticated user
*DeviceprofilesApi* | [**UpdateDeviceProfile**](docs/DeviceprofilesApi.md#updatedeviceprofile) | **PUT** /deviceprofiles/{deviceProfileId} | Update a device profile.
*DevicesApi* | [**CreateDeviceEvents**](docs/DevicesApi.md#createdeviceevents) | **POST** /devices/{deviceId}/events | Create Device Events.
*DevicesApi* | [**DeleteDevice**](docs/DevicesApi.md#deletedevice) | **DELETE** /devices/{deviceId} | Delete a Device.
*DevicesApi* | [**ExecuteDeviceCommands**](docs/DevicesApi.md#executedevicecommands) | **POST** /devices/{deviceId}/commands | Execute commands on device.
*DevicesApi* | [**GetDevice**](docs/DevicesApi.md#getdevice) | **GET** /devices/{deviceId} | Get a device's description.
*DevicesApi* | [**GetDeviceComponentStatus**](docs/DevicesApi.md#getdevicecomponentstatus) | **GET** /devices/{deviceId}/components/{componentId}/status | Get a device component's status.
*DevicesApi* | [**GetDeviceStatus**](docs/DevicesApi.md#getdevicestatus) | **GET** /devices/{deviceId}/status | Get the full status of a device.
*DevicesApi* | [**GetDeviceStatusByCapability**](docs/DevicesApi.md#getdevicestatusbycapability) | **GET** /devices/{deviceId}/components/{componentId}/capabilities/{capabilityId}/status | Get a capability's status.
*DevicesApi* | [**GetDevices**](docs/DevicesApi.md#getdevices) | **GET** /devices | List devices.
*DevicesApi* | [**InstallDevice**](docs/DevicesApi.md#installdevice) | **POST** /devices | Install a Device.
*DevicesApi* | [**UpdateDevice**](docs/DevicesApi.md#updatedevice) | **PUT** /devices/{deviceId} | Update a device.
*InstalledappsApi* | [**CreateInstalledAppEvents**](docs/InstalledappsApi.md#createinstalledappevents) | **POST** /installedapps/{installedAppId}/events | Create Installed App events.
*InstalledappsApi* | [**DeleteInstallation**](docs/InstalledappsApi.md#deleteinstallation) | **DELETE** /installedapps/{installedAppId} | Delete an installed app.
*InstalledappsApi* | [**GetInstallation**](docs/InstalledappsApi.md#getinstallation) | **GET** /installedapps/{installedAppId} | Get an installed app.
*InstalledappsApi* | [**GetInstallationConfig**](docs/InstalledappsApi.md#getinstallationconfig) | **GET** /installedapps/{installedAppId}/configs/{configurationId} | Get an installed app configuration.
*InstalledappsApi* | [**ListInstallationConfig**](docs/InstalledappsApi.md#listinstallationconfig) | **GET** /installedapps/{installedAppId}/configs | List an installed app's configurations.
*InstalledappsApi* | [**ListInstallations**](docs/InstalledappsApi.md#listinstallations) | **GET** /installedapps | List installed apps.
*LocationsApi* | [**CreateLocation**](docs/LocationsApi.md#createlocation) | **POST** /locations | Create a Location.
*LocationsApi* | [**DeleteLocation**](docs/LocationsApi.md#deletelocation) | **DELETE** /locations/{locationId} | Delete a Location.
*LocationsApi* | [**GetLocation**](docs/LocationsApi.md#getlocation) | **GET** /locations/{locationId} | Get a Location.
*LocationsApi* | [**ListLocations**](docs/LocationsApi.md#listlocations) | **GET** /locations | List Locations.
*LocationsApi* | [**UpdateLocation**](docs/LocationsApi.md#updatelocation) | **PUT** /locations/{locationId} | Update a Location.
*RoomsApi* | [**CreateRoom**](docs/RoomsApi.md#createroom) | **POST** /locations/{locationId}/rooms | Create a Room.
*RoomsApi* | [**DeleteRoom**](docs/RoomsApi.md#deleteroom) | **DELETE** /locations/{locationId}/rooms/{roomId} | Delete a Room.
*RoomsApi* | [**GetRoom**](docs/RoomsApi.md#getroom) | **GET** /locations/{locationId}/rooms/{roomId} | Get a Room.
*RoomsApi* | [**ListRooms**](docs/RoomsApi.md#listrooms) | **GET** /locations/{locationId}/rooms | List Rooms.
*RoomsApi* | [**UpdateRoom**](docs/RoomsApi.md#updateroom) | **PUT** /locations/{locationId}/rooms/{roomId} | Update a Room.
*RulesApi* | [**CreateRule**](docs/RulesApi.md#createrule) | **POST** /rules | Create a rule
*RulesApi* | [**DeleteRule**](docs/RulesApi.md#deleterule) | **DELETE** /rules/{ruleId} | Delete a rule
*RulesApi* | [**ExecuteRule**](docs/RulesApi.md#executerule) | **POST** /rules/execute/{ruleId} | Execute a rule
*RulesApi* | [**GetRule**](docs/RulesApi.md#getrule) | **GET** /rules/{ruleId} | Get a Rule
*RulesApi* | [**ListRules**](docs/RulesApi.md#listrules) | **GET** /rules | Rules list
*RulesApi* | [**UpdateRule**](docs/RulesApi.md#updaterule) | **PUT** /rules/{ruleId} | Update a rule
*ScenesApi* | [**ExecuteScene**](docs/ScenesApi.md#executescene) | **POST** /scenes/{sceneId}/execute | Execute Scene
*ScenesApi* | [**ListScenes**](docs/ScenesApi.md#listscenes) | **GET** /scenes | List Scenes
*SchedulesApi* | [**CreateSchedule**](docs/SchedulesApi.md#createschedule) | **POST** /installedapps/{installedAppId}/schedules | Save an installed app schedule.
*SchedulesApi* | [**DeleteSchedule**](docs/SchedulesApi.md#deleteschedule) | **DELETE** /installedapps/{installedAppId}/schedules/{scheduleName} | Delete a schedule.
*SchedulesApi* | [**DeleteSchedules**](docs/SchedulesApi.md#deleteschedules) | **DELETE** /installedapps/{installedAppId}/schedules | Delete all of an installed app's schedules.
*SchedulesApi* | [**GetSchedule**](docs/SchedulesApi.md#getschedule) | **GET** /installedapps/{installedAppId}/schedules/{scheduleName} | Get an installed app's schedule.
*SchedulesApi* | [**GetSchedules**](docs/SchedulesApi.md#getschedules) | **GET** /installedapps/{installedAppId}/schedules | List installed app schedules.
*SubscriptionsApi* | [**DeleteAllSubscriptions**](docs/SubscriptionsApi.md#deleteallsubscriptions) | **DELETE** /installedapps/{installedAppId}/subscriptions | Delete all of an installed app's subscriptions.
*SubscriptionsApi* | [**DeleteSubscription**](docs/SubscriptionsApi.md#deletesubscription) | **DELETE** /installedapps/{installedAppId}/subscriptions/{subscriptionId} | Delete an installed app's subscription.
*SubscriptionsApi* | [**GetSubscription**](docs/SubscriptionsApi.md#getsubscription) | **GET** /installedapps/{installedAppId}/subscriptions/{subscriptionId} | Get an installed app's subscription.
*SubscriptionsApi* | [**ListSubscriptions**](docs/SubscriptionsApi.md#listsubscriptions) | **GET** /installedapps/{installedAppId}/subscriptions | List an installed app's subscriptions.
*SubscriptionsApi* | [**SaveSubscription**](docs/SubscriptionsApi.md#savesubscription) | **POST** /installedapps/{installedAppId}/subscriptions | Create a subscription for an installed app.


<a name="documentation-for-models"></a>
## Documentation for Models

 - [Model.Action](docs/Action.md)
 - [Model.AdhocMessage](docs/AdhocMessage.md)
 - [Model.AdhocMessageTemplate](docs/AdhocMessageTemplate.md)
 - [Model.App](docs/App.md)
 - [Model.AppClassification](docs/AppClassification.md)
 - [Model.AppDeviceDetails](docs/AppDeviceDetails.md)
 - [Model.AppOAuth](docs/AppOAuth.md)
 - [Model.AppTargetStatus](docs/AppTargetStatus.md)
 - [Model.AppType](docs/AppType.md)
 - [Model.AppUISettings](docs/AppUISettings.md)
 - [Model.Argument](docs/Argument.md)
 - [Model.ArrayOperand](docs/ArrayOperand.md)
 - [Model.AttributeProperties](docs/AttributeProperties.md)
 - [Model.AttributePropertiesData](docs/AttributePropertiesData.md)
 - [Model.AttributePropertiesUnit](docs/AttributePropertiesUnit.md)
 - [Model.AttributeSchema](docs/AttributeSchema.md)
 - [Model.AttributeState](docs/AttributeState.md)
 - [Model.BetweenCondition](docs/BetweenCondition.md)
 - [Model.CapabilityAttribute](docs/CapabilityAttribute.md)
 - [Model.CapabilityAttributeEnumCommands](docs/CapabilityAttributeEnumCommands.md)
 - [Model.CapabilityCommand](docs/CapabilityCommand.md)
 - [Model.CapabilityReference](docs/CapabilityReference.md)
 - [Model.CapabilitySubscriptionDetail](docs/CapabilitySubscriptionDetail.md)
 - [Model.CapabilitySummary](docs/CapabilitySummary.md)
 - [Model.ChangesCondition](docs/ChangesCondition.md)
 - [Model.CommandAction](docs/CommandAction.md)
 - [Model.ComponentTranslations](docs/ComponentTranslations.md)
 - [Model.Condition](docs/Condition.md)
 - [Model.ConditionAggregationMode](docs/ConditionAggregationMode.md)
 - [Model.ConfigEntry](docs/ConfigEntry.md)
 - [Model.CreateAppRequest](docs/CreateAppRequest.md)
 - [Model.CreateAppResponse](docs/CreateAppResponse.md)
 - [Model.CreateDeviceProfileRequest](docs/CreateDeviceProfileRequest.md)
 - [Model.CreateInstalledAppEventsRequest](docs/CreateInstalledAppEventsRequest.md)
 - [Model.CreateLocationRequest](docs/CreateLocationRequest.md)
 - [Model.CreateOrUpdateLambdaSmartAppRequest](docs/CreateOrUpdateLambdaSmartAppRequest.md)
 - [Model.CreateOrUpdateWebhookSmartAppRequest](docs/CreateOrUpdateWebhookSmartAppRequest.md)
 - [Model.CreateRoomRequest](docs/CreateRoomRequest.md)
 - [Model.CronSchedule](docs/CronSchedule.md)
 - [Model.DashboardCardLifecycle](docs/DashboardCardLifecycle.md)
 - [Model.DateOperand](docs/DateOperand.md)
 - [Model.DateTimeOperand](docs/DateTimeOperand.md)
 - [Model.DayOfWeek](docs/DayOfWeek.md)
 - [Model.DeleteInstalledAppResponse](docs/DeleteInstalledAppResponse.md)
 - [Model.Device](docs/Device.md)
 - [Model.DeviceActivity](docs/DeviceActivity.md)
 - [Model.DeviceCategory](docs/DeviceCategory.md)
 - [Model.DeviceCommand](docs/DeviceCommand.md)
 - [Model.DeviceCommandsEvent](docs/DeviceCommandsEvent.md)
 - [Model.DeviceCommandsEventCommand](docs/DeviceCommandsEventCommand.md)
 - [Model.DeviceCommandsRequest](docs/DeviceCommandsRequest.md)
 - [Model.DeviceComponent](docs/DeviceComponent.md)
 - [Model.DeviceComponentReference](docs/DeviceComponentReference.md)
 - [Model.DeviceConfig](docs/DeviceConfig.md)
 - [Model.DeviceEvent](docs/DeviceEvent.md)
 - [Model.DeviceEventsRequest](docs/DeviceEventsRequest.md)
 - [Model.DeviceHealthDetail](docs/DeviceHealthDetail.md)
 - [Model.DeviceHealthEvent](docs/DeviceHealthEvent.md)
 - [Model.DeviceInstallRequest](docs/DeviceInstallRequest.md)
 - [Model.DeviceInstallRequestApp](docs/DeviceInstallRequestApp.md)
 - [Model.DeviceIntegrationType](docs/DeviceIntegrationType.md)
 - [Model.DeviceLifecycle](docs/DeviceLifecycle.md)
 - [Model.DeviceLifecycleDetail](docs/DeviceLifecycleDetail.md)
 - [Model.DeviceLifecycleEvent](docs/DeviceLifecycleEvent.md)
 - [Model.DeviceLifecycleMove](docs/DeviceLifecycleMove.md)
 - [Model.DeviceNetworkSecurityLevel](docs/DeviceNetworkSecurityLevel.md)
 - [Model.DeviceOperand](docs/DeviceOperand.md)
 - [Model.DeviceProfile](docs/DeviceProfile.md)
 - [Model.DeviceProfileReference](docs/DeviceProfileReference.md)
 - [Model.DeviceProfileStatus](docs/DeviceProfileStatus.md)
 - [Model.DeviceResults](docs/DeviceResults.md)
 - [Model.DeviceStateEvent](docs/DeviceStateEvent.md)
 - [Model.DeviceStatus](docs/DeviceStatus.md)
 - [Model.DeviceSubscriptionDetail](docs/DeviceSubscriptionDetail.md)
 - [Model.DthDeviceDetails](docs/DthDeviceDetails.md)
 - [Model.EndpointApp](docs/EndpointApp.md)
 - [Model.EqualsCondition](docs/EqualsCondition.md)
 - [Model.Error](docs/Error.md)
 - [Model.ErrorResponse](docs/ErrorResponse.md)
 - [Model.EventType](docs/EventType.md)
 - [Model.EveryAction](docs/EveryAction.md)
 - [Model.ExecutionResult](docs/ExecutionResult.md)
 - [Model.GenerateAppOAuthRequest](docs/GenerateAppOAuthRequest.md)
 - [Model.GenerateAppOAuthResponse](docs/GenerateAppOAuthResponse.md)
 - [Model.GetAppSettingsResponse](docs/GetAppSettingsResponse.md)
 - [Model.GreaterThanCondition](docs/GreaterThanCondition.md)
 - [Model.GreaterThanOrEqualsCondition](docs/GreaterThanOrEqualsCondition.md)
 - [Model.HubHealthDetail](docs/HubHealthDetail.md)
 - [Model.HubHealthEvent](docs/HubHealthEvent.md)
 - [Model.IconImage](docs/IconImage.md)
 - [Model.IfAction](docs/IfAction.md)
 - [Model.IfActionAllOf](docs/IfActionAllOf.md)
 - [Model.InstallConfiguration](docs/InstallConfiguration.md)
 - [Model.InstallConfigurationDetail](docs/InstallConfigurationDetail.md)
 - [Model.InstallConfigurationStatus](docs/InstallConfigurationStatus.md)
 - [Model.InstalledApp](docs/InstalledApp.md)
 - [Model.InstalledAppIconImage](docs/InstalledAppIconImage.md)
 - [Model.InstalledAppLifecycle](docs/InstalledAppLifecycle.md)
 - [Model.InstalledAppLifecycleError](docs/InstalledAppLifecycleError.md)
 - [Model.InstalledAppLifecycleEvent](docs/InstalledAppLifecycleEvent.md)
 - [Model.InstalledAppStatus](docs/InstalledAppStatus.md)
 - [Model.InstalledAppType](docs/InstalledAppType.md)
 - [Model.InstalledAppUi](docs/InstalledAppUi.md)
 - [Model.Interval](docs/Interval.md)
 - [Model.IntervalUnit](docs/IntervalUnit.md)
 - [Model.IrDeviceDetails](docs/IrDeviceDetails.md)
 - [Model.IrDeviceDetailsFunctionCodes](docs/IrDeviceDetailsFunctionCodes.md)
 - [Model.IsaResults](docs/IsaResults.md)
 - [Model.LambdaSmartApp](docs/LambdaSmartApp.md)
 - [Model.LessThanCondition](docs/LessThanCondition.md)
 - [Model.LessThanOrEqualsCondition](docs/LessThanOrEqualsCondition.md)
 - [Model.Link](docs/Link.md)
 - [Model.Links](docs/Links.md)
 - [Model.LocaleReference](docs/LocaleReference.md)
 - [Model.LocaleVariables](docs/LocaleVariables.md)
 - [Model.Location](docs/Location.md)
 - [Model.LocationAction](docs/LocationAction.md)
 - [Model.LocationAttribute](docs/LocationAttribute.md)
 - [Model.LocationOperand](docs/LocationOperand.md)
 - [Model.Message](docs/Message.md)
 - [Model.MessageConfig](docs/MessageConfig.md)
 - [Model.MessageTemplate](docs/MessageTemplate.md)
 - [Model.MessageType](docs/MessageType.md)
 - [Model.Mode](docs/Mode.md)
 - [Model.ModeConfig](docs/ModeConfig.md)
 - [Model.ModeEvent](docs/ModeEvent.md)
 - [Model.ModeSubscriptionDetail](docs/ModeSubscriptionDetail.md)
 - [Model.Notice](docs/Notice.md)
 - [Model.NoticeAction](docs/NoticeAction.md)
 - [Model.NoticeCode](docs/NoticeCode.md)
 - [Model.OnceSchedule](docs/OnceSchedule.md)
 - [Model.Operand](docs/Operand.md)
 - [Model.OperandAggregationMode](docs/OperandAggregationMode.md)
 - [Model.Owner](docs/Owner.md)
 - [Model.PagedApp](docs/PagedApp.md)
 - [Model.PagedApps](docs/PagedApps.md)
 - [Model.PagedDeviceProfiles](docs/PagedDeviceProfiles.md)
 - [Model.PagedDevices](docs/PagedDevices.md)
 - [Model.PagedInstallConfigurations](docs/PagedInstallConfigurations.md)
 - [Model.PagedInstalledApps](docs/PagedInstalledApps.md)
 - [Model.PagedLocation](docs/PagedLocation.md)
 - [Model.PagedLocations](docs/PagedLocations.md)
 - [Model.PagedMessageTemplate](docs/PagedMessageTemplate.md)
 - [Model.PagedRooms](docs/PagedRooms.md)
 - [Model.PagedRules](docs/PagedRules.md)
 - [Model.PagedSchedules](docs/PagedSchedules.md)
 - [Model.PagedSubscriptions](docs/PagedSubscriptions.md)
 - [Model.ParentType](docs/ParentType.md)
 - [Model.PermissionConfig](docs/PermissionConfig.md)
 - [Model.PredefinedMessage](docs/PredefinedMessage.md)
 - [Model.PrincipalType](docs/PrincipalType.md)
 - [Model.Room](docs/Room.md)
 - [Model.Rule](docs/Rule.md)
 - [Model.RuleAllOf](docs/RuleAllOf.md)
 - [Model.RuleExecutionResponse](docs/RuleExecutionResponse.md)
 - [Model.RuleMetadata](docs/RuleMetadata.md)
 - [Model.RuleRequest](docs/RuleRequest.md)
 - [Model.SceneAction](docs/SceneAction.md)
 - [Model.SceneArgument](docs/SceneArgument.md)
 - [Model.SceneCapability](docs/SceneCapability.md)
 - [Model.SceneCommand](docs/SceneCommand.md)
 - [Model.SceneComponent](docs/SceneComponent.md)
 - [Model.SceneConfig](docs/SceneConfig.md)
 - [Model.SceneDevice](docs/SceneDevice.md)
 - [Model.SceneDeviceGroup](docs/SceneDeviceGroup.md)
 - [Model.SceneDeviceGroupRequest](docs/SceneDeviceGroupRequest.md)
 - [Model.SceneDeviceRequest](docs/SceneDeviceRequest.md)
 - [Model.SceneLifecycle](docs/SceneLifecycle.md)
 - [Model.SceneLifecycleDetail](docs/SceneLifecycleDetail.md)
 - [Model.SceneLifecycleEvent](docs/SceneLifecycleEvent.md)
 - [Model.SceneMode](docs/SceneMode.md)
 - [Model.SceneModeRequest](docs/SceneModeRequest.md)
 - [Model.ScenePagedResult](docs/ScenePagedResult.md)
 - [Model.SceneRequest](docs/SceneRequest.md)
 - [Model.SceneSecurityModeRequest](docs/SceneSecurityModeRequest.md)
 - [Model.SceneSleepRequest](docs/SceneSleepRequest.md)
 - [Model.SceneSummary](docs/SceneSummary.md)
 - [Model.Schedule](docs/Schedule.md)
 - [Model.ScheduleRequest](docs/ScheduleRequest.md)
 - [Model.SecurityArmStateDetail](docs/SecurityArmStateDetail.md)
 - [Model.SecurityArmStateEvent](docs/SecurityArmStateEvent.md)
 - [Model.SignatureType](docs/SignatureType.md)
 - [Model.SimpleCondition](docs/SimpleCondition.md)
 - [Model.SimpleValue](docs/SimpleValue.md)
 - [Model.SingleOperandCondition](docs/SingleOperandCondition.md)
 - [Model.SleepAction](docs/SleepAction.md)
 - [Model.SmartAppDashboardCardEventRequest](docs/SmartAppDashboardCardEventRequest.md)
 - [Model.SmartAppEventRequest](docs/SmartAppEventRequest.md)
 - [Model.StandardSuccessResponse](docs/StandardSuccessResponse.md)
 - [Model.StringConfig](docs/StringConfig.md)
 - [Model.Subscription](docs/Subscription.md)
 - [Model.SubscriptionDelete](docs/SubscriptionDelete.md)
 - [Model.SubscriptionFilter](docs/SubscriptionFilter.md)
 - [Model.SubscriptionFilterTypes](docs/SubscriptionFilterTypes.md)
 - [Model.SubscriptionMode](docs/SubscriptionMode.md)
 - [Model.SubscriptionRequest](docs/SubscriptionRequest.md)
 - [Model.SubscriptionSource](docs/SubscriptionSource.md)
 - [Model.SubscriptionTarget](docs/SubscriptionTarget.md)
 - [Model.TimeOperand](docs/TimeOperand.md)
 - [Model.TimeReference](docs/TimeReference.md)
 - [Model.TimerEvent](docs/TimerEvent.md)
 - [Model.TimerType](docs/TimerType.md)
 - [Model.UpdateAppOAuthRequest](docs/UpdateAppOAuthRequest.md)
 - [Model.UpdateAppRequest](docs/UpdateAppRequest.md)
 - [Model.UpdateAppSettingsRequest](docs/UpdateAppSettingsRequest.md)
 - [Model.UpdateAppSettingsResponse](docs/UpdateAppSettingsResponse.md)
 - [Model.UpdateDeviceProfileRequest](docs/UpdateDeviceProfileRequest.md)
 - [Model.UpdateDeviceRequest](docs/UpdateDeviceRequest.md)
 - [Model.UpdateDeviceRequestComponents](docs/UpdateDeviceRequestComponents.md)
 - [Model.UpdateLocationRequest](docs/UpdateLocationRequest.md)
 - [Model.UpdateRoomRequest](docs/UpdateRoomRequest.md)
 - [Model.UpdateSignatureTypeRequest](docs/UpdateSignatureTypeRequest.md)
 - [Model.ViperDeviceDetails](docs/ViperDeviceDetails.md)
 - [Model.WebhookSmartApp](docs/WebhookSmartApp.md)


<a name="documentation-for-authorization"></a>
## Documentation for Authorization

<a name="Basic"></a>
### Basic

- **Type**: HTTP basic authentication

<a name="Bearer"></a>
### Bearer

- **Type**: OAuth
- **Flow**: implicit
- **Authorization URL**: https://auth-global.api.smartthings.com
- **Scopes**: 
  - r:installedapps:*: Read details about installed SmartApps, such as which devices have been configured for the installation. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - l:installedapps: View a list of installed SmartApps. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - w:installedapps:*: Create, update, or delete installed SmartApps. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - r:apps:*: Read details about a SmartApp. Only applicable for personal access tokens, and the scope is limited to the SmartApps associated with the token&#39;s account. 
  - w:apps:*: Create, update, or delete SmartApps. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - l:devices: View a list of devices. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - r:devices:*: Read details about a device, including device attribute state. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. This scope is required to create subscriptions. 
  - w:devices:*: Update details such as the device name, or delete a device. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - x:devices:*: Execute commands on a device. For SmartApp tokens, the scope is restricted to the location the SmartApp is installed into. For personal access tokens, the scope is limited to the account associated with the token. 
  - r:deviceprofiles: View details of device profiles associated with the account. Only applicable for personal access tokens. 
  - w:deviceprofiles: Create, update, or delete device profiles. Only applicable to personal access tokens, and the device profile must be owned by the same account associated with the token. 
  - i:deviceprofiles: Create devices of the type associated with the device profile. Only applicable for SmartApp tokens, and is requires the device profile and the SmartApp have the same account owner.
  - r:scenes:*: Read details about a scene. For personal access tokens, the scope is limited to the account associated with the token. 
  - x:scenes:*: Execute a scene. For personal access tokens, the scope is limited to the account associated with the token. 
  - r:schedules: Read details of scheduled executions. For SmartApp tokens, the scope is restricted to the installed SmartApp. For personal access tokens, the scope is limited to the account associated with the token. 
  - w:schedules: Create, update, or delete schedules. For SmartApp tokens, the scope is restricted to the installed SmartApp. For personal access tokens, the scope is limited to the account associated with the token. 
  - l:locations: View a list of locations. Only applicable for personal access tokens, and the scope is limited to the account associated with the token. 
  - r:locations:*: Read details of a location, such as geocoordinates and temperature scale. For SmartApp tokens, the scope is restricted to the installed SmartApp. For personal access tokens, the scope is limited to the account associated with the token. 
  - w:locations:*: Create, update, and delete locations. Only applicable for personal access tokens (the scope is limited to the account associated with the token). 
  - r:hubs: Read hubs.
  - r:security:locations:*:armstate: Read arm state in the given location.

