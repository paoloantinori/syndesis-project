# Integration Design

This is going to be a technical description of the logical idea behind an **Integration** and how that logical model is implemented.
It's not a comprehensive definition of how Syndesis work. That would take into account the definition of the pods that are deployed and their respective responsibilities.

Be aware that we currently don't have the logical model defined in any document or UML diagram. There is actually an ER Diagram [here](https://github.com/syndesisio/syndesis-rest/blob/master/docs/design/ui-domain-object-map.adoc), but it's not reflecting entirely the current status of the information architecture.

The entry point for out model is the `Integration` [entity](https://github.com/syndesisio/syndesis-rest/blob/master/model/src/main/java/io/syndesis/model/integration/Integration.java), that represents an interaction between systems.

This object incapsulates 2 kinds of information:
1. information about the definition of an Integration, that is what it has to do
2. runtime information about a specific Integration instance, for example `getStatus()` that tells you if the Integration is currently deployed and running

The real definition of what an `Integration` does, is delegated to the following set of entities:

- `Connector`
- `Connection`
- `Action`
- `Step`

##### Connector
Is used to define a set of common properties the use can define on a `Connection` entity, coupled with this `Connector`. Think of `Connector` as a configurable schema, used to define the shape of a corresponding `Connection` entity.  
Another responsibility of the `Connector` is to aggregate a list of `Action`s.  
An example of **Connector** can be the **Twitter** Connector, where you define some of the properties that are required by all the operation it will provide.  
So properties could be for example `user`, `password`, `token` and so on. Just understand that you are not setting the values for these keys here. You are just declaring what are the fields.

##### Connection
It can be seen as a specific instance of `Connector`. While on the `Connector` entity you were just declaring the fields that the specific Connector supported, here you are passing the corresponding values for thos fields.
For example it's here that you specify that the value for the field `user` is for instance `scott` and the value for the `password` field is `tiger`.

##### Action
`Action` represents a specific operation within the scope of a `Connector`. 
An `Action` has its own set of of properties.
And it's in the `Action` that there is the binding with the physical code library that is able to provide the connectivity with a specific system/technology: `CamelConnectorGAV`. In this field, you specify the coordinates of the Camel Connector component that this Action delegates its logical operations to.
In the Twitter example, an operation might be `postMessage()`.

##### Step
`Step` is a generic entity, that represents a pipeline operation in your Integration. It might be a transformation, a logging call or anything else.
Due to all this flexibility `Step` is actually a semantically poor entity. Everythin but it's type is optional: it can have references to an `Action` or a `Connection`, but it doesn't have to. If it needs them or not, depends on its `type` that is its most important information.
An important kind of `Step` is `endpoint`. Unluckily, there is not an enum that list these kinds. So the relationship is weak. You have to take care to use a type that the rest of the application is able to understand, because there is no preemptive way to validate your choice.


So the overall idea is that your `Integration` define 2 kind of information:
1. a catalog of technologies and configured endpoints
2. a pipeline of operations you need to perform, using the entities you have prepared at step 1

you define some `(Technology)Connector`, where you define all the properties you allow the developer to give a value to in a corresponding set of `Connection`s. You also link this `Connector` with a series of operations that are tied to this technology. These operations are represented by the class `Action`. In each `Action` you declare also eventual additional properties.


Before digging into the Technical Extension Design we need to be sure we have a correct understanding of what a standalone Integration is and which components it involves.

A logical **Integration** is implemented as a Camel Route, running in its own Camel Context, bootstrapped as a Spring Boot application, running in its own jvm, running in its own container, running in its own pod.

From a development workflow point of view, the starting point of the above relationship is the following:

Model https://github.com/syndesisio/syndesis-rest/blob/master/docs/design/ui-domain-object-map.adoc

1. a `.yaml`  in combination with Kubernetes Secrets for externalized variables, is used to define the integration.
2.  


