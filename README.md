# E-Commerce Product Management REST API - CRUD Implementation

**Student Name:** SHEDRICK BUCAGU ELISA 
**Student ID:** 26939  
**Course:** RESTful API Assignment  
**Institution:** AUCA (Adventist University of Central Africa)

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Technologies Used](#technologies-used)
3. [API Endpoints](#api-endpoints)
4. [Implementation](#implementation)
5. [Testing the API](#testing-the-api)

---

## Project Overview

This project implements a complete RESTful API for managing products in an e-commerce system. The API provides full CRUD (Create, Read, Update, Delete) operations with proper HTTP methods, status codes, and JSON responses.

### Features
- Create new products
- Retrieve all products or a single product by ID
- Update existing products
- Delete products
- Filter products by category

---

## Technologies Used

- **Java 17+**
- **Spring Boot 3.x**
- **Spring Data JPA**
- **Maven**
- **H2/MySQL Database**
- **RESTful Architecture**

---

## API Endpoints

### Base URL
```
http://localhost:8080/api/products
```

### Endpoints Summary

| Operation | HTTP Method | Endpoint | Description |
|-----------|-------------|----------|-------------|
| Create | POST | `/addProduct` | Add a new product |
| Read All | GET | `/getAllProducts` | Get all products |
| Read One | GET | `/getProduct/{id}` | Get product by ID |
| Update | PUT | `/updateProduct/{id}` | Update a product |
| Delete | DELETE | `/deleteProduct/{id}` | Delete a product |
| Filter | GET | `/getByCategory/{category}` | Get products by category |

---

## Implementation

### 1. ProductController.java

```java
package auca.ac.rw.restfullApiAssignment.controller;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import auca.ac.rw.restfullApiAssignment.modal.ecommerce.Product;
import auca.ac.rw.restfullApiAssignment.service.ProductService;

@RestController
@RequestMapping("/api/products")    
public class ProductController {

    @Autowired
    private ProductService productService;
    
    @PostMapping(value = "/addProduct", consumes = MediaType.APPLICATION_JSON_VALUE, 
                 produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> addNewProduct(@RequestBody Product product) {
        String saveProduct = productService.addNewProduct(product);
        if(saveProduct.equals("Product added successfully")) {
            return new ResponseEntity<>(saveProduct, HttpStatus.CREATED);
        } else {
            return new ResponseEntity<>(saveProduct, HttpStatus.CONFLICT);
        }
    }
    
    @GetMapping(value = "/getAllProducts", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<List<Product>> getAllProducts() {
        List<Product> products = productService.getAllProducts();
        return new ResponseEntity<>(products, HttpStatus.OK);
    }
    
    @GetMapping(value = "/getProduct/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> getProductById(@PathVariable Long id) {
        Product product = productService.getProductById(id);
        if(product != null) {
            return new ResponseEntity<>(product, HttpStatus.OK);
        } else {
            return new ResponseEntity<>("Product with id " + id + " not found", 
                                       HttpStatus.NOT_FOUND);
        }
    }
    
    @PutMapping(value = "/updateProduct/{id}", consumes = MediaType.APPLICATION_JSON_VALUE, 
                produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> updateProduct(@PathVariable Long id, @RequestBody Product product) {
        String updateResult = productService.updateProduct(id, product);
        if(updateResult.equals("Product updated successfully")) {
            return new ResponseEntity<>(updateResult, HttpStatus.OK);
        } else {
            return new ResponseEntity<>(updateResult, HttpStatus.NOT_FOUND);
        }
    }
    
    @DeleteMapping(value = "/deleteProduct/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> deleteProduct(@PathVariable Long id) {
        String deleteResult = productService.deleteProduct(id);
        if(deleteResult.equals("Product deleted successfully")) {
            return new ResponseEntity<>(deleteResult, HttpStatus.OK);
        } else {
            return new ResponseEntity<>(deleteResult, HttpStatus.NOT_FOUND);
        }
    }
    
    @GetMapping(value = "/getByCategory/{category}", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<List<Product>> getProductsByCategory(@PathVariable String category) {
        List<Product> products = productService.getProductsByCategory(category);
        return new ResponseEntity<>(products, HttpStatus.OK);
    }
}
```

---

### 2. ProductService.java

```java
package auca.ac.rw.restfullApiAssignment.service;

import java.util.List;
import java.util.Optional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import auca.ac.rw.restfullApiAssignment.modal.ecommerce.Product;
import auca.ac.rw.restfullApiAssignment.repository.ProductRepository;

@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;

    public String addNewProduct(Product product) {
        Optional<Product> existProduct = productRepository.findById(product.getId());
        if(existProduct.isPresent()) {
            return "Product with id " + product.getId() + " already exists";
        } else {
            productRepository.save(product);
            return "Product added successfully";
        }
    }
    
    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }
    
    public Product getProductById(Long id) {
        Optional<Product> product = productRepository.findById(id);
        return product.orElse(null);
    }
    
    public String updateProduct(Long id, Product updatedProduct) {
        Optional<Product> existingProduct = productRepository.findById(id);
        if(existingProduct.isPresent()) {
            Product product = existingProduct.get();
            product.setName(updatedProduct.getName());
            product.setDescription(updatedProduct.getDescription());
            product.setPrice(updatedProduct.getPrice());
            product.setCategory(updatedProduct.getCategory());
            product.setStockQuantity(updatedProduct.getStockQuantity());
            product.setBrand(updatedProduct.getBrand());
            productRepository.save(product);
            return "Product updated successfully";
        } else {
            return "Product with id " + id + " not found";
        }
    }
    
    public String deleteProduct(Long id) {
        Optional<Product> existingProduct = productRepository.findById(id);
        if(existingProduct.isPresent()) {
            productRepository.deleteById(id);
            return "Product deleted successfully";
        } else {
            return "Product with id " + id + " not found";
        }
    }
    
    public List<Product> getProductsByCategory(String category) {
        return productRepository.findByCategory(category);
    }
}
```

---

### 3. ProductRepository.java

```java
package auca.ac.rw.restfullApiAssignment.repository;

import java.util.List;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import auca.ac.rw.restfullApiAssignment.modal.ecommerce.Product;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByCategory(String category);
}
```

---

## Testing the API

### 1. CREATE - Add a Product

**Request:**
```http
POST http://localhost:8080/api/products/addProduct
Content-Type: application/json

{
    "id": 1001,
    "name": "Wireless Bluetooth Headphones",
    "description": "Premium noise-cancelling wireless headphones",
    "price": 89.99,
    "category": "Electronics",
    "stockQuantity": 150,
    "brand": "TechSound"
}
```


**Response (201 CREATED):**
```json
"Product added successfully"
```
SCREENSHOT FOR PRODUCT CREATION
<img width="1803" height="937" alt="Screenshot 2026-02-14 223930" src="https://github.com/user-attachments/assets/088919d0-ea57-442e-93b1-37cd323d0fb4" />

---

### 2. READ - Get All Products

**Request:**
```http
GET http://localhost:8080/api/products/getAllProducts
SCREEN SHOT

```
SCREENSHOT FOR DISPLAY FOR ALL PRODUCTS
<img width="1803" height="654" alt="Screenshot 2026-02-14 230926" src="https://github.com/user-attachments/assets/e00d346e-1d6b-4835-bbd2-546f81d5fab4" />

```

**Response (200 OK):**
```json
[
    {
        "id": 1001,
        "name": "Wireless Bluetooth Headphones",
        "description": "Premium noise-cancelling wireless headphones",
        "price": 89.99,
        "category": "Electronics",
        "stockQuantity": 150,
        "brand": "TechSound"
    }
]
```

---

### 3. READ - Get Single Product by ID

**Request:**
```http
GET http://localhost:8080/api/products/getProduct/1001

```
SCREENSHOT DISPLAY BY ID
<img width="1800" height="788" alt="Screenshot 2026-02-14 231035" src="https://github.com/user-attachments/assets/0edce55b-ac48-412f-aef4-899dd1feed4b" />

```

**Response (200 OK):**
```json
{
    "id": 1001,
    "name": "Wireless Bluetooth Headphones",
    "description": "Premium noise-cancelling wireless headphones",
    "price": 89.99,
    "category": "Electronics",
    "stockQuantity": 150,
    "brand": "TechSound"
}
```

---

### 4. UPDATE - Update a Product

**Request:**
```http
PUT http://localhost:8080/api/products/updateProduct/1001
Content-Type: application/json

{
    "name": "Wireless Bluetooth Headphones Pro",
    "description": "Updated premium headphones with 40-hour battery",
    "price": 99.99,
    "category": "Electronics",
    "stockQuantity": 200,
    "brand": "TechSound"
}
```

**Response (200 OK):**
```json
"Product updated successfully"
```
SCREEN SHOT FOR PRODUCT UPDATE
<img width="1805" height="902" alt="Screenshot 2026-02-14 232359" src="https://github.com/user-attachments/assets/6676d7cf-b491-4925-a5ed-767ee876474d" />

---

### 5. DELETE - Delete a Product

**Request:**
```http
DELETE http://localhost:8080/api/products/deleteProduct/1001
```

**Response (200 OK):**
```json
"Product deleted successfully"
```

SCREEN SHOT FOR PRODUCT DELETION
<img width="1798" height="701" alt="Screenshot 2026-02-14 232454" src="https://github.com/user-attachments/assets/6bafa0bd-a0a8-4dde-b769-5871b0237b45" />

---

### 6. FILTER - Get Products by Category

**Request:**
```http
GET http://localhost:8080/api/products/getByCategory/Electronics

```
<img width="1794" height="570" alt="Screenshot 2026-02-14 232256" src="https://github.com/user-attachments/assets/0b3a4af3-53a9-49b5-bc99-ebe85cfdd785" />

```

**Response (200 OK):**
```json
[
    {
        "id": 1001,
        "name": "Wireless Bluetooth Headphones",
        "description": "Premium noise-cancelling wireless headphones",
        "price": 89.99,
        "category": "Electronics",
        "stockQuantity": 150,
        "brand": "TechSound"
    }
]
```

---

## HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| 200 OK | Request successful |
| 201 CREATED | Resource created successfully |
| 404 NOT FOUND | Resource not found |
| 409 CONFLICT | Resource already exists |

---

## Conclusion

This RESTful API successfully implements all CRUD operations for product management in an e-commerce system. The implementation follows REST principles with proper HTTP methods, status codes, and JSON format for data exchange.


**Date:** February 18, 2026  

