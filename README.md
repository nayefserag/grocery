<h1 align="center"> Grocery and Order Management Microservice</h1>
<p align="center">
  <a href="http://nestjs.com/" target="blank">
    <img src="logo.jpg" width="500" alt="Nest Logo" style="border-radius: 15px;" />
  </a>
</p>


This service is part of a microservices architecture and handles grocery item management, grocery lists, and orders. It communicates with the **Auth Service** to validate user tokens and interacts with the **Notification Service** to send order-related notifications. It provides REST APIs to manage items, grocery lists, and orders. The service also integrates with RabbitMQ for asynchronous message handling and utilizes JWT-based authentication for securing the API.

## Key Features

- **Grocery Management**: Create, update, delete, and retrieve grocery items and grocery lists.
- **Order Management**: Create, update, and manage customer orders, including updating order statuses.
- **Communication with Auth Service**: Validates user tokens by communicating with the **Auth Service** to ensure secure access to the API endpoints.
- **Communication with Notification Service**: Sends order-related notifications (e.g., order confirmation) by communicating with the **Notification Service** through RabbitMQ and HTTP calls.
- **RabbitMQ Integration**: Facilitates asynchronous message handling for inter-service communication.
- **JWT Authentication**: Ensures that all endpoints are secured, allowing only authenticated users to interact with the service.
- **Structured Logging and Error Tracking**: Utilizes Winston for logging and Sentry for error tracking in production environments.

## Table of Contents

1. [Technologies Used](#technologies-used)
2. [File Structure](#file-structure)
3. [Inter-Service Communication](#inter-service-communication)
4. [RabbitMQ Integration](#rabbitmq-integration)
5. [Running the Service](#running-the-service)
6. [Endpoints](#endpoints)
7. [Development](#development)

---

### Technologies Used

- **NestJS**: A framework for building efficient, scalable Node.js server-side applications.
- **RabbitMQ**: Used for message queueing, facilitating asynchronous event-driven communication.
- **MySQL**: Relational database to store data for items, orders, and users.
- **TypeORM**: ORM for working with relational databases.
- **MongoDB/Mongoose**: (Optional) Used for storing unstructured data.
- **JWT (JSON Web Tokens)**: Secures the API endpoints by authenticating users.
- **Winston**: Logging library used for structured logging.
- **Sentry**: Error monitoring and performance tracking tool.

### File Structure

```plaintext
|-- README.md
|-- combined.log
|-- debug.log
|-- error.log
|-- nest-cli.json
|-- package-lock.json
|-- package.json
|-- postman_collection.json
|-- src
|   |-- app
|   |   |-- modules
|   |   |   |                               `-- api
|   |   |   |   |                           `-- api.module.ts
|   |   |   |   |                           `-- item
|   |   |   |   |   |                       `-- controller
|   |   |   |   |   |                       `-- item.controller.ts
|   |   |   |   |   `-- item.module.ts
|   |   |   |   `-- order
|   |   |   |       |-- controller
|   |   |   |       |                       `-- order.controller.ts
|   |   |   |       `-- order.module.ts
|   |   |   |-- application
|   |   |   |   |-- application.module.ts
|   |   |   |   |-- item
|   |   |   |   |   |-- model
|   |   |   |   |   |   |                   `-- item.dto.ts
|   |   |   |   |   |                       `-- list.dto.ts
|   |   |   |   |   `-- services
|   |   |   |   |       |                   `-- item.service.ts
|   |   |   |   |                           `-- list-items.service.ts
|   |   |   |   `-- orders
|   |   |   |       |-- model
|   |   |   |       |                       `-- order.dto.ts
|   |   |   |       `-- services
|   |   |   |                               `-- order.service.ts
|   |   |   |-- database
|   |   |   |   |-- data-source.ts
|   |   |   |   `-- database.module.ts
|   |   |   `-- infrastructure
|   |   |       |-- communicator
|   |   |       |   |-- communicator.module.ts
|   |   |       |   `-- notification.communicator.ts
|   |   |       |-- entites
|   |   |       |   |-- item.entity.ts
|   |   |       |   |-- list.item.entity.ts
|   |   |       |   `-- order.entity.ts
|   |   |       |-- infrastructure.module.ts
|   |   |       |-- repositories
|   |   |       |   |-- item
|   |   |       |   |                       `-- item.repository.ts
|   |   |       |   |-- list
|   |   |       |   |                       `-- items-list.repositry.ts
|   |   |       |   `-- order
|   |   |       |                           `-- order.repositry.ts
|   |   |       `-- sentry
|   |   |           `-- instrument.ts
|   |   |-- rabbitMQ
|   |   |   |-- connector.ts
|   |   |   |-- consumer-options.ts
|   |   |   |-- exchange.definition.ts
|   |   |   |-- queue-consumer.decorator.ts
|   |   |   |-- queue.difinition.ts
|   |   |   |-- rabbit-mq-consumer.ts
|   |   |   |-- rabbit-mq-publisher.ts
|   |   |   `-- rabbit-mq.module.ts
|   |   `-- shared
|   |       |-- enums
|   |       |   `-- order-status.enum.ts
|   |       |-- interfaces
|   |       |   `-- IEnvConfigInterface.ts
|   |       |-- module
|   |       |   |-- config-module
|   |       |   |   |-- config.module.ts
|   |       |   |   `-- config.service.ts
|   |       |   `-- guards
|   |       |       `-- jwt.guard.ts
|   |       `-- utils
|   |           |-- generator.helper.ts
|   |           `-- hash.helper.ts
|   |-- app.module.ts
|   `-- main.ts
|-- test
|   |-- app.e2e-spec.ts
|   `-- jest-e2e.json
|-- tsconfig.build.json
|-- tsconfig.json
`-- warn.log

```

### Inter-Service Communication

This service communicates with two other microservices to ensure smooth operations:

1. **Auth Service**: 
   - Before processing any request, the service communicates with the **Auth Service** to validate the JWT tokens provided by users. This ensures that only authenticated users can access or modify grocery items, lists, and orders.
   - This communication occurs through **HTTP requests** to the Auth Service’s validation endpoints.

2. **Notification Service**:
   - Once an order is created or updated, the service interacts with the **Notification Service** to send order-related notifications (e.g., order confirmation emails).
   - This communication is achieved through two methods:
     - **RabbitMQ** for asynchronous, event-driven notifications.
     - **HTTP calls** for direct, synchronous interactions when needed.

### RabbitMQ Integration

The service integrates with **RabbitMQ** for handling asynchronous communication between microservices. It listens to messages for events related to grocery items and orders. This allows the service to handle tasks like order processing in the background without blocking HTTP requests.

- **Order Events**: The service consumes messages from a RabbitMQ queue for order events such as order placement and status updates.
- **Grocery Events**: Grocery-related actions like list creation and item management can also be triggered by RabbitMQ events.

RabbitMQ configuration and consumers are managed in the `src/app/rabbitMQ/` directory, with handlers for different message types and queues.

### Running the Service

#### Prerequisites

- **Node.js**
- **MySQL**
- **RabbitMQ**
- **MongoDB** (if applicable)
- **Docker** (optional for containerized setup)

#### Steps

1. Install dependencies:
    ```bash
    npm install
    ```

2. Set up environment variables (e.g., `.env`):
    ```plaintext
    DATABASE_URL=mysql://user:password@localhost:3306/grocery_service
    RABBITMQ_URL=amqp://localhost
    AUTH_SERVICE_URL=http://auth-service-url/validate-token
    NOTIFICATION_SERVICE_URL=http://notification-service-url/
    JWT_SECRET=your_jwt_secret
    ```

3. Run the service:
    ```bash
    npm start
    ```

4. Run in development mode with live-reloading:
    ```bash
    npm run start:dev
    ```

5. For production build:
    ```bash
    npm run build
    npm run start:prod
    ```

### Endpoints

#### Grocery Management

- **POST** `/grocery/items`: Create a new grocery item.
- **GET** `/grocery/items`: Get all grocery items.
- **GET** `/grocery/items/:id`: Get a specific grocery item by ID.
- **PATCH** `/grocery/items/:id`: Update a grocery item by ID.
- **DELETE** `/grocery/items/:id`: Delete a grocery item by ID.

- **POST** `/grocery/lists`: Create a new grocery list.
- **GET** `/grocery/lists`: Get all grocery lists.
- **GET** `/grocery/lists/:id`: Get a specific grocery list by ID.
- **PATCH** `/grocery/lists/:id`: Update a grocery list by ID.
- **DELETE** `/grocery/lists/:id`: Delete a grocery list by ID.

#### Order Management

- **POST** `/orders`: Create a new order.
- **GET** `/orders`: Get all orders.
- **GET** `/orders/:id`: Get a specific order by ID.
- **PATCH** `/orders/:id/status`: Update the status of an order.
- **DELETE** `/orders/:id`: Delete an order by ID.

### Development

- **Testing**: Unit and e2e tests are configured using Jest.
    ```bash
    npm run test
    ```

- **Linting**: Use ESLint to check for code quality and formatting.
    ```bash
    npm run lint
    ```

- **Docker**: You can also run the service in Docker:
    ```bash
    docker build -t grocery-service .
    docker run -p 3000:3000 grocery-service
    ```

---
