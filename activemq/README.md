# LAB 1
In this lab we will install ActiveMQ, get it running and take a look at the Web GUI.
We will also take a look at the installation and some settings.

## Install ActiveMQ
* Download the software from the following URL: http://activemq.apache.org/download.html
* Unpack the compressed archive to some good location
  * Will be referred to as: [activemq_home]
* Start the broker into console mode
  * ```# [activemq_home]/bin/activemq console```
* Log in to the web GUI
  * URL: http://localhost:8161
  * UID: admin
  * PW: admin
* Check the Queues page
  * (should be empty)

## Check out the installation directory [activemq_home]
* Take a look at the file: ```[activemq_home]/conf/activemq.xml```
  * What do we recognize in this configuration?
  * Look at the persistence configuration. ```<persistenceAdapter/>```
  * another interesting section is ```<systemUsage/>```
  * how about ```<transportConnectors/>``` and ```<import resource="jetty.xml"/>```
