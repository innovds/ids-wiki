# REST API Standards and Norms

## URI Definition

- Use the plural form of the targeted object name regardless of the action: `/api/v1/cars` or `/api/v1/people`.
- Use hyphens (`-`) to separate words, not underscores (`_`). Avoid `camelCase`: **`/api/v1/gear-boxes`**.
- Use nouns in URIs; the HTTP method defines the action: **`/api/v1/cars`**.
- URIs should be in lowercase letters.
- Use versioning in URIs to handle breaking changes: **`/api/v1`**.

## HTTP Methods

- **POST**: Creates a new record. Returns `201 Created` and the URI of the new resource in the `Location` header. Not idempotent.
- **PUT**: Completely updates an object. Returns `200 OK`. Idempotent.
- **PATCH**: Partially updates an object. Returns `200 OK`. Idempotent.
- **GET**: Retrieves objects or lists. Returns `200 OK`. Must not modify records.
- **DELETE**: Deletes an object. Returns `204 No Content`.

## CRUD Actions

### Create a New Object

`POST /api/v1/cars`
- The object is passed in the request body (`@RequestBody Car car`).

### Complete Update of an Object
`PUT /api/v1/cars/{id}`
- The identifier is in the path (`@PathVariable Long id`).
- The updated object is in the request body (`@RequestBody Car car`).

### Partial Update of an Object
`PATCH /api/v1/cars/{id}`
- The identifier is in the path (`@PathVariable Long id`).
- The update fields are in the request body (`@RequestBody Car car` or specific fields).

### Delete an Object

`DELETE /api/v1/cars/{id}`
- The identifier is in the path (`@PathVariable Long id`).

### Get All Objects

`GET /api/v1/cars`
- Returns a list of cars. If none, returns an empty list with `200 OK`.

### Get Paged Objects

`GET /api/v1/cars?page=0&size=10`
- Handles pagination using `Pageable` and returns a `PagedModel`.

### Get an Object by ID

`GET /api/v1/cars/{id}`
- The identifier is in the path (`@PathVariable Long id`).
- Returns the object or `404 Not Found` if not found.

## Complex Cases

### Cars by Number of Seats

`GET /api/v1/cars-by-seats?seats=4`
- Filter by seats using query parameters. Returns a list with `200 OK`.

### Cars by Owner

#### Using Cars API

`GET /api/v1/cars-by-owner-id?ownerId=1`
- Filter by owner ID. Returns a list with `200 OK`.

#### Using Persons API

`GET /api/v1/persons/{id}/cars`
- Person ID is in the path (`@PathVariable Long id`). Returns `404 Not Found` if the person does not exist.

## View Usage

- Define lite objects for returning subsets of information.
- Use the lite object name in the path: `GET /api/v1/car-views/{id}`. Returns `404 Not Found` if the car is absent.

## Error Handling

- Standardize error responses: `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `500 Internal Server Error`.
- Use consistent error response structures with error codes and messages.

## Security Considerations

- Implement authentication and authorization (e.g., OAuth2, JWT).
- Ensure endpoints are secured appropriately.

## Quick Summary

- **Create**: `POST /api/v1/cars` with object in request body.
- **Read**:
    - `GET /api/v1/cars/{id}` for one row.
    - `GET /api/v1/cars` for all rows.
- **Update**:
    - `PUT /api/v1/cars/{id}` for complete update.
    - `PATCH /api/v1/cars/{id}` for partial update.
- **Delete**: `DELETE /api/v1/cars/{id}`.

_Keep these simple and readable rules in mind for CRUD operations._
