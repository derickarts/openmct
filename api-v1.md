# Building Applications With Open MCT

## Scope and purpose of this document

This document is intended to serve as reference material for a developer who 
wishes to develop an application based on Open MCT. It will provide details of 
the public facing API necessary to extend the Open MCT platform meet common use 
cases such as integrating with a telemetry source. 

The best place to start is with the Open MCT tutorials. These will walk you through the 
process of getting up and running with Open MCT, as well as addressing some common 
developer use cases.

## Building From Source 

The latest version of Open MCT is available from [our GitHub repository](https://github.com/). 
If you have [node.js](https://nodejs.org/en/) installed, an easy way to get and build 
Open MCT is as an npm dependency 

```
npm install nasa/openmct
```

This will fetch the Open MCT source from our GitHub repository, and build 
a minified version that can be included in your application. The output of the 
build process is placed in a `dist` folder, which can be copied out to another 
location as needed. The content of this folder will include a minified javascript 
file named `openmct.js` as well as  

## Starting an Open MCT application

To start a minimally functional Open MCT application, it is necessary to include 
the Open MCT source, enable some basic plugins, and bootstrap the application. 
The tutorials walk through the process of getting Open MCT up and running from scratch,
but provided below is a minimal HTML template that includes Open MCT, installs 
some basic plugins, and bootstraps the application. It assumes that Open MCT is 
installed as a node module, as described in [Building From Source](#building-from-source). 
This approach includes openmct using a simple script tag, resulting in a global 
variable named `openmct`. This `openmct` object is used to make API calls. 
Open MCT is packaged as a UMD (Universal Module Definition) module, so common 
script loaders are supported.

```
<!DOCTYPE html>
<html>
<head>
    <title>Open MCT</title>
    <script src="node_modules/openmct/dist/openmct.js"></script>
</head>
<body>
    <script>
        openmct.setAssetPath('node_modules/openmct/dist');
        openmct.install(openmct.plugins.localStorage);
        openmct.install(openmct.plugins.myItems);
        openmct.install(openmct.plugins.UTCTimeSystem());
        openmct.install(openmct.plugins.espresso);
        openmct.start();
    </script>
</body>
</html>

```
Open MCT requires certain assets to be available to it, such as html templates, 
images, and css. To tell Open MCT where these are located, simply call the 
`setAssetPath` function. Typically this will be the same location as the `openmct.js` 
library is included from.

Open MCT does not require any plugins to be installed, but there are some plugins 
bundled with the application that provide UI, persistence, and other default 
configuration which are necessary to be able to do anything with the application 
initially. Any of these plugins could be replaced with a custom plugin however. 
The included plugins are documented in the [Included Plugins](#included-plugins) section.  


## Plugins

### Defining and Installing a new Plugin
```
openmct.install(function install(openmctAPI) {
    // Do things here
    // ...
});
```

A plugin is simply a function that when installed is invoked with one parameter - 
the openmct API. A common approach that we use is to define a plugin as a function 
that returns an installation function, this allows configuration to be specified 
when the plugin is included.

eg.
```
openmct.install(openmct.plugins.Elasticsearch("http://localhost:8002/openmct"));
```
This approach can be seen in all of the [plugins provided with Open MCT](https://github.com/nasa/openmct/blob/master/src/plugins/plugins.js).

## Domain Objects and Identifiers

_Domain Objects_ are the basic entities that represent domain knowledge in Open MCT.
The temperature sensor on a solar panel, an overlay plot comparing 
the results of all temperature sensors, the command dictionary for a spacecraft,
the individual commands in that dictionary, the "My Documents" folder:
All of these things are domain objects.

A _Domain Object_ is simply a javascript object with some standard attributes.  
An example of a _Domain Object_ is the My Items object which is a folder in 
which a user can persist any objects that they create. The My Items object 
looks like this: 

``` javascript
{
    identifier: {
        namespace: ""
        key: "mine"
    }
    name:"My Items",
    type:"folder",
    location:"ROOT",
    composition: []
}
```

### Object Attributes

The main attributes to note are the `identifier`, and `type` attributes.
* `identifier`: A composite key that provides a universally unique identifier for 
this object. The `namespace` and `key` are used to identify the object, and the key 
must be unique within the namespace. 
* `type`: All objects in Open MCT have a type. Types allow you to form an 
ontology of knowledge and provide an abstraction for grouping, visualizing, and 
interpreting data. Details on how to define a new object type are provided below. 
Open MCT uses a number of builtin types. Typically you are going to want to 
define your own if extending Open MCT.

### Domain Object Types

Custom types may be registered via the addType function on the Type registry.
[openmct.types]{@link module:openmct.MCT#types}:
```
openmct.types.addType('my-type', {
    label: "My Type",
    description: "This is a type that I added!",
    creatable: true
});
```

The Open MCT tutorials provide a step-by-step example of defining a new object
type.

## Root Objects

In many cases, you'd like a certain object (or a certain hierarchy of
objects) to be accessible from the top level of the application (the
tree on the left-hand side of Open MCT.) It is typical to expose a telemetry
dictionary as a hierarchy of telemetry-providing domain objects in this
fashion.

To do so, use the [`addRoot`]{@link module:openmct.ObjectAPI#addRoot} method
of the [object API]{@link module:openmct.ObjectAPI}:

```
openmct.objects.addRoot({
        key: "my-key", 
        namespace: "my-namespace" 
    });
```

Root objects are loaded just like any other objects, i.e. via an object
provider.

## Object Providers



## Composition Providers

The "composition" of a domain object is the list of objects it contains,
as shown (for example) in the tree for browsing. Open MCT provides a
[default solution](#default-composition-provider) for composition, but there
may be cases where you want to provide the composition of a certain object
(or type of object) dynamically.

### Adding Composition Providers

You may want to populate a hierarchy under a custom root-level object based on 
the contents of a telemetry dictionary. To do this, you can add a new 
CompositionProvider:

```
openmct.composition.addProvider({
    appliesTo: function (domainObject) {
        return domainObject.type === 'my-type';
    },
    load: function (domainObject) {
        return Promise.resolve(myDomainObjects);
    }
});
```

### Default Composition Provider

The default composition provider applies to any domain object with
a `composition` property. The value of `composition` should be an
array of identifiers, e.g.:

```js
var domainObject = {
    name: "My Object",
    type: 'folder',
    composition: [
        {
            id: 'foo:412229c3-922c-444b-8624-736d85516247',
            namespace: 'foo'
        },
        {
            key: 'd6e0ce02-5b85-4e55-8006-a8a505b64c75',
            namespace: 'foo'
        }
    ]
};
```

## Telemetry Providers

## Included Plugins

Open MCT is packaged along with a few general-purpose plugins:

* `openmct.plugins.CouchDB` is an adapter for using CouchDB for persistence
  of user-created objects. This is a constructor that takes the URL for the
  CouchDB database as a parameter, e.g.
  `openmct.install(new openmct.plugins.CouchDB('http://localhost:5984/openmct'))`
* `openmct.plugins.Elasticsearch` is an adapter for using Elasticsearch for
  persistence of user-created objects. This is a
  constructor that takes the URL for the Elasticsearch instance as a
  parameter, e.g.
  `openmct.install(new openmct.plugins.CouchDB('http://localhost:9200'))`.
  Domain objects will be indexed at `/mct/domain_object`.
* `openmct.plugins.Espresso` and `openmct.plugins.Snow` are two different
  themes (dark and light) available for Open MCT. Note that at least one
  of these themes must be installed for Open MCT to appear correctly.
* `openmct.plugins.LocalStorage` provides persistence of user-created
  objects in browser-local storage. This is particularly useful in
  development environments.
* `openmct.plugins.MyItems` adds a top-level folder named "My Items"
  when the application is first started, providing a place for a
  user to store created items.
* `openmct.plugins.UTCTimeSystem` provides a default time system for Open MCT.

Generally, you will want to either install these plugins, or install
different plugins that provide persistence and an initial folder
hierarchy. Installation is [as described previously](#installing-plugins):

eg.
```
openmct.install(openmct.plugins.LocalStorage());
openmct.install(openmct.plugins.MyItems());
```
