---
# Mandatory fields.
title: Manage endpoints and routes
titleSuffix: Azure Digital Twins
description: See how to set up and manage endpoints and event routes for Azure Digital Twins data
author: baanders
ms.author: baanders # Microsoft employees only
ms.date: 7/30/2021
ms.topic: how-to
ms.service: digital-twins

# Optional fields. Don't forget to remove # if you need a field.
# ms.custom: can-be-multiple-comma-separated
# ms.reviewer: MSFT-alias-of-reviewer
# manager: MSFT-alias-of-manager-or-PM-counterpart
---

# Manage endpoints and routes in Azure Digital Twins

In Azure Digital Twins, you can route [event notifications](concepts-event-notifications.md) to downstream services or connected compute resources. This is done by first setting up **endpoints** that can receive the events. You can then create [event routes](concepts-route-events.md) that specify which events generated by Azure Digital Twins are delivered to which endpoints.

This article walks you through the process of creating endpoints and routes using the [Azure portal](https://portal.azure.com), the [REST APIs](/rest/api/azure-digitaltwins/), the [.NET (C#) SDK](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true), and the [Azure Digital Twins CLI](/cli/azure/dt).

## Prerequisites

* You'll need an **Azure account**, which [can be set up for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* You'll need an **Azure Digital Twins instance** in your Azure subscription. If you don't have an instance already, you can create one using the steps in [Set up an instance and authentication](how-to-set-up-instance-portal.md). Have the following values from setup handy to use later in this article:
    - Instance name
    - Resource group

You can find these details in the [Azure portal](https://portal.azure.com) after setting up your instance. Log into the portal and search for the name of your instance in the portal search bar.
 
:::image type="content" source="media/how-to-manage-routes/search-field-portal.png" alt-text="Screenshot of Azure portal search bar." lightbox="media/how-to-manage-routes/search-field-portal.png":::

Select your instance from the results to see these details in the Overview for your instance:

:::image type="content" source="media/how-to-manage-routes/instance-details.png" alt-text="Screenshot of the Overview page for an Azure Digital Twins instance in the Azure portal. The name and resource group are highlighted.":::

Follow the instructions below if you intend to use the Azure CLI while following this guide.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## Create an endpoint for Azure Digital Twins

These are the supported types of endpoints that you can create for your instance:
* [Event Grid](../event-grid/overview.md) topic
* [Event Hubs](../event-hubs/event-hubs-about.md) hub
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) topic

>[!NOTE]
> For Event Grid endpoints, only event grid **topics** are supported. Event grid **domains** are not supported as endpoints.

For more information on the different endpoint types, see [Choose between Azure messaging services](../event-grid/compare-messaging-services.md).

This section explains how to create an endpoint using the [Azure portal](https://portal.azure.com) or the [Azure CLI](/cli/azure/dt/endpoint). You can also manage endpoints with the [DigitalTwinsEndpoint control plane APIs](/rest/api/digital-twins/controlplane/endpoints).

### Prerequisite: Create endpoint resources

To link an endpoint to Azure Digital Twins, the event grid topic, event hub, or Service Bus topic that you're using for the endpoint needs to exist already.

Use the following chart to see what resources should be set up before creating your endpoint.

| Endpoint type | Required resources (linked to creation instructions) |
| --- | --- |
| Event Grid endpoint | [event grid topic](../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic)<br/>*event schema must be Event Grid Schema or Cloud Event Schema v1.0 |
| Event Hubs endpoint | [Event&nbsp;Hubs&nbsp;namespace](../event-hubs/event-hubs-create.md)<br/><br/>[event hub](../event-hubs/event-hubs-create.md)<br/><br/>(Optional) [authorization rule](../event-hubs/authorize-access-shared-access-signature.md) for key-based authentication | 
| Service Bus endpoint | [Service Bus namespace](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md)<br/><br/>[Service Bus topic](../service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md)<br/><br/> (Optional) [authorization rule](../service-bus-messaging/service-bus-authentication-and-authorization.md#shared-access-signature) for key-based authentication|

### Create the endpoint 

Once you have created the endpoint resources, you can use them for an Azure Digital Twins endpoint. 

# [Portal](#tab/portal) 

To create a new endpoint, go to your instance's page in the [Azure portal](https://portal.azure.com) (you can find the instance by entering its name into the portal search bar).

1. From the instance menu, select _Endpoints_. Then from the *Endpoints* page that follows, select *+ Create an endpoint*. This will open the *Create an endpoint* page, where you'll fill in the fields in the following steps.

    :::image type="content" source="media/how-to-manage-routes/create-endpoint-event-grid.png" alt-text="Screenshot of creating an endpoint of type Event Grid in the Azure portal." lightbox="media/how-to-manage-routes/create-endpoint-event-grid.png":::

1. Enter a **Name** for your endpoint and choose the **Endpoint type**.

1. Complete the other details that are required for your endpoint type, including your subscription and the endpoint resources described [above](#prerequisite-create-endpoint-resources).
    1. For Event Hub and Service Bus endpoints only, you must select an **Authentication type**. You can use key-based authentication with a pre-created authorization rule, or identity-based authentication if you'll be using the endpoint with a [managed identity](concepts-security.md#managed-identity-for-accessing-other-resources) for your Azure Digital Twins instance. 

    :::row:::
        :::column:::
            :::image type="content" source="media/how-to-manage-routes/create-endpoint-event-hub-authentication.png" alt-text="Screenshot of creating an endpoint of type Event Hub in the Azure portal." lightbox="media/how-to-manage-routes/create-endpoint-event-hub-authentication.png":::
        :::column-end:::
        :::column:::
        :::column-end:::
    :::row-end:::

1. Finish creating your endpoint by selecting _Save_.

>[!IMPORTANT]
> In order to successfully use identity-based authentication for your endpoint, you'll need to create a managed identity for your instance by following the steps in [Route events with a managed identity](how-to-route-with-managed-identity.md).

After creating your endpoint, you can verify that the endpoint was successfully created by checking the notification icon in the top Azure portal bar: 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes/create-endpoint-notifications.png" alt-text="Screenshot of the notification to verify the creation of an endpoint in the Azure portal.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

If the endpoint creation fails, observe the error message and retry after a few minutes.

You can also view the endpoint that was created back on the *Endpoints* page for your Azure Digital Twins instance.

Now the event grid, event hub, or Service Bus topic is available as an endpoint inside of Azure Digital Twins, under the name you chose for the endpoint. You'll typically use that name as the target of an **event route**, which you'll create [later in this article](#create-an-event-route).

# [CLI](#tab/cli) 

The following examples show how to create endpoints using the [az dt endpoint create](/cli/azure/dt/endpoint/create) command for the [Azure Digital Twins CLI](/cli/azure/dt). Replace the placeholders in the commands with the details of your own resources.

To create an Event Grid endpoint:

```azurecli-interactive
az dt endpoint create eventgrid --endpoint-name <Event-Grid-endpoint-name> --eventgrid-resource-group <Event-Grid-resource-group-name> --eventgrid-topic <your-Event-Grid-topic-name> --dt-name <your-Azure-Digital-Twins-instance-name>
```

To create an Event Hubs endpoint (key-based authentication):
```azurecli-interactive
az dt endpoint create eventhub --endpoint-name <Event-Hub-endpoint-name> --eventhub-resource-group <Event-Hub-resource-group> --eventhub-namespace <Event-Hub-namespace> --eventhub <Event-Hub-name> --eventhub-policy <Event-Hub-policy> --dt-name <your-Azure-Digital-Twins-instance-name>
```

To create a Service Bus topic endpoint (key-based authentication):
```azurecli-interactive 
az dt endpoint create servicebus --endpoint-name <Service-Bus-endpoint-name> --servicebus-resource-group <Service-Bus-resource-group-name> --servicebus-namespace <Service-Bus-namespace> --servicebus-topic <Service-Bus-topic-name> --servicebus-policy <Service-Bus-topic-policy> --dt-name <your-Azure-Digital-Twins-instance-name>
```

After successfully running these commands, the event grid, event hub, or Service Bus topic will be available as an endpoint inside of Azure Digital Twins, under the name you supplied with the `--endpoint-name` argument. You'll typically use that name as the target of an **event route**, which you'll create [later in this article](#create-an-event-route).

#### Create an endpoint with identity-based authentication

You can also create an endpoint that has identity-based authentication, to use the endpoint with a [managed identity](concepts-security.md#managed-identity-for-accessing-other-resources). This option is only available for Event Hubs and Service Bus-type endpoints (it's not supported for Event Grid).

The CLI command to create this type of endpoint is below. You'll need the following values to plug into the placeholders in the command:
* the Azure resource ID of your Azure Digital Twins instance
* an endpoint name
* an endpoint type
* the endpoint resource's namespace
* the name of the event hub or Service Bus topic
* the location of your Azure Digital Twins instance

```azurecli-interactive
az resource create --id <Azure-Digital-Twins-instance-Azure-resource-ID>/endpoints/<endpoint-name> --properties '{\"properties\": { \"endpointType\": \"<endpoint-type>\", \"authenticationType\": \"IdentityBased\", \"endpointUri\": \"sb://<endpoint-namespace>.servicebus.windows.net\", \"entityPath\": \"<name-of-event-hub-or-Service-Bus-topic>\"}, \"location\":\"<instance-location>\" }' --is-full-object
```

---

### Create an endpoint with dead-lettering

When an endpoint can't deliver an event within a certain time period or after trying to deliver the event a certain number of times, it can send the undelivered event to a storage account. This process is known as **dead-lettering**.

You can set up the necessary storage resources using the [Azure portal](https://ms.portal.azure.com/#home) or the [Azure Digital Twins CLI](/cli/azure/dt). However, to create an endpoint with dead-lettering enabled, you'll need use the [Azure Digital Twins CLI](/cli/azure/dt) or [control plane APIs](concepts-apis-sdks.md#overview-control-plane-apis).

To learn more about dead-lettering, see [Event routes](concepts-route-events.md#dead-letter-events). For instructions on how to set up an endpoint with dead-lettering, continue through the rest of this section.

#### Set up storage resources

Before setting the dead-letter location, you must have a [storage account](../storage/common/storage-account-create.md?tabs=azure-portal) with a [container](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) set up in your Azure account. 

You'll provide the URI for this container when creating the endpoint later. The dead-letter location will be provided to the endpoint as a container URI with a [SAS token](../storage/common/storage-sas-overview.md). That token needs `write` permission for the destination container within the storage account. The fully formed **dead letter SAS URI** will be in the format of: `https://<storage-account-name>.blob.core.windows.net/<container-name>?<SAS-token>`.

Follow the steps below to set up these storage resources in your Azure account, to prepare to set up the endpoint connection in the next section.

1. Follow the steps in [Create a storage account](../storage/common/storage-account-create.md?tabs=azure-portal) to create a **storage account** in your Azure subscription. Make a note of the storage account name to use it later.
1. Follow the steps in [Create a container](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) to create a **container** within the new storage account. Make a note of the container name to use it later.

##### Create a SAS token

Next, create a **SAS token** for your storage account that the endpoint can use to access it.

# [Portal](#tab/portal)

1. Start by navigating to your storage account in the [Azure portal](https://ms.portal.azure.com/#home) (you can find it by name with the portal search bar).
1. In the storage account page, choose the _Shared access signature_ link in the left navigation bar to start setting up the SAS token.

    :::image type="content" source="./media/how-to-manage-routes/generate-sas-token-1.png" alt-text="Screenshot of the storage account page in the Azure portal." lightbox="./media/how-to-manage-routes/generate-sas-token-1.png":::

1. On the *Shared access signature page*, under *Allowed services* and *Allowed resource types*, select whatever settings you want. You'll need to select at least one box in each category. Under *Allowed permissions*, choose **Write** (you can also select other permissions if you want).
1. Set whatever values you want for the remaining settings.
1. When you're finished, select the _Generate SAS and connection string_ button to generate the SAS token. 

    :::image type="content" source="./media/how-to-manage-routes/generate-sas-token-2.png" alt-text="Screenshot of the storage account page in the Azure portal showing all the setting selection to generate a SAS token." lightbox="./media/how-to-manage-routes/generate-sas-token-2.png"::: 

1. This will generate several SAS and connection string values at the bottom of the same page, underneath the setting selections. Scroll down to view the values, and use the *Copy to clipboard* icon to copy the **SAS token** value. Save it to use later.

    :::image type="content" source="./media/how-to-manage-routes/copy-sas-token.png" alt-text="Screenshot of the storage account page in the Azure portal highlighting how to copy the SAS token to use in the dead-letter secret." lightbox="./media/how-to-manage-routes/copy-sas-token.png":::

# [CLI](#tab/cli)

1. Retrieve your storage account keys using the following command and copy the value for either one of your keys:

    ```azurecli
    az storage account keys list --account-name <storage-account-name>
    ```

1. Select an expiration date and generate the SAS token for your storage account using the following command:

    ```azurecli
    az storage account generate-sas --account-name <storage-account-name> --account-key <storage-account-key> --expiry <expiration-date> --services bfqt --resource-types o --permissions w
    ```

    The output of this command is the SAS token. Copy the SAS token value to use later.

    > [!NOTE]
    > This command includes "**b**lob", "**f**ile", "**q**ueue", and "**t**able" *services*; an "**o**bject" *resource type*; and allows "**w**rite" *permissions*.
    >
    > For more information about the `az storage account generate-sas` command and its parameters, see the [Azure CLI reference](/cli/azure/storage/account#az_storage_account_generate_sas).

---

#### Create the dead-letter endpoint

# [Portal](#tab/portal)

In order to create an endpoint with dead-lettering enabled, you must use the [CLI commands](/cli/azure/dt) or [control plane APIs](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) to create your endpoint, rather than the Azure portal.

For instructions on how to do this with the Azure CLI, switch to the CLI tab for this section.

# [CLI](#tab/cli)

To create an endpoint that has dead-lettering enabled, add the following dead letter parameter to the [az dt endpoint create](/cli/azure/dt/endpoint/create) command for the [Azure Digital Twins CLI](/cli/azure/dt).

The value for the parameter is the **dead letter SAS URI** made up of the storage account name, container name, and SAS token that you gathered in the [previous section](#set-up-storage-resources). This parameter creates the endpoint with key-based authentication.

```azurecli
--deadletter-sas-uri https://<storage-account-name>.blob.core.windows.net/<container-name>?<SAS-token>
```

Add this parameter to the end of the endpoint creation commands from the [Create the endpoint](#create-the-endpoint) section earlier to create an endpoint of your desired type that has dead-lettering enabled.

Alternatively, you can create dead letter endpoints using the [Azure Digital Twins control plane APIs](concepts-apis-sdks.md#overview-control-plane-apis) instead of the CLI. To do this, view the [DigitalTwinsEndpoint documentation](/rest/api/digital-twins/controlplane/endpoints/digitaltwinsendpoint_createorupdate) to see how to structure the request and add the dead letter parameters.

#### Create a dead-letter endpoint with identity-based authentication

You can also create a dead-lettering endpoint that has identity-based authentication, to use the endpoint with a [managed identity](concepts-security.md#managed-identity-for-accessing-other-resources). This option is only available for Event Hubs and Service Bus-type endpoints (it's not supported for Event Grid).

To create this type of endpoint, use the same CLI command from earlier to [create an endpoint with identity-based authentication](#create-an-endpoint-with-identity-based-authentication), with an extra field in the JSON payload for a `deadLetterUri`.

Here are the values you'll need to plug into the placeholders in the command:
* the Azure resource ID of your Azure Digital Twins instance
* an endpoint name
* an endpoint type
* the endpoint resource's namespace
* the name of the event hub or Service Bus topic
* **dead letter SAS URI** details: storage account name, container name
* the location of your Azure Digital Twins instance

```azurecli-interactive
az resource create --id <Azure-Digital-Twins-instance-Azure-resource-ID>/endpoints/<endpoint-name> --properties '{\"properties\": { \"endpointType\": \"<endpoint-type>\", \"authenticationType\": \"IdentityBased\", \"endpointUri\": \"sb://<endpoint-namespace>.servicebus.windows.net\", \"entityPath\": \"<name-of-event-hub-or-Service-Bus-topic>\", \"deadLetterUri\": \"https://<storage-account-name>.blob.core.windows.net/<container-name>\"}, \"location\":\"<instance-location>\" }' --is-full-object
```

---

#### Message storage schema

Once the endpoint with dead-lettering is set up, dead-lettered messages will be stored in the following format in your storage account:

`<container>/<endpoint-name>/<year>/<month>/<day>/<hour>/<event-ID>.json`

Dead-lettered messages will match the schema of the original event that was intended to be delivered to your original endpoint.

Here is an example of a dead-letter message for a [twin create notification](concepts-event-notifications.md#digital-twin-lifecycle-notifications):

```json
{
  "specversion": "1.0",
  "id": "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "<your-instance>.api.<your-region>.da.azuredigitaltwins-test.net",
  "data": {
    "$dtId": "<your-instance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "$etag": "W/\"xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
    "TwinData": "some sample",
    "$metadata": {
      "$model": "dtmi:test:deadlettermodel;1",
      "room": {
        "lastUpdateTime": "2020-10-14T01:11:49.3576659Z"
      }
    }
  },
  "subject": "<your-instance>xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "time": "2020-10-14T01:11:49.3667224Z",
  "datacontenttype": "application/json",
  "traceparent": "00-889a9094ba22b9419dd9d8b3bfe1a301-f6564945cb20e94a-01"
}
```

## Create an event route

To actually send data from Azure Digital Twins to an endpoint, you'll need to define an **event route**. These routes let developers wire up event flow, throughout the system and to downstream services. A single route can allow multiple notifications and event types to be selected. Read more about event routes in [Routing Azure Digital Twins events](concepts-route-events.md).

**Prerequisite**: You need to create endpoints as described earlier in this article before you can move on to creating a route. You can proceed to creating an event route once your endpoints are finished setting up.

>[!NOTE]
>If you have recently deployed your endpoints, validate that they're finished deploying **before** attempting to use them for a new event route. If route deployment fails because the endpoints aren't ready, wait a few minutes and try again.
>
> If you are scripting this flow, you may want to account for this by building in 2-3 minutes of wait time for the endpoint service to finish deploying before moving on to route setup.

A route definition can contain these elements:
* The route name you want to use
* The name of the endpoint you want to use
* A filter that defines which events are sent to the endpoint
    - To disable the route so that no events are sent, use a filter value of `false`
    - To enable a route that has no specific filtering, use a filter value of `true`
    - For details on any other type of filter, see the [Filter events](#filter-events) section below

If there is no route name, no messages are routed outside of Azure Digital Twins. 
If there is a route name and the filter is `true`, all messages are routed to the endpoint. 
If there is a route name and a different filter is added, messages will be filtered based on the filter.

Event routes can be created with the [Azure portal](https://portal.azure.com), [EventRoutes data plane APIs](/rest/api/digital-twins/dataplane/eventroutes), or [az dt route CLI commands](/cli/azure/dt/route). The rest of this section walks through the creation process.

# [Portal](#tab/portal2)

To create an event route, go to the details page for your Azure Digital Twins instance in the [Azure portal](https://portal.azure.com) (you can find the instance by entering its name into the portal search bar).

From the instance menu, select _Event routes_. Then from the *Event routes* page that follows, select *+ Create an event route*. 

On the *Create an event route* page that opens up, choose at minimum:
* A name for your route in the _Name_ field
* The _Endpoint_ you want to use to create the route 

For the route to be enabled, you must also **Add an event route filter** of at least `true`. (Leaving the default value of `false` will create the route, but no events will be sent to it.) To do this, toggle the switch for the _Advanced editor_ to enable it, and write `true` in the *Filter* box.

:::image type="content" source="media/how-to-manage-routes/create-event-route-no-filter.png" alt-text="Screenshot of creating an event route for your instance in the Azure portal." lightbox="media/how-to-manage-routes/create-event-route-no-filter.png":::

When finished, select the _Save_ button to create your event route.

# [CLI](#tab/cli2)

Routes can be managed using the [az dt route](/cli/azure/dt/route) commands for the Azure Digital Twins CLI. 

For more information about using the CLI and what commands are available, see [Azure Digital Twins CLI command set](concepts-cli.md).

# [.NET SDK](#tab/sdk2)

This section shows how to create an event route using the [.NET (C#) SDK](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true).

`CreateOrReplaceEventRouteAsync` is the SDK call that is used to add an event route. Here is an example of its usage:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="CreateEventRoute":::
    
> [!TIP]
> All SDK functions come in synchronous and asynchronous versions.

#### Event route sample SDK code

The following sample method shows how to create, list, and delete an event route with the C# SDK:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/eventRoute_operations.cs" id="FullEventRouteSample":::

---

## Filter events

As described above, routes have a **filter** field. If the filter value on your route is `false`, no events will be sent to your endpoint. 

After enabling the minimal filter of `true`, endpoints will receive a variety of events from Azure Digital Twins:
* Telemetry fired by [digital twins](concepts-twins-graph.md) using the Azure Digital Twins service API
* Twin property change notifications, fired on property changes for any twin in the Azure Digital Twins instance
* Life-cycle events, fired when twins or relationships are created or deleted

You can restrict the types of events being sent by defining a more-specific filter.

>[!NOTE]
> Filters are **case-sensitive** and need to match the payload case. 
>
> For telemetry filters, this means that the casing needs to match the casing in the telemetry sent by the device, not necessarily the casing defined in the twin's model.

# [Portal](#tab/portal3)

To add an event filter while you are creating an event route, use the "Add an event route filter" section of the *Create an event route* page. 

You can either select from some basic common filter options, or use the advanced filter options to write your own custom filters.

### Use the basic filters

To use the basic filters, expand the _Event types_ option and select the checkboxes corresponding to the events you want to send to your endpoint. 

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes/create-event-route-filter-basic-1.png" alt-text="Screenshot of creating an event route with a basic filter in the Azure portal, highlighting the checkboxes of the events.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

This will auto-populate the filter text box with the text of the filter you've selected:

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes/create-event-route-filter-basic-2.png" alt-text="Screenshot of creating an event route with a basic filter in the Azure portal, highlighting the auto-populated filter text after selecting the events.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

### Use the advanced filters

Alternatively, you can use the advanced filter option to write your own custom filters.

To create an event route with advanced filter options, toggle the switch for the _Advanced editor_ to enable it. You can then write your own event filters in the *Filter* box:

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-manage-routes/create-event-route-filter-advanced.png" alt-text="Screenshot of creating an event route with an advanced filter in the Azure portal.":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

# [API](#tab/api)

You can use the APIs to write custom filters. To add a filter, you can use a PUT request to `https://<Your-Azure-Digital-Twins-host-name>/eventRoutes/<event-route-name>?api-version=2020-10-31` with the following body:

:::code language="json" source="~/digital-twins-docs-samples/api-requests/filter.json":::

---

### Supported route filters

Here are the supported route filters.

| Filter name | Description | Filter text schema | Supported values | 
| --- | --- | --- | --- |
| True / False | Allows creating a route with no filtering, or disabling a route so no events are sent | `<true/false>` | `true` = route is enabled with no filtering <br> `false` = route is disabled |
| Type | The [type of event](concepts-route-events.md#types-of-event-messages) flowing through your digital twin instance | `type = '<event-type>'` | Here are the possible event type values: <br>`Microsoft.DigitalTwins.Twin.Create` <br> `Microsoft.DigitalTwins.Twin.Delete` <br> `Microsoft.DigitalTwins.Twin.Update`<br>`Microsoft.DigitalTwins.Relationship.Create`<br>`Microsoft.DigitalTwins.Relationship.Update`<br> `Microsoft.DigitalTwins.Relationship.Delete` <br> `microsoft.iot.telemetry`  |
| Source | Name of Azure Digital Twins instance | `source = '<host-name>'`| Here are the possible host name values: <br> **For notifications**: `<your-Digital-Twins-instance>.api.<your-region>.digitaltwins.azure.net` <br> **For telemetry**: `<your-Digital-Twins-instance>.api.<your-region>.digitaltwins.azure.net/<twin-ID>`|
| Subject | A description of the event in the context of the event source above | `subject = '<subject>'` | Here are the possible subject values: <br>**For notifications**: The subject is `<twin-ID>` <br> or a URI format for subjects, which are uniquely identified by multiple parts or IDs:<br>`<twin-ID>/relationships/<relationship-ID>`<br> **For telemetry**: The subject is the component path (if the telemetry is emitted from a twin component), such as `comp1.comp2`. If the telemetry is not emitted from a component, then its subject field is empty. |
| Data schema | DTDL model ID | `dataschema = '<model-dtmi-ID>'` | **For telemetry**: The data schema is the model ID of the twin or the component that emits the telemetry. For example, `dtmi:example:com:floor4;2` <br>**For notifications (create/delete)**: Data schema can be accessed in the notification body at `$body.$metadata.$model`. <br>**For notifications (update)**: Data schema can be accessed in the notification body at `$body.modelId`|
| Content type | Content type of data value | `datacontenttype = '<content-type>'` | The content type is `application/json` |
| Spec version | The version of the event schema you are using | `specversion = '<version>'` | The version must be `1.0`. This indicates the CloudEvents schema version 1.0 |
| Notification body | Reference any property in the `data` field of a notification | `$body.<property>` | See [Event notifications](concepts-event-notifications.md) for examples of notifications. Any property in the `data` field can be referenced using `$body`

>[!NOTE]
> Azure Digital Twins currently doesn't support filtering events based on fields within an array. This includes filtering on properties within a `patch` section of a [digital twin change notification](concepts-event-notifications.md#digital-twin-change-notifications).

The following data types are supported as values returned by references to the data above:

| Data type | Example |
|-|-|-|
|**String**| `STARTS_WITH($body.$metadata.$model, 'dtmi:example:com:floor')` <br> `CONTAINS(subject, '<twin-ID>')`|
|**Integer**|`$body.errorCode > 200`|
|**Double**|`$body.temperature <= 5.5`|
|**Bool**|`$body.poweredOn = true`|
|**Null**|`$body.prop != null`|

The following operators are supported when defining route filters:

|Family|Operators|Example|
|-|-|-|
|Logical|AND, OR, ( )|`(type != 'microsoft.iot.telemetry' OR datacontenttype = 'application/json') OR (specversion != '1.0')`|
|Comparison|<, <=, >, >=, =, !=|`$body.temperature <= 5.5`

The following functions are supported when defining route filters:

|Function|Description|Example|
|--|--|--|
|STARTS_WITH(x,y)|Returns true if the value `x` starts with the string `y`.|`STARTS_WITH($body.$metadata.$model, 'dtmi:example:com:floor')`|
|ENDS_WITH(x,y) | Returns true if the value `x` ends with the string `y`.|`ENDS_WITH($body.$metadata.$model, 'floor;1')`|
|CONTAINS(x,y)| Returns true if the value `x` contains the string `y`.|`CONTAINS(subject, '<twin-ID>')`|

When you implement or update a filter, the change may take a few minutes to be reflected in the data pipeline.

## Monitor event routes

Routing metrics such as count, latency, and failure rate can be viewed in the [Azure portal](https://portal.azure.com/). 

From the portal homepage, search for your Azure Digital Twins instance to pull up its details. Select the **Metrics** option from the Azure Digital Twins instance's navigation menu on the left to bring up the *Metrics* page.

:::image type="content" source="media/troubleshoot-metrics/azure-digital-twins-metrics.png" alt-text="Screenshot showing the metrics page for Azure Digital Twins.":::

From here, you can view the metrics for your instance and create custom views.

For more on viewing Azure Digital Twins metrics, see [Troubleshooting: Metrics](troubleshoot-metrics.md).

## Next steps

Read about the different types of event messages you can receive:
* [Event notifications](concepts-event-notifications.md)