# Java-Spring-Boot
Calculator age and dob
import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.data.jpa.repository.JpaRepository;

import org.springframework.stereotype.Service;

import org.springframework.web.bind.annotation.*;

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.http.HttpStatus;

import jakarta.persistence.*;

import jakarta.validation.constraints.NotBlank;

import jakarta.validation.constraints.NotNull;

import java.time.LocalDate;

import java.time.Period;

import java.util.Optional;

// --- 1. Main Application Class ---

@SpringBootApplication

public class UserManagementApp {

// This starts the Spring Boot application.

// It will automatically configure a REST API and an in-memory H2 database.

public static void main(String[] args) {

SpringApplication.run(UserManagementApp.class, args);

System.out.println("\nApplication is running! Access the API at http://localhost:8080");

System.out.println("Use H2 Console to view database: http://localhost:8080/h2-console");

}

}

// --- 2. Data Model (Entity) ---

@Entity

@Table(name = "users")

class User {

@Id

@GeneratedValue(strategy = GenerationType.IDENTITY)

private Long id;

@NotBlank(message = "Name is required")

private String name;

@NotNull(message = "Date of Birth (dob) is required")

private LocalDate dob; // Stores the date of birth

// Standard Getters and Setters

public Long getId() { return id; }

public void setId(Long id) { this.id = id; }

public String getName() { return name; }

public void setName(String name) { this.name = name; }

public LocalDate getDob() { return dob; }

public void setDob(LocalDate dob) { this.dob = dob; }

}

// --- 3. DTO (Data Transfer Object) for Output ---

class UserResponse {

private Long id;

private String name;

private LocalDate dob;

private int age; // Dynamically calculated age

public UserResponse(User user, int age) {

this.id = user.getId();

this.name = user.getName();

this.dob = user.getDob();

this.age = age;

}

// Standard Getters
public Long getId() { return id; }

public String getName() { return name; }

public LocalDate getDob() { return dob; }

public int getAge() { return age; }

}

// --- 4. Repository (Data Access Layer) ---

// Spring Data JPA handles all CRUD operations automatically

interface UserRepository extends JpaRepository<User, Long> {

}

// --- 5. Service (Business Logic Layer) ---

@Service

class UserService {

@Autowired

private UserRepository userRepository;

/**

* Calculates the age based on the date of birth.

* @param dob The user's date of birth.

* @return The calculated age in years.

*/

public int calculateAge(LocalDate dob) {

if (dob == null) {

return 0;

}

// Use Java Time API for accurate date difference calculation

return Period.between(dob, LocalDate.now()).getYears();

}

/**

* Saves a new user to the database.

*/

public User saveUser(User user) {

return userRepository.save(user);

}

/**

* Finds a user by ID and calculates their age.

* @param id The user ID.

* @return An Optional containing the UserResponse DTO, or empty if not found.

*/

public Optional<UserResponse> findUserWithAge(Long id) {

Optional<User> userOptional = userRepository.findById(id);

if (userOptional.isEmpty()) {

return Optional.empty();

}

User user = userOptional.get();

int age = calculateAge(user.getDob());

return Optional.of(new UserResponse(user, age));

}

}

// --- 6. Controller (API Endpoint Layer) ---

@RestController

@RequestMapping("/users")

class UserController {

@Autowired

private UserService userService;

// POST /users: Create User Endpoint
@PostMapping

@ResponseStatus(HttpStatus.CREATED)

public User createUser(@RequestBody User user) {

return userService.saveUser(user);

}

// GET /users/{id}: Fetch User with Dynamic Age Calculation Endpoint

@GetMapping("/{id}")

public UserResponse getUser(@PathVariable Long id) {

return userService.findUserWithAge(id)

.orElseThrow(() -> new ResourceNotFoundException("User not found with ID: " + id));

}

}

// --- 7. Custom Exception Handler ---

@ResponseStatus(HttpStatus.NOT_FOUND)

class ResourceNotFoundException extends RuntimeException {

public ResourceNotFoundException(String message) {

super(message);

}

}
