# Spring Web Layered Architecture Rules

## Core Principles

### Request Parameter Specifications
1. **Prohibit using MyBatis table entities as interface request parameters**: Interface request parameters (Request DTO) must use independent DTO (Data Transfer Object) classes, and cannot directly use MyBatis generated entity classes or table mapping entity classes
2. **Separate request and response**: Request parameters (Request DTO) and response parameters (Response DTO) should be defined separately and cannot be mixed
3. **Entity class responsibility separation**: Database entity classes (Entity/DO) are only used for the data persistence layer and should not be exposed to the Controller layer
4. **Unified response format restriction**: Only interface return values (Controller layer) can use unified response format structures (such as `code`, `msg`, `data`), and such structures must not be passed in the Service layer

## Layered Architecture Specifications

### Controller Layer
1. **Use Request DTO**: Controller method parameters must use dedicated Request DTO classes
2. **Use Response DTO**: Controller method return values must use dedicated Response DTO classes
3. **Parameter validation**: Use Bean Validation annotations on Request DTO for parameter validation
4. **Unified response format**: Controller layer can use unified response format (e.g., `Result<T>` with `code`, `msg`, `data` fields) for interface return values

### Service Layer
1. **Receive DTO parameters**: Service methods receive Request DTO as parameters
2. **Return DTO results**: Service methods return Response DTO or business objects
3. **Entity conversion**: The Service layer is responsible for converting DTO to Entity, or Entity to DTO
4. **Prohibit unified response format**: Service layer must not use or return unified response format structures (such as `Result<T>` with `code`, `msg`, `data`). Service methods should return business objects or DTOs directly, and the Controller layer is responsible for wrapping them into unified response format

### Mapper/DAO Layer
1. **Use Entity objects**: The Mapper layer only operates on database entity classes (Entity/DO)
2. **Not exposed to upper layers**: Entity objects should not be directly returned to the Controller layer

## DTO Design Specifications

### Request DTO
1. **Naming convention**: Use `{BusinessName}Request` or `{BusinessName}DTO` naming, e.g., `UserCreateRequest`, `OrderQueryDTO`
2. **Field design**: Only include fields required by the interface, exclude database technical fields (such as `id`, `created_at`, `updated_at`, etc., unless business requires)
3. **Validation annotations**: Use `@NotNull`, `@NotBlank`, `@Size`, `@Min`, `@Max`, `@Email` and other annotations for parameter validation
4. **Documentation annotations**: Use `@ApiModel`, `@ApiModelProperty` and other annotations to provide API documentation

### Response DTO
1. **Naming convention**: Use `{BusinessName}Response` or `{BusinessName}VO` naming, e.g., `UserResponse`, `OrderVO`
2. **Field design**: Only include fields that need to be returned to the frontend, can include calculated fields, formatted fields, etc.
3. **Sensitive information**: Do not include sensitive information (such as passwords, internal status, etc.)
4. **Documentation annotations**: Use `@ApiModel`, `@ApiModelProperty` and other annotations to provide API documentation

### Entity/DO
1. **Naming convention**: Use `{TableName}Entity` or `{TableName}DO` naming, e.g., `UserEntity`, `OrderDO`
2. **Single responsibility**: Only used for database mapping, does not contain business logic
3. **Complete fields**: Contains all fields of the database table
4. **Annotation usage**: Use MyBatis related annotations (such as `@Table`, `@Column`, `@Id`, etc.)

## Conversion Specifications

### DTO and Entity Conversion
1. **Use conversion tools**: Recommend using MapStruct, BeanUtils, or custom conversion methods
2. **Conversion location**: Perform DTO and Entity conversion in the Service layer
3. **Avoid circular dependencies**: DTO and Entity should not reference each other

### Conversion Example
```java
// Service layer conversion example
@Service
public class UserService {
    
    public UserResponse createUser(UserCreateRequest request) {
        // Convert Request DTO to Entity
        UserEntity entity = convertToEntity(request);
        
        // Call Mapper to save
        userMapper.insert(entity);
        
        // Convert Entity to Response DTO
        return convertToResponse(entity);
    }
    
    private UserEntity convertToEntity(UserCreateRequest request) {
        UserEntity entity = new UserEntity();
        entity.setUsername(request.getUsername());
        entity.setEmail(request.getEmail());
        // ... other field conversions
        return entity;
    }
    
    private UserResponse convertToResponse(UserEntity entity) {
        UserResponse response = new UserResponse();
        response.setId(entity.getId());
        response.setUsername(entity.getUsername());
        response.setEmail(entity.getEmail());
        // ... other field conversions
        return response;
    }
}
```

## Examples

### Wrong Example ❌
```java
// Controller layer - Wrong: directly using Entity
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public UserEntity createUser(@RequestBody UserEntity user) {
        // Wrong: directly using Entity as request parameter
        return userService.createUser(user);
    }
}
```

### Correct Example ✅
```java
// Request DTO
@Data
@ApiModel("User Create Request")
public class UserCreateRequest {
    @NotBlank(message = "Username cannot be empty")
    @ApiModelProperty("Username")
    private String username;
    
    @NotBlank(message = "Email cannot be empty")
    @Email(message = "Email format is incorrect")
    @ApiModelProperty("Email")
    private String email;
    
    @NotBlank(message = "Password cannot be empty")
    @Size(min = 6, max = 20, message = "Password length must be between 6-20")
    @ApiModelProperty("Password")
    private String password;
}

// Response DTO
@Data
@ApiModel("User Response")
public class UserResponse {
    @ApiModelProperty("User ID")
    private Long id;
    
    @ApiModelProperty("Username")
    private String username;
    
    @ApiModelProperty("Email")
    private String email;
    
    @ApiModelProperty("Created at")
    private LocalDateTime createdAt;
}

// Entity
@Data
@Table(name = "users")
public class UserEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    private String passwordHash;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

// Unified Response Format
@Data
public class Result<T> {
    private Integer code;
    private String msg;
    private T data;
    
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMsg("success");
        result.setData(data);
        return result;
    }
    
    public static <T> Result<T> error(Integer code, String msg) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMsg(msg);
        return result;
    }
}

// Controller layer - Correct: using DTO and unified response format
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    @ApiOperation("Create User")
    public Result<UserResponse> createUser(@Valid @RequestBody UserCreateRequest request) {
        // Service returns business object, Controller wraps it into unified response format
        UserResponse response = userService.createUser(request);
        return Result.success(response);
    }
}
```

### Unified Response Format Usage

#### Wrong Example ❌
```java
// Service layer - Wrong: returning unified response format
@Service
public class UserService {
    
    public Result<UserResponse> createUser(UserCreateRequest request) {
        // Wrong: Service layer should not return unified response format
        UserEntity entity = convertToEntity(request);
        userMapper.insert(entity);
        UserResponse response = convertToResponse(entity);
        return Result.success(response);
    }
}
```

#### Correct Example ✅
```java
// Service layer - Correct: returning business object
@Service
public class UserService {
    
    public UserResponse createUser(UserCreateRequest request) {
        // Correct: Service returns business object directly
        UserEntity entity = convertToEntity(request);
        userMapper.insert(entity);
        return convertToResponse(entity);
    }
}

// Controller layer - Correct: wrapping into unified response format
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    @ApiOperation("Create User")
    public Result<UserResponse> createUser(@Valid @RequestBody UserCreateRequest request) {
        // Correct: Controller wraps Service result into unified response format
        UserResponse response = userService.createUser(request);
        return Result.success(response);
    }
}
```

## Notes

1. **Performance considerations**: DTO conversion will bring certain performance overhead, but this is a necessary architectural layering cost
2. **Code reuse**: Can use tools like MapStruct to automatically generate conversion code, reducing manual writing
3. **Field mapping**: Note that DTO and Entity field names may differ and need correct mapping
4. **Nested objects**: For nested objects, DTO conversion is also required, cannot directly use Entity
5. **Batch operations**: Request parameters for batch operations should also use DTO, not Entity collections
6. **Query parameters**: Query interface parameters should also use dedicated Query DTO, not directly use Entity
7. **Unified response format**: Only Controller layer can use unified response format (e.g., `Result<T>`), Service layer must return pure business objects or DTOs
