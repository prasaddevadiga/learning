

# Spring boot microservice project setup documentation

Let me create that comprehensive report for you right now. Since you're coming from a mobile development background and want to understand microservices with Spring Boot from scratch, I'll provide:

## Complete Spring Boot Microservices Project - Production Ready

### Project Structure

```
spring-boot-microservices/
├── docker-compose.yml
├── pom.xml (parent)
├── README.md
├── config-server/
├── discovery-server/ (Eureka)
├── gateway-service/
├── catalog-service/
├── inventory-service/
├── orders-service/
├── payments-service/
└── common-lib/
```


### 1. Parent POM Configuration

**File: `pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.shop</groupId>
    <artifactId>microservices-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <spring-boot.version>3.2.0</spring-boot.version>
        <spring-cloud.version>2023.0.0</spring-cloud.version>
    </properties>
    
    <modules>
        <module>common-lib</module>
        <module>config-server</module>
        <module>discovery-server</module>
        <module>gateway-service</module>
        <module>catalog-service</module>
        <module>inventory-service</module>
        <module>orders-service</module>
        <module>payments-service</module>
    </modules>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```


### 2. Common Library (Shared DTOs and Utilities)

**File: `common-lib/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>common-lib</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
        </dependency>
    </dependencies>
</project>
```

**File: `common-lib/src/main/java/com/shop/common/dto/ProductDto.java`**

```java
package com.shop.common.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import java.math.BigDecimal;

public record ProductDto(
        @JsonProperty("id") Long id,
        @NotBlank(message = "Product name is required")
        @JsonProperty("name") String name,
        @NotNull @Positive(message = "Price must be positive")
        @JsonProperty("price") BigDecimal price,
        @NotBlank(message = "SKU is required")
        @JsonProperty("sku") String sku,
        @JsonProperty("description") String description
) {}
```

**File: `common-lib/src/main/java/com/shop/common/dto/OrderDto.java`**

```java
package com.shop.common.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.NotNull;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public record OrderDto(
        @JsonProperty("id") Long id,
        @NotNull(message = "Customer ID is required")
        @JsonProperty("customerId") Long customerId,
        @NotEmpty(message = "Order items cannot be empty")
        @JsonProperty("items") List<OrderItemDto> items,
        @JsonProperty("totalAmount") BigDecimal totalAmount,
        @JsonProperty("status") String status,
        @JsonProperty("createdAt") LocalDateTime createdAt
) {}
```

**File: `common-lib/src/main/java/com/shop/common/dto/OrderItemDto.java`**

```java
package com.shop.common.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Positive;
import java.math.BigDecimal;

public record OrderItemDto(
        @JsonProperty("id") Long id,
        @NotBlank(message = "SKU is required")
        @JsonProperty("sku") String sku,
        @Positive(message = "Quantity must be positive")
        @JsonProperty("quantity") Integer quantity,
        @JsonProperty("unitPrice") BigDecimal unitPrice,
        @JsonProperty("totalPrice") BigDecimal totalPrice
) {}
```


### 3. Configuration Server

**File: `config-server/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>config-server</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `config-server/src/main/java/com/shop/config/ConfigServerApplication.java`**

```java
package com.shop.config;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**File: `config-server/src/main/resources/application.yml`**

```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config
        git:
          uri: https://github.com/your-username/microservices-config
          clone-on-start: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,refresh
```


### 4. Discovery Server (Eureka)

**File: `discovery-server/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>discovery-server</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `discovery-server/src/main/java/com/shop/discovery/DiscoveryServerApplication.java`**

```java
package com.shop.discovery;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

**File: `discovery-server/src/main/resources/application.yml`**

```yaml
server:
  port: 8761

spring:
  application:
    name: discovery-server

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

management:
  endpoints:
    web:
      exposure:
        include: health,info
```


### 5. API Gateway Service

**File: `gateway-service/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>gateway-service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `gateway-service/src/main/java/com/shop/gateway/GatewayApplication.java`**

```java
package com.shop.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

**File: `gateway-service/src/main/resources/application.yml`**

```yaml
server:
  port: 8080

spring:
  application:
    name: gateway-service
  config:
    import: "optional:configserver:http://localhost:8888"
  cloud:
    gateway:
      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods: "*"
            allowedHeaders: "*"
      routes:
        - id: catalog-service
          uri: lb://catalog-service
          predicates:
            - Path=/api/catalog/**
          filters:
            - StripPrefix=2
            - name: CircuitBreaker
              args:
                name: catalog-service
                fallbackUri: forward:/fallback
        - id: inventory-service
          uri: lb://inventory-service
          predicates:
            - Path=/api/inventory/**
          filters:
            - StripPrefix=2
            - name: CircuitBreaker
              args:
                name: inventory-service
                fallbackUri: forward:/fallback
        - id: orders-service
          uri: lb://orders-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=2
            - name: CircuitBreaker
              args:
                name: orders-service
                fallbackUri: forward:/fallback
        - id: payments-service
          uri: lb://payments-service
          predicates:
            - Path=/api/payments/**
          filters:
            - StripPrefix=2
            - name: CircuitBreaker
              args:
                name: payments-service
                fallbackUri: forward:/fallback
        - id: discovery-server
          uri: http://localhost:8761
          predicates:
            - Path=/eureka/web
          filters:
            - SetPath=/

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    fetch-registry: true
    register-with-eureka: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,gateway

resilience4j:
  circuitbreaker:
    instances:
      catalog-service:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        wait-duration-in-open-state: 10s
        failure-rate-threshold: 50
      inventory-service:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        wait-duration-in-open-state: 10s
        failure-rate-threshold: 50
```

**File: `gateway-service/src/main/java/com/shop/gateway/controller/FallbackController.java`**

```java
package com.shop.gateway.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FallbackController {
    
    @GetMapping("/fallback")
    public ResponseEntity<String> fallback() {
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                .body("Service is currently unavailable. Please try again later.");
    }
}
```


### 6. Catalog Service

**File: `catalog-service/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>catalog-service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>com.shop</groupId>
            <artifactId>common-lib</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-graphql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `catalog-service/src/main/java/com/shop/catalog/CatalogServiceApplication.java`**

```java
package com.shop.catalog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class CatalogServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(CatalogServiceApplication.class, args);
    }
}
```

**File: `catalog-service/src/main/java/com/shop/catalog/entity/Product.java`**

```java
package com.shop.catalog.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "products")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;
    
    @Column(nullable = false, unique = true)
    private String sku;
    
    @Column(length = 1000)
    private String description;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

**File: `catalog-service/src/main/java/com/shop/catalog/repository/ProductRepository.java`**

```java
package com.shop.catalog.repository;

import com.shop.catalog.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    Optional<Product> findBySku(String sku);
    
    List<Product> findByNameContainingIgnoreCase(String name);
    
    @Query("SELECT p FROM Product p WHERE p.price BETWEEN :minPrice AND :maxPrice")
    List<Product> findByPriceRange(BigDecimal minPrice, BigDecimal maxPrice);
    
    boolean existsBySku(String sku);
}
```

**File: `catalog-service/src/main/java/com/shop/catalog/service/ProductService.java`**

```java
package com.shop.catalog.service;

import com.shop.catalog.entity.Product;
import com.shop.catalog.repository.ProductRepository;
import com.shop.common.dto.ProductDto;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class ProductService {
    
    private final ProductRepository productRepository;
    
    public List<ProductDto> getAllProducts() {
        log.info("Fetching all products");
        return productRepository.findAll()
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public Optional<ProductDto> getProductById(Long id) {
        log.info("Fetching product with id: {}", id);
        return productRepository.findById(id)
                .map(this::mapToDto);
    }
    
    public Optional<ProductDto> getProductBySku(String sku) {
        log.info("Fetching product with sku: {}", sku);
        return productRepository.findBySku(sku)
                .map(this::mapToDto);
    }
    
    public List<ProductDto> searchProducts(String name) {
        log.info("Searching products with name containing: {}", name);
        return productRepository.findByNameContainingIgnoreCase(name)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public List<ProductDto> getProductsByPriceRange(BigDecimal minPrice, BigDecimal maxPrice) {
        log.info("Fetching products with price between {} and {}", minPrice, maxPrice);
        return productRepository.findByPriceRange(minPrice, maxPrice)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    @Transactional
    public ProductDto createProduct(ProductDto productDto) {
        log.info("Creating new product with sku: {}", productDto.sku());
        
        if (productRepository.existsBySku(productDto.sku())) {
            throw new IllegalArgumentException("Product with SKU " + productDto.sku() + " already exists");
        }
        
        Product product = Product.builder()
                .name(productDto.name())
                .price(productDto.price())
                .sku(productDto.sku())
                .description(productDto.description())
                .build();
        
        Product savedProduct = productRepository.save(product);
        log.info("Product created successfully with id: {}", savedProduct.getId());
        
        return mapToDto(savedProduct);
    }
    
    @Transactional
    public Optional<ProductDto> updateProduct(Long id, ProductDto productDto) {
        log.info("Updating product with id: {}", id);
        
        return productRepository.findById(id)
                .map(existingProduct -> {
                    existingProduct.setName(productDto.name());
                    existingProduct.setPrice(productDto.price());
                    existingProduct.setDescription(productDto.description());
                    
                    Product updatedProduct = productRepository.save(existingProduct);
                    log.info("Product updated successfully with id: {}", updatedProduct.getId());
                    
                    return mapToDto(updatedProduct);
                });
    }
    
    @Transactional
    public boolean deleteProduct(Long id) {
        log.info("Deleting product with id: {}", id);
        
        if (productRepository.existsById(id)) {
            productRepository.deleteById(id);
            log.info("Product deleted successfully with id: {}", id);
            return true;
        }
        
        log.warn("Product not found with id: {}", id);
        return false;
    }
    
    private ProductDto mapToDto(Product product) {
        return new ProductDto(
                product.getId(),
                product.getName(),
                product.getPrice(),
                product.getSku(),
                product.getDescription()
        );
    }
}
```

**File: `catalog-service/src/main/java/com/shop/catalog/controller/ProductController.java`**

```java
package com.shop.catalog.controller;

import com.shop.catalog.service.ProductService;
import com.shop.common.dto.ProductDto;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.math.BigDecimal;
import java.util.List;

@RestController
@RequestMapping("/products")
@RequiredArgsConstructor
@Slf4j
public class ProductController {
    
    private final ProductService productService;
    
    @GetMapping
    public ResponseEntity<List<ProductDto>> getAllProducts() {
        List<ProductDto> products = productService.getAllProducts();
        return ResponseEntity.ok(products);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ProductDto> getProductById(@PathVariable Long id) {
        return productService.getProductById(id)
                .map(product -> ResponseEntity.ok(product))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/sku/{sku}")
    public ResponseEntity<ProductDto> getProductBySku(@PathVariable String sku) {
        return productService.getProductBySku(sku)
                .map(product -> ResponseEntity.ok(product))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<ProductDto>> searchProducts(@RequestParam String name) {
        List<ProductDto> products = productService.searchProducts(name);
        return ResponseEntity.ok(products);
    }
    
    @GetMapping("/price-range")
    public ResponseEntity<List<ProductDto>> getProductsByPriceRange(
            @RequestParam BigDecimal minPrice,
            @RequestParam BigDecimal maxPrice) {
        List<ProductDto> products = productService.getProductsByPriceRange(minPrice, maxPrice);
        return ResponseEntity.ok(products);
    }
    
    @PostMapping
    public ResponseEntity<ProductDto> createProduct(@Valid @RequestBody ProductDto productDto) {
        try {
            ProductDto createdProduct = productService.createProduct(productDto);
            return ResponseEntity.status(HttpStatus.CREATED).body(createdProduct);
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest().build();
        }
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ProductDto> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody ProductDto productDto) {
        return productService.updateProduct(id, productDto)
                .map(product -> ResponseEntity.ok(product))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        boolean deleted = productService.deleteProduct(id);
        return deleted ? ResponseEntity.noContent().build() : ResponseEntity.notFound().build();
    }
}
```

**File: `catalog-service/src/main/java/com/shop/catalog/graphql/ProductGraphQLController.java`**

```java
package com.shop.catalog.graphql;

import com.shop.catalog.service.ProductService;
import com.shop.common.dto.ProductDto;
import lombok.RequiredArgsConstructor;
import org.springframework.graphql.data.method.annotation.Argument;
import org.springframework.graphql.data.method.annotation.MutationMapping;
import org.springframework.graphql.data.method.annotation.QueryMapping;
import org.springframework.stereotype.Controller;

import java.math.BigDecimal;
import java.util.List;

@Controller
@RequiredArgsConstructor
public class ProductGraphQLController {
    
    private final ProductService productService;
    
    @QueryMapping
    public List<ProductDto> products() {
        return productService.getAllProducts();
    }
    
    @QueryMapping
    public ProductDto product(@Argument Long id) {
        return productService.getProductById(id).orElse(null);
    }
    
    @QueryMapping
    public ProductDto productBySku(@Argument String sku) {
        return productService.getProductBySku(sku).orElse(null);
    }
    
    @QueryMapping
    public List<ProductDto> searchProducts(@Argument String name) {
        return productService.searchProducts(name);
    }
    
    @QueryMapping
    public List<ProductDto> productsByPriceRange(@Argument BigDecimal minPrice, @Argument BigDecimal maxPrice) {
        return productService.getProductsByPriceRange(minPrice, maxPrice);
    }
    
    @MutationMapping
    public ProductDto createProduct(@Argument String name, @Argument BigDecimal price, 
                                   @Argument String sku, @Argument String description) {
        ProductDto productDto = new ProductDto(null, name, price, sku, description);
        return productService.createProduct(productDto);
    }
    
    @MutationMapping
    public ProductDto updateProduct(@Argument Long id, @Argument String name, 
                                   @Argument BigDecimal price, @Argument String description) {
        ProductDto productDto = new ProductDto(id, name, price, null, description);
        return productService.updateProduct(id, productDto).orElse(null);
    }
    
    @MutationMapping
    public Boolean deleteProduct(@Argument Long id) {
        return productService.deleteProduct(id);
    }
}
```

**File: `catalog-service/src/main/resources/graphql/schema.graphqls`**

```graphql
type Product {
    id: ID!
    name: String!
    price: Float!
    sku: String!
    description: String
}

type Query {
    products: [Product!]!
    product(id: ID!): Product
    productBySku(sku: String!): Product
    searchProducts(name: String!): [Product!]!
    productsByPriceRange(minPrice: Float!, maxPrice: Float!): [Product!]!
}

type Mutation {
    createProduct(name: String!, price: Float!, sku: String!, description: String): Product!
    updateProduct(id: ID!, name: String!, price: Float!, description: String): Product
    deleteProduct(id: ID!): Boolean!
}
```

**File: `catalog-service/src/main/resources/application.yml`**

```yaml
server:
  port: 8081

spring:
  application:
    name: catalog-service
  config:
    import: "optional:configserver:http://localhost:8888"
  datasource:
    url: jdbc:postgresql://localhost:5432/catalog_db
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
  graphql:
    graphiql:
      enabled: true
      path: /graphiql

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    fetch-registry: true
    register-with-eureka: true
  instance:
    prefer-ip-address: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  tracing:
    sampling:
      probability: 1.0

logging:
  level:
    com.shop.catalog: DEBUG
    org.springframework.graphql: DEBUG
```


### 7. Inventory Service

**File: `inventory-service/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>inventory-service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>com.shop</groupId>
            <artifactId>common-lib</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `inventory-service/src/main/java/com/shop/inventory/InventoryServiceApplication.java`**

```java
package com.shop.inventory;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class InventoryServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(InventoryServiceApplication.class, args);
    }
}
```

**File: `inventory-service/src/main/java/com/shop/inventory/entity/Inventory.java`**

```java
package com.shop.inventory.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.time.LocalDateTime;

@Entity
@Table(name = "inventory")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Inventory {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String sku;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @Column(name = "reserved_quantity", nullable = false)
    private Integer reservedQuantity;
    
    @Version
    private Long version;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    @UpdateTimestamp
    private LocalDateTime updatedAt;
    
    public Integer getAvailableQuantity() {
        return quantity - reservedQuantity;
    }
    
    public boolean canReserve(Integer requestedQuantity) {
        return getAvailableQuantity() >= requestedQuantity;
    }
}
```

**File: `inventory-service/src/main/java/com/shop/inventory/dto/InventoryDto.java`**

```java
package com.shop.inventory.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

public record InventoryDto(
        @JsonProperty("id") Long id,
        @NotBlank(message = "SKU is required")
        @JsonProperty("sku") String sku,
        @NotNull @Min(value = 0, message = "Quantity must be non-negative")
        @JsonProperty("quantity") Integer quantity,
        @JsonProperty("reservedQuantity") Integer reservedQuantity,
        @JsonProperty("availableQuantity") Integer availableQuantity
) {}
```

**File: `inventory-service/src/main/java/com/shop/inventory/dto/ReservationRequest.java`**

```java
package com.shop.inventory.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;

public record ReservationRequest(
        @NotBlank(message = "SKU is required")
        @JsonProperty("sku") String sku,
        @NotNull @Min(value = 1, message = "Quantity must be positive")
        @JsonProperty("quantity") Integer quantity
) {}
```

**File: `inventory-service/src/main/java/com/shop/inventory/repository/InventoryRepository.java`**

```java
package com.shop.inventory.repository;

import com.shop.inventory.entity.Inventory;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface InventoryRepository extends JpaRepository<Inventory, Long> {
    
    Optional<Inventory> findBySku(String sku);
    
    List<Inventory> findBySkuIn(List<String> skus);
    
    @Query("SELECT i FROM Inventory i WHERE i.quantity - i.reservedQuantity < :threshold")
    List<Inventory> findLowStockItems(@Param("threshold") Integer threshold);
    
    @Modifying
    @Query("UPDATE Inventory i SET i.reservedQuantity = i.reservedQuantity + :quantity " +
           "WHERE i.sku = :sku AND i.quantity - i.reservedQuantity >= :quantity")
    int reserveStock(@Param("sku") String sku, @Param("quantity") Integer quantity);
    
    @Modifying
    @Query("UPDATE Inventory i SET i.reservedQuantity = i.reservedQuantity - :quantity " +
           "WHERE i.sku = :sku AND i.reservedQuantity >= :quantity")
    int releaseReservedStock(@Param("sku") String sku, @Param("quantity") Integer quantity);
    
    @Modifying
    @Query("UPDATE Inventory i SET i.quantity = i.quantity - :quantity, " +
           "i.reservedQuantity = i.reservedQuantity - :quantity " +
           "WHERE i.sku = :sku AND i.reservedQuantity >= :quantity")
    int confirmStock(@Param("sku") String sku, @Param("quantity") Integer quantity);
}
```

**File: `inventory-service/src/main/java/com/shop/inventory/service/InventoryService.java`**

```java
package com.shop.inventory.service;

import com.shop.inventory.dto.InventoryDto;
import com.shop.inventory.dto.ReservationRequest;
import com.shop.inventory.entity.Inventory;
import com.shop.inventory.repository.InventoryRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class InventoryService {
    
    private final InventoryRepository inventoryRepository;
    
    public List<InventoryDto> getAllInventory() {
        log.info("Fetching all inventory items");
        return inventoryRepository.findAll()
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public Optional<InventoryDto> getInventoryBySku(String sku) {
        log.info("Fetching inventory for sku: {}", sku);
        return inventoryRepository.findBySku(sku)
                .map(this::mapToDto);
    }
    
    public List<InventoryDto> getInventoryBySkus(List<String> skus) {
        log.info("Fetching inventory for skus: {}", skus);
        return inventoryRepository.findBySkuIn(skus)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public List<InventoryDto> getLowStockItems(Integer threshold) {
        log.info("Fetching low stock items with threshold: {}", threshold);
        return inventoryRepository.findLowStockItems(threshold)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    @Transactional
    public InventoryDto createOrUpdateInventory(InventoryDto inventoryDto) {
        log.info("Creating/updating inventory for sku: {}", inventoryDto.sku());
        
        Optional<Inventory> existingInventory = inventoryRepository.findBySku(inventoryDto.sku());
        
        Inventory inventory;
        if (existingInventory.isPresent()) {
            inventory = existingInventory.get();
            inventory.setQuantity(inventoryDto.quantity());
        } else {
            inventory = Inventory.builder()
                    .sku(inventoryDto.sku())
                    .quantity(inventoryDto.quantity())
                    .reservedQuantity(0)
                    .build();
        }
        
        Inventory savedInventory = inventoryRepository.save(inventory);
        log.info("Inventory saved successfully for sku: {}", savedInventory.getSku());
        
        return mapToDto(savedInventory);
    }
    
    @Transactional
    public boolean reserveStock(ReservationRequest request) {
        log.info("Attempting to reserve {} units of sku: {}", request.quantity(), request.sku());
        
        int updatedRows = inventoryRepository.reserveStock(request.sku(), request.quantity());
        boolean reserved = updatedRows > 0;
        
        if (reserved) {
            log.info("Successfully reserved {} units of sku: {}", request.quantity(), request.sku());
        } else {
            log.warn("Failed to reserve {} units of sku: {} - insufficient stock", 
                    request.quantity(), request.sku());
        }
        
        return reserved;
    }
    
    @Transactional
    public boolean releaseReservedStock(ReservationRequest request) {
        log.info("Releasing {} reserved units of sku: {}", request.quantity(), request.sku());
        
        int updatedRows = inventoryRepository.releaseReservedStock(request.sku(), request.quantity());
        boolean released = updatedRows > 0;
        
        if (released) {
            log.info("Successfully released {} reserved units of sku: {}", request.quantity(), request.sku());
        } else {
            log.warn("Failed to release {} reserved units of sku: {}", request.quantity(), request.sku());
        }
        
        return released;
    }
    
    @Transactional
    public boolean confirmStock(ReservationRequest request) {
        log.info("Confirming {} units of sku: {}", request.quantity(), request.sku());
        
        int updatedRows = inventoryRepository.confirmStock(request.sku(), request.quantity());
        boolean confirmed = updatedRows > 0;
        
        if (confirmed) {
            log.info("Successfully confirmed {} units of sku: {}", request.quantity(), request.sku());
        } else {
            log.warn("Failed to confirm {} units of sku: {}", request.quantity(), request.sku());
        }
        
        return confirmed;
    }
    
    public boolean isInStock(String sku, Integer quantity) {
        log.info("Checking stock availability for sku: {} quantity: {}", sku, quantity);
        
        return inventoryRepository.findBySku(sku)
                .map(inventory -> inventory.canReserve(quantity))
                .orElse(false);
    }
    
    private InventoryDto mapToDto(Inventory inventory) {
        return new InventoryDto(
                inventory.getId(),
                inventory.getSku(),
                inventory.getQuantity(),
                inventory.getReservedQuantity(),
                inventory.getAvailableQuantity()
        );
    }
}
```

**File: `inventory-service/src/main/java/com/shop/inventory/controller/InventoryController.java`**

```java
package com.shop.inventory.controller;

import com.shop.inventory.dto.InventoryDto;
import com.shop.inventory.dto.ReservationRequest;
import com.shop.inventory.service.InventoryService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/inventory")
@RequiredArgsConstructor
@Slf4j
public class InventoryController {
    
    private final InventoryService inventoryService;
    
    @GetMapping
    public ResponseEntity<List<InventoryDto>> getAllInventory() {
        List<InventoryDto> inventory = inventoryService.getAllInventory();
        return ResponseEntity.ok(inventory);
    }
    
    @GetMapping("/{sku}")
    public ResponseEntity<InventoryDto> getInventoryBySku(@PathVariable String sku) {
        return inventoryService.getInventoryBySku(sku)
                .map(inventory -> ResponseEntity.ok(inventory))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping("/check-multiple")
    public ResponseEntity<List<InventoryDto>> getInventoryBySkus(@RequestBody List<String> skus) {
        List<InventoryDto> inventory = inventoryService.getInventoryBySkus(skus);
        return ResponseEntity.ok(inventory);
    }
    
    @GetMapping("/low-stock")
    public ResponseEntity<List<InventoryDto>> getLowStockItems(@RequestParam(defaultValue = "10") Integer threshold) {
        List<InventoryDto> inventory = inventoryService.getLowStockItems(threshold);
        return ResponseEntity.ok(inventory);
    }
    
    @PostMapping
    public ResponseEntity<InventoryDto> createOrUpdateInventory(@Valid @RequestBody InventoryDto inventoryDto) {
        InventoryDto savedInventory = inventoryService.createOrUpdateInventory(inventoryDto);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedInventory);
    }
    
    @PostMapping("/reserve")
    public ResponseEntity<Void> reserveStock(@Valid @RequestBody ReservationRequest request) {
        boolean reserved = inventoryService.reserveStock(request);
        return reserved ? ResponseEntity.ok().build() : ResponseEntity.badRequest().build();
    }
    
    @PostMapping("/release")
    public ResponseEntity<Void> releaseReservedStock(@Valid @RequestBody ReservationRequest request) {
        boolean released = inventoryService.releaseReservedStock(request);
        return released ? ResponseEntity.ok().build() : ResponseEntity.badRequest().build();
    }
    
    @PostMapping("/confirm")
    public ResponseEntity<Void> confirmStock(@Valid @RequestBody ReservationRequest request) {
        boolean confirmed = inventoryService.confirmStock(request);
        return confirmed ? ResponseEntity.ok().build() : ResponseEntity.badRequest().build();
    }
    
    @GetMapping("/check/{sku}")
    public ResponseEntity<Boolean> isInStock(@PathVariable String sku, @RequestParam Integer quantity) {
        boolean inStock = inventoryService.isInStock(sku, quantity);
        return ResponseEntity.ok(inStock);
    }
}
```

**File: `inventory-service/src/main/resources/application.yml`**

```yaml
server:
  port: 8082

spring:
  application:
    name: inventory-service
  config:
    import: "optional:configserver:http://localhost:8888"
  datasource:
    url: jdbc:postgresql://localhost:5433/inventory_db
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    fetch-registry: true
    register-with-eureka: true
  instance:
    prefer-ip-address: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  tracing:
    sampling:
      probability: 1.0

logging:
  level:
    com.shop.inventory: DEBUG
```


### 8. Orders Service (with OpenFeign for inter-service communication)

**File: `orders-service/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>orders-service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>com.shop</groupId>
            <artifactId>common-lib</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-graphql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `orders-service/src/main/java/com/shop/orders/OrdersServiceApplication.java`**

```java
package com.shop.orders;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class OrdersServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrdersServiceApplication.class, args);
    }
}
```

**File: `orders-service/src/main/java/com/shop/orders/entity/Order.java`**

```java
package com.shop.orders.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

@Entity
@Table(name = "orders")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "customer_id", nullable = false)
    private Long customerId;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items;
    
    @Column(name = "total_amount", nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}
```

**File: `orders-service/src/main/java/com/shop/orders/entity/OrderItem.java`**

```java
package com.shop.orders.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;

@Entity
@Table(name = "order_items")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class OrderItem {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
    
    @Column(nullable = false)
    private String sku;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @Column(name = "unit_price", nullable = false, precision = 10, scale = 2)
    private BigDecimal unitPrice;
    
    @Column(name = "total_price", nullable = false, precision = 10, scale = 2)
    private BigDecimal totalPrice;
}
```

**File: `orders-service/src/main/java/com/shop/orders/entity/OrderStatus.java`**

```java
package com.shop.orders.entity;

public enum OrderStatus {
    PENDING,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

**File: `orders-service/src/main/java/com/shop/orders/client/CatalogServiceClient.java`**

```java
package com.shop.orders.client;

import com.shop.common.dto.ProductDto;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "catalog-service")
public interface CatalogServiceClient {
    
    @GetMapping("/products/sku/{sku}")
    ProductDto getProductBySku(@PathVariable("sku") String sku);
}
```

**File: `orders-service/src/main/java/com/shop/orders/client/InventoryServiceClient.java`**

```java
package com.shop.orders.client;

import com.shop.inventory.dto.ReservationRequest;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(name = "inventory-service")
public interface InventoryServiceClient {
    
    @GetMapping("/inventory/check/{sku}")
    Boolean isInStock(@PathVariable("sku") String sku, @RequestParam("quantity") Integer quantity);
    
    @PostMapping("/inventory/reserve")
    void reserveStock(@RequestBody ReservationRequest request);
    
    @PostMapping("/inventory/release")
    void releaseReservedStock(@RequestBody ReservationRequest request);
    
    @PostMapping("/inventory/confirm")
    void confirmStock(@RequestBody ReservationRequest request);
}
```

**File: `orders-service/src/main/java/com/shop/orders/client/PaymentsServiceClient.java`**

```java
package com.shop.orders.client;

import com.shop.orders.dto.PaymentRequest;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient(name = "payments-service")
public interface PaymentsServiceClient {
    
    @PostMapping("/payments")
    void processPayment(@RequestBody PaymentRequest request);
}
```

**File: `orders-service/src/main/java/com/shop/orders/dto/PaymentRequest.java`**

```java
package com.shop.orders.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;

import java.math.BigDecimal;

public record PaymentRequest(
        @NotNull(message = "Order ID is required")
        @JsonProperty("orderId") Long orderId,
        @NotNull @Positive(message = "Amount must be positive")
        @JsonProperty("amount") BigDecimal amount,
        @JsonProperty("paymentMethod") String paymentMethod,
        @JsonProperty("reference") String reference
) {}
```

**File: `orders-service/src/main/java/com/shop/orders/repository/OrderRepository.java`**

```java
package com.shop.orders.repository;

import com.shop.orders.entity.Order;
import com.shop.orders.entity.OrderStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;

@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    List<Order> findByCustomerId(Long customerId);
    
    List<Order> findByStatus(OrderStatus status);
    
    List<Order> findByCustomerIdAndStatus(Long customerId, OrderStatus status);
    
    @Query("SELECT o FROM Order o WHERE o.createdAt BETWEEN :startDate AND :endDate")
    List<Order> findOrdersByDateRange(@Param("startDate") LocalDateTime startDate, 
                                     @Param("endDate") LocalDateTime endDate);
    
    @Query("SELECT COUNT(o) FROM Order o WHERE o.customerId = :customerId")
    Long countOrdersByCustomerId(@Param("customerId") Long customerId);
}
```

**File: `orders-service/src/main/java/com/shop/orders/service/OrderService.java`**

```java
package com.shop.orders.service;

import com.shop.common.dto.OrderDto;
import com.shop.common.dto.OrderItemDto;
import com.shop.common.dto.ProductDto;
import com.shop.inventory.dto.ReservationRequest;
import com.shop.orders.client.CatalogServiceClient;
import com.shop.orders.client.InventoryServiceClient;
import com.shop.orders.client.PaymentsServiceClient;
import com.shop.orders.dto.PaymentRequest;
import com.shop.orders.entity.Order;
import com.shop.orders.entity.OrderItem;
import com.shop.orders.entity.OrderStatus;
import com.shop.orders.repository.OrderRepository;
import feign.FeignException;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final CatalogServiceClient catalogServiceClient;
    private final InventoryServiceClient inventoryServiceClient;
    private final PaymentsServiceClient paymentsServiceClient;
    
    public List<OrderDto> getAllOrders() {
        log.info("Fetching all orders");
        return orderRepository.findAll()
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public Optional<OrderDto> getOrderById(Long id) {
        log.info("Fetching order with id: {}", id);
        return orderRepository.findById(id)
                .map(this::mapToDto);
    }
    
    public List<OrderDto> getOrdersByCustomerId(Long customerId) {
        log.info("Fetching orders for customer: {}", customerId);
        return orderRepository.findByCustomerId(customerId)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public List<OrderDto> getOrdersByStatus(OrderStatus status) {
        log.info("Fetching orders with status: {}", status);
        return orderRepository.findByStatus(status)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    @Transactional
    public OrderDto createOrder(OrderDto orderDto) {
        log.info("Creating new order for customer: {}", orderDto.customerId());
        
        try {
            // Step 1: Validate and get product details
            List<OrderItemDto> validatedItems = validateAndEnrichOrderItems(orderDto.items());
            
            // Step 2: Check inventory and reserve stock
            reserveInventoryForItems(validatedItems);
            
            // Step 3: Calculate total amount
            BigDecimal totalAmount = calculateTotalAmount(validatedItems);
            
            // Step 4: Create and save order
            Order order = createOrderEntity(orderDto.customerId(), validatedItems, totalAmount);
            Order savedOrder = orderRepository.save(order);
            
            // Step 5: Process payment
            processPaymentForOrder(savedOrder);
            
            // Step 6: Confirm inventory
            confirmInventoryForItems(validatedItems);
            
            // Step 7: Update order status
            savedOrder.setStatus(OrderStatus.CONFIRMED);
            savedOrder = orderRepository.save(savedOrder);
            
            log.info("Order created successfully with id: {}", savedOrder.getId());
            return mapToDto(savedOrder);
            
        } catch (Exception e) {
            log.error("Failed to create order for customer: {}", orderDto.customerId(), e);
            // Release any reserved inventory
            releaseReservedInventory(orderDto.items());
            throw new RuntimeException("Failed to create order: " + e.getMessage(), e);
        }
    }
    
    @Transactional
    public Optional<OrderDto> updateOrderStatus(Long orderId, OrderStatus newStatus) {
        log.info("Updating order {} status to {}", orderId, newStatus);
        
        return orderRepository.findById(orderId)
                .map(order -> {
                    order.setStatus(newStatus);
                    Order updatedOrder = orderRepository.save(order);
                    log.info("Order {} status updated to {}", orderId, newStatus);
                    return mapToDto(updatedOrder);
                });
    }
    
    @Transactional
    public boolean cancelOrder(Long orderId) {
        log.info("Cancelling order: {}", orderId);
        
        return orderRepository.findById(orderId)
                .map(order -> {
                    if (order.getStatus() == OrderStatus.PENDING || order.getStatus() == OrderStatus.CONFIRMED) {
                        // Release reserved inventory
                        order.getItems().forEach(item -> {
                            try {
                                ReservationRequest request = new ReservationRequest(item.getSku(), item.getQuantity());
                                inventoryServiceClient.releaseReservedStock(request);
                            } catch (Exception e) {
                                log.warn("Failed to release inventory for sku: {}", item.getSku(), e);
                            }
                        });
                        
                        order.setStatus(OrderStatus.CANCELLED);
                        orderRepository.save(order);
                        log.info("Order {} cancelled successfully", orderId);
                        return true;
                    } else {
                        log.warn("Cannot cancel order {} with status: {}", orderId, order.getStatus());
                        return false;
                    }
                })
                .orElse(false);
    }
    
    private List<OrderItemDto> validateAndEnrichOrderItems(List<OrderItemDto> items) {
        return items.stream()
                .map(item -> {
                    try {
                        ProductDto product = catalogServiceClient.getProductBySku(item.sku());
                        if (product == null) {
                            throw new IllegalArgumentException("Product not found for SKU: " + item.sku());
                        }
                        
                        BigDecimal totalPrice = product.price().multiply(BigDecimal.valueOf(item.quantity()));
                        
                        return new OrderItemDto(
                                item.id(),
                                item.sku(),
                                item.quantity(),
                                product.price(),
                                totalPrice
                        );
                    } catch (FeignException e) {
                        log.error("Failed to get product details for SKU: {}", item.sku(), e);
                        throw new RuntimeException("Product service unavailable for SKU: " + item.sku());
                    }
                })
                .toList();
    }
    
    private void reserveInventoryForItems(List<OrderItemDto> items) {
        for (OrderItemDto item : items) {
            try {
                // Check if item is in stock
                Boolean inStock = inventoryServiceClient.isInStock(item.sku(), item.quantity());
                if (!Boolean.TRUE.equals(inStock)) {
                    throw new RuntimeException("Insufficient stock for SKU: " + item.sku());
                }
                
                // Reserve the stock
                ReservationRequest request = new ReservationRequest(item.sku(), item.quantity());
                inventoryServiceClient.reserveStock(request);
                
            } catch (FeignException e) {
                log.error("Failed to reserve inventory for SKU: {}", item.sku(), e);
                throw new RuntimeException("Inventory service unavailable for SKU: " + item.sku());
            }
        }
    }
    
    private void confirmInventoryForItems(List<OrderItemDto> items) {
        for (OrderItemDto item : items) {
            try {
                ReservationRequest request = new ReservationRequest(item.sku(), item.quantity());
                inventoryServiceClient.confirmStock(request);
            } catch (Exception e) {
                log.warn("Failed to confirm inventory for SKU: {}", item.sku(), e);
            }
        }
    }
    
    private void releaseReservedInventory(List<OrderItemDto> items) {
        for (OrderItemDto item : items) {
            try {
                ReservationRequest request = new ReservationRequest(item.sku(), item.quantity());
                inventoryServiceClient.releaseReservedStock(request);
            } catch (Exception e) {
                log.warn("Failed to release reserved inventory for SKU: {}", item.sku(), e);
            }
        }
    }
    
    private BigDecimal calculateTotalAmount(List<OrderItemDto> items) {
        return items.stream()
                .map(OrderItemDto::totalPrice)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    private Order createOrderEntity(Long customerId, List<OrderItemDto> items, BigDecimal totalAmount) {
        Order order = Order.builder()
                .customerId(customerId)
                .totalAmount(totalAmount)
                .status(OrderStatus.PENDING)
                .build();
        
        List<OrderItem> orderItems = items.stream()
                .map(item -> OrderItem.builder()
                        .order(order)
                        .sku(item.sku())
                        .quantity(item.quantity())
                        .unitPrice(item.unitPrice())
                        .totalPrice(item.totalPrice())
                        .build())
                .toList();
        
        order.setItems(orderItems);
        return order;
    }
    
    private void processPaymentForOrder(Order order) {
        try {
            PaymentRequest paymentRequest = new PaymentRequest(
                    order.getId(),
                    order.getTotalAmount(),
                    "CREDIT_CARD",
                    "ORDER-" + order.getId() + "-" + UUID.randomUUID().toString().substring(0, 8)
            );
            
            paymentsServiceClient.processPayment(paymentRequest);
            log.info("Payment processed successfully for order: {}", order.getId());
            
        } catch (FeignException e) {
            log.error("Payment processing failed for order: {}", order.getId(), e);
            throw new RuntimeException("Payment processing failed");
        }
    }
    
    private OrderDto mapToDto(Order order) {
        List<OrderItemDto> itemDtos = order.getItems().stream()
                .map(item -> new OrderItemDto(
                        item.getId(),
                        item.getSku(),
                        item.getQuantity(),
                        item.getUnitPrice(),
                        item.getTotalPrice()
                ))
                .toList();
        
        return new OrderDto(
                order.getId(),
                order.getCustomerId(),
                itemDtos,
                order.getTotalAmount(),
                order.getStatus().name(),
                order.getCreatedAt()
        );
    }
}
```

**File: `orders-service/src/main/java/com/shop/orders/controller/OrderController.java`**

```java
package com.shop.orders.controller;

import com.shop.common.dto.OrderDto;
import com.shop.orders.entity.OrderStatus;
import com.shop.orders.service.OrderService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/orders")
@RequiredArgsConstructor
@Slf4j
public class OrderController {
    
    private final OrderService orderService;
    
    @GetMapping
    public ResponseEntity<List<OrderDto>> getAllOrders() {
        List<OrderDto> orders = orderService.getAllOrders();
        return ResponseEntity.ok(orders);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<OrderDto> getOrderById(@PathVariable Long id) {
        return orderService.getOrderById(id)
                .map(order -> ResponseEntity.ok(order))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/customer/{customerId}")
    public ResponseEntity<List<OrderDto>> getOrdersByCustomerId(@PathVariable Long customerId) {
        List<OrderDto> orders = orderService.getOrdersByCustomerId(customerId);
        return ResponseEntity.ok(orders);
    }
    
    @GetMapping("/status/{status}")
    public ResponseEntity<List<OrderDto>> getOrdersByStatus(@PathVariable OrderStatus status) {
        List<OrderDto> orders = orderService.getOrdersByStatus(status);
        return ResponseEntity.ok(orders);
    }
    
    @PostMapping
    public ResponseEntity<OrderDto> createOrder(@Valid @RequestBody OrderDto orderDto) {
        try {
            OrderDto createdOrder = orderService.createOrder(orderDto);
            return ResponseEntity.status(HttpStatus.CREATED).body(createdOrder);
        } catch (RuntimeException e) {
            log.error("Failed to create order", e);
            return ResponseEntity.badRequest().build();
        }
    }
    
    @PutMapping("/{id}/status")
    public ResponseEntity<OrderDto> updateOrderStatus(@PathVariable Long id, @RequestParam OrderStatus status) {
        return orderService.updateOrderStatus(id, status)
                .map(order -> ResponseEntity.ok(order))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> cancelOrder(@PathVariable Long id) {
        boolean cancelled = orderService.cancelOrder(id);
        return cancelled ? ResponseEntity.noContent().build() : ResponseEntity.notFound().build();
    }
}
```

**File: `orders-service/src/main/resources/graphql/schema.graphqls`**

```graphql
type Order {
    id: ID!
    customerId: ID!
    items: [OrderItem!]!
    totalAmount: Float!
    status: String!
    createdAt: String!
}

type OrderItem {
    id: ID!
    sku: String!
    quantity: Int!
    unitPrice: Float!
    totalPrice: Float!
}

input OrderInput {
    customerId: ID!
    items: [OrderItemInput!]!
}

input OrderItemInput {
    sku: String!
    quantity: Int!
}

type Query {
    orders: [Order!]!
    order(id: ID!): Order
    ordersByCustomer(customerId: ID!): [Order!]!
    ordersByStatus(status: String!): [Order!]!
}

type Mutation {
    createOrder(input: OrderInput!): Order!
    updateOrderStatus(id: ID!, status: String!): Order
    cancelOrder(id: ID!): Boolean!
}
```

**File: `orders-service/src/main/resources/application.yml`**

```yaml
server:
  port: 8083

spring:
  application:
    name: orders-service
  config:
    import: "optional:configserver:http://localhost:8888"
  datasource:
    url: jdbc:postgresql://localhost:5434/orders_db
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
  graphql:
    graphiql:
      enabled: true
      path: /graphiql

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    fetch-registry: true
    register-with-eureka: true
  instance:
    prefer-ip-address: true

feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  tracing:
    sampling:
      probability: 1.0

logging:
  level:
    com.shop.orders: DEBUG
    org.springframework.graphql: DEBUG
```


### 9. Payments Service

**File: `payments-service/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>microservices-parent</artifactId>
        <groupId>com.shop</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    
    <artifactId>payments-service</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**File: `payments-service/src/main/java/com/shop/payments/PaymentsServiceApplication.java`**

```java
package com.shop.payments;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentsServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentsServiceApplication.class, args);
    }
}
```

**File: `payments-service/src/main/java/com/shop/payments/entity/Payment.java`**

```java
package com.shop.payments.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.hibernate.annotations.CreationTimestamp;

import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "payments")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Payment {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "order_id", nullable = false)
    private Long orderId;
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal amount;
    
    @Column(name = "payment_method", nullable = false)
    private String paymentMethod;
    
    @Column(nullable = false, unique = true)
    private String reference;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private PaymentStatus status;
    
    @Column(name = "transaction_id")
    private String transactionId;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;
}
```

**File: `payments-service/src/main/java/com/shop/payments/entity/PaymentStatus.java`**

```java
package com.shop.payments.entity;

public enum PaymentStatus {
    PENDING,
    COMPLETED,
    FAILED,
    REFUNDED
}
```

**File: `payments-service/src/main/java/com/shop/payments/dto/PaymentDto.java`**

```java
package com.shop.payments.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public record PaymentDto(
        @JsonProperty("id") Long id,
        @NotNull(message = "Order ID is required")
        @JsonProperty("orderId") Long orderId,
        @NotNull @Positive(message = "Amount must be positive")
        @JsonProperty("amount") BigDecimal amount,
        @JsonProperty("paymentMethod") String paymentMethod,
        @JsonProperty("reference") String reference,
        @JsonProperty("status") String status,
        @JsonProperty("transactionId") String transactionId,
        @JsonProperty("createdAt") LocalDateTime createdAt
) {}
```

**File: `payments-service/src/main/java/com/shop/payments/repository/PaymentRepository.java`**

```java
package com.shop.payments.repository;

import com.shop.payments.entity.Payment;
import com.shop.payments.entity.PaymentStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

@Repository
public interface PaymentRepository extends JpaRepository<Payment, Long> {
    
    Optional<Payment> findByReference(String reference);
    
    List<Payment> findByOrderId(Long orderId);
    
    List<Payment> findByStatus(PaymentStatus status);
    
    boolean existsByReference(String reference);
    
    @Query("SELECT p FROM Payment p WHERE p.createdAt BETWEEN :startDate AND :endDate")
    List<Payment> findPaymentsByDateRange(@Param("startDate") LocalDateTime startDate, 
                                         @Param("endDate") LocalDateTime endDate);
    
    @Query("SELECT SUM(p.amount) FROM Payment p WHERE p.status = :status AND p.createdAt BETWEEN :startDate AND :endDate")
    BigDecimal getTotalAmountByStatusAndDateRange(@Param("status") PaymentStatus status,
                                                 @Param("startDate") LocalDateTime startDate,
                                                 @Param("endDate") LocalDateTime endDate);
}
```

**File: `payments-service/src/main/java/com/shop/payments/service/PaymentService.java`**

```java
package com.shop.payments.service;

import com.shop.payments.dto.PaymentDto;
import com.shop.payments.entity.Payment;
import com.shop.payments.entity.PaymentStatus;
import com.shop.payments.repository.PaymentRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class PaymentService {
    
    private final PaymentRepository paymentRepository;
    private final PaymentProcessor paymentProcessor;
    
    public List<PaymentDto> getAllPayments() {
        log.info("Fetching all payments");
        return paymentRepository.findAll()
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public Optional<PaymentDto> getPaymentById(Long id) {
        log.info("Fetching payment with id: {}", id);
        return paymentRepository.findById(id)
                .map(this::mapToDto);
    }
    
    public Optional<PaymentDto> getPaymentByReference(String reference) {
        log.info("Fetching payment with reference: {}", reference);
        return paymentRepository.findByReference(reference)
                .map(this::mapToDto);
    }
    
    public List<PaymentDto> getPaymentsByOrderId(Long orderId) {
        log.info("Fetching payments for order: {}", orderId);
        return paymentRepository.findByOrderId(orderId)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    public List<PaymentDto> getPaymentsByStatus(PaymentStatus status) {
        log.info("Fetching payments with status: {}", status);
        return paymentRepository.findByStatus(status)
                .stream()
                .map(this::mapToDto)
                .toList();
    }
    
    @Transactional
    public PaymentDto processPayment(PaymentDto paymentDto) {
        log.info("Processing payment for order: {} with reference: {}", 
                paymentDto.orderId(), paymentDto.reference());
        
        // Check for duplicate payment reference
        if (paymentRepository.existsByReference(paymentDto.reference())) {
            log.warn("Payment with reference {} already exists", paymentDto.reference());
            throw new IllegalArgumentException("Payment with this reference already exists");
        }
        
        // Create payment record
        Payment payment = Payment.builder()
                .orderId(paymentDto.orderId())
                .amount(paymentDto.amount())
                .paymentMethod(paymentDto.paymentMethod())
                .reference(paymentDto.reference())
                .status(PaymentStatus.PENDING)
                .build();
        
        Payment savedPayment = paymentRepository.save(payment);
        
        try {
            // Process payment through external payment gateway (simulated)
            String transactionId = paymentProcessor.processPayment(savedPayment);
            
            // Update payment with transaction ID and success status
            savedPayment.setTransactionId(transactionId);
            savedPayment.setStatus(PaymentStatus.COMPLETED);
            savedPayment = paymentRepository.save(savedPayment);
            
            log.info("Payment processed successfully with transaction ID: {}", transactionId);
            
        } catch (Exception e) {
            log.error("Payment processing failed for reference: {}", paymentDto.reference(), e);
            
            // Update payment status to failed
            savedPayment.setStatus(PaymentStatus.FAILED);
            savedPayment = paymentRepository.save(savedPayment);
            
            throw new RuntimeException("Payment processing failed: " + e.getMessage(), e);
        }
        
        return mapToDto(savedPayment);
    }
    
    @Transactional
    public Optional<PaymentDto> refundPayment(Long paymentId) {
        log.info("Processing refund for payment: {}", paymentId);
        
        return paymentRepository.findById(paymentId)
                .filter(payment -> payment.getStatus() == PaymentStatus.COMPLETED)
                .map(payment -> {
                    try {
                        // Process refund through payment gateway (simulated)
                        paymentProcessor.refundPayment(payment);
                        
                        payment.setStatus(PaymentStatus.REFUNDED);
                        Payment refundedPayment = paymentRepository.save(payment);
                        
                        log.info("Payment {} refunded successfully", paymentId);
                        return mapToDto(refundedPayment);
                        
                    } catch (Exception e) {
                        log.error("Refund processing failed for payment: {}", paymentId, e);
                        throw new RuntimeException("Refund processing failed: " + e.getMessage(), e);
                    }
                });
    }
    
    public BigDecimal getTotalRevenue(LocalDateTime startDate, LocalDateTime endDate) {
        log.info("Calculating total revenue between {} and {}", startDate, endDate);
        BigDecimal total = paymentRepository.getTotalAmountByStatusAndDateRange(
                PaymentStatus.COMPLETED, startDate, endDate);
        return total != null ? total : BigDecimal.ZERO;
    }
    
    private PaymentDto mapToDto(Payment payment) {
        return new PaymentDto(
                payment.getId(),
                payment.getOrderId(),
                payment.getAmount(),
                payment.getPaymentMethod(),
                payment.getReference(),
                payment.getStatus().name(),
                payment.getTransactionId(),
                payment.getCreatedAt()
        );
    }
}
```

**File: `payments-service/src/main/java/com/shop/payments/service/PaymentProcessor.java`**

```java
package com.shop.payments.service;

import com.shop.payments.entity.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.UUID;

@Component
@Slf4j
public class PaymentProcessor {
    
    public String processPayment(Payment payment) {
        log.info("Processing payment for order: {} amount: {}", 
                payment.getOrderId(), payment.getAmount());
        
        // Simulate payment processing delay
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // Simulate payment processing with 95% success rate
        if (Math.random() < 0.95) {
            String transactionId = "TXN-" + UUID.randomUUID().toString();
            log.info("Payment processed successfully with transaction ID: {}", transactionId);
            return transactionId;
        } else {
            log.error("Payment processing failed for order: {}", payment.getOrderId());
            throw new RuntimeException("Payment gateway declined the transaction");
        }
    }
    
    public void refundPayment(Payment payment) {
        log.info("Processing refund for payment: {} transaction: {}", 
                payment.getId(), payment.getTransactionId());
        
        // Simulate refund processing delay
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        // Simulate refund processing with 98% success rate
        if (Math.random() < 0.98) {
            log.info("Refund processed successfully for payment: {}", payment.getId());
        } else {
            log.error("Refund processing failed for payment: {}", payment.getId());
            throw new RuntimeException("Refund processing failed");
        }
    }
}
```

**File: `payments-service/src/main/java/com/shop/payments/controller/PaymentController.java`**

```java
package com.shop.payments.controller;

import com.shop.payments.dto.PaymentDto;
import com.shop.payments.entity.PaymentStatus;
import com.shop.payments.service.PaymentService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

@RestController
@RequestMapping("/payments")
@RequiredArgsConstructor
@Slf4j
public class PaymentController {
    
    private final PaymentService paymentService;
    
    @GetMapping
    public ResponseEntity<List<PaymentDto>> getAllPayments() {
        List<PaymentDto> payments = paymentService.getAllPayments();
        return ResponseEntity.ok(payments);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<PaymentDto> getPaymentById(@PathVariable Long id) {
        return paymentService.getPaymentById(id)
                .map(payment -> ResponseEntity.ok(payment))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/reference/{reference}")
    public ResponseEntity<PaymentDto> getPaymentByReference(@PathVariable String reference) {
        return paymentService.getPaymentByReference(reference)
                .map(payment -> ResponseEntity.ok(payment))
                .orElse(ResponseEntity.notFound().build());
    }
    
    @GetMapping("/order/{orderId}")
    public ResponseEntity<List<PaymentDto>> getPaymentsByOrderId(@PathVariable Long orderId) {
        List<PaymentDto> payments = paymentService.getPaymentsByOrderId(orderId);
        return ResponseEntity.ok(payments);
    }
    
    @GetMapping("/status/{status}")
    public ResponseEntity<List<PaymentDto>> getPaymentsByStatus(@PathVariable PaymentStatus status) {
        List<PaymentDto> payments = paymentService.getPaymentsByStatus(status);
        return ResponseEntity.ok(payments);
    }
    
    @PostMapping
    public ResponseEntity<PaymentDto> processPayment(@Valid @RequestBody PaymentDto paymentDto) {
        try {
            PaymentDto processedPayment = paymentService.processPayment(paymentDto);
            return ResponseEntity.status(HttpStatus.CREATED).body(processedPayment);
        } catch (IllegalArgumentException e) {
            log.error("Invalid payment request", e);
            return ResponseEntity.badRequest().build();
        } catch (RuntimeException e) {
            log.error("Payment processing failed", e);
            return ResponseEntity.status(HttpStatus.PAYMENT_REQUIRED).build();
        }
    }
    
    @PostMapping("/{id}/refund")
    public ResponseEntity<PaymentDto> refundPayment(@PathVariable Long id) {
        return paymentService.refundPayment(id)
                .map(payment -> ResponseEntity.ok(payment))
                .orElse(ResponseEntity.badRequest().build());
    }
    
    @GetMapping("/revenue")
    public ResponseEntity<BigDecimal> getTotalRevenue(
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) LocalDateTime startDate,
            @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME) LocalDateTime endDate) {
        BigDecimal revenue = paymentService.getTotalRevenue(startDate, endDate);
        return ResponseEntity.ok(revenue);
    }
}
```

**File: `payments-service/src/main/resources/application.yml`**

```yaml
server:
  port: 8084

spring:
  application:
    name: payments-service
  config:
    import: "optional:configserver:http://localhost:8888"
  datasource:
    url: jdbc:postgresql://localhost:5435/payments_db
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
    fetch-registry: true
    register-with-eureka: true
  instance:
    prefer-ip-address: true

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  tracing:
    sampling:
      probability: 1.0

logging:
  level:
    com.shop.payments: DEBUG
```


### 10. Docker Compose Configuration

**File: `docker-compose.yml`**

```yaml
version: '3.8'

services:
  # PostgreSQL Databases
  catalog-db:
    image: postgres:16-alpine
    container_name: catalog-db
    environment:
      POSTGRES_DB: catalog_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - catalog_data:/var/lib/postgresql/data
    networks:
      - microservices-network

  inventory-db:
    image: postgres:16-alpine
    container_name: inventory-db
    environment:
      POSTGRES_DB: inventory_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5433:5432"
    volumes:
      - inventory_data:/var/lib/postgresql/data
    networks:
      - microservices-network

  orders-db:
    image: postgres:16-alpine
    container_name: orders-db
    environment:
      POSTGRES_DB: orders_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5434:5432"
    volumes:
      - orders_data:/var/lib/postgresql/data
    networks:
      - microservices-network

  payments-db:
    image: postgres:16-alpine
    container_name: payments-db
    environment:
      POSTGRES_DB: payments_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5435:5432"
    volumes:
      - payments_data:/var/lib/postgresql/data
    networks:
      - microservices-network

  # Config Server
  config-server:
    build:
      context: ./config-server
      dockerfile: Dockerfile
    container_name: config-server
    ports:
      - "8888:8888"
    environment:
      SPRING_PROFILES_ACTIVE: docker
    healthcheck:
      test: "curl --fail --silent localhost:8888/actuator/health | grep UP || exit 1"
      interval: 20s
      timeout: 5s
      retries: 5
      start_period: 40s
    networks:
      - microservices-network

  # Discovery Server
  discovery-server:
    build:
      context: ./discovery-server
      dockerfile: Dockerfile
    container_name: discovery-server
    ports:
      - "8761:8761"
    environment:
      SPRING_PROFILES_ACTIVE: docker
    depends_on:
      config-server:
        condition: service_healthy
    healthcheck:
      test: "curl --fail --silent localhost:8761/actuator/health | grep UP || exit 1"
      interval: 20s
      timeout: 5s
      retries: 5
      start_period: 40s
    networks:
      - microservices-network

  # API Gateway
  gateway-service:
    build:
      context: ./gateway-service
      dockerfile: Dockerfile
    container_name: gateway-service
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://discovery-server:8761/eureka
    depends_on:
      discovery-server:
        condition: service_healthy
    networks:
      - microservices-network

  # Catalog Service
  catalog-service:
    build:
      context: ./catalog-service
      dockerfile: Dockerfile
    container_name: catalog-service
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://catalog-db:5432/catalog_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: password
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://discovery-server:8761/eureka
    depends_on:
      - catalog-db
      discovery-server:
        condition: service_healthy
    networks:
      - microservices-network

  # Inventory Service
  inventory-service:
    build:
      context: ./inventory-service
      dockerfile: Dockerfile
    container_name: inventory-service
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://inventory-db:5432/inventory_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: password
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://discovery-server:8761/eureka
    depends_on:
      - inventory-db
      discovery-server:
        condition: service_healthy
    networks:
      - microservices-network

  # Orders Service
  orders-service:
    build:
      context: ./orders-service
      dockerfile: Dockerfile
    container_name: orders-service
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://orders-db:5432/orders_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: password
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://discovery-server:8761/eureka
    depends_on:
      - orders-db
      discovery-server:
        condition: service_healthy
    networks:
      - microservices-network

  # Payments Service
  payments-service:
    build:
      context: ./payments-service
      dockerfile: Dockerfile
    container_name: payments-service
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://payments-db:5432/payments_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: password
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://discovery-server:8761/eureka
    depends_on:
      - payments-db
      discovery-server:
        condition: service_healthy
    networks:
      - microservices-network

volumes:
  catalog_data:
  inventory_data:
  orders_data:
  payments_data:

networks:
  microservices-network:
    driver: bridge
```


### 11. Dockerfile Template (use for each service)

**File: `catalog-service/Dockerfile` (example - repeat for all services)**

```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copy the JAR file
COPY target/*.jar app.jar

# Create a non-root user
RUN addgroup --system spring && adduser --system spring --ingroup spring
USER spring:spring

# Expose the port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```


### 12. Build and Run Instructions

**File: `README.md`**

```markdown
# Spring Boot Microservices - Complete Implementation

## Architecture Overview

This project demonstrates a production-ready microservices architecture using Spring Boot 3, featuring:

- **API Gateway**: Route management, load balancing, and circuit breaker patterns
- **Service Discovery**: Eureka server for dynamic service registration
- **Configuration Management**: Centralized config server
- **Database per Service**: PostgreSQL databases with proper isolation
- **Inter-Service Communication**: OpenFeign clients with fallback mechanisms
- **GraphQL Support**: Modern API query language alongside REST
- **Observability**: Health checks, metrics, and distributed tracing

## Services

| Service | Port | Database | Purpose |
|---------|------|----------|---------|
| Gateway | 8080 | - | API Gateway and routing |
| Discovery | 8761 | - | Service registry |
| Config | 8888 | - | Configuration management |
| Catalog | 8081 | 5432 | Product management |
| Inventory | 8082 | 5433 | Stock management |
| Orders | 8083 | 5434 | Order processing |
| Payments | 8084 | 5435 | Payment processing |

## Prerequisites

- **Java 17+**
- **Maven 3.9+**
- **Docker & Docker Compose**
- **8GB+ RAM** (for running all services)

## Quick Start

### 1. Clone and Build
```

git clone <repository-url>
cd spring-boot-microservices

# Build all services

mvn clean package -DskipTests

```

### 2. Run with Docker Compose
```


# Start all services

docker-compose up --build

# Or run in background

docker-compose up -d --build

```

### 3. Verify Services
```


# Check all services are up

curl http://localhost:8761

# API Gateway health

curl http://localhost:8080/actuator/health

# Test catalog service

curl http://localhost:8080/api/catalog/products

```

## API Testing

### REST API Examples

#### Create a Product
```

curl -X POST http://localhost:8080/api/catalog/products \
-H "Content-Type: application/json" \
-d '{
"name": "iPhone 15",
"price": 999.99,
"sku": "IPHONE15",
"description": "Latest iPhone model"
}'

```

#### Add Inventory
```

curl -X POST http://localhost:8080/api/inventory/inventory \
-H "Content-Type: application/json" \
-d '{
"sku": "IPHONE15",
"quantity": 100
}'

```

#### Create Order
```

curl -X POST http://localhost:8080/api/orders/orders \
-H "Content-Type: application/json" \
-d '{
"customerId": 1,
"items": [
{
"sku": "IPHONE15",
"quantity": 2
}
]
}'

```

### GraphQL Examples

Access GraphQL Playground at: `http://localhost:8080/api/catalog/graphiql`

#### Query Products
```

query {
products {
id
name
price
sku
description
}
}

```

#### Create Product
```

mutation {
createProduct(
name: "Samsung Galaxy S24"
price: 899.99
sku: "GALAXY_S24"
description: "Latest Samsung flagship"
) {
id
name
price
sku
}
}

```

## Key Concepts Explained

### 1. Microservices Architecture

**What it is**: Breaking a large application into smaller, independent services.

**Benefits**:
- **Scalability**: Scale individual services based on demand
- **Technology Diversity**: Each service can use different tech stacks
- **Fault Isolation**: If one service fails, others continue working
- **Team Independence**: Different teams can work on different services

### 2. Service Discovery with Eureka

**What it does**: Automatically registers and discovers services.

**Why important**: Services can find each other without hardcoding URLs.

```

// Instead of hardcoding URLs
String url = "http://catalog-service:8081/products";

// Services discover each other automatically
@FeignClient(name = "catalog-service")
public interface CatalogServiceClient {
@GetMapping("/products")
List<Product> getAllProducts();
}

```

### 3. API Gateway Pattern

**Purpose**: Single entry point for all client requests.

**Benefits**:
- **Centralized Routing**: All requests go through one place
- **Cross-cutting Concerns**: Authentication, logging, rate limiting
- **Protocol Translation**: Convert between different protocols
- **Load Balancing**: Distribute requests across service instances

### 4. Database per Service

**Principle**: Each service owns its data.

**Implementation**:
```


# Catalog Service

spring:
datasource:
url: jdbc:postgresql://localhost:5432/catalog_db

# Orders Service

spring:
datasource:
url: jdbc:postgresql://localhost:5434/orders_db

```

**Benefits**:
- **Data Isolation**: Services can't directly access other service data
- **Technology Choice**: Each service can use optimal database
- **Independent Scaling**: Scale databases independently

### 5. Inter-Service Communication

**OpenFeign**: Declarative HTTP client for service-to-service calls.

```

@FeignClient(name = "inventory-service")
public interface InventoryServiceClient {
@GetMapping("/inventory/check/{sku}")
Boolean isInStock(@PathVariable String sku, @RequestParam Integer quantity);
}

```

**Circuit Breaker**: Prevents cascade failures.

```

resilience4j:
circuitbreaker:
instances:
catalog-service:
failure-rate-threshold: 50
wait-duration-in-open-state: 10s

```

### 6. Configuration Management

**Spring Cloud Config**: Centralized configuration.

```


# application.yml in config server

catalog-service:
database:
max-connections: 20
cache:
ttl: 300

```

Services fetch config at startup, allowing changes without redeployment.

### 7. GraphQL vs REST

| Aspect | REST | GraphQL |
|--------|------|---------|
| Endpoints | Multiple (`/products`, `/orders`) | Single (`/graphql`) |
| Data Fetching | Fixed response structure | Client specifies fields |
| Over/Under-fetching | Common problem | Solved by design |
| Caching | HTTP caching | Query-level caching |

### 8. Transaction Management

**Local Transactions**: Within single service boundaries.

```

@Transactional
public OrderDto createOrder(OrderDto orderDto) {
// All database operations succeed or fail together
Order order = orderRepository.save(order);
orderItems.forEach(item -> orderItemRepository.save(item));
return mapToDto(order);
}

```

**Distributed Transactions**: Across multiple services using Saga pattern.

```

// Order Service
@Transactional
public OrderDto createOrder(OrderDto orderDto) {
try {
// 1. Reserve inventory
inventoryService.reserveStock(items);

        // 2. Process payment
        paymentService.processPayment(paymentRequest);
        
        // 3. Create order
        Order order = orderRepository.save(order);
        
        // 4. Confirm inventory
        inventoryService.confirmStock
