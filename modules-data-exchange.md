# Modules integration

The Hit2Gap Core Platform (BEMServer) is a middleware that facilitates the communication between data captured on-site from monitored buildings and smart modules.
It is based on standard web technologies:

1. Semantic web technologies to describe monitored data
2. REST APIs to enable communication between the different entities

For a module to be integrated in an BEMServer environment, it needs to be registered in an instance of BEMServer, to get some rights on it, to be able to get data from BEMServer and potentially to push some, and if necessary to have its frontend integrated in the Hit2Gap web portal.

In this file, we focus on data exchange between modules and the Core Platform; front-end integration and security aspects are described in the other files of the current directory.

## Module Registration

The first step for a module developer to get its module interacting with an BEMServer instance is to get it registered on it. In order to do so, various steps needs to be handled, by the module developer itself, and the BEMServer administrator.

### Module developer

The module developer needs to provide information about its module to either the administrator, or the BEMServer directly.
In the second case, this can be done through the following steps:

* inform the module (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/services)
```
POST https://h2g-platform-core.nobatek.com/api/v0/services/ 
```
The operation returns an UUID (Unique Identifier) for the module

* inform one or many model(s) for the module (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/models)
```
POST https://h2g-platform-core.nobatek.com/api/v0/models/
```
The operation returns an UUID (Unique Identifier) for the model. The UUID of the module is required.

* inform one/many output(s) for the module (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/outputs)
An output can be of two forms: it is either a time series, or an event. If the module generate predicted values, simulation outputs... then the output is a time series. If the module generates some information about the status of the monitored building, then it generates an event.
The UUID of both the module and the model are required.
```
POST https://h2g-platform-core.nobatek.com/api/v0/outputs/events/
POST https://h2g-platform-core.nobatek.com/api/v0/outputs/timeseries/
```

Beware these initial steps are only used to give a description of the module, and its outputs. All the steps are obviously not mandatory. In particular, if the module is a visualization module (produces no outputs), no model and output need to be created.
After this initial step, no time series or event is created, but the module is allowed to create some.

### BEMServer administrator

In parallel, the BEMServer administrator is in charge of defining a profile for the module developer, based on the information provided by the latter. This information are :
- provide the developer with an identifier
- assign him a scope [role, sites] so that the module can have an access to the data he needs on the different sites, based on its role.
Depending on the whether the module is running in a scheduled mode or on-demand, a certificate will be generated for the module.

## Get data from BEMServer

Data are obtained using ```GET``` operations on the different APIs describing sites and measures. Data are structured in a hierarchical mode, and separated in the following categories

### Building description

```
Site                        (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/sites)
|--Building                 (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/buildings)
   |--Floor                 (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/floors)
      |--Slab               (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/slabs)
      |--Space              (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/spaces)
         |--Façade          (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/facades)
            |--Window       (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/windows)
```
Additionally the concept of ```Zone``` can be used, as a logical grouping of spaces, to ease data description (fro instance, heating system can be associated to a group of rooms, rather than to a specific floor/building...).

### Measures description

```
Sensor                      (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/sensors)
  |--Measure                (https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/measures)
```
The description of both sensors and measures is based on the fact that a single sensor can have the possibility of performing more than one measures.
A sensor is associated with the UUID of site/building/floor/space that identifies the physical position of the sensor itself.
**Beware that the term sensor refers to a device performing a measure. As such, it can refer to a meter, a smart phone... **

A measure can also be associated with some UUIDs referring to a location in a building, that refers to the location for which the measure is made.
For instance, a meter can located at the basement (location for sensor) but measure the energy consumption for heating the first floor (location for measure).

### Events and Time Series

Events are generated by modules. Time Series can be generated either by modules, or by sensors.

#### Events

Events are accessed performing a GET operation on the corresponding API:
```
https://h2g-platform-core.nobatek.com/api/v0/events/
```
The online documentation provides the list of filters that can be used. In particular, the dates at which the events are generated is of particular importance.

##### Time Series

Time Series are accessed performing a GET operation on the corresponding API:
```
https://h2g-platform-core.nobatek.com/api/v0/timeseries/{timeseries_id}
```
Where ```timeseries_id``` uniquely identifies a measure/module output.
**IN THE HIT2GAP PROJECT, THIS ID CORRESPONDS TO THE VALUE IN THE ```external_id``` field of the measure/module output description in the API**

## Pushing values to an BEMServer instance

Once the module, its models and outputs are fully informed, pushing data to an instance of openBMs is straightforward.

For events, perform a POST on the event API:
```
POST https://h2g-platform-core.nobatek.com/api/v0/events/
```
The full documentation is provided at https://h2g-platform-core.nobatek.com/api/v0/api-docs/redoc#tag/events/paths/~1events~1/post.
**For each of the element for which an ID is required (```site_id```, ```building_id```, ```floor_id```, ```space_id```, ```sensor_ids```) the UUID of the corresponding element must be used.
Fields ```application``` and ```model``` must also refer to the UUID of the module and of its model.**

For Time Series, use a PATCH operation on the time series API (using the external_id of the measure/module output):
```
PATCH https://h2g-platform-core.nobatek.com/api/v0/timeseries/{timeseries_id}
```
