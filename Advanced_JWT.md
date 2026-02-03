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
package in.maddy;

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
 
package in.maddy.entities;

import java.io.Serializable;

public class AuthenticationResponse implements Serializable{

	private static final long serialVersionUID = 1L;
	
	private final String accessToken;
    private final String refreshToken;

    public AuthenticationResponse(String accessToken, String refreshToken) {
        this.accessToken = accessToken;
        this.refreshToken = refreshToken;
    }

	public String getAccessToken() {
		return accessToken;
	}

	public String getRefreshToken() {
		return refreshToken;
	}

	 
}


```

---

## 🌐 REST Controller

### `HelloRequestController.java`

```java
 
package in.maddy.rest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.web.bind.annotation.*;

import in.maddy.entities.AuthenticationRequest;
import in.maddy.entities.AuthenticationResponse;
import in.maddy.utils.JwtUtils;

 

@RestController
public class HelloRequestController {

    private static final Logger logger = LoggerFactory.getLogger(HelloRequestController.class);

    private final AuthenticationManager authenticationManager;
    private final UserDetailsService userDetailsService;
    private final JwtUtils jwtUtils;
    
    // ✅ Constructor Injection (Best Practice)
    public HelloRequestController(AuthenticationManager authenticationManager,
                                  UserDetailsService userDetailsService,
                                  JwtUtils jwtUtils) {
        this.authenticationManager = authenticationManager;
        this.userDetailsService = userDetailsService;
        this.jwtUtils = jwtUtils;
    }

    @GetMapping("/hello")
    public String firstPage() {
        logger.info("Accessed /hello endpoint");
        return "Hello World";
    }

    @PostMapping("/authenticate")
    public ResponseEntity<?> createAuthenticationToken(
            @RequestBody AuthenticationRequest authenticationRequest) throws Exception {

        logger.info("Authentication attempt for user: {}", authenticationRequest.getUsername());

        try {
            authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(
                            authenticationRequest.getUsername(),
                            authenticationRequest.getPassword()
                    )
            );
            logger.info("Authentication successful for user: {}", authenticationRequest.getUsername());
        } catch (BadCredentialsException e) {
            logger.error("Authentication failed for user: {}", authenticationRequest.getUsername(), e);
            return ResponseEntity
                    .status(HttpStatus.UNAUTHORIZED)
                    .body("Invalid username or password");
        }

        // ✅ Load user properly
        UserDetails userDetails =
                userDetailsService.loadUserByUsername(
                        authenticationRequest.getUsername()
                );
        logger.debug("Loaded user details: {}", userDetails.getUsername());

        // ✅ Generate JWT
        String accessToken = jwtUtils.generateAccessToken(userDetails);
        String refreshToken = jwtUtils.generateRefreshToken(userDetails);
        logger.info("JWT generated for user: {}", userDetails.getUsername());

        return ResponseEntity.ok(new AuthenticationResponse(accessToken, refreshToken));
    }
  
    @PostMapping("/refresh")
    public ResponseEntity<?> refresh(@RequestParam String refreshToken) {

    	 try {
             String username = jwtUtils.extractUsername(refreshToken);
             UserDetails user =
                     userDetailsService.loadUserByUsername(username);

             String newAccessToken =
                     jwtUtils.generateAccessToken(user);

             return ResponseEntity.ok(newAccessToken);

         } catch (Exception ex) {
             logger.error("Invalid refresh token", ex);
             return ResponseEntity
                     .status(HttpStatus.UNAUTHORIZED)
                     .body("Invalid refresh token");
         }
    }

}


```

---

## 👤 User Details Service

### `MyUserDetailsService.java`

```java
package in.maddy.service;
 
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
public class MyUserDetailsService implements UserDetailsService {

    private static final Logger logger =
            LoggerFactory.getLogger(MyUserDetailsService.class);

    private final PasswordEncoder passwordEncoder;

    public MyUserDetailsService(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        logger.info("Authenticating user: {}", username);

        if (!"admin".equals(username)) {
            logger.warn("Authentication failed. User not found: {}", username);
            throw new UsernameNotFoundException("User not found");
        }

        return User.builder()
                .username("admin")
                .password(passwordEncoder.encode("admin@123")) // BCrypt
                .authorities(List.of(
                        new SimpleGrantedAuthority("ROLE_ADMIN")
                ))
                .accountExpired(false)
                .accountLocked(false)
                .credentialsExpired(false)
                .disabled(false)
                .build();
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
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import in.maddy.filters.JwtRequestFilter;

import java.util.List;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class WebSecurityConfig {


    @Autowired
    private AuthenticationEntryPoint authenticationEntryPoint;

    // ---------------- SECURITY FILTER CHAIN ----------------
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http,JwtRequestFilter jwtRequestFilter,UserDetailsService userDetailsService) throws Exception {

        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable())
            .exceptionHandling(ex ->
                ex.authenticationEntryPoint(authenticationEntryPoint)
            )
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(
                    "/authenticate",
                    "/v3/api-docs/**",
                    "/swagger-ui/**",
                    "/swagger-ui.html"
                ).permitAll()
                .anyRequest().authenticated()
            )
            .authenticationProvider(authenticationProvider(userDetailsService))
            .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    // ---------------- AUTH MANAGER ----------------
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    // ---------------- AUTH PROVIDER ----------------
    @Bean
    public DaoAuthenticationProvider authenticationProvider(UserDetailsService userDetailsService) {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    // ---------------- PASSWORD ENCODER ----------------
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // ✅ ENTERPRISE STANDARD
    }

    // ---------------- CORS CONFIG ----------------
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("http://localhost:8080"));
        config.setAllowedMethods(List.of("GET","POST","PUT","DELETE","OPTIONS"));
        config.setAllowedHeaders(List.of("Authorization","Content-Type"));
        config.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source =
                new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);

        return source;
    }
}

```

---

### `JwtKeyConfig.java`

```java
package in.maddy.security;


import java.nio.file.Files;
import java.nio.file.Path;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

 
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 

@Configuration
public class JwtKeyConfig {

    @Bean
    public KeyPair keyPair() throws Exception {

        String publicKeyPem = Files.readString(
                Path.of("src/main/resources/keys/public_key.pem")
        );

        String privateKeyPem = Files.readString(
                Path.of("src/main/resources/keys/private_key.pem")
        );

        publicKeyPem = publicKeyPem
                .replace("-----BEGIN PUBLIC KEY-----", "")
                .replace("-----END PUBLIC KEY-----", "")
                .replaceAll("\\s", "");

        privateKeyPem = privateKeyPem
                .replace("-----BEGIN PRIVATE KEY-----", "")
                .replace("-----END PRIVATE KEY-----", "")
                .replaceAll("\\s", "");

        KeyFactory keyFactory = KeyFactory.getInstance("RSA");

        PublicKey publicKey = keyFactory.generatePublic(
                new X509EncodedKeySpec(Base64.getDecoder().decode(publicKeyPem))
        );

        PrivateKey privateKey = keyFactory.generatePrivate(
                new PKCS8EncodedKeySpec(Base64.getDecoder().decode(privateKeyPem))
        );

        return new KeyPair(publicKey, privateKey);
    }
}

```
---
### `JwtAuthenticationEntryPoint.java`

```java
package in.maddy.security;

import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException authException
    ) throws IOException, ServletException {

        response.sendError(
                HttpServletResponse.SC_UNAUTHORIZED,
                "Unauthorized: JWT token is missing or invalid"
        );
    }
}

```
---
### `GlobalExceptionhandler.java`

```java
package in.maddy.security;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import io.jsonwebtoken.JwtException;

@RestControllerAdvice
public class GlobalExceptionHandler {

	 @ExceptionHandler(BadCredentialsException.class)
	    public ResponseEntity<String> handleBadCredentials() {
	        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
	                .body("Invalid username or password");
	    }

	    @ExceptionHandler(JwtException.class)
	    public ResponseEntity<String> handleJwtError() {
	        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
	                .body("Invalid or expired JWT");
	    }
}

```
---
## 🔐 Jwt Utils Configuration

### `JwtUtils.java`

```java
package in.maddy.utils;

import java.security.KeyPair;
import java.security.PublicKey;
import java.util.Date;

 
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;

import io.jsonwebtoken.*;

@Service
public class JwtUtils {

    private static final String ISSUER = "ashokit-auth-server";
    private static final String AUDIENCE = "ashokit-api";
    private static final long ACCESS_TOKEN_EXP  = 15 * 60 * 1000; // 15 min
    private static final long REFRESH_TOKEN_EXP = 7 * 24 * 60 * 60 * 1000;
    
    
    private final KeyPair keyPair;
    
    public JwtUtils(KeyPair keyPair) {
        this.keyPair = keyPair;
    }
    
    public String generateAccessToken(UserDetails userDetails) {
        return generateToken(userDetails, ACCESS_TOKEN_EXP,"ACCESS");
    }

    public String generateRefreshToken(UserDetails userDetails) {
        return generateToken(userDetails, REFRESH_TOKEN_EXP,"REFRESH");
    }

    // ----------------- GENERATE TOKEN -----------------
    public String generateToken(UserDetails userDetails, long expiry,String tokenType) {
        return Jwts.builder()
                .setSubject(userDetails.getUsername()) // no PII
                .setIssuer(ISSUER)
                .setAudience(AUDIENCE)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + expiry))
                .signWith(keyPair.getPrivate(), SignatureAlgorithm.RS256)
                .compact();
    }

    // ----------------- VALIDATE TOKEN -----------------
    public Claims validateAndParseToken(String token) {

        JwtParser parser = Jwts.parserBuilder()
                .requireIssuer(ISSUER)
                .requireAudience(AUDIENCE)
                .setSigningKey(getPublicKey())
                .build();

        Jws<Claims> jws = parser.parseClaimsJws(token);

        // Explicit algorithm validation
        String alg = jws.getHeader().getAlgorithm();
        if (!SignatureAlgorithm.RS256.getValue().equals(alg)) {
            throw new JwtException("Invalid signing algorithm");
        }

        return jws.getBody();
    }

    public String extractUsername(String token) {
        return validateAndParseToken(token).getSubject();
    }

    private PublicKey getPublicKey() {
        return keyPair.getPublic();
    }
    
    public boolean isRefreshToken(String token) {
        return "REFRESH".equals(
                validateAndParseToken(token).get("type")
        );
    }
}


```

---
## 🔐 Dependency Configuration

### `Dependencies`

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		
		<dependency>
		    <groupId>io.jsonwebtoken</groupId>
		    <artifactId>jjwt-api</artifactId>
		    <version>0.11.5</version>
		</dependency>
		
		<dependency>
		    <groupId>io.jsonwebtoken</groupId>
		    <artifactId>jjwt-impl</artifactId>
		    <version>0.11.5</version>
		    <scope>runtime</scope>
		</dependency>
		
		<dependency>
		    <groupId>io.jsonwebtoken</groupId>
		    <artifactId>jjwt-jackson</artifactId>
		    <version>0.11.5</version>
		    <scope>runtime</scope>
		</dependency>
		<dependency>
		    <groupId>org.springframework.boot</groupId>
		    <artifactId>spring-boot-starter-validation</artifactId>
		</dependency>

		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

---

## 📂 Folder Structure

```
51_SB_MS_Security_JWT_Advance/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── in/
│   │   │       └── maddy/
│   │   │           ├── Application.java
│   │   │           ├── ServletInitializer.java
│   │   │           ├── entities/
│   │   │           │   ├── AuthenticationRequest.java
│   │   │           │   └── AuthenticationResponse.java
│   │   │           ├── filters/
│   │   │           │   └── JwtRequestFilter.java
│   │   │           ├── rest/
│   │   │           │   └── HelloRequestController.java
│   │   │           ├── security/
│   │   │           │   ├── GlobalExceptionHandler.java
│   │   │           │   ├── JwtAuthenticationEntryPoint.java
│   │   │           │   ├── JwtKeyConfig.java
│   │   │           │   └── WebSecurityConfig.java
│   │   │           ├── service/
│   │   │           │   └── MyUserDetailsService.java
│   │   │           └── utils/
│   │   │               └── JwtUtils.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── keys/
│   │       │   ├── private_key.pem
│   │       │   └── public_key.pem
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/
├── target/
│   ├── generated-sources/
│   │   └── annotations/
│   └── generated-test-sources/
│       └── test-annotations/
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
├── JRE System Library [JavaSE-17]
└── Maven Dependencies
```

---

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

 
