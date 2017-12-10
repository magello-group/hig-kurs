# Lab CAMEL CXF REST

---

## Goal

The goal is to understand:

-	How does camelContext use CXF REST endpoints?
-	What needs to be configured in blueprint.xml for Camel & CXF?

## Exercise

###	Prerequisites

Item       | Value
---------- | ---
%SMX_HOME% | Location of ServiceMix installation

###	Install prebuilt feature

SMX
```
feature:install examples-camel-cxf-rest
bundle:list
```

### Firefox - check that service is alive
```
localhost:8181/cxf
```
- Q: On which port is the REST service available?
- A: 8989

### 1.6.2.3	Run prebuilt client
```
cd %SMX_HOME%/examples/camel/camel-cxf-rest/camel-cxf-rest-client
mvn compile exec:java
```
- Q: Did a POST, GET & DELETE get executed?

### Analyze code in IDE

#### Import project
```
File > Import > Existing Maven Projects > %SMX_HOME%/examples/camel/camel-cxf-rest/pom.xml
```

#### Analyze “camel-cxf-rest-service” sub project

- Class Person
  - Q: Does it have any JAXB annotations?

- Class ServiceHandler
  - Q: What is the purpose of the class?
  - A: Holds and manages a list of Person.

- File blueprint.xml

|     |
| --- |
| Q: How many routes does it define in the camel context? |
| A: 3 |
| |
| Q: Which kind of endpoint do the routes consume? |
| A: direct-vm |
| |
| Q: What is the difference between ```direct``` & ```direct-vm``` endpoints? **Tip:** Check it out on [camel.apache.org][1]. |
| A: ```direct-vm``` can be used to connect existing routes in the same camel context, as well as from other camel contexts in the same JVM. |
| |
| Q: ```<bean method="getPerson" ref="serviceHandler"/>``` ... How does it know on which class it should call the method? |
| A: Attribute ```ref``` |
| |
| Q: What is ```${header.id}```? |
| A: This is Apache Commons OGNL (Object-Graph Navigation Language) notation. It accesses header with the key **id** in the IN message of the message exchange. |
| |
| Q: What does ```<transform><simple>${header.id}</simple></transform>``` do? |
| A: It replaces the body of the IN message with the contents from header **id**. |
| |

#### Analyze “camel-cxf-rest-route” sub project

##### blueprint.xml 

| Item |     |
| ---  | --- |
| ```<cxf:rsServer>```  | Q: What does attribute ```loggingFeatureEnabled=”true”``` mean? |
|                       | A: It enables the CXF logging interceptors. |
| | |
| ```<camelContext>```    | Q: Why does it send it to ```<recipientList>``` instead of ```<to>```? |
|                       | A: ```<to>``` is not capable to route to a dynamic URI. ```<recipientList>``` can. See [camel.apache.org][2] |
| | |
| ```<cm:property-placeholder>``` | Q: What is ```cxfrs``` in ```cxfrs:bean:rsServer```? What is the purpose of component **cxfrs**? |
| | A: The **cxfrs** component provides integration of camel with Apache CXF for connecting to JAX-RS services hosted in CXF. **Tip:** Check it out on [camel.apache.org][3]. |
| | |
| | Q: What is ```bean``` in ```cxfrs:bean:rsServer```? What is the purpose of component **bean**? |
| | A: The **bean** component binds beans to Camel message exchanges. |
| | |
| | Q: What is ```rsServer``` in ```cxfrs:bean:rsServer```? |
| | A: ```rsServer``` is the ID of the actual bean being called. |




  [1]: http://camel.apache.org/direct-vm.html
  [2]: http://camel.apache.org/how-to-use-a-dynamic-uri-in-to.html
  [3]: http://camel.apache.org/cxfrs.html
  
