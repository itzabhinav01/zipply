# Uber Clone Backend Documentation

This document provides an in-depth explanation of the backend architecture for this Uber Clone application. The backend is built with Node.js and the Express framework, following a structured and scalable Model-View-Controller (MVC) pattern, extended with a Service layer.

## Core Concepts

The backend is designed to be modular, with a clear separation of concerns. This makes the codebase easier to understand, maintain, and scale.

### How a Request is Handled (The Data Flow)

When the frontend application sends a request to the backend, it goes on a specific journey through the codebase:

1.  **`server.js`**: This is the main entry point. It starts the HTTP server and makes the application listen for incoming requests on a specific port.
2.  **`app.js`**: This file is the heart of the Express application. It loads all the necessary middleware (for things like CORS, parsing JSON, and handling cookies) and connects the routing files.
3.  **`/routes`**: This directory acts as the primary switchboard. Based on the request URL (e.g., `/users`, `/rides`), it directs the request to the appropriate controller function to handle the specific task.
4.  **`/middlewares`**: These are checkpoint functions. Before a request reaches its designated controller, it can pass through one or more middleware functions. A common use case is an `auth.middleware.js` which checks if the user is logged in and has permission to access the requested resource.
5.  **`/controllers`**: This is where the main business logic resides. The controller receives the request, validates the input, calls services to perform the necessary operations, and sends a response back to the client.
6.  **`/services`**: This directory contains reusable business logic. Instead of putting all the logic in the controller, complex operations (like creating a user or finding a nearby driver) are abstracted into services. This keeps controllers clean and logic reusable.
7.  **`/models`**: This directory defines the data structure (schema) for the application. For example, a `user.model.js` file defines that a user must have a name, email, and password. It also contains functions for interacting with the database for that specific data type (e.g., hashing passwords, generating tokens).
8.  **`/db`**: This folder contains the database connection logic.
9.  **`socket.js`**: This file manages real-time, two-way communication between the server and the client using WebSockets. This is essential for features like live location tracking of drivers on the map.

---

## Folder Structure Explained

-   **`server.js`**: Starts the server.
-   **`app.js`**: Configures and wires together the entire Express application.
-   **`.env`**: A file (not checked into version control) that stores environment variables like database connection strings and API keys.
-   **`/db`**: Contains the database connection setup file (`db.js`).
-   **`/routes`**: Defines the API endpoints. Each file (e.g., `user.routes.js`, `ride.routes.js`) groups related routes. These files link a specific URL path and HTTP method to a controller function.
-   **`/middlewares`**: Contains middleware functions. For example, `auth.middleware.js` protects routes by verifying authentication tokens.
-   **`/controllers`**: Contains the core logic for handling requests. For example, `user.controller.js` has functions like `registerUser`, `loginUser`, and `getUserProfile`.
-   **`/services`**: Contains abstracted business logic. For example, `user.service.js` might have a `createUser` function that the `user.controller` calls.
-   **`/models`**: Defines the Mongoose schemas for the database collections (e.g., `user.model.js`, `ride.model.js`). These schemas enforce data structure and can also contain helpful methods related to the data.

---

## Example Data Flow: User Registration

To make this concrete, let's trace the journey of a user registration request:

1.  **The Request:** The user fills out a registration form on the frontend, which sends a `POST` request to `/users/register` with the user's name, email, and password in the request body.

2.  **`app.js` -> `user.routes.js`:** `app.js` receives the request and, seeing the `/users` prefix, passes it to the `user.routes.js` file. This router matches the `/register` path and sees that it needs to call the `registerUser` function from the user controller. It may also have validation middleware that runs first to ensure the email is valid and the password is long enough.

3.  **`user.controller.js` (`registerUser` function):**
    a. It checks for any validation errors.
    b. It checks if a user with the given email already exists in the database to prevent duplicates.
    c. It securely hashes the plain-text password by calling a method from the `user.model.js`.
    d. It calls the `createUser` function in the `user.service.js`, passing the user data.
    e. Once the user is created, it generates a JSON Web Token (JWT) for authentication.
    f. It sends a `201 Created` response back to the frontend, including the new user's data and the JWT.

This structured flow ensures that each part of the application has a single, clear responsibility, making the backend robust, secure, and easy to manage.
