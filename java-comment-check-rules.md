# Java Comment Check Rules

## Comment Specifications

### Standard Comment Formats
1. **Single-line comments**: Use `//` prefix for single-line comments
2. **Multi-line comments**: Use `/* */` to wrap multi-line comments
3. **Documentation comments**: Use `/** */` to wrap JavaDoc documentation
4. **Prohibit non-standard comments**: `@data` is not a standard Java comment format and should be removed

## Comment Check Rules

### Prohibit Using @data as Comment
1. **@data is not a standard comment**: `@data` is not a standard Java comment format and should not appear in code comments
2. **Remove @data comments**: If `@data` related comments exist in code, they should be deleted
3. **Distinguish annotations from comments**: `@Data` (capitalized) is a Lombok annotation, not a comment, and should not be used in comments

## Code Examples

### Wrong Example ❌
```java
// @data User information
public class User {
    private Long id;
    private String name;
}

// @data Order information
@Data
public class Order {
    private Long id;
    private String orderNo;
}

/* @data Product information */
public class Product {
    private Long id;
    private String name;
}

/**
 * @data User service interface
 */
public interface UserService {
    User getUserById(Long id);
}
```

### Correct Example ✅
```java
// User information
public class User {
    private Long id;
    private String name;
}

// Order information
@Data
public class Order {
    private Long id;
    private String orderNo;
}

/* Product information */
public class Product {
    private Long id;
    private String name;
}

/**
 * User service interface
 */
public interface UserService {
    User getUserById(Long id);
}
```

## Standard Comment Format Examples

### Single-line Comments
```java
// This is a single-line comment
public class User {
    // User ID
    private Long id;
    
    // Username
    private String name;
}
```

### Multi-line Comments
```java
/*
 * This is a multi-line comment
 * Can write multiple lines of content
 */
public class User {
    /*
     * User information class
     * Contains basic user information
     */
    private Long id;
}
```

### JavaDoc Documentation Comments
```java
/**
 * User information class
 * 
 * @author Author Name
 * @version 1.0
 * @since 2024-01-01
 */
public class User {
    /**
     * User ID
     * @return User ID
     */
    private Long id;
    
    /**
     * Get user information
     * @param id User ID
     * @return User object
     */
    public User getUserById(Long id) {
        return null;
    }
}
```

## Notes

1. **Distinguish annotations from comments**:
   - `@Data` (capitalized) is a Lombok annotation used to automatically generate getter/setter methods
   - `@data` (lowercase) is not a standard comment format and should not appear in comments

2. **Comment content standards**:
   - Comments should clearly describe the functionality and purpose of the code
   - Avoid using non-standard comment formats
   - Use standard JavaDoc tags (such as `@param`, `@return`, `@throws`, etc.)

3. **Code review**:
   - Check for non-standard comments like `@data` during code review
   - Clean up and correct non-standard comments in a timely manner

4. **IDE configuration**:
   - Configure IDE code inspection rules to automatically detect and prompt non-standard comments
   - Use code formatting tools to unify comment styles
