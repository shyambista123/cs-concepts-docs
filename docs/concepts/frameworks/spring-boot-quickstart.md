### Spring Boot Quickstart

#### 1. Install Prerequisites

Make sure you have the following installed:

- **Java Development Kit (JDK)**: Version 8 or higher. Check if it's installed:

  ```bash
  java -version
  ```

  Download it from [oracle.com](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) if necessary.

- **Maven**: A build tool for Java projects. Check if it's installed:

  ```bash
  mvn -version
  ```

  Download it from [maven.apache.org](https://maven.apache.org/download.cgi) if needed.

#### 2. Create a New Spring Boot Project

Use the Spring Initializr to generate a new Spring Boot project:

1. Go to [start.spring.io](https://start.spring.io/).
2. Choose **Maven Project**.
3. Select your preferred **Java version**.
4. Fill in the **Group** and **Artifact** (e.g., `com.example` and `demo`).
5. Select **Dependencies**:
   - Spring Web
   - Spring Data JPA
   - H2 Database (for an in-memory database)
6. Click **Generate** to download a ZIP file containing your project.

#### 3. Extract and Navigate to Your Project

Unzip the downloaded file and navigate to the project directory:

```bash
cd demo
```

#### 4. Open the Project in Your IDE

Open the project in your favorite IDE (like IntelliJ IDEA or Eclipse). If using IntelliJ, import the project as a Maven project.

#### 5. Create a Simple Model

In `src/main/java/com/example/demo`, create a class called `Item.java`:

```java
package com.example.demo;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Item {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
}
```

#### 6. Create a Repository

Create an interface called `ItemRepository.java`:

```java
package com.example.demo;

import org.springframework.data.jpa.repository.JpaRepository;

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

#### 7. Create a Controller

Create a class called `ItemController.java`:

```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/items")
public class ItemController {

    @Autowired
    private ItemRepository itemRepository;

    @GetMapping
    public List<Item> getAllItems() {
        return itemRepository.findAll();
    }

    @PostMapping
    public Item createItem(@RequestBody Item item) {
        return itemRepository.save(item);
    }
}
```

#### 8. Run the Application

Run the main class located at `src/main/java/com/example/demo/DemoApplication.java`:

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

You can run the application via your IDE or using Maven:

```bash
mvn spring-boot:run
```

#### 9. Test Your API with Postman

**GET All Items:**

1. Open Postman.
2. Set the request type to **GET**.
3. Enter the URL: `http://localhost:8080/items`.
4. Click **Send**.
5. You should see an empty array (`[]`) as no items have been added yet.

**POST a New Item:**

1. Change the request type to **POST**.
2. Enter the URL: `http://localhost:8080/items`.
3. Go to the **Body** tab.
4. Select **raw** and choose **JSON** from the dropdown.
5. Enter the following JSON:

   ```json
   {
       "name": "Item1",
       "description": "This is Item 1"
   }
   ```

6. Click **Send**.
7. You should see the created item in the response, including its ID.

**Verify Item Creation:**

1. Set the request type to **GET**.
2. Enter the URL: `http://localhost:8080/items`.
3. Click **Send**.
4. You should see the list with your newly created item.

### Conclusion

You've successfully created a Spring Boot application with a model, repository, and RESTful controller. You also tested your API using Postman. 

For more detailed documentation, visit the official Spring Boot documentation at [spring.io](https://spring.io/projects/spring-boot). Happy coding!