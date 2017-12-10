Spring
======

1. Ladda ner och installera senaste versionen av Tomcat 8 (https://tomcat.apache.org/download-80.cgi)
2. Anpassa klassen se.hig.springapplication.SpringApplicationInitializer så den startar Springapplikationen med
   annotation-driven application-context (https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/htmlsingle/#mvc-container-config)
3. Skapa en Annotations driven application-context (https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/htmlsingle/#beans-java-instantiating-container)

Nu har ni en spring applikation! Nu ska vi göra en enkel REST-applikation med properties

1. Använd instruktionerna i (https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/htmlsingle/#beans-environment)
   för att skapa en @PropertySource som pekar på din properties fil
2. Skapa en property "se.hig.from" som innehåller en vem vi ska hälsa från
3. Skapa en enkel RestController som tar som inparameter ett namn, och som returnerar en json-sträng "Hello <namn>, I am <värde från properties-filen>"
