# LAB #2

* Create a spring boot project with camel and activemq support.
  * (i.e. using spring boot initializer)
* Add the broker-url to your application.properties file
  * ```spring.activemq.broker-url=tcp://localhost:61616```
* Configure your application to keep on running
  * ```camel.springboot.main-run-controller=true```
* Create a class for your routes.
  * remember the following:
    * @Component annotation 
    * extending RouteBuilder
    * @Override annotation on the configure() method.
  * Create a route 
    * that will listen to a file directory.
    * send the contents of any file in this directory into a queue and deletes the files when they are consumed.
* Start the application using ‘mvn spring-boot:run’
* look for a directory with the name you specified in the route
  * This will be in the root of your project folder.
* Create a textfile in the root of your project folder containing some text.
* copy the file into the directory (keeping a copy in the root since the file will be deleted once consumed)
* Check your queue status in the ActiveMQ web GUI
  * Also inspect the message contents.
* Stop the application
* Create another route
  * that will consume messages from the queue you used in the other route and send the message contents to the log / standard output.
