# Orange_Task

### How to build

- If you have eclipse installed you can import a maven project if not use pom.xml to get the dependencies and the code in Toy0Store/src/ folder to bilud the project.

- Change the database credentials in "src/main/resources/application.properties." Example:
```sh
spring.datasource.ur=jdbc:mysql://localhost:3306/toystore
spring.datasource.username=root
spring.datasource.password=
```
- Run the project as a Java application

### Usage 
- List all categories 
```sh
	Send GET request to /categories
```


- List one category  
```sh
	Send GET request to /categories/{id}
```
- Add category  
```sh
    Send Post request to /categories
    
    # the Post request contains a JSON file represnting
    # a category. example:  
    
    {
        "name": "board Games",
        "id": 3
    }
    
  # note: id can be null or any number the database auto increament the id
```
- Update category  
```sh
    Send PUT request to /categories/{id}
 	
    # the PUT request contains a JSON file represnting
    # a category. example:   
    
    {
        "name": "board Games",
        "id": 3
    }
    
  # note: id can be null or any number the category with the id in the url gets modified
```


- Delete category  
```sh
	Send DELETE request to /categories/{id}
````
- List all products
```sh
	Send GET request to /products
````
- List one product
```sh
	Send GET request to /products/{id}
````
- List products by category
```sh
	Send GET request to categories/{id}/products
````

- Add product  
```sh
    Send Post request to /products
    
    # the Post request contains a JSON file represnting
    # a product. example:  
    
   {
        "categoryID": 4,
        "name": "fifa18",
        "id": 2,
        "description": "football",
        "price": 1000
    }
    
  # note: id can be null or any number the database auto increament the id categoryID must be provided
```

- Update product  
```sh
    Send PUT request to /products/{id}
 	
    # the PUT request contains a JSON file represnting
    # a product. example:   
   
   {
        "categoryID": 4,
        "name": "fifa18",
        "id": 2,
        "description": "football",
        "price": 1000
    }
    
  # note: id can be null or any number the category with the id in the url gets modified
```

- Delete product  
```sh
	Send DELETE request to /products/{id}
````
