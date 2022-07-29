## Dynamics 365 external data caching in Azure

Often in the course of Dynamics 365 implementations, there are requirements to integrate the system with customer portals or some other external solutions hosted in different clouds or on premise infrastructure. This usually will lead to a tight-coupled integration which will potentially be affected by [Throttling prioritization](https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/data-entities/priority-based-throttling).

Since the data in Dynamics 365 as in any ERP or CRM system contains a lot of static master data, which changes rarely if not at all. Keeping a cached copy of this data outside of Dynamics 365 system will limit the number of API calls the system will receive when this data needs to be retrieved by the external application, reducing both the throttling change and the workload for the system.   

### Solution overview

The high-level view of the solution can be displayed in the following diagram.
TODO add image

The redis cache acts as a buffer between the external system and Dynamics 365. 
The API management self hosted gateway feature provides the option to keep this cache closer to the client solution.
This will greatly improve the response time for the client application, which are usually very strict in the NFR.

The communication between API management and Dynamics 365 can easily be achieved using the following sample policy from official documentation [Use OAuth2 for authorization between the gateway and a backend](https://docs.microsoft.com/en-us/azure/api-management/policies/use-oauth2-for-authorization)

The azure function will execute the cache refresh if the data is modified in the backend Dynamics 365 system. A simple sample code on how that can be achieved is provided [here](https://github.com/NicolaeAlexandruPanait/Dynamics-365-external-data-caching-in-Azure/blob/main/http_azure_redis_func.txt). There are classes required. One is for initializing the azure function parameters which will contain the redis database connection string. The other will be the actual function runtime that will clear the cache. This is just a prototype but the logic can be extended depending on the business requirement.

This artefact could be optional if the data does not change at all, or it changes so rarely that having a manual process will be fit for purpose. 

The [Business data events](https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/business-events/data-events) can be levereged to add a message to a service bus queue and use the [Service bus trigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus-trigger?tabs=in-process%2Cextensionv5&pivots=programming-language-csharp) for azure functions.

### Let's see it in action

For testing purpose I've create a simple API management, which contains 2 api calls. One is to query the Dynamics 365 items list. 
As it can be seen in the picture below this will point to D365Url/data/ReleasedProductsV2. The API definition will need to have the input as ItemId and the LegalEntity. 

![D365 first call](https://user-images.githubusercontent.com/25058196/181766303-f4a5e12b-03b3-46f1-a6f2-975f1b17ee1d.png)

For the first API call, there will not be any data stored in the cache so it will result in a miss. This means that the call will be re-routed to the backend Dynamics 365 system.

Subsequent calls with the same input parameters will result in a hit. In this scenario, the cache result will be a hit.

![D365 subsequent calls](https://user-images.githubusercontent.com/25058196/181767517-c3ee1ba1-3bad-4919-8c2b-526b2613daf5.png)

The second API will point to the Azure function with the code sample provided before: [sample azure http function](https://github.com/NicolaeAlexandruPanait/Dynamics-365-external-data-caching-in-Azure/blob/main/http_azure_redis_func.txt)

Supposing there is a change in Dynamics 365 for that item. That would be equivalent with triggering the second API which will clear the cache for the specified item id.

The next D365 API call will be then rerouted again to the Dynamics 365 system and the new record data will be added to the cache.

