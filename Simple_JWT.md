# 🔐 Spring Boot JWT Authentication Application

## 📌 Project Overview

This project demonstrates a **stateless authentication system** using **Spring Boot, Spring Security, and JWT (JSON Web Tokens)**.
Every protected request is validated using a JWT token without maintaining any server-side session.

---

## 🛠️ Tech Stack

* Java
* Spring Boot
* Spring Security
* JWT (jjwt)
* Maven

---

## 🚀 Application Bootstrap

### `Application.java`

Entry point of the Spring Boot application.

```java
package in.jwt;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### `ServletInitializer.java`

Used for deploying the application as a **WAR** to external servlet containers.

```java
package in.maddy;

import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```

---

## 🔄 JWT Authentication Flow

```
Client → /authenticate → JWT generated
Client → sends JWT in Authorization header
JWT Filter → validates token
Security Context → authenticated user
Protected APIs → accessible
```

---

## 🔐 JWT Request Filter

### `JwtRequestFilter.java`

This filter runs **once per request** and validates the JWT token.

```java
package in.maddy.filters;

import java.io.IOException;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import in.maddy.security.MyUserDetailsService;
import in.maddy.utils.JwtUtils;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    @Autowired
    private MyUserDetailsService myUserDetailsService;

    @Autowired
    private JwtUtils jwtUtils;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        final String authHeader = request.getHeader("Authorization");

        String username = null;
        String jwt = null;

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            jwt = authHeader.substring(7);
            username = jwtUtils.extractUsername(jwt);
        }

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails = myUserDetailsService.loadUserByUsername(username);

            if (jwtUtils.validateToken(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails, null, userDetails.getAuthorities());

                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

---

## 🧾 Authentication Models

### `AuthenticationRequest.java`

```java
package in.maddy.models;

import java.io.Serializable;

public class AuthenticationRequest implements Serializable {

    private static final long serialVersionUID = 1L;

    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### `AuthenticationResponse.java`

```java
package in.maddy.models;

import java.io.Serializable;

public class AuthenticationResponse implements Serializable {

    private static final long serialVersionUID = 1L;

    private final String jwt;

    public AuthenticationResponse(String jwt) {
        this.jwt = jwt;
    }

    public String getJwt() {
        return jwt;
    }
}
```

---

## 🌐 REST Controller

### `HelloRequestController.java`

```java
package in.maddy.rest;

import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.web.bind.annotation.*;

import in.maddy.models.AuthenticationRequest;
import in.maddy.models.AuthenticationResponse;
import in.maddy.utils.JwtUtils;

@RestController
public class HelloRequestController {

    private final AuthenticationManager authenticationManager;
    private final UserDetailsService userDetailsService;
    private final JwtUtils jwtUtils;

    public HelloRequestController(AuthenticationManager authenticationManager,
                                  UserDetailsService userDetailsService,
                                  JwtUtils jwtUtils) {
        this.authenticationManager = authenticationManager;
        this.userDetailsService = userDetailsService;
        this.jwtUtils = jwtUtils;
    }

    @GetMapping("/hello")
    public String firstPage() {
        return "Hello World";
    }

    @PostMapping("/authenticate")
    public ResponseEntity<?> createAuthenticationToken(
            @RequestBody AuthenticationRequest authenticationRequest) throws Exception {

        try {
            authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(
                            authenticationRequest.getUsername(),
                            authenticationRequest.getPassword()));
        } catch (BadCredentialsException e) {
            throw new Exception("Incorrect username or password", e);
        }

        UserDetails userDetails = userDetailsService
                .loadUserByUsername(authenticationRequest.getUsername());

        String jwt = jwtUtils.generateToken(userDetails.getUsername());
        return ResponseEntity.ok(new AuthenticationResponse(jwt));
    }
}
```

---

## 👤 User Details Service

### `MyUserDetailsService.java`

```java
package in.maddy.security;

import java.util.ArrayList;

import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class MyUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return new User("admin", "admin@123", new ArrayList<>());
    }
}
```

---

## 🔐 Spring Security Configuration

### `WebSecurityConfig.java`

```java
package in.maddy.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import in.maddy.filters.JwtRequestFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@SuppressWarnings("deprecation")
public class WebSecurityConfig {

    @Autowired
    private JwtRequestFilter jwtRequestFilter;

    @Autowired
    private UserDetailsService myUserDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/authenticate").permitAll()
                .anyRequest().authenticated())
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(myUserDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public PasswordEncoder passwordEncoder(
```
 

 

### 1️⃣ Application Bootstrap

**`Application.java`**

* Entry point of the Spring Boot application
* Uses `@SpringBootApplication` to enable:

  * Auto-configuration
  * Component scanning
  * Spring Boot setup

**`ServletInitializer.java`**

* Extends `SpringBootServletInitializer`
* Allows deployment as a **WAR file** on external servers (Tomcat, etc.)
* Important for production-ready apps

---

### 2️⃣ JWT Authentication Flow (High Level)

```
Client → /authenticate → JWT generated
Client → sends JWT in Authorization header
JWT Filter → validates token
Security Context → authenticated user
Protected APIs → accessible
```

---

### 3️⃣ JWT Filter Layer

**`JwtRequestFilter`**

* Runs **once per request**
* Responsibilities:

  * Read `Authorization` header
  * Extract JWT (`Bearer <token>`)
  * Extract username from token
  * Validate token
  * Set authentication in `SecurityContextHolder`

✔ Best practices used:

* `OncePerRequestFilter`
* Avoids re-authentication if context already exists
* Uses `UsernamePasswordAuthenticationToken` correctly

---

### 4️⃣ Authentication Models

**`AuthenticationRequest`**

* Accepts:

  * `username`
  * `password`
* Used in `/authenticate` API

**`AuthenticationResponse`**

* Returns:

  * `jwt` token
* Immutable response object (good design)

---

### 5️⃣ REST Controller

**`HelloRequestController`**

Endpoints:

* `GET /hello`

  * Protected API
  * Requires valid JWT
* `POST /authenticate`

  * Public API
  * Validates credentials
  * Generates JWT

Flow inside `/authenticate`:

1. Authenticate using `AuthenticationManager`
2. Load user via `UserDetailsService`
3. Generate JWT
4. Return token to client

---

### 6️⃣ User Details Service

**`MyUserDetailsService`**

* Implements `UserDetailsService`
* Currently returns **in-memory static user**:

  ```
  username: admin
  password: admin@123
  ```

📌 This is perfect for:

* Learning
* Demos
* POCs

(In real projects → DB-based users)

---

### 7️⃣ Spring Security Configuration

**`WebSecurityConfig`**

Key configurations:

* ❌ CSRF disabled (required for stateless JWT)
* ✅ `/authenticate` → public
* 🔒 All other endpoints → secured
* 🚫 Session creation disabled (STATELESS)
* 🔑 Custom JWT filter added
* 🔐 DAO Authentication Provider used

Password Encoding:

* `NoOpPasswordEncoder`

  * Used for simplicity
  * ⚠️ Not recommended for production

---

### 8️⃣ JWT Utility Class

**`JwtUtils`**

Responsibilities:

* Generate JWT token
* Validate JWT token
* Extract:

  * Username
  * Expiration
* Uses:

  * HS256 algorithm
  * 256-bit secret key
  * 1-hour expiration

✔ Proper use of `io.jsonwebtoken (jjwt)` library

---

## 📘 README.md (Based on Your Code)

You can **directly copy-paste** this into `README.md` 👇

---

```md
# 🔐 Spring Boot JWT Authentication Application

## 📌 Project Overview
This project demonstrates a **stateless authentication system** using **Spring Boot, Spring Security, and JWT (JSON Web Tokens)**.  
It secures REST APIs by validating JWT tokens on every request without using server-side sessions.

---

## 🚀 Features
- JWT-based authentication
- Stateless security (no HTTP session)
- Custom JWT filter
- Secure API access using Spring Security
- Clean and modular architecture
- Ready for REST API integration

---

## 🛠️ Tech Stack
- Java
- Spring Boot
- Spring Security
- JWT (jjwt library)
- Maven

---

## 🔄 Authentication Flow
1. Client sends credentials to `/authenticate`
2. Server validates username & password
3. JWT token is generated
4. Client sends JWT in `Authorization` header
5. JWT filter validates token
6. Access is granted to protected APIs

---

## 📂 Package Structure
```

in.maddy
├── filters
│   └── JwtRequestFilter.java
├── models
│   ├── AuthenticationRequest.java
│   └── AuthenticationResponse.java
├── rest
│   └── HelloRequestController.java
├── security
│   ├── MyUserDetailsService.java
│   └── WebSecurityConfig.java
├── utils
│   └── JwtUtils.java
├── Application.java
└── ServletInitializer.java

````

---

## 🔐 API Endpoints

### 1️⃣ Authenticate User
**POST** `/authenticate`

**Request Body**
```json
{
  "username": "admin",
  "password": "admin@123"
}
````

**Response**

```json
{
  "jwt": "eyJhbGciOiJIUzI1NiJ9..."
}
```

---

### 2️⃣ Protected API

**GET** `/hello`

**Header**

```
Authorization: Bearer <JWT_TOKEN>
```

**Response**

```
Hello World
```

---

## 👤 Default User (In-Memory)

```
Username: admin
Password: admin@123
```

---

## ⚙️ Security Configuration

* CSRF disabled
* Stateless session policy
* Custom JWT filter
* DAO authentication provider
* NoOp password encoder (for demo purposes)

---

## ⏳ Token Details

* Algorithm: HS256
* Expiration Time: 1 Hour
* Secret Key: 256-bit key

---

## ⚠️ Notes

* Password encoder is `NoOpPasswordEncoder` (not for production)
* User data is in-memory
* Can be easily extended with database authentication

---

### 1️⃣ Architecture Diagram (BIG IMPACT)

Clients & reviewers **love visuals**.

```md
## 🧱 System Architecture

Client
  ↓ (Credentials)
Authentication API
  ↓ (JWT)
JWT Filter
  ↓
Spring Security Context
  ↓
Protected REST APIs
```

Or you can add an image later:

```md
![JWT Architecture](docs/jwt-architecture.png)
```

---

### 2️⃣ Sequence Flow (Interview Gold ⭐)

Shows you *understand* what’s happening internally.

```md
## 🔄 Request Lifecycle

1. Client sends login request to `/authenticate`
2. AuthenticationManager validates credentials
3. JWT token is generated
4. Token is returned to client
5. Client sends token with each request
6. JwtRequestFilter validates token
7. SecurityContext is populated
8. Protected API is executed
```

---

### 3️⃣ Security Design Decisions (Very Professional)

This screams **“real backend developer”**.

```md
## 🔐 Security Design Decisions

- JWT used to ensure stateless authentication
- CSRF disabled due to REST-based stateless APIs
- Sessions disabled to improve scalability
- Custom filter used for token validation
- Authentication context stored per request
```

---

### 4️⃣ Error Handling Section

Even simple projects look senior with this.

```md
## ❌ Error Handling

- Invalid credentials → 401 Unauthorized
- Missing token → 403 Forbidden
- Expired token → Access denied
- Malformed token → Authentication failure
```

---

### 5️⃣ How to Run Locally (Must-Have for GitHub)

This is almost mandatory.

````md
## ▶️ Run Project Locally

1. Clone the repository
```bash
git clone https://github.com/your-username/jwt-auth-app.git
````

2. Navigate to project

```bash
cd jwt-auth-app
```

3. Run the application

```bash
mvn spring-boot:run
```

4. Application runs at:

```
http://localhost:8080
```

````

---

### 6️⃣ Testing with Postman / Curl (Client Friendly)
Shows practical usage.

```md
## 🧪 API Testing

Use Postman or curl:

```bash
curl -X POST http://localhost:8080/authenticate \
-H "Content-Type: application/json" \
-d '{"username":"admin","password":"admin@123"}'
````

````

---

### 7️⃣ Limitations (Honesty = Trust)
Very underrated but powerful.

```md
## ⚠️ Limitations

- In-memory authentication only
- No password encryption
- No refresh token mechanism
- No role-based authorization
````

---

### 8️⃣ Production Readiness Checklist ✅

This is 🔥 for interviews.

```md
## 🚀 Production Readiness Checklist

- [ ] Replace NoOpPasswordEncoder with BCrypt
- [ ] Move users to database
- [ ] Externalize JWT secret
- [ ] Add refresh tokens
- [ ] Enable HTTPS
- [ ] Add logging & monitoring
```

---

### 9️⃣ API Contract Summary

Makes it feel enterprise-level.

```md
## 📄 API Contract

| Method | Endpoint        | Access  | Description              |
|------|----------------|--------|--------------------------|
| POST | /authenticate  | Public | Generate JWT token       |
| GET  | /hello         | Secure | Sample protected API     |
```

---

### 🔑 My Top 5 (If You Add Only a Few)

If you want **maximum impact with minimum effort**, add these:

1. Architecture Diagram
2. Request Lifecycle
3. How to Run Locally
4. Production Readiness Checklist
5. Security Design Decisions

---

## 📈 Future Enhancements

* Database-backed users (JPA + MySQL)
* Refresh token support
* Role-based authorization
* Password encryption (BCrypt)
* Global exception handling

---

## 👨‍💻 Author

**Shivam Sharma**
Java | Spring Boot | Backend Developer

```

 
