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


