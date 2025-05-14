# REST API Design for Library Catalogue System (Books)

---

## Functional and Non-Functional Requirements

### Functional Requirements

1. **Book Management**:
   - The API must allow users to create, read, update, and delete books.
   - Users must be able to retrieve a list of books with filtering by title, author, genre, and publication year.
   - Users must be able to retrieve details of a specific book, including its authors and genres.

2. **Author Management**:
   - The API must support CRUD operations for authors.
   - Users must be able to retrieve a list of authors.
   - Users must be able to retrieve books associated with a specific author.

3. **Genre Management**:
   - The API must support CRUD operations for genres.
   - Users must be able to retrieve a list of genres.
   - Users must be able to retrieve books associated with a specific genre.

4. **Search and Filtering**:
   - The API must allow filtering books by:
     - Title (partial match)
     - Author ID
     - Genre ID
     - Publication year
   - Sorting must be supported on fields like title and publication date (ascending/descending).

5. **Pagination**:
   - All list endpoints must support pagination with parameters for page number and items per page.
   - The response must include pagination metadata (total items, current page, page size, next/previous page links).

6. **Relationship Navigation**:
   - The API must provide links to related resources using HATEOAS principles.

7. **Authentication**:
   - The API must require authentication for creating, updating, and deleting resources (books, authors, genres).
   - Public access must be allowed for reading operations.

### Non-Functional Requirements

1. **Performance**:
   - The API must respond to GET requests within 500ms under normal load.
   - Pagination must be implemented to limit the response size and improve performance for large datasets.

2. **Scalability**:
   - The API must handle up to 1,000 concurrent users without significant performance degradation.
   - The design must support horizontal scaling.

3. **Security**:
   - Authentication must use JSON Web Tokens (JWT) passed in the `Authorization` header.
   - The API must return appropriate error codes (401 Unauthorized, 403 Forbidden) for authentication/authorization failures.
   - Input validation must be enforced to prevent injection attacks.

4. **Reliability**:
   - The API must return meaningful status codes for all operations.
   - Error messages must be descriptive.

5. **Caching**:
   - GET endpoints must support caching using ETag headers for conditional requests.
   - Responses must include `Cache-Control` headers (e.g., `private, max-age=3600`) to cache responses for 1 hour, assuming moderate update frequency.

6. **Usability**:
   - The API must follow REST best practices, using intuitive endpoint names.
   - Responses must include HATEOAS links to related resources for easy navigation.

7. **Maintainability**:
   - The API design must use consistent conventions.
   - Documentation of endpoints, parameters, and responses must be clear.

8. **Interoperability**:
   - The API must use `application/json` as the content type for requests and responses.
   - It must adhere to HTTP standards and the Richardson Maturity Model (Level 3) for RESTful design.

---

## Description of Entities and Operations

### Entities

The library catalogue system revolves around the following key entities:

1. **Book**
   - **Attributes**:
     - `id`: Unique identifier (integer or UUID)
     - `title`: Title of the book (string)
     - `ISBN`: International Standard Book Number (string)
     - `publication_date`: Date of publication (ISO 8601 string)
     - `description`: Brief summary or abstract (string)
   - **Relationships**:
     - `authors`: Many-to-many relationship with Author
     - `genres`: Many-to-many relationship with Genre

2. **Author**
   - **Attributes**:
     - `id`: Unique identifier
     - `name`: Full name of the author (string)
     - `biography`: Short bio of the author (string)
   - **Relationships**:
     - `books`: Many-to-many relationship with Book

3. **Genre**
   - **Attributes**:
     - `id`: Unique identifier
     - `name`: Name of the genre (string)
     - `description`: Brief description of the genre (string)
   - **Relationships**:
     - `books`: Many-to-many relationship with Book

### Operations

The API supports the following operations:

- **Book Operations**:
  - **Create**: Add a new book (authenticated users only).
  - **Read**: Retrieve a list of books or details of a specific book, with filtering and pagination for lists.
  - **Update**: Modify an existing book’s details (authenticated users only).
  - **Delete**: Remove a book (authenticated users only).

- **Author Operations**:
  - **Create**: Add a new author (authenticated users only).
  - **Read**: Retrieve a list of authors or details of a specific author, with pagination for lists.
  - **Update**: Modify an author’s details (authenticated users only).
  - **Delete**: Remove an author (authenticated users only).
  - **Related Read**: Retrieve books written by a specific author.

- **Genre Operations**:
  - **Create**: Add a new genre (authenticated users only).
  - **Read**: Retrieve a list of genres or details of a specific genre, with pagination for lists.
  - **Update**: Modify a genre’s details (authenticated users only).
  - **Delete**: Remove a genre (authenticated users only).
  - **Related Read**: Retrieve books in a specific genre.

---

## REST API Description

### Base URL
All endpoints are prefixed with `/api`.

### Media Type
The API uses `application/json` for request and response bodies.

### Endpoints

#### Books

- **GET /api/books**
  - **Description**: Retrieves a paginated list of books with optional filtering and sorting.
  - **Query Parameters**:
    - `page`: Page number (integer, default: 1)
    - `size`: Items per page (integer, default: 20)
    - `sort`: Sort field and direction (e.g., `title,asc` or `publication_date,desc`)
    - `author_id`: Filter by author ID (integer)
    - `genre_id`: Filter by genre ID (integer)
    - `title`: Filter by title keyword (string, partial match)
    - `publication_year`: Filter by publication year (integer)
  - **Response**:
    - **200 OK**: Successfully retrieved the list.
      ```json
      {
        "data": [
          {
            "id": 1,
            "title": "Book One",
            "links": {
              "self": "/api/books/1"
            }
          },
          {
            "id": 2,
            "title": "Book Two",
            "links": {
              "self": "/api/books/2"
            }
          }
        ],
        "pagination": {
          "total": 100,
          "page": 1,
          "size": 20,
          "next": "/api/books?page=2&size=20",
          "prev": null
        }
      }
      ```
    - **400 Bad Request**: Invalid query parameters (e.g., negative page number).
      ```json
      {"error": "Invalid input", "details": ["page must be positive"]}
      ```
  - **Headers**:
    - `ETag`: Entity tag for caching
    - `Cache-Control`: `private, max-age=3600`

- **GET /api/books/{id}**
  - **Description**: Retrieves details of a specific book.
  - **Path Parameter**:
    - `id`: Book identifier
  - **Response**:
    - **200 OK**: Successfully retrieved the book.
      ```json
      {
        "id": 1,
        "title": "Book Title",
        "ISBN": "1234567890",
        "publication_date": "2020-01-01",
        "description": "A description",
        "authors": [
          {"id": 10, "name": "Author One", "links": {"self": "/api/authors/10"}}
        ],
        "genres": [
          {"id": 20, "name": "Fiction", "links": {"self": "/api/genres/20"}}
        ],
        "links": {
          "self": "/api/books/1",
          "authors": "/api/books/1/authors",
          "genres": "/api/books/1/genres"
        }
      }
      ```
    - **404 Not Found**: Book not found.
      ```json
      {"error": "Book not found"}
      ```
  - **Headers**:
    - `ETag`: For caching
    - `Cache-Control`: `private, max-age=3600`

- **POST /api/books**
  - **Description**: Creates a new book.
  - **Authentication**: Required (JWT in `Authorization` header)
  - **Request Body**:
    ```json
    {
      "title": "New Book",
      "ISBN": "1234567890",
      "publication_date": "2023-01-01",
      "description": "A new book",
      "author_ids": [10, 11],
      "genre_ids": [20]
    }
    ```
  - **Response**:
    - **201 Created**: Book created.
      - **Headers**: `Location: /api/books/1`
      - **Body**: Created book details (as in GET /books/{id})
    - **400 Bad Request**: Invalid input.
      ```json
      {"error": "Invalid input", "details": ["title is required"]}
      ```
    - **401 Unauthorized**: Missing or invalid token.
      ```json
      {"error": "Authentication required"}
      ```
    - **403 Forbidden**: User lacks permission.
      ```json
      {"error": "Permission denied"}
      ```

- **PUT /api/books/{id}**
  - **Description**: Updates an existing book.
  - **Authentication**: Required
  - **Path Parameter**:
    - `id`: Book identifier
  - **Request Body**: Same as POST, with updated fields
  - **Response**:
    - **200 OK**: Book updated, returns updated book.
    - **400 Bad Request**: Invalid input.
    - **401 Unauthorized**: Missing or invalid token.
    - **403 Forbidden**: Permission denied.
    - **404 Not Found**: Book not found.

- **DELETE /api/books/{id}**
  - **Description**: Deletes a book.
  - **Authentication**: Required
  - **Path Parameter**:
    - `id`: Book identifier
  - **Response**:
    - **204 No Content**: Book deleted.
    - **401 Unauthorized**: Missing or invalid token.
    - **403 Forbidden**: Permission denied.
    - **404 Not Found**: Book not found.

#### Authors

- **GET /api/authors**
  - **Description**: Retrieves a paginated list of authors.
  - **Query Parameters**: `page`, `size`, `sort` (e.g., `name,asc`)
  - **Response**:
    - **200 OK**: List of authors with pagination metadata (similar to GET /api/books).
  - **Headers**: `ETag`, `Cache-Control: private, max-age=3600`

- **GET /api/authors/{id}**
  - **Description**: Retrieves details of a specific author.
  - **Response**:
    - **200 OK**:
      ```json
      {
        "id": 10,
        "name": "Author One",
        "biography": "Bio text",
        "links": {
          "self": "/api/authors/10",
          "books": "/api/authors/10/books"
        }
      }
      ```
    - **404 Not Found**: Author not found.

- **POST /api/authors**
  - **Description**: Creates a new author.
  - **Authentication**: Required
  - **Request Body**:
    ```json
    {"name": "Author Name", "biography": "Bio text"}
    ```
  - **Response**:
    - **201 Created**: Author created.
    - **400 Bad Request**: Invalid input.
    - **401 Unauthorized**, **403 Forbidden**: Authentication/permission issues.

- **PUT /api/authors/{id}**
  - **Description**: Updates an author.
  - **Authentication**: Required
  - **Response**: Similar to POST (200 OK on success).

- **DELETE /api/authors/{id}**
  - **Description**: Deletes an author.
  - **Authentication**: Required
  - **Response**: 204 No Content on success, 404 Not Found if author doesn’t exist.

- **GET /api/authors/{id}/books**
  - **Description**: Retrieves a paginated list of books by the author.
  - **Query Parameters**: `page`, `size`
  - **Response**:
    - **200 OK**: List of books with pagination metadata.
    - **404 Not Found**: Author not found.

#### Genres

- **GET /api/genres**
  - **Description**: Retrieves a paginated list of genres.
  - **Query Parameters**: `page`, `size`, `sort` (e.g., `name,asc`)
  - **Response**:
    - **200 OK**: List of genres with pagination metadata.
  - **Headers**: `ETag`, `Cache-Control: private, max-age=3600`

- **GET /api/genres/{id}**
  - **Description**: Retrieves details of a specific genre.
  - **Response**:
    - **200 OK**:
      ```json
      {
        "id": 20,
        "name": "Fiction",
        "description": "Fictional works",
        "links": {
          "self": "/api/genres/20",
          "books": "/api/genres/20/books"
        }
      }
      ```
    - **404 Not Found**: Genre not found.

- **POST /api/genres**
  - **Description**: Creates a new genre.
  - **Authentication**: Required
  - **Request Body**:
    ```json
    {"name": "New Genre", "description": "Genre description"}
    ```
  - **Response**:
    - **201 Created**: Genre created.
    - **400 Bad Request**: Invalid input.
    - **401 Unauthorized**, **403 Forbidden**: Authentication/permission issues.

- **PUT /api/genres/{id}**
  - **Description**: Updates a genre.
  - **Authentication**: Required
  - **Response**: Similar to POST (200 OK on success).

- **DELETE /api/genres/{id}**
  - **Description**: Deletes a genre.
  - **Authentication**: Required
  - **Response**: 204 No Content on success, 404 Not Found if genre doesn’t exist.

- **GET /api/genres/{id}/books**
  - **Description**: Retrieves a paginated list of books in the genre.
  - **Query Parameters**: `page`, `size`
  - **Response**:
    - **200 OK**: List of books with pagination metadata.
    - **404 Not Found**: Genre not found.

### Additional Features

- **Authentication**:
  - Uses JSON Web Tokens (JWT) in the `Authorization` header (e.g., `Bearer <token>`).
  - Required for POST, PUT, and DELETE operations.
  - Public GET operations do not require authentication.

- **Error Handling**:
  - **400 Bad Request**: Invalid input with details in response body.
  - **401 Unauthorized**: Missing or invalid authentication.
  - **403 Forbidden**: Insufficient permissions.
  - **404 Not Found**: Resource not found.
  - **500 Internal Server Error**: Unexpected server issues.

- **Pagination**:
  - Applied to all list endpoints using `page` and `size` parameters.
  - Response includes `pagination` object with `total`, `page`, `size`, `next`, and `prev` links.

- **Filtering**:
  - Supported on GET /api/books with parameters like `author_id`, `genre_id`, `title`, and `publication_year`.

- **Caching**:
  - **ETag**: Included in GET responses for conditional requests (e.g., `If-None-Match`).
  - **Cache-Control**: Set to `private, max-age=3600` (1 hour) for GET responses.

- **Richardson Maturity Model**:
  - **Level 2**: Proper use of HTTP verbs and status codes.
  - **Level 3**: HATEOAS implemented via `links` in responses.
