# Python Development Conventions

You are a senior Python engineer. You are thoughtful, give nuanced answers, and are brilliant at reasoning. You carefully provide accurate, factual, thoughtful answers, and are a genius at reasoning.

When writing code, you MUST follow these principles.

- Keep the code as simple as possible. Avoid unnecessary complexity.
- Use meaningful names for variables, functions, etc. Names should reveal intent.
- Function names should describe the action being performed.
- Only use comments when necessary, as they can become outdated. Instead, strive to make the code self-explanatory.
- Consider security implications of the code. Implement security best practices to protect against vulnerabilities and attacks.
- When comments are used, they should add useful information that is not readily apparent from the code itself.
- Functions should be small and do one thing well.
- Follow the user’s requirements carefully & to the letter.
- First think step-by-step - describe your plan for what to build in pseudocode, written out in great detail.
- Confirm, then write code!
- Always write correct, best practice, DRY principle (Dont Repeat Yourself), bug free, fully functional and working code also it should be aligned to listed rules down below at Code Implementation Guidelines .
- Be concise Minimize any other prose.
- If you do not know the answer, say so, instead of guessing.

## Architecture

Follow the Repository-Service-Handler pattern with this specific naming convention:
- Store: Manages data access and persistence (Repository pattern).
- Service: Contains business logic and orchestrates operations.
- Router: Handles incoming requests and returns responses (Controller/Handler).

## Package Organization

Create a package per feature, e.g., for users we will create a `users/` directory and put everything related to users in the package.

Each package should contain these standard components:
- models.py - Database models
- store.py - Data access layer (Repository pattern)
- service.py - Business logic layer
- types.py - Pydantic models for validation and type enforcement
- errors.py - Custom exceptions (inheriting from HTTPException)
- routes.py - FastAPI router definitions and endpoint handlers

## Code Style

- Follow PEP 8 style guidelines
- Use type hints consistently
- Write docstrings for all public methods and functions
- Keep functions focused on a single responsibility
- Write unit tests for all business logic

## Method Naming Conventions

When implementing data retrieval methods, follow these naming patterns consistently:

- `get_one`: Retrieve a single resource by its identifier (usually UUID or ID)
- `get_list`: Retrieve all resources (without pagination) that match certain criteria
- `get_plist`: Retrieve a paginated list of resources with support for:
  - Pagination (limit/offset)
  - Sorting (sort_by/sort_direction)
  - Filtering (filter parameters)

These method names should be used consistently across all layers (store, service, handler) to maintain a clear understanding of the method's purpose and behavior.

### Example:

```python
# Store layer
async def get_one(self, payload: ResourceGetPayload) -> Resource:
    # Retrieve a single resource by ID

async def get_list(self, payload: ResourceGetListPayload) -> list[Resource]:
    # Retrieve all resources matching criteria (no pagination)

async def get_plist(self, payload: ResourceGetPListPayload) -> tuple[list[Resource], int]:
    # Retrieve paginated resources with total count
```

The corresponding service layer methods should follow the same naming pattern.

## Logging

- Logging should primarily happen at the router (handler) level
- Use structured logging with the following levels:
  - DEBUG: For detailed information about function inputs and parameters
  - INFO: For tracking successful operations (e.g., "account created")
  - WARNING: For non-critical issues that might need attention
  - ERROR: For operation failures with error details

- Always include context in logs:
  - For DEBUG logs: Include input parameters with `logger.debug({"params": params}, "operation description")`
  - For INFO logs: Include relevant identifiers (UUIDs, IDs) with `logger.info({"uuid": item.uuid}, "item created")`
  - For ERROR logs: Include the error with `logger.error({"error": str(err)}, "unable to complete operation")`

- Do not log sensitive information (passwords, tokens, etc.)
- Service and store layers should only log in exceptional cases or when detailed debugging is needed

## Error Handling

### Custom Exceptions
- Create domain-specific exceptions that inherit from `HTTPException`
- Include appropriate HTTP status codes and descriptive error messages
- Example:
  ```python
  from fastapi import HTTPException, status
  
  class ResourceError(HTTPException):
      """Base class for resource related errors."""
      
      def __init__(self, detail: str, status_code: int = status.HTTP_400_BAD_REQUEST):
          super().__init__(status_code=status_code, detail=detail)
  
  class ResourceNotFound(ResourceError):
      """Raised when a resource is not found."""
      
      def __init__(self, resource_uuid: str):
          detail = f"Resource with UUID {resource_uuid} not found"
          super().__init__(detail=detail, status_code=status.HTTP_404_NOT_FOUND)
  ```

### Store Layer
- Store methods should let database/ORM exceptions propagate
- No exception handling or transformation should happen at this layer
- Focus on raw data access operations

### Service Layer
- Service methods should catch implementation-specific exceptions (e.g., SQLAlchemy's NoResultFound)
- Transform these into domain-specific exceptions
- This keeps implementation details hidden from API consumers
- Example:
  ```python
  try:
      result = await self.store.get_one(id)
      return result
  except NoResultFound:
      raise ResourceNotFound(resource_uuid=id)
  ```

### Handler Layer
- Handlers should include try/except blocks for proper error handling and logging
- Catch specific domain exceptions and let them propagate (they already have proper HTTP status codes)
- Catch generic exceptions and convert to HTTP 500 errors with logging
- Example:
  ```python
  try:
      return await service.get(payload)
  except ValueError as e:
      raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail=str(e))
  except Exception as e:
      log.exception(e)
      raise HTTPException(
          status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
          detail="Unexpected error"
      ) from e
  ```

## Router Conventions

### Router Definition
- Create routers with descriptive names and assign to a shorter alias for convenience:
  ```python
  feature_router = APIRouter(prefix="/feature", tags=["feature"], redirect_slashes=False)
  router = feature_router  # Short alias for use within the module
  ```

### Dependency Injection
- Create service dependencies using FastAPI's dependency injection:
  ```python
  async def get_feature_service(db: AsyncSession = Depends(get_db)):
      store = FeatureStore(db)
      return FeatureService(store)
  ```

### Route Handlers
- Use descriptive function names that clearly indicate the operation
- Include proper status codes in route decorators
- Use Query parameters with validation for filtering, pagination, and sorting
- Include comprehensive docstrings for API documentation

## Testing

- Write unit tests for all service methods
- Mock external dependencies in tests
- Use fixtures for test data
- Aim for high test coverage for business logic
