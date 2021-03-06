---
layout: single
title: Release Notes
sidebar: 
  nav: "docs"
---


### 0.10.M1

This is a new major release of the BIG IoT Consumer and Provider Lib. 

**NOTE: Since the Marketplace API (between Consumer/Provider Lib and Marketplace) has changed, this Lib release will work only  with Marketplace release 0.10.M1.**  

Several new features have been introduces (see below) and changes to the Programming API are implied. See details here: 

* Creating a Offering Description using the Programming API (see new methods `withName()`, `withCategory()`): 
    ```java
      RegistrableOfferingDescription offeringDescription = provider.createOfferingDescription("BasicDemoParkingSpotProvider")
                .withName("Basic Demo Parking Offering")
                .withCategory("urn:big-iot:ParkingSpaceCategory")
                .inRegion(BoundingBox.create(Location.create(42.1, 9.0), Location.create(43.2, 10.0)))
                .addInputData("longitude", new RDFType("schema:longitude"), ValueType.NUMBER)
                ...
                .addOutputData("status", new RDFType("datex:parkingSpaceStatus"), ValueType.TEXT)
                ...
                .withPrice(Euros.amount(0.001)).withPricingModel(PricingModel.PER_ACCESS)
                .withLicenseType(LicenseType.OPEN_DATA_LICENSE)
                .withAccessRequestHandler(accessCallbackDummy);
  ```
  
* Creating an Offering Query using the Programming API (see new methods `withName()`, `withCategory()`):
   ```java
     OfferingQuery query = OfferingQuery.create("BasicDemoParkingSpotQuery")
                .withName("Basic Demo Parking Query")
                .withCategory("urn:big-iot:ParkingSpaceCategory")
                .inRegion(BoundingBox.create(Location.create(42.2, 9.1), Location.create(43.0, 9.8)))
                .addInputData(new RDFType("schema:longitude"), ValueType.NUMBER)
                ...
                .addOutputData(new RDFType("datex:parkingSpaceStatus"), ValueType.TEXT)
                ...
                .withPricingModel(BigIotTypes.PricingModel.PER_ACCESS).withMaxPrice(Euros.amount(0.1))
                .withLicenseType(LicenseType.OPEN_DATA_LICENSE);
  ```
 
***NOTE: The most important change is that from this release onwards, Offering Descriptions and OfferingQueries created in code via the Lib can only use existing semantic offering types or categories.***
   * ***A full list of already defined and supported semantic categories is available [here](https://big-iot.github.io/categories/). However, via the Marketplace user interface, you can also propose new categories during creation of an offering. Those "proposed" categories  can then also be used in your code.***
   * ***New semantic types for input and output data can be directly created in program code. Just use the keyword '"proposed"' in front of your new type (e.g. '"proposed:randomValue"').***

***Highlighted features and changes since 0.9.9:***

* Support **specification of a time period (from-to) in offering descriptions / queries**. This allows Providers to define the time period in a data offering, and Consumers to define a time period in the offering query.
   * Sample code extract for this optional feature (applicable to offering descriptions and queries): 
       ```
       .withTimePeriod(new DateTime(2017, 1, 1, 0, 0, 0), new DateTime())
       ```
* Support **specification of a geographic region in offering descriptions / queries**. This allows Providers to define the geographic region by means of a region name, city name, or a bounding box in a data offering, and Consumers to define a geographic region in the offering query.
   * Sample code extract for this optional feature (applicable to offering descriptions and queries): 
       ```
       .inRegion(BoundingBox.create(Location.create(42.1, 9.0), Location.create(43.2, 10.0)))
       .inRegion("Germany")
       .inCity("Barcelona")
       ```
* Support **offerings with restricted access**. This allows Providers to restrict discoverability of an offering (and thus also the ability to subscribe to and access the offering) to a set of organizations.
   * Sample code extract for this optional feature (applicable to offering descriptions): 
       ```
       .restrictedToOrganizations("Bosch", "ATOS")
       ```
* **Accounting support** in Provider and Consumer Lib for access interface. This allows Providers and Consumers to track the access to/of data offerings based on the # of bytes and data records transfered between Providers and Consumers on a subscription + access session basis. Both the Provider and Consumer Lib writes these accounting records in a CSV log file in the directory: `accounting`. It creates different log files for each Provider and Consumer instance based on the name `accounting/<ProviderId>.log` or `accounting/<ConsumerId>.log`

* Support for **project-internal licenses** of offerings (e.g. `.withLicenseType(LicenseType.PROJECT_INTERNAL_USE_ONLY)`)

* Support for **external BIG IoT provider endpoints** has been added. This allows a Provider to register an offering that is hosted on an external BIG IoT provider. The endpoint can be specified and registered as follows:
    ```java
        RegistrableOfferingDescription offeringDescription =
                OfferingDescription.createOfferingDescription("DemoParkingOfferingWithExternalProvider")
                        .withName("Demo Parking Offering With External Provider")
                        ...
                        .withLicenseType(LicenseType.PROJECT_INTERNAL_USE_ONLY);
    
        Endpoints endpoints = Endpoints.create(offeringDescription)
                                       .withEndpointUri("https://127.0.0.1:9443/test");

        provider.register(offeringDescription, endpoints);
    ```
       
 * Further example provider and consumer Java programs are provided [here](https://github.com/BIG-IoT/example-projects/tree/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples).
 
       
### 0.9.9

NOTE: Update to this version requires a minor change to the Programming API (a new BIG IoT Excdeption needs to be handled). All other changes are backward compatibile.

***Highlighted features and changes since 0.9.8:***

* Support **specification of desired Input and Output Data in Offering Queries**. This allows Consumers to discover Offerings not only based on the semantic categories, city, price and license, but also based on concrete semantic Input and Output types. 
   * Sample code extract: `OfferingQuery query = OfferingQuery.create("ParkingQuery")
                ... 
                .addInputData(new RDFType("schema:longitude"), ValueType.NUMBER)
                .addOutputData(new RDFType("schema:longitude"), ValueType.NUMBER)
                ...`
   * A full Consumer example can is shown [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleConsumer.java).      
* Support Consumers to **discover Offerings by Query ID**, based on a Query created via the Web Portal.
   * `List<SubscribableOfferingDescriptionCore> list = consumer.discoverByIdFuture("... Query ID ...").get();`
* Support Consumers to **subscribe to and access Offerings by Offering ID**.
   * Subscription: `Offering offering = consumer.subscribeByOfferingId("... Offering ID ...").get();`
   * Access: `AccessResponse response = offering.accessOneTime(AccessParameters.create()...).get();`
   * A full Consumer example can is shown [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleConsumerSubscriptionById.java).
* Support for **BIG IoT properties files** has been added to simplify the management and update of Provider and Consumer IDs, Secrets, etc. (incl. application specific properties). How this feature can be used is illustrated here:
   * Load propertes files: `BridgeIotProperties prop = BridgeIotProperties.load("example.properties");`
   * Use standard BIG IoT properties:
      `ProviderSpark provider = ProviderSpark.create(prop.PROVIDER_ID, prop.MARKETPLACE_URI, prop.PROVIDER_DNS_NAME, prop.PROVIDER_PORT);
      provider.authenticate(prop.PROVIDER_SECRET);`
   * Access application specific properties: `prop.getProperty("defaultFetchInterval");`
   * Full examples on how to use this for a Consumer and Provider is shown [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleConsumer.java) (Consumer) and [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleProvider.java) (Provider).
* Minor fixes:
   * Marketplace access token is automatically updated prior to expiration.
   * Socket Timeouts have been increased to: 10sec connect, 10sec write and 10sec read.
   * Optimized `.discover()` and `.discoveryContinuous()`: Consumer Lib creates the query on the marketplace only once - subsequent calls only execute the query. 
   

### 0.9.8

NOTE: No update to Programming API required. 

***Highlighted features and changes since 0.9.7:***

* Support Consumers to use `.discoverContinuous(...)` by defining callback functions and a frequency in seconds. See an example consumer [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleConsumerDiscoverContinuous.java).
* Added example provider for usage of access stream feature (introduced in version 0.9.7). See [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleProviderAccessStream.java). 
  * Note: Access streams support providers that offer real-time data as streams in a way that the Provider Lib takes care of the stream handling, i.e. the provider passes newly arriving data points to the Provider Lib, which internally takes care of the queuing and stream management and takes care that each consumer obtains the  data points only once per session - despite multiple access requests.
* Bug Fixes:
  * Disabled automatic connection failure handling in HTTP Client (to avoid message storms on Marketplace after a restart).
  * Fixed response object handling on accessContinuous to avoid leaks.
  * Fixed problem with automatic resubscription of Offerings by Consumers.


### 0.9.7

NOTE: Update to this version requires a minor change to the Programming API (see the `AccessRequestHandler` below). All other changes are backward compatibile.

***Highlighted features and changes since 0.9.6:***

* Extended Consumer Lib to also access Offerings with an HTTP endpoint (before only HTTPS was supported)
* Extended AccessRequestHandler to also provide the Subscriber Id and Consumer information in the callback function
   * `public BigIotHttpResponse processRequestHandler (OfferingDescription offeringDescription, Map<String,Object> inputData)`   --> 
   * `public BigIotHttpResponse processRequestHandler (OfferingDescription offeringDescription, Map<String,Object> inputData, String subscriberId, String consumerInfo)`
* Introduces Provider Lib functionality for *Access Streams* - access streams support providers that offer real-time data as streams in a way that the Provider Lib takes care of the stream handling, i.e. the provider passes newly arriving data points to the Provider Lib, which internally takes care of the queuing and stream management and takes care that each consumer obtains the  data points only once per session - despite multiple access requests.
   * Providers can *queue* stream data in an `RegisteredOffering` with the `.queue()` method
   * For such offerings, the Provider Lib will directly serve those queued data to consumers upon an access requests - without the access callback function
   * `.flush()` will flush queued data in the access stream
   * an `AccessStreamFilterHandler` can be provided during offering creation (with `.withAccessStreamFilterHandler()`) to support input parameter filtering
* Extended Consumer Lib trust manager to trust besides the Provider Lib endpoints also any Provider end point with a certificate that is trusted by the Java VM 
* Improved Lib stability of Marketplece interactions, e.g. re-registration will continue after Marketplace crash
* Extended `BigIotHttpResponse.okay().withBody()` to also allow passing of JSONObjects or JSONArrays in the response
* Bug fixes:
   * `.inCity()` or `.inRegion()` is now optional for Offering Descriptions
   * `.withPricingModel(PricingModel.FREE)` is now also supported by the Provider Lib supported
   * `.withPrice()` and `.withPriceModel()` are now optional and a Free Pricing Model is chosed by default
   * the access response `.map()` function works now also when the response includes unexpected property names - those are simnply ignored
   

### 0.9.6

*Highlighted features and changes since 0.8.0. Checkout [more-java-examples](https://github.com/BIG-IoT/example-projects/tree/master/more-java-examples) on GitHub*

* Renamed package prefix from `org.bigiot.lib` to `org.eclipse.bigiot.lib`
* Renamed `addInputDataElement` to `addInputData` in `RegistrableOfferingDescription`
* Renamed term `AccoutingType` to `PricingModel` in all methods
* Introduced Complex Types
  * On provider side, use `addInputData(/*..*/`, `IOData.createMembers()./*..*/)` to configure your offering
  * On consumer side, use `AccessParameters.create().addNameValue("myComplex", AccessParameters.create()./*..*/)`
* Added support for consumer side integrated offerings (integration mode 2) with the following degrees of freedom
  * HTTP POST, GET, PUT and COAP  
  * XML and JSON encoding
  * Parameters in path, query, body and body template
* Introduced `OfferingSelector` for defining a rule set to select a single offering out of the discovery response
* Map the access response to your POJO 
  * Automatic mapping: `List<MyPojo> list  = res.get().map(MyPojo.class)`
  * Source code annoated remapping: `List<MyPojoAnnoated> list  = res.get().map(MyPojoAnnotated.class)`
  * Programmatic remapping: `List pojo = res.get().map(MyCustomPojo.class, OutputMapping.create().addTypeMapping("schema:geoCoordinates", "myCoordinate")/*..*/)`  
* Introduced HTTP proxy configuration, use `provider.setProxy("127.0.0.1",3128)` and `provider.addProxyBypass("172.17.17.100")`)
* Supported automatic renewal of offerings
* Specification of route for offering registration is now optional
* Enabled offering queries including a subset of search parameters (search for all type of offerings in Barcelona)
* Added `inCity` for creating offering descriptions and offering queries
