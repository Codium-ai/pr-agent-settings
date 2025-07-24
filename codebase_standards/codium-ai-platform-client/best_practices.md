<b>Pattern 1: Simplify "If" statements by extracting complex conditions into well-named variables for better readability and maintainability.</b>

Example code before:
```typescript
if (user.age >= 18 && user.hasValidMembership && !user.hasUsedDiscount) {
  applyDiscount();
}
```

Example code after:
```typescript
const isUserEligibleForDiscount =
  user.age >= 18 && user.hasValidMembership && !user.hasUsedDiscount;

if (isUserEligibleForDiscount) {
  applyDiscount();
}
```


<b>Pattern 2: Keep logic out of hooks by extracting it into separate, testable utility functions to improve code organization and testability.</b>

Example code before:
```typescript
const usePathSegments = (): string[] => {
  const location = useLocation();
  return location.pathname.split('/').filter(Boolean);
};
```

Example code after:
```typescript
// utils/url.ts
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

// hooks/usePathSegments.ts
const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```


<b>Pattern 3: Always add JSDoc comments to utility functions and hooks to improve code documentation and developer experience.</b>

Example code before:
```typescript
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```

Example code after:
```typescript
// utils/url.ts
/**
 * Extracts path segments from a URL pathname
 * @param pathname - The URL pathname to extract segments from
 * @returns Array of non-empty path segments
 */
const extractPathSegments = (pathname: string): string[] => {
  return pathname.split('/').filter(Boolean);
};

// hooks/usePathSegments.ts
/**
 * Hook that returns the current URL path segments
 * @returns Array of non-empty path segments from the current URL
 */
const usePathSegments = (): string[] => {
  const location = useLocation();
  return extractPathSegments(location.pathname);
};
```


<b>Pattern 4: Use named exports instead of default exports to improve code maintainability and enable better IDE refactoring support.</b>

Example code before:
```typescript
// utils/math.ts
const add = (a: number, b: number): number => a + b;
const subtract = (a: number, b: number): number => a - b;

export default { add, subtract };

// hooks/useCounter.ts
const useCounter = (initialValue: number) => {
  const [count, setCount] = useState(initialValue);
  return { count, setCount };
};

export default useCounter;
```

Example code after:
```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;
export const subtract = (a: number, b: number): number => a - b;

// hooks/useCounter.ts
export const useCounter = (initialValue: number) => {
  const [count, setCount] = useState(initialValue);
  return { count, setCount };
};
```


<b>Pattern 5: Always use barrel files (index files) to centralize exports and simplify import statements across the codebase.</b>

Example code before:
```typescript
// utils/math/add.ts
export const add = (a: number, b: number): number => a + b;

// utils/math/subtract.ts
export const subtract = (a: number, b: number): number => a - b;

// Usage in other files
import { add } from '@/utils/math/add';
import { subtract } from '@/utils/math/subtract';
```

Example code after:
```typescript
// utils/math/add.ts
export const add = (a: number, b: number): number => a + b;

// utils/math/subtract.ts
export const subtract = (a: number, b: number): number => a - b;

// utils/math/index.ts
export * from './add';
export * from './subtract';

// Usage in other files
import { add, subtract } from '@/utils/math';
```


<b>Pattern 6: Unit tests should be co-located with the code they test to improve maintainability and make the relationship between code and tests clear.</b>

Example code before:
```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

// tests/utils/math.test.ts
import { add } from '../../../../utils/math';

describe('add', () => {
  // Test code here
});
```

Example code after:
```typescript
// utils/math.ts
export const add = (a: number, b: number): number => a + b;

// utils/math.spec.ts
import { add } from './math';

describe('add', () => {
  // Test code here
});
```


Best Practices Guidelines:
======
Repository best practices:
==================
## Overview

This guide outlines the standards for developing Java APIs at WHOOP. Following these rules ensures consistent API implementation, proper error handling, and maintainable code.

## Framework Guidelines

1. **Dropwizard Framework**
   - Follow Dropwizard patterns for:
     - REST endpoints
     - Health checks
     - API resources
   - Example:
     ```java
     @Path("/v1/health")
     public class HealthCheck  {

         @GET
         public Result check() {
             return Result.builder().setStatus(Status.HEALTHY).build();
         }
     }
     ```

## Pagination Implementation

2. **Token-Based Pagination**
   - Use tokens for all list endpoints
   - Implement using `OffsetTokenUtil`
   - Example:
     ```java
     @Path("/v1/myresource")
     public class MyResource {
         private final OffsetTokenUtil tokenUtil;
         
         @Inject
         public MyResource(@Named(OFFSET_TOKEN_UTIL) OffsetTokenUtil tokenUtil) {
             this.tokenUtil = tokenUtil;
         }

         @GET
         @Path("/")
         public PaginatedRecords<MyType> myEndpoint(
             @QueryParam(NEXT_TOKEN) String tokenString,
             @QueryParam(LIMIT) @DefaultValue(DEFAULT_PAGE_SIZE) @Max(DEFAULT_PAGE_LIMIT) int limit) {
             OffsetToken token = tokenUtil.parseToken(tokenString);
             List<MyType> collection = myTypeDao.get(token.getOffset(), limit);

             Optional<String> nextToken = tokenUtil.create(token.getOffset(), limit, collection.size());
             return PaginatedRecords.<MyType>builder()
                     .setRecords(collection)
                     .setNextToken(nextToken)
                     .build();
         }
     }
     ```

## API Resource Best Practices

3. **HTTP Standards**
   - Use proper HTTP methods:
     - GET: Retrieve resources
     - POST: Create resources
     - PUT: Update resources
     - DELETE: Remove resources
   - Include appropriate status codes
   - Use proper content types (application/json)
   - Version your APIs (e.g., /v1/...)

4. **Authentication & Authorization**
   - Use proper annotations:
     - `@ValidateWhoopToken` - WHOOP App Authentication
     - `@ValidateThirdParty` - Third Party Authentication
     - `@ValidateWhoopRole` - Internal Employee Role Authentication
     - `@NoAuth` - No Authentication
   - Inject auth info via:
     - `@TrustedUserId` - Injects a user_id
     - `@TrustedRole` - Inject a users roles
     - `@TrustedInternalGoogleEmail` - Injects a users email
   - Never accept `userId` from clients directly, use `@TrustedUserId`

5. **Resource Structure**
   - Prefer thin `@Resource` classes
   - Push logic to service layers
   - Use proper path naming conventions
   - Include proper documentation
   - Example:
     ```java
     @Path("/v1/users")
     @Produces(MediaType.APPLICATION_JSON)
     @Consumes(MediaType.APPLICATION_JSON)
     public class UserResource {
         private final UserService userService;

         @Inject
         public UserResource(UserService userService) {
             this.userService = userService;
         }

         @GET
         @Path("/{id}")
         @ValidateWhoopToken
         public User getUser(@PathParam("id") String id) {
             return userService.getUser(id);
         }
     }
     ```

## Response Handling

6. **Response Structure**
   - Use immutable models for responses
   - Include proper error responses
   - Use consistent naming conventions
   - Include proper validation annotations
   - Use proper serialization/deserialization annotations

7. **Error Handling**
   - Use proper exception mappers
   - Include meaningful error messages
   - Use proper HTTP status codes from `com.google.api.client.http.HttpStatusCodes`
   - Log errors appropriately
   - Include proper error tracking
   - Example:
     ```java
     
     public Response handleNotFoundException(NotFoundException e) {
         return Response.status(Status.NOT_FOUND)
               .entity(new ErrorResponse(e.getMessage()))
               .build();
     }
     ```

## API Versioning

8. **Resource-Level Versioning**
   - Use for any breaking change
   - Copy resource class to `v2`, keep `v1` intact
   - Move all touched endpoints
   - Only version models if their schema changes
   - Mark old versions with:
     ```java
     @Deprecated(since = "2025-03-01", forRemoval = true)
     @Operation(description = "Use /v2/xyz instead. Will be removed 2025-06-01.")
     ```

## BFF Endpoints

9. **Domain Layer Service and Presenter Pattern**
   - Always use the domain service -> presenter pattern for BFF endpoints
   - Domain service handles business logic and returns domain objects
   - Presenter transforms domain objects to API response models
   - Keep the resource layer thin and focused on HTTP concerns
   - Example:
     ```java
     @GET
     @Path("/resource")
     @ValidateWhoopToken
     public ApiResponseModel getResource() {
         // 1. Get authenticated user context
         int userId = trustedUserId.get();
         
         // 2. Call domain service to get business domain object
         DomainObject domain = domainService.getDomainObject(userId);
         
         // 3. Use presenter to transform domain object to API response
         ApiResponseModel response = presenter.present(domain);
         
         // 4. Return the API response
         return response;
     }
     ```

## Best Practices

1. **Documentation**
   - Include OpenAPI/Swagger annotations
   - Document all parameters and responses
   - Provide usage examples
   - Keep documentation up to date

2. **Testing**
   - Write integration tests for all endpoints
   - Test error scenarios
   - Verify pagination behavior
   - Test authentication and authorization 



 ## Overview

This guide outlines the standards for developing Data Access Objects (DAOs) in Java at WHOOP. Following these rules ensures consistent database access patterns, proper error handling, and maintainable code.

## DAO Creation Guidelines

1. **Table Coverage**
   - Create DAOs for all new database tables
   - Exception: many-to-many (join) tables can be handled by either side's DAO
   - Example:
     ```java
     // For a users table
     public interface UserDao {
         @SqlQuery("SELECT * FROM users WHERE user_id = :user_id")
         Optional<User> getById(@Bind("user_id") int userId);
     }
     ```

## Implementation Patterns

2. **Interface-Based Pattern**
   - Use JDBI SQL annotations
   - Follow interface-first design
   - Example:
     ```java
     public interface MyDao {
         @SqlQuery("SELECT COUNT(*) FROM my_table")
         int getCount();
     }
     ```

3. **Guice Integration**
   - Register DAOs in Guice modules using JDBI
   - Use consistent naming conventions
   - Example:
     ```java
     @Provides
     MyDao provideDao(@Named("myproject.jdbi") Jdbi jdbi) {
         return jdbi.onDemand(MyDao.class);
     }
     ```

## SQL Operations

4. **JDBI Annotations**
   - Use appropriate annotations for operations:
     - `@SqlQuery` for SELECT operations
     - `@SqlUpdate` for INSERT, UPDATE, DELETE operations
     - `@SqlBatch` for batch operations
   - Example:
     ```java
     @SqlUpdate("INSERT INTO users (name, email) VALUES (:name, :email)")
     void insert(@BindWithRosetta User user);

     @SqlBatch("INSERT INTO users (name, email) VALUES (:name, :email)")
     void insertBatch(@BindWithRosetta List<User> users);
     ```

## Naming Conventions

5. **Interface and Method Naming**
   - Interface name: `[EntityName]Dao`
   - Methods: Use descriptive verbs (e.g., `getById`, `insert`, `update`)
   - Use clear, specific names that describe the operation
   - Avoid accessor patterns in field names (get, set, is)
   - Use snake_case for SQL query parameter bindings
   - Example:
     ```java
     public interface UserDao {
         Optional<User> findById(@Bind("user_id") int userId);
         List<User> findByEmail(@Bind("email_address") String email);
         void insert(@BindWithRosetta User user);
     }
     ```

## Parameter Binding

6. **SQL Parameter Binding**
   - Use `@Bind` for named parameters with snake_case in SQL
   - Use `@BindWithRosetta` for object properties
   - Use `@BindList` for IN clauses
   - Example:
     ```java
     @SqlQuery(
       "SELECT * " +
       "  FROM users " +
       " WHERE user_id = :user_id " +
       "   AND email_address = :email_address")
     User getUserByIdAndEmail(
         @Bind("user_id") int userId,
         @Bind("email_address") String email);
     ```

## Error Handling

7. **Result Handling**
   - Use `Optional` for single-result queries that might return null
   - Use `List` for multiple results
   - Handle SQL exceptions appropriately
   - Never call `.get()` on an Optional without verifying it is present/empty
   - Example:
     ```java
     @SqlQuery("SELECT * FROM users WHERE user_id = :user_id")
     Optional<User> findById(@Bind("user_id") int userId);

     // Usage
     User user = dao.findById(userId)
       .orElseThrow(() -> new NotFoundException("User not found"));
     ```

## Transaction Management

9. **Transaction Guidelines**
   - Annotate methods with `@Transactional` for atomic operations
   - Keep transaction boundaries narrow
   - Handle transaction rollbacks appropriately
   - Example:
     ```java
     @Transactional
     public void transferMoney(int fromAccount, int toAccount, BigDecimal amount) {
         accountDao.debit(fromAccount, amount);
         accountDao.credit(toAccount, amount);
     }
     ```

## SQL Conventions

10. **Database Standards**
    - Use `TEXT` for character-based data types (including enums)
    - Use `BIGSERIAL` (int in Java) for IDs instead of UUID or long
    - Avoid stored procedures and triggers
    - Use changesets in migration files
    - Keep SQL queries simple and readable with proper indentation
    - Example:
      ```sql
      CREATE TABLE users (
          user_id BIGSERIAL PRIMARY KEY,
          name TEXT NOT NULL,
          email_address TEXT NOT NULL,
          created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
      );
      ```

## SQL Migrations

11. **Migration Guidelines**
    - Use Liquibase for database migrations
    - Create migrations in `src/main/resources/migrations`
    - Follow naming convention:
      - `create_table_<table_name>.sql` for table creation
      - `alter_table_<table_name>_<alteration>.sql` for table alterations (e.g., `alter_table_customers_add_column_status.sql`)
    - Use Liquibase formatted SQL with changesets
    - **IMPORTANT**: Changeset IDs must be globally unique across all migrations to the database
    - Example:
      ```sql
      --liquibase formatted sql
      --changeset xbrett:create_table_customers
      CREATE
          TABLE
              customers(
                  id uuid NOT NULL DEFAULT uuid_generate_v4() CONSTRAINT customers_pk PRIMARY KEY,
                  address_id uuid CONSTRAINT customers_addresses_id_fk REFERENCES addresses,
                  user_id INTEGER,
                  first_name text,
                  last_name text,
                  email text,
                  phone text
              );
      ```

12. **Migration Best Practices**
    - Each changeset should be idempotent
    - Use meaningful descriptions
    - Test migrations locally before committing
    - Ensure changeset IDs are unique across all migration files
    - Always use `IF NOT EXISTS` when adding new columns or indexes to prevent errors
    - For index creation, use `runInTransaction: false` in the changeset comment
    - Create all indexes concurrently to avoid locking tables
    - Example:
      ```sql
      --liquibase formatted sql
      --changeset xbrett:alter_table_customers_add_column_status runInTransaction: false
      ALTER TABLE customers ADD COLUMN IF NOT EXISTS status text NOT NULL DEFAULT 'active';
      
      -- Create index concurrently to avoid locking
      CREATE INDEX CONCURRENTLY IF NOT EXISTS customers_statu_idx ON customers(status);
      ```

13. **Add Migration to migrations.xml**
    - After creating a migration file, add it to the `migrations.xml` file
    - Place the migration in the correct order based on dependencies
    - Include a descriptive comment explaining the purpose of the migration
    - Example:
      ```xml
      <?xml version="1.0" encoding="UTF-8" standalone="no"?>
      <databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                         xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">
          <!-- Existing migrations -->
          <include relativeToChangelogFile="true" file="migrations/create_table_customers.sql"/>
          <include relativeToChangelogFile="true" file="migrations/alter_table_customers_add_column_status.sql"/>
          <!-- Add your new migration here -->
          <include relativeToChangelogFile="true" file="migrations/alter_table_customers_add_index.sql"/>
      </databaseChangeLog>
      ```
    - Always add new migrations at the end of the file
    - Ensure the file path is correct relative to the `migrations.xml` file
    - Use the `relativeToChangelogFile="true"` attribute to ensure paths are resolved correctly
    - Always test the migration locally before deploying to DEV/PROD

## Best Practices

1. **Performance**
   - Use batch operations for bulk inserts/updates
   - Keep transactions as short as possible
   - Use appropriate indexes
   - Monitor query performance

2. **Maintenance**
   - Document complex queries
   - Keep SQL in migration files
   - Use consistent naming across related tables

3. **Security**
   - Use parameter binding to prevent SQL injection
   - Validate input before database operations
   - Use appropriate access controls
   - Follow principle of least privilege 



 ## Overview 

Describes the proper way to generate models that will be stored in and retrieved from our databases

## Database Pattern (Egg Pattern)

5. **Three-Interface Structure**
   - Core interface: Contains shared fields
   - Egg interface: Extends Core, adds builder
   - Full entity: Extends Core, adds DB fields

   Example implementation:
   ```java
   // Core interface - shared fields
   public interface UserCore {
       String getName();
       String getEmail();
   }

   // Egg interface - extends Core, adds builder
   @Value.Immutable
   @WhoopStyle
   public interface UserEgg extends UserCore {
       @ExcludeFromCoverageGeneratedReport
       class Builder extends ImmutableUserEgg.Builder {}

       @ExcludeFromCoverageGeneratedReport
       static Builder builder() {
           return new Builder();
       }
   }

   // Full entity - extends Core, adds DB fields
   @Value.Immutable
   @WhoopStyle
   public interface User extends UserCore {
       UUID getId();
       
       @JsonFormat(pattern = OFFSET_DATE_TIME_FORMAT)
       OffsetDateTime getCreatedAt();

       @JsonFormat(pattern = OFFSET_DATE_TIME_FORMAT)
       OffsetDateTime getUpdatedAt();

       @ExcludeFromCoverageGeneratedReport
       class Builder extends ImmutableUser.Builder {}

       @ExcludeFromCoverageGeneratedReport
       static Builder builder() {
           return new Builder();
       }
   }
   ```

## Best Practices

1. **Field Types**
   - Use appropriate types for IDs (UUID)
   - Use proper date/time types with formatting
   - Keep fields immutable and final

2. **Documentation**
   - Add `@Schema` annotations for field descriptions
   - Document any special serialization requirements 



 ## Overview

This guide outlines the layered backend architecture that decouples data access, business logic, and presentation to break up monolithic Resource classes and decouple clients from services' internal models. Following these rules ensures proper separation of concerns, improved testability, and decoupling between internal and external data models.

## Three-Tier Architecture

The BFF architecture consists of three distinct tiers, each with specific responsibilities:

1. **Data Tier**
   - Abstracts away data access and encapsulates implementation details
   - Handles data format, source system, and caching strategy concerns
   - Not responsible for making business decisions based on data
   - Concrete components: Repositories, Clients, DAOs
   - Example:
     ```java
     public class FeatureMetadataClient {
         public FeatureMetadata getFeatureMetadataById(final String id) {
             final LegacyFeatureMetadata metadata = apiCall("https://inventory.com/item/" + id);
             
             return FeatureMetadata.builder()
                     .setId(id)
                     .setName(metadata.getName())
                     .setCreatedAt(convertUglyDate(metadata.getCreatedAt()))
                     .setType(convertFeatureCode(metadata.getFeatureCode()))
                     .build();
         }
     }
     ```

2. **Business Logic Tier**
   - Uses data tier abstractions to gather data and make decisions
   - Enforces system constraints and business rules using Domain Model language
   - Handles product requirements and domain-specific logic
   - Concrete components: Services
   - Example:
     ```java
     public class FeatureService {
         private final FeatureMetadataClient metadataClient;
         private final FeatureDao featureDao;
         
         public FeatureModel getFeatureDomainModel(final FeatureRequest request) throws IOException {
             final FeatureMetadata metadata = metadataClient.get(request.getId());
             final Feature feature = featureDao.get(request.getId());
             
             if (isUnreleasedFeature(feature)) {
                 throw new UnreleasedFeatureException("This feature is experimental!");
             }
             
             return FeatureModel.builder().setMetadata(metadata).setFeature(feature).build();
         }
     }
     ```

3. **Presentation Tier**
   - Decouples clients from internal Domain Model
   - Renders display of internal models for external consumption
   - Handles presentation-only concerns (translations, formatting, etc.)
   - Ensures only relevant fields are returned to clients
   - Concrete components: Presenters
   - Example:
     ```java
     public class FeatureGraphPresenter implements Presenter<FeatureModel, FeatureResponse> {
         private final Translator translator;
         
         public FeatureGraphResponse present(final FeatureModel domainModel) throws IOException {
             String title = translator.getBundleString(MY_TITLE_KEY);
             String description = translator.getBundleString(MY_DESCRIPTION_KEY);
             
             return FeatureGraphResponse.builder()
                     .setTitle(title)
                     .setDescription(description)
                     .setXAxisLabel(domainModel.getMetadata().getName())
                     .build();
         }
     }
     ```

## Resource Class Coordination

4. **Resource Classes**
   - Should NOT contain business logic or generate response models
   - Coordinate the flow between tiers: request -> service -> presenter
   - Follow the pattern: receive request, call service, call presenter, return response
   - Example:
     ```java
     @GET
     @Path("/feature/{id}/graph")
     public FeatureResponse getFeature(@QueryParam("id") String id) throws IOException {
         final FeatureRequest request = FeatureRequest.builder().setId(id).build();
         final FeatureModel feature = featureService.getFeatureDomainModel(request);
         final FeatureGraphResponse response = featureGraphPresenter.present(feature);
         return response;
     }
     ```

## Best Practices

1. **Avoid Monolithic Classes**
   - Break up large Resource classes that intermix data access, business logic, and presentation
   - Separate concerns across the three tiers to improve testability and maintainability
   - Centralize data access to enable easier refactoring and parallelization

2. **Decouple Internal and External Models**
   - Never expose internal database models directly to API clients
   - Use presentation tier to transform domain models into client-safe responses

3. **Encapsulate Data Access Details**
   - Hide data source implementation details within the data tier
   - Abstract away data format conversions, caching strategies, and external API peculiarities
   - Make data tier components easily mockable for unit testing 



 ## Overview

This guide outlines the standards for creating and using immutable models in Java at WHOOP. Following these rules ensures consistent model implementation, proper serialization, and maintainable code.

## Style Guidelines

1. **Use WHOOP Style**
   - Always use `@WhoopStyle` for new immutable classes
   - Do not use `@HubspotStyle` or `@WhoopLegacyStyle`
   - Do not make any fields nullable. Instead, use java.util.Optional for values which might not be present.
   - Example:
     ```java
     @Value.Immutable
     @WhoopStyle
     @JsonSerialize(as = ImmutableUser.class)
     @JsonDeserialize(as = ImmutableUser.class)
     public interface User {
         String getName();
         String getEmail();
         Optional<String> getPhoneNumber();

         @ExcludeFromCoverageGeneratedReport
         class Builder extends ImmutableUser.Builder {}

         @ExcludeFromCoverageGeneratedReport
         static Builder builder() {
             return new Builder();
         }
     }
     ```

## Serialization

2. **Jackson Annotations**
   - Include both serialization and deserialization annotations
   - Ensures proper JSON handling
   - Example:
     ```java
     @Value.Immutable
     @WhoopStyle
     @JsonSerialize(as = ImmutableUser.class)
     @JsonDeserialize(as = ImmutableUser.class)
     public interface User {
         String getName();

         @ExcludeFromCoverageGeneratedReport
         class Builder extends ImmutableUser.Builder {}

         @ExcludeFromCoverageGeneratedReport
         static Builder builder() {
             return new Builder();
         }
     }
     ```

## Builder Pattern

3. **Standard Builder Implementation**
   - Provide a static builder method
   - Include inner Builder class
   - Mark with coverage exclusion
   - Example:
     ```java
     @Value.Immutable
     @WhoopStyle
     @JsonSerialize(as = ImmutableUser.class)
     @JsonDeserialize(as = ImmutableUser.class)
     public interface User {
         String getName();

         @ExcludeFromCoverageGeneratedReport
         class Builder extends ImmutableUser.Builder {}

         @ExcludeFromCoverageGeneratedReport
         static Builder builder() {
             return new Builder();
         }
     }
     ```

4. **Coverage Exclusions**
   - Always mark immutable builder code with `@ExcludeFromCoverageGeneratedReport`
   - Applies to both Builder class and builder() method
   - Prevents generated code from affecting coverage metrics 



 ## Logger Setup

1. **SLF4J Usage**
   - Always use SLF4J for logging
   - Import `org.slf4j.Logger` and `org.slf4j.LoggerFactory`
   - Example:
     ```java
     import org.slf4j.Logger;
     import org.slf4j.LoggerFactory;

     public class MyService {
         private static final Logger LOGGER = LoggerFactory.getLogger(MyService.class);
     }
     ```

2. **Logger Initialization**
   - Initialize loggers as private static final fields
   - Use the current class as the logger name
   - Example:
     ```java
     private static final Logger LOGGER = LoggerFactory.getLogger(CurrentClass.class);
     ```

## Logging Patterns

3. **Exception Logging**
   - Log exceptions appropriately with `LOGGER.error()`
   - Include both message and exception
   - Example:
     ```java
     try {
         // Some operation
     } catch (Exception e) {
         LOGGER.error("Failed to process request: {}", requestId, e);
         throw new ServiceException("Request processing failed", e);
     }
     ```

4. **Log Levels**
   - Use appropriate log levels:
     - ERROR: For errors that need immediate attention
     - WARN: For potentially harmful situations
     - INFO: For general operational information
     - DEBUG: For detailed information for debugging
     - TRACE: For most detailed information
   - Example:
     ```java
     LOGGER.error("Critical error: {}", errorMessage);
     LOGGER.warn("Warning: {}", warningMessage);
     LOGGER.info("Operation completed: {}", operationId);
     LOGGER.debug("Debug info: {}", debugInfo);
     LOGGER.trace("Trace info: {}", traceInfo);
     ```

5. **Contextual Logging**
   - Include relevant context in log messages
   - Use parameterized logging to avoid string concatenation
   - Example:
     ```java
     // Good
     LOGGER.info("User {} performed action {} with result {}", userId, action, result);

     // Bad - Avoid string concatenation
     LOGGER.info("User " + userId + " performed action " + action + " with result " + result);
     ```

## Best Practices

1. **Security**
   - Never log sensitive information (passwords, tokens, etc.)
   - Mask sensitive data in logs
   - Example:
     ```java
     // Good
     LOGGER.info("User {} accessed resource", userId);

     // Bad - Don't log sensitive data
     LOGGER.info("User {} accessed resource with token {}", userId, authToken);
     ```

3. **Maintenance**
   - Keep log messages consistent and meaningful
   - Use consistent formatting across the codebase
   - Include enough context for debugging
   - Example:
     ```java
     // Good - Consistent format with context
     LOGGER.info("[Operation={}] User {} completed task {} in {}ms", 
         operationName, userId, taskId, duration);

     // Bad - Inconsistent format, missing context
     LOGGER.info("Task completed"); 



 ## Overview

This guide outlines the standards for using models throughout the Java codebase. For rules on creating model interfaces and implementations, refer to the Java Immutables guide. Following these rules ensures consistent model usage patterns, proper data flow, and maintainable code.

## Using the Builder Pattern

1. **Always Use Builders for Creation**
   - Never create models using constructors directly
   - Always use the builder pattern for model creation
   - Example:
     ```java
     // Good
     User user = User.builder()
         .setId("123")
         .setName("John Doe")
         .setEmail("john@example.com")
         .build();
     
     // Bad - avoid direct construction, this does not work and will cause compilaion errors
     // UserModel user = new UserModel("123", "John Doe", "john@example.com");
     ```

2. **Fluent Builder Chaining**
   - Chain setter calls for readability
   - Set optional fields only when needed
   - Example:
     ```java
     Order order = Order.builder()
         .setOrderId("123")
         .setTotal(new BigDecimal("99.99"))
         .setItems(items)
         // Optional fields only when needed, do not box them in an Optional.of(...) call
         .setNotes("Rush delivery")
         .build();
     ```

## Modifying Existing Models

3. **Copy-and-Modify Pattern**
   - Always use the `from()` method to create a new instance from an existing model
   - Never try to modify immutable model fields directly
   - Example:
     ```java
     // Original model
     Customer customer = getCustomer();
     
     // Create modified version
     Customer updatedCustomer = Customer.builder()
         .from(customer)
         .setEmail("new-email@example.com")
         .build();
         
     // Never attempt to modify the original
     // customer.setEmail("new-email@example.com"); // Will not compile
     ```

## Model Traversal and Transformation

5. **Safe Field Access**
   - Always use getters to access model fields
   - Handle Optional fields safely
   - Example:
     ```java
     // Safe handling of optional fields
     String email = user.getEmail().orElseThrow(IllegalStateException.class);
     
     // Process only if present
     user.getEmail().ifPresent(emailAddress -> {
         sendVerificationEmail(emailAddress);
     });
     ```

6. **Model Transformations**
   - Create dedicated mappers or transformers for converting between model types
   - Keep transformation logic out of models themselves
   - Example:
     ```java
     // In a dedicated transformer class
     public class UserTransformer {
         public static UserDTO toDTO(UserModel model) {
             return UserDTO.builder()
                 .setId(model.getId())
                 .setName(model.getName())
                 .setEmail(model.getEmail().orElse(null))
                 .build();
         }
         
         public static User fromDTO(UserDTO dto) {
             return User.builder()
                 .setId(dto.getId())
                 .setName(dto.getName())
                 .setEmail(dto.getEmail())
                 .build();
         }
     }
     ```

## Collection Handling

7. **Immutable Collections**
   - Treat collection fields as immutable
   - Create new collections when modifying content
   - Example:
     ```java
     // Adding an item to a list in a model
     List<LineItem> updatedItems = new ArrayList<>(order.getItems());
     updatedItems.add(newItem);
     
     Order updatedOrder = Order.builder()
         .from(order)
         .setItems(updatedItems)
         .build();
     ```

8. **Collection Transformations**
   - Use Stream API for transforming collections of models
   - Example:
     ```java
     // Transform a list of models
     List<UserDTO> userDtos = users.stream()
         .map(UserTransformer::toDTO)
         .collect(Collectors.toList());
     ```

## Best Practices

1. **Consistency Across Layers**
   - Use consistent model types within a layer (e.g., API, service, data)
   - Transform models at layer boundaries
   - Example:
     ```java
     // In API layer - transforming API request to domain model
     UserModel user = UserTransformer.fromRequest(request);
     
     // Pass domain model to service layer
     UserModel savedUser = userService.createUser(user);
     
     // Transform back to API response at boundary
     return UserTransformer.toResponse(savedUser); 



 ## Overview

This guide outlines the standard Maven module organization and scope guidelines for Java projects at WHOOP. Following these rules ensures consistent project structure, clear module boundaries, and proper code organization.

## Module Organization

### Core Modules

1. **API Module** (`template-service-api`)
   - Contains API code and endpoints
   - Deployable service
   - Private and isolated scope
   - Example:
     ```java
     // template-service-api/src/main/java/com/whoop/template/api/
     public class TemplateResource {
         @GET
         @Path("/v1/templates")
         public Response getTemplates() {
             // API implementation
         }
     }
     ```

2. **Client Module** (`template-service-client`)
   - Public API client for external projects
   - Public scope
   - Never import in your own API module
   - Example:
     ```java
     // template-service-client/src/main/java/com/whoop/template/client/
     public class TemplateClient {
         public PaginatedRecord<Template> getTemplates(String nextToken, int limit) {
             // Client implementation
         }
     }
     ```

3. **Data Module** (`template-service-data`)
   - Data access layer and DAOs
   - Private and shareable within project
   - Example:
     ```java
     // template-service-data/src/main/java/com/whoop/template/data/
     public class TemplateDao {
         @SqlQuery("SELECT * FROM template where id = :id")
         public Optional<Template> findById(@Bind("id") UUID id) {
             // Database access
         }
     }
     ```

### Supporting Modules

4. **Integration Tests** (`template-service-int-test`)
   - Integration test suite
   - Deployable
   - Private and isolated scope
   - Uses client module for API access

5. **Jobs Module** (`template-service-jobs`)
   - Queue/Cron workers
   - Deployable
   - Private and isolated scope
   - Uses client module for API access

6. **Stream Module** (`template-service-stream`)
   - Kafka consumers
   - Deployable
   - Private and isolated scope
   - Uses client module for API access

### Shared Modules

7. **Models Module** (`template-service-models`)
   - Public immutable models
   - Public scope
   - Used across all modules
   - Example:
     ```java
     // template-service-models/src/main/java/com/whoop/template/models/
     @WhoopStyle
     @Value.Immutable
     public interface Template {
         UUID getId();
         String getName();
     }
     ```

8. **Common Module** (`template-service-common`)
   - Internal shared utilities
   - Private and shareable within project
   - Used by multiple internal modules

9. **Library Module** (`template-service-library`)
   - External logic without network calls
   - Public scope
   - Special use cases only:
     - WHOOP app component presentation logic
     - Preventing circular dependencies
     - Common abstract code for client call overrides

## Module Scope Guidelines

### Private and Isolated (Never Shared)
- API module
- Integration test module
- Jobs module
- Stream module

### Private and Shareable (Within Project Only)
- Data module
- Common module

### Public (Can Be Shared Externally)
- Models module
- Client module
- Library module

## Project Configuration

1. **Configuration Files**
   - Use `config/dropwizard` for Dropwizard API YAML files
   - Maintain proper `.gitignore`
   - Include comprehensive `README.md`

2. **Model Placement**
   - All public-facing models must go in the models module
   - Internal models can be placed in the data module

3. **Client Module Usage**
   - Jobs, stream, and integration tests must use API via client module
   - Move shared code to common module if needed
   - Never import your project's client module in your own API module

4. **Library Module Guidelines**
   - Default to exposing public logic via API module
   - Use only for logic that cannot be exposed over network boundaries 



 ## Overview

This guide outlines the standards for implementing translations and internationalization (i18n) in Java at WHOOP. Following these rules ensures consistent translation implementation, proper localization support, and maintainable multilingual code.

## Supported Languages

1. **Primary Languages**
   - **English (US)** - Default language (no locale suffix)
   - **Spanish** - `_es` suffix
   - **French** - `_fr` suffix 
   - **German** - `_de` suffix
   - **Italian** - `_it` suffix
   - **Portuguese** - `_pt` suffix

2. **Language Priority**
   - Always implement English first as the default
   - Ensure all other languages have complete translations
   - Never leave translation keys untranslated in non-English files

## Translation Bundle Structure

3. **TranslationBundleInfo Pattern**
   - Extend `TranslationBundleInfo` for type-safe translation access
   - Define enum for all translation keys
   - Bundle name should match properties file prefix
   - Example:
     ```java
     import com.whoop.apicommons.translation.cache.TranslationBundleInfo;

     public class MyFeatureBundleInfo extends TranslationBundleInfo {
         
         public MyFeatureBundleInfo() {
             super(Keys.values());
         }
         
         @Override
         public String getBundleName() {
             return "MyFeature";
         }
         
         public enum Keys {
             TITLE_TEXT,
             DESCRIPTION_TEXT,
             ACTION_BUTTON_TEXT,
             ERROR_MESSAGE_TEXT
         }
     }
     ```

## Properties File Format

4. **File Naming Convention**
   - Default English: `BundleName.properties`
   - Localized versions: `BundleName_[locale].properties`
   - Place all files in `src/main/resources/lang/`
   - Examples:
     - `MyFeature.properties` (English default)
     - `MyFeature_es.properties` (Spanish)
     - `MyFeature_fr.properties` (French)

5. **Properties File Structure**
   - Include header comment with language identifier
   - Group related strings together by UI context or feature area
   - Add comments that provide context about what the strings represent on screen
   - Use empty lines to separate logical groups
   - Include information about where strings appear in the UI
   - Example:
     ```properties
     # My Feature Strings - English (US)
     
     # Main screen header section - displayed at top of feature landing page
     TITLE_TEXT=Welcome to My Feature
     DESCRIPTION_TEXT=This feature provides amazing functionality.
     
     # Primary action buttons - shown in bottom action bar
     ACTION_BUTTON_TEXT=Get Started
     CANCEL_BUTTON_TEXT=Cancel
     
     # Error handling - displayed in modal dialogs and inline alerts
     ERROR_MESSAGE_TEXT=Something went wrong. Please try again.
     NETWORK_ERROR_TEXT=Unable to connect. Check your internet connection.
     
     # Onboarding flow - step-by-step introduction screens
     ONBOARDING_WELCOME_TITLE=Get Started with My Feature
     ONBOARDING_STEP1_BODY=First, we'll walk you through the basics.
     ONBOARDING_NEXT_BUTTON=Continue
     ```

## Key Naming Conventions

6. **Translation Key Format**
   - Use UPPER_SNAKE_CASE for all keys
   - Keys must exactly match enum values in BundleInfo class
   - Use descriptive, hierarchical naming
   - Examples:
     ```properties
     # Good examples
     TILE_ACTION_TEXT=Get Advanced Labs
     TILE_ONBOARDING_TITLE=Advanced labs is now available
     TILE_EMPTY_BODY=Get clinician-reviewed labs and actionable insights.
     ERROR_NETWORK_TIMEOUT=Network request timed out
     
     # Bad examples
     text1=Some text              # Not descriptive
     tileActionText=Get Labs      # Wrong case format
     tile_action=Get Labs         # Missing context suffix
     ```

7. **Key Naming Patterns**
   - Use context prefixes: `TILE_`, `MODAL_`, `BUTTON_`, `ERROR_`
   - Use content suffixes: `_TITLE`, `_BODY`, `_TEXT`, `_MESSAGE`
   - Be specific about UI element: `_ACTION_TEXT`, `_CANCEL_TEXT`
   - Group related keys with common prefixes

## Translation Content Guidelines

8. **Content Standards**
   - Keep English text concise and clear
   - Avoid technical jargon when possible
   - Use consistent terminology across all features
   - Include context in comments for translators
   - Example:
     ```properties
     # Button text for primary action (appears on blue button)
     TILE_ACTION_TEXT=GET ADVANCED LABS
     
     # Title shown when user first sees the feature (max 50 characters)
     TILE_ONBOARDING_TITLE=Advanced labs is now available
     ```

9. **Localization Considerations**
   - Account for text expansion in other languages (German ~30% longer)
   - Avoid concatenating translated strings
   - Use placeholders for dynamic content
   - Consider cultural differences in messaging tone

## File Organization

10. **Directory Structure**
    
    **Properties Files Location:**
    ```
    src/main/resources/lang/
    ├── MyFeature.properties                 # English (default)
    ├── MyFeature_es.properties              # Spanish
    ├── MyFeature_fr.properties              # French
    ├── MyFeature_de.properties              # German
    ├── MyFeature_it.properties              # Italian
    └── MyFeature_pt.properties              # Portuguese
    ```
    
    **Bundle Classes Location:**
    ```
    src/main/java/com/whoop/[feature]/common/strings/
    └── MyFeatureBundleInfo.java              # Bundle info class
    ```
    
    **Complete Project Structure Example:**
    ```
    advanced-labs-service-common/
    ├── src/main/java/com/whoop/advancedlabs/common/strings/
    │   └──  AdvancedLabsEntrypointBundleInfo.java
    └── src/main/resources/lang/
        ├──  AdvancedLabsEntrypoint.properties
        ├──  AdvancedLabsEntrypoint_es.properties
        ├──  AdvancedLabsEntrypoint_fr.properties
        ├──  AdvancedLabsEntrypoint_de.properties
        ├──  AdvancedLabsEntrypoint_it.properties
        └──  AdvancedLabsEntrypoint_pt.properties
    ```

11. **Bundle Organization**
    - One bundle per major feature or UI component
    - Keep bundles focused and cohesive
    - Place BundleInfo classes in a `[feature]/common/strings/` package
    - Use descriptive bundle names that match feature names
    - BundleInfo class name should match properties file prefix + "BundleInfo"

## Best Practices

1. **Development Workflow**
   - Always add English translations first
   - Update all language files when adding new keys
   - Validate enum keys match properties keys exactly
   - Test with different locales during development

2. **Code Integration**
   - Inject translation services where needed
   - Use type-safe enum keys, never string literals
   - Handle missing translations gracefully
   - Log warnings for missing translation keys 



 ## Overview

This guide outlines the standards for writing unit tests in Java at WHOOP. Following these rules ensures consistent test implementation, proper assertion usage, and maintainable test code.

## Test Method Naming

1. **Method Name Convention**
   - Follow the pattern: `methodName_setupForTestCase_expectedResult`
   - Start with the method being tested
   - Include setup context after first underscore
   - End with expected result after second underscore
   - Example:
     ```java
     @Test
     public void createUser_withValidEmail_returnsUser() {
         // Test implementation
     }

     @Test
     public void updateUser_withInvalidId_throwsNotFoundException() {
         // Test implementation
     }
     ```

## Assertions

2. **Use AssertJ**
   - Always use AssertJ assertions (`org.assertj.core.api.Assertions.assertThat()`)
   - Do not use JUnit assertions
   - Example:
     ```java
     // Good
     assertThat(result).isPresent();
     assertThat(result.get().getValue()).isEqualTo(expectedValue);

     // Bad - Don't use JUnit assertions
     Assert.assertNotNull(result);
     Assert.assertEquals(expectedValue, result.getValue());
     ```

3. **Assertion Types**
   - Use `AssertionsForClassTypes` for `assertThat` calls
   - Never use specialized assertion imports like:
     - `AssertionsForInterfaceTypes`
     - `Java6Assertions`
   - Example:
     ```java
     // Good
     import static org.assertj.core.api.AssertionsForClassTypes.assertThat;

     // Bad - Don't use these
     import static org.assertj.core.api.AssertionsForInterfaceTypes.assertThat;
     import static org.assertj.core.api.Java6Assertions.assertThat;
     ```

4. **Assert on Complete Objects**
   - Always assert on complete objects rather than individual fields
   - Create expected objects using builders
   - Example:
     ```java
     // Good
     WaitlistFooter expected = WaitlistFooter.builder()
         .setDescriptionText("Description")
         .setButtonText("Button")
         .build();
     assertThat(result).isEqualTo(expected);

     // Bad - Don't assert on individual fields
     assertThat(result.getDescriptionText()).isEqualTo("Description");
     assertThat(result.getButtonText()).isEqualTo("Button");
     ```

## Mock Verification

5. **Use Strict Stubbing for Mocks**
   - All test files that use Mockito mocks must declare the rule:
   - This enforces that all stubs are used and helps avoid silently broken tests
   - Example:
     ```java
     @Rule
     public MockitoRule rule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS);
     ```

6. **Mock Interactions**
   - Use `verify()` to check mock interactions
   - Verify only necessary interactions
   - Do not use the `eq` argument matcher if all arguments are `eq`. This is only needed when we are using other matchers in `doReturn` or `verify`.
   - Example:
     ```java
     @Test
     public void processOrder_withValidOrder_callsPaymentService() {
         // Arrange
         Order order = Order.builder().setEmail("test@whoop.com").build();

         // Act
         underTest.processOrder(order);

         // Assert
         verify(paymentService).processPayment(order);
     }
     ```

7. **Mock Setup**
   - Always use `doReturn().when()` over `when().thenReturn()` for mock setup
   - This provides better handling of method stubbing, especially for void methods
   - Never use `any()`, `anyString()`, `anyInt()`, etc for call verfications, use the exact value expected instead.
   - Example:
     ```java
     // Good
     doReturn("Translated Text").when(translator).getTranslation(trasnationId);

     // Bad - Don't use when().thenReturn()
     when(translator.getTranslation(translationId))).thenReturn("Translated Text");
     ```

8. **Translations in Tests**
   - Use real translations with US locale preferences when possible
   - Set up localization preferences with US locale using EASY_RANDOM
   - Example:
     ```java
     private static final LocalizationPreferences LOCALIZATION_PREFERENCES =
         EASY_RANDOM.nextObject(LocalizationPreferences.class)
             .withPreferredLanguages(Locale.US)
             .withFormattingLocale(Locale.US);

## Test Data

1. **EasyRandom Configuration**
   - Use EasyRandom for generating test data for models and final classes
   - Configure EasyRandom properly for immutables
   - Example:
     ```java
     private final EasyRandom EASY_RANDOM = new EasyRandom(new EasyRandomParameters()
            .scanClasspathForConcreteTypes(true)
            .ignoreRandomizationErrors(true));
     ```

2. **Prefer Randomized Instances Over Nulls in Tests**
   - Never hardcode `null` in tests
   - Use realistic data with:
     ```java
     EASY_RANDOM.nextObject(YourType.class);
     ```
   - You can inject them via a lambda when needed:
     ```java
     () -> YOUR_INSTANCE
     ```

3. **Working with Immutables**
   - Never mock Immubales or any model classes.
   - Always configure EasyRandom with:
     - `ignoreRandomizationErrors(true)` to handle immutable constraints
     - `scanClasspathForConcreteTypes(true)` to find concrete implementations
   - Example usage:
     ```java
     @Test
     public void processUser_withValidUser_returnsProcessedUser() {
         // Arrange
         User user = EASY_RANDOM.nextObject(User.class);

         // Act
         User result = underTest.processUser(user);

         // Assert
         assertThat(result).isNotNull();
         assertThat(result.getId()).isEqualTo(user.getId());
     }
     ```

3. **Test Data Best Practices**
   - Use EasyRandom for complex objects
   - Override specific fields when needed:
     ```java
     User user = EASY_RANDOM.nextObject(User.class);
     User modifiedUser = User.builder()
         .from(user)
         .setEmail("specific@whoop.com")
         .build();
     ```
   - Keep test data creation consistent across tests
   - Use meaningful values for required fields

4. **Test Data Fixtures**
- Rather than re-creating an Immutable model within every test, prefer to define it once within a test fixture.
- The test fixture should contain a static method named `builder` which returns the Immutable with default, testing values for all field.
- You can utilize `EASY_RANDOM` within the fixture where it makes sense (for example, to provide a complex field).

```java
public class UserTestFixture {
    public static User.Builder builder() {
        return User.builder()
            .setId(123L)
            .setName("Test User")
            .setEmail("test@whoop.com")
            .setIsActive(true);
    }

    public static User build() {
        return builder().build();
    }
}

// Example usage in a test:
@Test
public void testUserProcessing() {
    // Arrange
    User testUser = UserTestFixture.builder()
        .setEmail("override@whoop.com") // Override specific fields if needed
        .build();

    // Act & Assert
    // ... test logic using testUser
}
```

## Running Tests

1. **Maven Test Commands**
   - Use these commands to run tests efficiently
   - Skips linting, autoformatting, and other checks
   - Shows only test failures and stack traces

2. **Command Options**
   ```bash
   # Run all tests in module
   mvn -q test -pl <module-name> -Denforcer.skip -Dspotless.skip=true -Djacoco.skip=true -Dmaven.resources.skip=true -o -T1C -DforkCount=1 -DreuseForks=true -Dcheckstyle.skip

   # Run specific test class
   mvn -q test -pl <module-name> -Dtest=<TestClassName> -Denforcer.skip -Dspotless.skip=true -Djacoco.skip=true -Dmaven.resources.skip=true -o -T1C -DforkCount=1 -DreuseForks=true -Dcheckstyle.skip

   # Run specific test method
   mvn -q test -pl <module-name> -Dtest=<TestClassName>#<testMethodName> -Denforcer.skip -Dspotless.skip=true -Djacoco.skip=true -Dmaven.resources.skip=true -o -T1C -DforkCount=1 -DreuseForks=true -Dcheckstyle.skip
   ```

3. **Command Parameters**
   - `-q`: Quiet mode, reduces output
   - `-pl <module-name>`: Specifies the module to test
   - `-Dtest=<TestClassName>`: Specifies the test class to run
   - `-Dtest=<TestClassName>#<testMethodName>`: Specifies the test method to run
   - `-Denforcer.skip`: Skips enforcer checks
   - `-Dspotless.skip=true`: Skips code formatting
   - `-Djacoco.skip=true`: Skips code coverage
   - `-Dmaven.resources.skip=true`: Skips resource processing
   - `-o`: Offline mode
   - `-T1C`: Uses 1 thread per CPU core
   - `-DforkCount=1`: Uses 1 fork for tests
   - `-DreuseForks=true`: Reuses forks for tests
   - `-Dcheckstyle.skip`: Skips checkstyle

## Test Structure

5. **Standard Test Layout**
   - Follow clear setup, execution, and assertion pattern
   - Use consistent class structure
   - Example:
     ```java
     public class SomeServiceTest {
     
         // always use MockitoRule to instantiate mockito
         @Rule
         public MockitoRule rule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS);
     
         // EasyRandom instantiation
         private final EasyRandom EASY_RANDOM = new EasyRandom(new EasyRandomParameters()
             .scanClasspathForConcreteTypes(true)
             .ignoreRandomizationErrors(true));

         private final UUID ID = EASY_RANDOM.next(UUID.class);

         // Constants
         private static final String CONSTANT = "value";

         // Test objects
         private static final SomeClassEgg EGG = EASY_RANDOM.next(SomeClassEgg.builder().getClass())
             .setValue(CONSTANT)
             .build();
         
         // Mocks
         @Mock
         private SomeDao someDao;

         // Class under test
         private SomeService underTest;

         @Before
         public void setUp() {

             underTest = new SomeService(someDao);
             // Global mock setup
             doReturn(Optional.empty()).when(someDao).findById(ID);
         }

         @Test
         public void create_insertAnyNumber_returnsValueIsEqualToConstant() {
             // Test-specific mock setup
             doReturn(1).when(someDao).insert(EGG);

             // Execute
             Optional<SomeClass> result = underTest.create(CONSTANT);

             // Assert
             assertThat(result).isPresent();
             assertThat(result.get().getValue()).isEqualTo(CONSTANT);

             // Verify
             verify(someDao).insert(EGG);
         }
     }
     ```

## Best Practices

1. **Test Organization**
   - Group related tests together
   - Use descriptive constant names
   - Keep test objects immutable
   - Use meaningful variable names

2. **Mock Usage**
   - Mock only external dependencies
   - Use `@Mock` for dependencies
   - Initialize underTest class and set up common mock behavior in `@Before`

3. **Test Coverage**
   - Test happy path scenarios
   - Test error cases and edge conditions
   - Verify all important interactions
   - Keep tests focused and single-purpose

