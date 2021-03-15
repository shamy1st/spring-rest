# Spring REST
**REST**: REpresentational State Transfer. / REST API calls over HTTP.

* lightweit approach for communicating between applications. / Language independent.
* data format -> any, XML, JSON
* **JSON**: JavaScript Object Notation.
* REST API = RESTful API = REST Web Services = RESTful Web Services = REST Services = RESTful Services

---

### JSON

* **Data Binding**: convert JSON data to java POJO.
  * also known as Mapping, Serialization/Deserialization, Marshalling/Unmarshalling.

* Spring uses Jackson project to handle data binding.

* Jackson Data Binding API

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>

---

### REST over HTTP

--     | url                         | description
-------|-----------------------------|-----------------------------
POST   | /api/customers              | create new customer
GET    | /api/customers              | read a list of customers
GET    | /api/customers/{customerId} | read a single customer
PUT    | /api/customers              | update an existing customer
DELETE | /api/customers/{customerId} | delete an existing customer

        @RestController
        @RequestMapping("/api")
        public class CustomerRestController {

            // autowire the CustomerService
            @Autowired
            private CustomerService customerService;
            
            // add mapping for GET /customers
            @GetMapping("/customers")
            public List<Customer> getCustomers() {
                return customerService.getCustomers();
            }
            
            // add mapping for GET /customers/{customerId}
            @GetMapping("/customers/{customerId}")
            public Customer getCustomer(@PathVariable int customerId) {
                Customer theCustomer = customerService.getCustomer(customerId);
                
                if (theCustomer == null) {
                    throw new CustomerNotFoundException("Customer id not found - " + customerId);
                }
                
                return theCustomer;
            }
            
            // add mapping for POST /customers  - add new customer
            @PostMapping("/customers")
            public Customer addCustomer(@RequestBody Customer theCustomer) {
                // also just in case the pass an id in JSON ... set id to 0
                // this is force a save of new item ... instead of update
                
                theCustomer.setId(0);
                
                customerService.saveCustomer(theCustomer);
                
                return theCustomer;
            }
            
            // add mapping for PUT /customers - update existing customer
            @PutMapping("/customers")
            public Customer updateCustomer(@RequestBody Customer theCustomer) {
                customerService.saveCustomer(theCustomer);
                
                return theCustomer;
            }
            
            // add mapping for DELETE /customers/{customerId} - delete customer
            @DeleteMapping("/customers/{customerId}")
            public String deleteCustomer(@PathVariable int customerId) {
                Customer tempCustomer = customerService.getCustomer(customerId);
                
                // throw exception if null
                
                if (tempCustomer == null) {
                    throw new CustomerNotFoundException("Customer id not found - " + customerId);
                }
                        
                customerService.deleteCustomer(customerId);
                
                return "Deleted customer id - " + customerId;
            }	
        }

### Exception Handling

1. **create custom error response**

        public class CustomerErrorResponse {

            private int status;
            private String message;
            private long timeStamp;
            
            public CustomerErrorResponse() {
                
            }

            public CustomerErrorResponse(int status, String message, long timeStamp) {
                this.status = status;
                this.message = message;
                this.timeStamp = timeStamp;
            }

            public int getStatus() {
                return status;
            }

            public void setStatus(int status) {
                this.status = status;
            }

            public String getMessage() {
                return message;
            }

            public void setMessage(String message) {
                this.message = message;
            }

            public long getTimeStamp() {
                return timeStamp;
            }

            public void setTimeStamp(long timeStamp) {
                this.timeStamp = timeStamp;
            }
        }

2. **create custom Student Exception**

        public class CustomerNotFoundException extends RuntimeException {

            public CustomerNotFoundException() {
            }

            public CustomerNotFoundException(String message) {
                super(message);
            }

            public CustomerNotFoundException(Throwable cause) {
                super(cause);
            }

            public CustomerNotFoundException(String message, Throwable cause) {
                super(message, cause);
            }

            public CustomerNotFoundException(String message, Throwable cause, boolean enableSuppression,
                    boolean writableStackTrace) {
                super(message, cause, enableSuppression, writableStackTrace);
            }

        }

3. **update REST service to throw exception** (AOP implementation)


        @ControllerAdvice
        public class CustomerRestExceptionHandler {

            // Add an exception handler for CustomerNotFoundException
            
            @ExceptionHandler
            public ResponseEntity<CustomerErrorResponse> handleException(CustomerNotFoundException exc) {
                
                // create CustomerErrorResponse
                
                CustomerErrorResponse error = new CustomerErrorResponse(
                                                    HttpStatus.NOT_FOUND.value(),
                                                    exc.getMessage(),
                                                    System.currentTimeMillis());
                
                // return ResponseEntity
                
                return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
            }
            
            
            // Add another exception handler ... to catch any exception (catch all)

            @ExceptionHandler
            public ResponseEntity<CustomerErrorResponse> handleException(Exception exc) {
                
                // create CustomerErrorResponse
                
                CustomerErrorResponse error = new CustomerErrorResponse(
                                                    HttpStatus.BAD_REQUEST.value(),
                                                    exc.getMessage(),
                                                    System.currentTimeMillis());
                
                // return ResponseEntity
                
                return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
            }
            
        }

## RESTful Web Services with Spring and Spring Boot

### Project Creation

![](https://github.com/shamy1st/spring-rest/blob/main/images/project-creation.png)

        application.properties:
            spring.datasource.url=jdbc:h2:mem:testdb
            spring.datasource.driverClassName=org.h2.Driver
            spring.datasource.username=sa
            spring.datasource.password=password
            spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
            spring.jpa.show-sql=true
            spring.h2.console.enable=true
            
            management.endpoints.web.exposure.exclude=*

            spring.data.rest.basePath=/api
            spring.data.rest.default-page-size=3

        data.sql
            INSERT INTO Users (id, name) VALUES (1, 'Ahmed');
            INSERT INTO Users (id, name) VALUES (2, 'Mohamed');
            INSERT INTO Users (id, name) VALUES (3, 'Abdalla');
            INSERT INTO Users (id, name) VALUES (4, 'Hassan');
            INSERT INTO Users (id, name) VALUES (5, 'Elshamy');
            INSERT INTO Users (id, name) VALUES (6, 'Amr');
            INSERT INTO Users (id, name) VALUES (7, 'Mona');
            INSERT INTO Users (id, name) VALUES (8, 'Manal');

        @Entity(name="Users")
        @Data
        public class User {
            @Id
            private int id;
            @Column
            private String name;

            public User() {

            }
        }

        @RepositoryRestResource
        public interface UserRepository extends JpaRepository<User, Integer> {

        }

### Swagger Documentation Format

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>3.0.0</version>
        </dependency>


        @Configuration
        @EnableSwagger2
        public class SwaggerConfig {

            @Bean
            public Docket api() {
                return new Docket(DocumentationType.SWAGGER_2)
                    .select()
                    .apis(RequestHandlerSelectors.any())
                    .paths(PathSelectors.any())
                    .build();
            }
        }

        http://localhost:8080/v2/api-docs
        



