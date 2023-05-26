# Moqui/Shopify Integration.

<img src="https://cdn.shopify.com/shopifycloud/help/assets/sharing/share-image-generic-bd3ce342a910c2489b672b00e45c74b1b1548662c41448e456547fa5b6e0f585.png" width="248">

## Introduction
Moqui Framework and Artifacts is is a fully fledged ERP solution with Fulfilment capabilities. 
Shopify is a very popular Storefront used by many Ecommerce companies. 
This integration allows products, collections, orders, inventory, payments and refunds to be synced between Shopify and Moqui. 
It requires that you have a store set up in Shopify, and a basic knowledge of the Moqui Framework, and entities such as:

- mantle.product.Product
- mantle.product.store.ProductStore
- mantle.product.category.ProductCategory
- mantle.order.OrderHeader
- mantle.order.OrderItem
- mantle.facility.Facility
- mantle.product.asset.Asset
- moqui.service.job.ServiceJob
- moqui.service.message.SystemMessage

For more detailed information on the Moqui Framework, please read the documentation available here:
https://www.moqui.org/docs/framework

## Generate Shopify API credentials

To allow moqui to communicate with the Shopify API for your store, you need to create a 'Custom App' in the shopify admin:
 1. From your Shopify admin, click Settings > Apps and sales channels.
 2. Click Develop apps.
 3. Click Allow custom app development.
 4. Click Create a custom app.
 5. In the modal window, enter the App name (e.g 'Moqui') and select an App developer.

Click Create app, and assign the following scopes:
```
write_assigned_fulfillment_orders, read_assigned_fulfillment_orders, write_customers, read_customers, write_discounts, read_discounts, write_fulfillments, read_fulfillments, write_inventory, read_inventory, write_locations, read_locations, write_merchant_managed_fulfillment_orders, read_merchant_managed_fulfillment_orders, write_orders, read_orders, write_products, read_products, write_custom_fulfillment_services, read_custom_fulfillment_services
```

6. Click "install app"

https://help.shopify.com/en/manual/apps/app-types/custom-apps?shpxid=d49fae40-17FA-4911-FF2D-EE18EA231E81

Go to the "API Credentials" tab and take note of the API key and secret key.

### Create a default 'Location' in Shopify
 1. From your Shopify admin, click Settings > Locations
 2. Click on the name of the default location
 3. Modify the 'Location name' to match the name of your 'Facility' in Moqui. 

 In the example, we call the location 'Ziziwork Retail Warehouse'


## Configure Moqui to read/write Orders and Fulfillments
The first step is to configure a SystemMessageRemote that will allow calls to the Shopify REST API:
```
<moqui.service.message.SystemMessageRemote systemMessageTypeId="ShopifySystemMessageType" 
        systemMessageRemoteId="" productStoreId="" username="" password=""/>
```
Give your remote a name, the productStoreId that it will be interacting with, and the API credentials that were noted from Shopify. The stores subdomain should be configured in the  'username' field of the SystemMessageRemote and the 'API Key' should be configured as the password. See the demo data for an example.

Run the following service to configure the Facility in Moqui with the previously created Location in Shopify:
```
mantle.shopify.ShopifyServices.configure#FacilityFromShopifyLocation
```


## Services to Sync Products and Categories
There are 2 services that can be called to download products and collections from Shopify down to Moqui.
```
mantle.shopify.ShopifyProductServices.download#ShopifyProducts
mantle.shopify.ShopifyProductServices.download#ShopifyCollections
```
Both services generate SystemMessages that need to be consumed. If this is not happening automatically, ensure the consume_AllReceivedSystemMessages_frequent is running.


There are 2 services that can be called to upload products and categories from Moqui to Shopify
```
mantle.shopify.ShopifyProductServices.upload#MoquiStoreProducts
mantle.shopify.ShopifyProductServices.upload#MoquiStoreCategories
```
Choose which platform is the source of truth for Product data and then set up service jobs to poll one of the sets of services. There are demo service jobs set up fro each one. Choose a frequency that makse sense given how much your product data changes during BAU. Run the jobs manually when need data synced immediately.

## Services to Sync Orders, Fulfilments and Inventory
There is a service that can be polled to download orders from Shopify:
```
mantle.shopify.ShopifyOrderServices.download#ShopifyOrders
```

There is a demo job that polls this service called: poll_shopify_orders_download

When an order is fulfilled in Moqui, a SystemMessage is queued to POST to the shopify fulfilments API 
```
mantle.shopify.ShopifyProductServices.queue#ShopifyFulfillmentSystemMessage
```

Fulfilments are queued as SystemMessage to allow retries if the API call fails.


When an order is fulfilled in Moqui, or Inventory is received in the Moqui Facility, services are triggered to update inventory in Shopify
```
mantle.shopify.ShopifyProductServices.update#ItemInventory

```
This API call updates one Item per call, with obvious limitations around the Shopify REST API rate limits. To avoid this limitation, a new endpoint would need to be developed to call the Shopify Graph API, and update multiple items per API call.

TODO Complete the help for the refund services.




