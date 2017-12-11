
## 1. Bringing Spring Boot & Apache CXF up and running

Our first goal should be to get Spring Boot up together with Apache CXF. As a starting point, I love to use the Spring Initializr. Just choose “Web” and optionally “DevTools”. After importing the resulting project into our IDE, we have to add the correct dependency for Apache CXF. If you use Maven as I do, I added the dependencies cxf–rt–frontend–jaxws and cxf–rt-transports-http along with the current CXF version 3.1.4 to my pom.xml. After our build tool has imported both libraries and some dependencies, we can add two spring beans to our HigCxfApplication.java, which will initialize CXF completely:

```java
@SpringBootApplication
public class SimpleBootCxfApplication {
 
    public static void main(String[] args) {
	SpringApplication.run(SimpleBootCxfApplication.class, args);
    }
 
    @Bean
    public ServletRegistrationBean dispatcherServlet() {
        return new ServletRegistrationBean(new CXFServlet(), "/soap-api/*");
    }
 
    @Bean(name=Bus.DEFAULT_BUS_ID)
    public SpringBus springBus() {      
        return new SpringBus();
    }
}
```

The CXFServlet will process all SOAP requests that reach our URI /soap-api/* and the cxf-SpringBus gets the CXF framework up and running, with all needed CXF modules – see CXF´s architecture. As soon as we start our HigCxfApplication.java (simple “Run as…” is enough), Spring Boot initializes its embedded Tomcat, registers the CXFServlet, and we can type the following URL into our Browser http://localhost:8080/soap-api. We´ve done well if CXF says:

No services have been found.
…as there are no services deployed until now 🙂

## 2. From WSDL to Java…
To reach our “no XML” goal, we could use a XML databinding framework such as Java Architecture for XML Binding (JAXB). In combination with the “Java API for XML Web Services” (JAX-WS) we have a comfortable chance to provide SOAP web services with Java standard tools – the reference implementation (RI) is part of the Java runtime and can be used out-of-the-box.

Again everything will be reproducable, as we extend our example from step 1. The running example sources can be found in the project step2_wsdl_2_java_maven.

The structure of the mentioned web service example http://wsf.cdyne.com/WeatherWS/Weather.asmx?WSDL is not comparable with our Enterprise WSDLs out there. As I said, I extended this example until it was more comparable with the bigger WSDLs, especially thinking about “the how” – not really the actual size.

If you can hardly remember what this WSDL thingy was… Just remember one thing: read it from bottom to the top. 🙂

Throw out unnecessary things…

Our example WeatherService has many wsdl:ports that connect to their own wsdl:binding each, which leads to unnecessary complexity. So in our derived web service there´s only one wsdl:port:

```xml
<wsdl:service name="Weather">
	<wsdl:port name="WeatherService" binding="weather:WeatherService">
		<soap:address location="http://localhost:8095/soap-api/WeatherSoapService_1.0"/>
	</wsdl:port>
</wsdl:service>
```

This implies that while we have three web service methods, they are just defined once – and not repeated three times because of the many ports:

```xml
<wsdl:operation name=“GetWeatherInformation“>…</wsdl:operation>
<wsdl:operation name=“GetCityForecastByZIP“>…</wsdl:operation>
<wsdl:operation name=“GetCityWeatherByZIP“>…</wsdl:operation>
```

The wsdl files are located under the folder wsdl

The wsdl:portType finally defines what the (XML) requests and reponses of our web service methods will look like – and how they will act in error situations.

### Nested XSD imports…

Following the definition of the wsdl:messages element, the fragments of the XML schema are referenced. Here´s the biggest difference between our derived example and the original WeatherService:

Our WSDL imports the central Weather1.0.xsd, which again imports weather-general.xsd and weather-exception.xsd.

And there are more imports in those XSDs. The effort was necessary to emulate the considerably bigger and more complex web services that are used in the field out there. Not really reaching that size, our service helps us show many techniques that matter to get things working. I was really anxious if my choosen toolchain could handle that WSDL. It wasn´t really a problem. We´ll see it step by step.

### WSDL-2-Java (finally!)

Because our WSDL describes our web service API contract-first, our dependent Java classes should always represent the current state of the WSDL. It should therefore regularly be generated from it. Furthermore, as our WSDL describes all aspects of our API, we don´t want to check in those generated Java classes into our version control system.

These requirements are easily implemented using a Maven plugin which will generate all necessary bindings and classes in the generate-sources phase, which includes all the technical and the functional classes our web service needs to live.

If you have a look at the already recommended getting started guides, the jaxb2-maven-plugin is used in most of them. If you look a bit further, you´ll find lots of plugins and corresponding discussions, which one is the best. But because we decided to use JAX-WS, the usage of the Maven plugin of the JAX-WS-commons project seems to be a good choice.

But be careful: The JAX WS Maven plugin is back under mojohaus goverance. You can track the development progress on Github. Because of this we´ll use the more recent groupId org.codehaus.mojo instead of org.jvnet.jax-ws-commons in our Maven poms.

Configuring the Maven plugin

The configuration of the jaxws–Maven–plugin shouldn´t be underestimated. So let´s look at the build section of our pom:

```xml
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>jaxws-maven-plugin</artifactId>
	<version>2.4.1</version>
	<configuration>...</configuration>
</plugin>
```

Starting from the <configuration>-tag it´s getting interesting:

```xml
<configuration>
	<wsdlUrls>
		<wsdlUrl>src/main/resources/service-api-definition/Weather1.0.wsdl</wsdlUrl>
	</wsdlUrls>
	<sourceDestDir>target/generated-sources/wsdlimport/Weather1.0</sourceDestDir>
	<vmArgs>
		<vmArg>-Djavax.xml.accessExternalSchema=all</vmArg>
	</vmArgs>
</configuration>
```

The <wsdlUrl> defines where our WSDL resides as resource and the <sourceDestDir> decides where to put the generated Java classes. Because we´ve chosen a realistic example, this configuration wouldn’t work for our WSDL with this bunch of imported and nested XSDs. So we have to add a <vmArg>: -Djavax.xml.accessExternalSchema=all makes sure that no XML schema is forgotten.

After the necessary definition of the Maven goal wsimport, we use a second plugin: the build-helper-maven-plugin to add the generated Java classes to our classpath. Now we can use them like any other class in our project. If you want to give it a try, just run

```
mvn clean generate-sources
```

on commandline after you got the project from step2_wsdl_2_java_maven. This should generate all necessary classes into the folder target/generated-sources/wsdlimport/Weather1.0. If you inspect the result, you should recognize the similarity between the package-structure and how the sample-XSDs are structured.

Finally don´t forget to prevent the generated Java classes from beeing checked in into your version control system, as we don´t want to have them there. If you use Git, you can simply put the /target-Folder into your .gitignore – if it´s not already there.

## 3. a running SOAP-Endpoint

This next step will finally bring our first SOAP end point to life. So let´s extend our project from step2.

As we now begin to extend our configuration, we should grant our project its own @Configuration-annotated Class. There we´ll initialize CXF and our first end point. As a consequence, our Application class is reduced to the minimum necessary to fire up Spring Boot. Additionally even with SpringBoot we can use a @ComponentScan to fasten scanning of Spring beans and components.

Again we see the beans SpringBus and ServletRegistrationBean inside our @Configuration-Class. To configure the Endpoint, we need two additional beans. Let´s start by defining the service end point interface (SEI):

```java
@Bean
public WeatherService weatherService() {
	return new WeatherServiceEndpoint();
}
```

The SEI implementing class WeatherServiceEndpoint is not generated and we have to create one manually. This class represents the place where the functional implementation begins. But within this step we only have to create this class so that we can instantiate it inside our bean defintion.

The second bean to define is javax.xml.ws.Endpoint. This is the point where the Apache CXF docs get really annoying because there´s not really a description to define all necessary beans without XML. But hey, this is where this tutorial comes in handy. 🙂

The crucial point is to return an instance of org.apache.cxf.jaxws.EndpointImpl, which we forward to the SpringBus and our WeatherServiceEndpoint via constructor-arg:

```java
@Bean
public Endpoint endpoint() {
	EndpointImpl endpoint = new EndpointImpl(springBus(), weatherService());
	endpoint.publish("/WeatherSoapService_1.0");
	endpoint.setWsdlLocation("Weather1.0.wsdl");
	return endpoint;
}
```

Furthermore we have to use the .publish-Method of our org.apache.cxf.jaxws.EndpointImpl to define the last part of our WebService-URI.

If you now fire up our application, as you´re used to with SpringBoot, a browser should show our WeatherService beneath “Available SOAP services”, when we point it to http://localhost:8080/soap-api/ – including all three available web service methods.

SoapUI is available for download [here](https://www.soapui.org/)

As part of the next step, we´ll see how we can call our web service from within a unit or integration test. At this current step, a test call with SoapUI should do. If you start SoapUI and paste our WSDLs URI into the corresponding field inside “New SOAP Project”, everything necessary should be generated to start a real SOAP request against our end point. If you give it a try, you´ll notice an error-free response that doesn´t contain much for the moment.

So finally, our first SOAP end point with SpringBoot, Apache CXF and JAX-WS is up and running. 