#Spring 4 MVC REST Controller Service Example (JSON CRUD Tutorial) 

A step by step tutorial to understand Spring 4 MVC REST API and to create RESTful service using Spring 4.

Spring 4 MVC REST provides powerful APIs to built complete RESTful services. Let us understand the core concept and create simple web application using Maven and make our CRUD based REST service.

Getting started with Spring 4 MVC REST Controller
For this project we will use following tools and technologies.

* Java 1.8
* Spring MVC 4.3.13.RELEASE
* Tomcat 7
* Maven 3
* POSTMan (optional)

The demo REST application will have Customer resource. This customer resource can be accessed using standard GET, POST, PUT, DELETE http methods. We will create below REST endpoints for this project.

| REST Endpoint	  | HTTP Method	| Description
| -------------   | ----------- | -----------
| /customers	    | GET	        | Returns the list of customers
| /customers/{id}	| GET	        | Returns customer detail for given customer {id}
| /customers	    | POST	      | Creates new customer from the post data
| /customers/{id}	| PUT	        | Replace the details for given customer {id}
| /customers/{id}	| DELETE      |	Delete the customer for given customer {id}

##1. Create a new Maven Project

If you are using Eclipse then you can use M2Eclipse plugin. Check the tutorial Spring 4 MVC Hello World and follow the section 1.

Alternatively if you want to generate Maven webapp using mvn command then follow these steps.

```sh
mvn archetype:create 
  -DgroupId=se.hig.spring
  -DartifactId=SpringRest
  -DarchetypeArtifactId=maven-archetype-webapp
```

This will generate maven application with default project directory structure. You can then run following command and convert the project in Eclipse project.

mvn eclipse:eclipse
And then simply import the project in Eclipse.

##2. Add Spring 4 MVC Maven dependencies (Update pom.xml)

Project structure is created. Now let’s start and add first the maven dependencies for Spring 4 MVC REST in our pom.xml file.

Update pom.xml file and add following dependencies.

#####pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>se.hig.spring</groupId>
	<artifactId>Spring4Rest</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>Spring 4 Rest Service CRUD Example</name>
	<url>http://maven.apache.org</url>
	<properties>
		<java-version>1.8</java-version>
		<springframework.version>4.3.13.RELEASE</springframework.version>
		<jackson.version>2.7.5</jackson.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.0.1</version>
			<scope>provided</scope>
		</dependency>
	</dependencies>
	<build>
		<finalName>HelloWorld</finalName>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.tomcat.maven</groupId>
					<artifactId>tomcat7-maven-plugin</artifactId>
					<version>2.2</version>
					<configuration>
						<path>/springrest</path>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>
</project>
```
 
After updating pom.xml, Eclipse’s maven plugin should start resolving the dependencies.

##3. Set Annotation based Configuration for Spring 4 MVC REST

For this Spring 4 MVC REST tutorial we are going to use Spring’s Java based configuration or annotation based configuration instead of old XML configuration. So now let us add the Java Configuration required to bootstrap Spring 4 MVC REST in our webapp.

Create AppConfig.java file under /src folder. Give appropriate package name to your file. We are using @EnableWebMvc, @ComponentScan and @Configuration annotations. These will bootstrap the spring mvc application and set package to scan controllers and resources.

#####/src/main/java/net/viralpatel/spring/config/AppConfig.java
```java
package se.hig.spring.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "se.hig.spring")
public class AppConfig {

}
```

##4. Set Servlet 3 Java Configuration

Create AppInitializer class under config package. This class will replace web.xml and it will map the spring’s dispatcher servlet and bootstrap it.

#####/src/main/java/net/viralpatel/spring/config/AppInitializer.java
```java
package se.hig.spring.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class[] getRootConfigClasses() {
		return new Class[] { AppConfig.class };
	}

	@Override
	protected Class[] getServletConfigClasses() {
		return null;
	}

	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}

}
```

We have configured the dispatcher servlet using standard Java based configuration instead of the older web.xml. Thus web.xml is no longer required and we can simply delete it.

##5. Create the Customer Model

Next let us create Customer model class that will have few properties such as firstName, lastName, email etc. This bean will hold customer information.

#####/src/main/java/net/viralpatel/spring/model/Customer.java
```java
package se.hig.spring.model;

import java.util.Date;

public class Customer {

	private Long id;
	private String firstName;
	private String lastName;
	private String email;
	private String mobile;
	private Date dateOfBirth;

	public Customer(long id, String firstName, String lastName, String email, String mobile) {
		this.id = id;
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
		this.mobile = mobile;
		this.dateOfBirth = new Date();
	}

	public Customer() {
	}

	//..Getter and setter methods

}
```

##6. Create the Dummy Customer Data Access Object (DAO)

Instead of storing the customer data in database and to make this example simple, we will create a dummy data access object that will store customer details in a list. This DAO class can be easily replaced with Spring Data DAO or custom DAO. But for this example we will keep it easy.

The CustomerDAO contains methods list(), get(), create(), update() and delete() to perform CRUD operation on customers.

#####/src/main/java/net/viralpatel/spring/dao/CustomerDAO.java
```java
package se.hig.spring.dao;

import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Component;

import se.hig.spring.model.Customer;

@Component
public class CustomerDAO {

	// Dummy database. Initialize with some dummy values.
	private static List customers;
	{
		customers = new ArrayList();
		customers.add(new Customer(101, "John", "Doe", "djohn@gmail.com", "121-232-3435"));
		customers.add(new Customer(201, "Russ", "Smith", "sruss@gmail.com", "343-545-2345"));
		customers.add(new Customer(301, "Kate", "Williams", "kwilliams@gmail.com", "876-237-2987"));
	}

	/**
	 * Returns list of customers from dummy database.
	 * 
	 * @return list of customers
	 */
	public List list() {
		return customers;
	}

	/**
	 * Return customer object for given id from dummy database. If customer is
	 * not found for id, returns null.
	 * 
	 * @param id
	 *            customer id
	 * @return customer object for given id
	 */
	public Customer get(Long id) {

		for (Customer c : customers) {
			if (c.getId().equals(id)) {
				return c;
			}
		}
		return null;
	}

	/**
	 * Create new customer in dummy database. Updates the id and insert new
	 * customer in list.
	 * 
	 * @param customer
	 *            Customer object
	 * @return customer object with updated id
	 */
	public Customer create(Customer customer) {
		customer.setId(System.currentTimeMillis());
		customers.add(customer);
		return customer;
	}

	/**
	 * Delete the customer object from dummy database. If customer not found for
	 * given id, returns null.
	 * 
	 * @param id
	 *            the customer id
	 * @return id of deleted customer object
	 */
	public Long delete(Long id) {

		for (Customer c : customers) {
			if (c.getId().equals(id)) {
				customers.remove(c);
				return id;
			}
		}

		return null;
	}

	/**
	 * Update the customer object for given id in dummy database. If customer
	 * not exists, returns null
	 * 
	 * @param id
	 * @param customer
	 * @return customer object with id
	 */
	public Customer update(Long id, Customer customer) {

		for (Customer c : customers) {
			if (c.getId().equals(id)) {
				customer.setId(c.getId());
				customers.remove(c);
				customers.add(customer);
				return customer;
			}
		}

		return null;
	}

}
```

##7. Create the Customer REST Controller

Now let us create CustomerRestController class. This class is annotated with @RestController annotation. Also note that we are using new annotations @GetMapping, @PostMapping, @PutMapping and @DeleteMapping instead of standard @RequestMapping. These annotations are available since Spring MVC 4.3 and are standard way of defining REST endpoints. They act as wrapper to @RequestMapping. For example @GetMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.GET).

#####/src/main/java/net/viralpatel/spring/controller/CustomerRestController.java
```java
package se.hig.spring.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import se.hig.spring.dao.CustomerDAO;
import se.hig.spring.model.Customer;

@RestController
public class CustomerRestController {

	
	@Autowired
	private CustomerDAO customerDAO;

	
	@GetMapping("/customers")
	public List getCustomers() {
		return customerDAO.list();
	}

	@GetMapping("/customers/{id}")
	public ResponseEntity getCustomer(@PathVariable("id") Long id) {

		Customer customer = customerDAO.get(id);
		if (customer == null) {
			return new ResponseEntity("No Customer found for ID " + id, HttpStatus.NOT_FOUND);
		}

		return new ResponseEntity(customer, HttpStatus.OK);
	}

	@PostMapping(value = "/customers")
	public ResponseEntity createCustomer(@RequestBody Customer customer) {

		customerDAO.create(customer);

		return new ResponseEntity(customer, HttpStatus.OK);
	}

	@DeleteMapping("/customers/{id}")
	public ResponseEntity deleteCustomer(@PathVariable Long id) {

		if (null == customerDAO.delete(id)) {
			return new ResponseEntity("No Customer found for ID " + id, HttpStatus.NOT_FOUND);
		}

		return new ResponseEntity(id, HttpStatus.OK);

	}

	@PutMapping("/customers/{id}")
	public ResponseEntity updateCustomer(@PathVariable Long id, @RequestBody Customer customer) {

		customer = customerDAO.update(id, customer);

		if (null == customer) {
			return new ResponseEntity("No Customer found for ID " + id, HttpStatus.NOT_FOUND);
		}

		return new ResponseEntity(customer, HttpStatus.OK);
	}
}
```

##8. Download Tomcat and start application

Get the latest version of Tomcat 8 here - (https://tomcat.apache.org/download-80.cgi)

Unzip somewhere on your hard drive

Copy the war-file you have just created into the webapps folder and start tomcat

Test the application using the url: http://localhost:8080/{app-name}/customers