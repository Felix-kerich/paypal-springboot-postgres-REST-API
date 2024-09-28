# PayPal REST API with Spring Boot and PostgreSQL

This project integrates PayPal's REST API with a Spring Boot application, allowing users to make payments via PayPal and save payment details to a PostgreSQL database. The project is containerized using Docker, and also includes pgAdmin for managing the PostgreSQL database.

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Environment Variables](#environment-variables)
  - [Running with Docker](#running-with-docker)
  - [Spring Boot Configuration](#spring-boot-configuration)
- [API Endpoints](#api-endpoints)
- [Database Schema](#database-schema)
- [Docker Setup](#docker-setup)
- [PayPal Configuration](#paypal-configuration)
- [Project Structure](#project-structure)
- [Contributing](#contributing)

## Features

- **PayPal Integration**: Accept payments using PayPal.
- **Save Payment Details**: After successful payments, transaction details are stored in PostgreSQL.
- **Dockerized**: Easy setup using Docker and Docker Compose with PostgreSQL and pgAdmin.
- **Cross-Origin Requests**: CORS configuration for frontend communication (default for `http://localhost:3000`).

## Tech Stack

- **Backend**: Java 17, Spring Boot 3
- **Payment API**: PayPal REST API
- **Database**: PostgreSQL
- **Docker**: Docker & Docker Compose
- **Database Management**: pgAdmin

## Getting Started

### Prerequisites

Ensure you have the following installed on your machine:
- **Docker & Docker Compose**
- **Java 17+**
- **Maven**

### Environment Variables

Create an `.env` file in the root directory to provide the necessary environment variables:

```bash
# PayPal API credentials
PAYPAL_CLIENT_ID=your-paypal-client-id
PAYPAL_CLIENT_SECRET=your-paypal-client-secret
PAYPAL_MODE=sandbox # Use 'live' for production

# pgAdmin credentials
PGADMIN_DEFAULT_EMAIL=pgadmin@pgadmin.org
PGADMIN_DEFAULT_PASSWORD=pgadmin

# Database credentials
POSTGRES_USER=username
POSTGRES_PASSWORD=password
```

## Running with Docker

1. **Clone the repository**:

    ```bash
    git clone https://github.com/your-username/paypal-springboot-postgres.git
    cd paypal-springboot-postgres-REST-API
    ```

2. **Build the Spring Boot application**:

    ```bash
    mvn clean package
    ```

3. **Run Docker containers**:

    ```bash
    docker-compose up --build
    ```

    This will start:
    - **PostgreSQL** on port `5432`
    - **pgAdmin** on port `5040` (Accessible at [http://localhost:5040](http://localhost:5040))

4. **Access pgAdmin**:
   - Open [http://localhost:5040](http://localhost:5040) in your browser.
   - Login using the email and password from the `.env` file.
   - Add a new server with the following details:
     - **Host**: `postgres`
     - **Port**: `5432`
     - **Username**: The value of `POSTGRES_USER`
     - **Password**: The value of `POSTGRES_PASSWORD`

## Spring Boot Configuration

Add the following settings in your `application.yml` file located in `src/main/resources`:

```yaml
spring:
  application:
    name: paypal-rest-api
  jackson:
    property-naming-strategy: SNAKE_CASE  
  datasource:
    url: jdbc:postgresql://localhost:5432/your_database_name
    username: username
    password: password
  jpa:
    properties:
      hibernate:
        jdbc:
          lob:
            non_contextual_creation: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
    show-sql: true
    hibernate:
      ddl-auto: update  

paypal:
  client-id: your_paypal_client_id
  client-secret: your_paypal_client_secret
  mode: sandbox  # Change to 'live' for production

logging:
  level:
    org.springframework.security: DEBUG

server:
  port: 8081
```

Replace the placeholders with your actual PayPal credentials and database information.

## API Endpoints

### Create PayPal Payment

- **Endpoint**: `POST /api/paypal/payment/create`
- **Request Body**:

```json
{
  "amount": 10.0,
  "currency": "USD",
  "method": "paypal",
  "description": "TestPayment"
}
```

- **Response**: A URL for payment approval on PayPal.

### Execute PayPal Payment

- **Endpoint**: `POST /api/paypal/payment/success?paymentId={paymentId}&PayerID={payerId}`
- Executes the payment after PayPal approval.

### Cancel Payment

- **Endpoint**: `GET /api/paypal/payment/cancel`
- Returns a message indicating the payment was canceled.

## Database Schema

Below is a sample schema for storing payment details in PostgreSQL based on the `PaymentEntity` class:

```sql
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    paypal_payment_id VARCHAR(255) NOT NULL,
    intent VARCHAR(50) NOT NULL,
    state VARCHAR(50) NOT NULL,
    cart VARCHAR(255),
    payment_method VARCHAR(50) NOT NULL,
    payer_status VARCHAR(50) NOT NULL,
    payer_id VARCHAR(255) NOT NULL,
    payer_email VARCHAR(255) NOT NULL,
    payer_first_name VARCHAR(100) NOT NULL,
    payer_last_name VARCHAR(100) NOT NULL,
    currency VARCHAR(10) NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    transaction_id VARCHAR(255),
    transaction_state VARCHAR(50),
    transaction_fee DECIMAL(10, 2),
    payment_mode VARCHAR(50),
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    update_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

```

This schema can be modified based on additional payment details you'd like to store.

## Docker Setup

Here's the `docker-compose.yml` used to set up PostgreSQL and pgAdmin:

```yaml
version: '3.8'

services:
  postgres:
    container_name: ms_ps_sql_paypal_payment
    image: postgres
    environment:
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
      PGDATA: /var/lib/postgresql/data
    volumes:
      - postgres:/var/lib/postgresql/data
    ports:
      - 5432:5432
    networks:
      - microservices-net
    restart: unless-stopped

  pgadmin:
    container_name: ms_pgadmin_paypal_payment
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-pgadmin@pgadmin.org}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-pgadmin}
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    ports:
      - 5040:80
    networks:
      - microservices-net
    restart: unless-stopped

networks:
  microservices-net:
    driver: bridge

volumes:
  postgres:
  pgadmin:
```

## PayPal Configuration

To configure PayPal, follow these steps:

1. Log in to your [PayPal Developer Dashboard](https://developer.paypal.com/).
2. Create a new application to get your `client-id` and `client-secret`.
3. Set your mode to `sandbox` for testing or `live` for production.
4. Update your `application.yml` with these credentials.

## Project Structure

```bash
src
├── main
│   ├── java
│   │   └── com
│   │       └── payment
│   │           └── paypal_rest_api
│   │               ├── config   # CORS and security configurations
│   │               ├── controller  # REST controllers
│   │               ├── service  # PayPal service layer
│   │               └── model  # Entity classes for storing data
│   └── resources
│       ├── application.yml  # Spring Boot configuration
│       └── templates  # HTML templates (if needed)
└── test
    └── java
        └── com
            └── payment
                └── paypal_rest_api
                    └── controller  # Unit tests for controllers
```

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a new branch.
3. Make your changes and commit them.
4. Push to your branch.
5. Create a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
