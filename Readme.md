# Payment Processing System

A microservices-based payment processing system that handles payment transactions, integrates with Stripe, and manages payment status updates through webhooks.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Technologies](#technologies)
- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Frontend Integration](#frontend-integration)
- [Testing](#testing)
- [Deployment](#deployment)
- [Security](#security)

## 🎯 Overview

This system provides a complete payment processing solution with:

- **Transaction Management**: Create and track payment transactions
- **Stripe Integration**: Secure payment processing through Stripe
- **Webhook Handling**: Automatic payment status updates
- **Database Persistence**: MySQL-based data storage
- **Microservices Architecture**: Scalable and maintainable design

### Payment Flow

```
1. Frontend → Payment Processing Service (Create Transaction)
2. Payment Processing Service → Stripe Provider Service (Initiate Payment)
3. Stripe Provider Service → Stripe API (Create Checkout Session)
4. User → Stripe Checkout Page (Complete Payment)
5. Stripe → Stripe Provider Service (Webhook Notification)
6. Stripe Provider Service → Payment Processing Service (Update Status)
7. Payment Processing Service → Database (Store Final Status)
```

## 🏗️ Architecture

### Microservices

#### 1. Payment Processing Service (Port: 8080)

Core service managing payment transactions and database operations.

```
payment-processing-service/
├── src/main/java/com/hulkhiretech/payments/
│   ├── config/              # Application configuration
│   ├── constant/            # Enums and constants
│   ├── controller/          # REST API controllers
│   ├── dao/                 # Data access layer
│   ├── dto/                 # Data transfer objects
│   ├── entity/              # Database entities
│   ├── exception/           # Exception handling
│   ├── service/             # Business logic
│   └── util/                # Utility classes
├── src/main/resources/
│   └── application.properties
└── pom.xml
```

#### 2. Stripe Provider Service (Port: 8081)

Handles all Stripe API communications and webhook processing.

```
stripe-provider-expire-session/
├── src/main/java/com/hulkhiretech/payments/
│   ├── config/              # Application configuration
│   ├── controller/          # REST API controllers
│   ├── service/             # Business logic
│   ├── stripe/              # Stripe models
│   ├── http/                # HTTP client utilities
│   └── util/                # Utility classes
├── src/main/resources/
│   └── application.properties
└── pom.xml
```

## 💻 Technologies

### Backend Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Java | 21 | Programming language |
| Spring Boot | 3.4+ | Application framework |
| Maven | 3.6+ | Build tool |
| MySQL | 8+ | Database |
| JDBC Template | - | Data access |
| ModelMapper | - | Object mapping |
| Lombok | - | Code generation |
| JUnit 5 | - | Testing |
| Mockito | - | Mocking framework |

### Payment Integration

- **Stripe API**: Payment processing
- **Stripe Java SDK**: Webhook validation
- **RestClient**: HTTP communication

## 🚀 Getting Started

### Prerequisites

- Java 21 JDK installed
- Maven 3.6+ installed
- MySQL 8+ running
- Stripe account with API keys

### Database Setup

**1. Create Database**

```sql
CREATE DATABASE payments;
```

**2. Create Transaction Table**

```sql
CREATE TABLE payments.Transaction (
    id INT AUTO_INCREMENT PRIMARY KEY,
    userId INT NOT NULL,
    paymentMethodId INT NOT NULL,
    providerId INT NOT NULL,
    paymentTypeId INT NOT NULL,
    txnStatusId INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    merchantTransactionReference VARCHAR(255),
    txnReference VARCHAR(255) UNIQUE NOT NULL,
    providerReference VARCHAR(255),
    errorCode VARCHAR(50),
    errorMessage VARCHAR(255),
    creationDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    retryCount INT DEFAULT 0,
    INDEX idx_txnReference (txnReference),
    INDEX idx_userId (userId)
);
```

### Configuration

#### Payment Processing Service

Create `src/main/resources/application-local.properties`:

```properties
# Server Configuration
server.port=8080

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/payments
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Stripe Provider Configuration
stripe-provider.create.payment.url=http://localhost:8081/payments

# Logging Configuration
logging.level.root=INFO
logging.level.com.hulkhiretech.payments=DEBUG
```

#### Stripe Provider Service

Create `src/main/resources/application-local.properties`:

```properties
# Server Configuration
server.port=8081

# Stripe API Configuration
stripe.api.key=sk_test_your_stripe_secret_key
stripe.create.session.url=https://api.stripe.com/v1/checkout/sessions
stripe.get.session.url=https://api.stripe.com/v1/checkout/sessions/{id}
stripe.expire.session.url=https://api.stripe.com/v1/checkout/sessions/{id}/expire
stripe.webhook.secret=whsec_your_webhook_secret

# Processing Service Configuration
processing.notification.url=http://localhost:8080/v1/payments/notifications

# Logging Configuration
logging.level.root=INFO
logging.level.com.hulkhiretech.payments=DEBUG
```

### Running the Services

**1. Start Payment Processing Service**

```bash
cd payment-processing-service
mvn clean install
mvn spring-boot:run -Dspring.profiles.active=local
```

**2. Start Stripe Provider Service**

```bash
cd stripe-provider-expire-session
mvn clean install
mvn spring-boot:run -Dspring.profiles.active=local
```

**3. Verify Services are Running**

- Payment Processing Service: http://localhost:8080/actuator/health
- Stripe Provider Service: http://localhost:8081/actuator/health

## 📡 API Documentation

### Payment Processing Service (Port 8080)

#### Create Transaction

```http
POST /v1/payments
Content-Type: application/json

{
  "userId": 123,
  "paymentMethodId": 1,
  "providerId": 1,
  "paymentTypeId": 1,
  "amount": 99.99,
  "currency": "USD",
  "merchantTransactionReference": "ORDER-12345"
}
```

**Response:**
```json
{
  "txnReference": "TXN-20250101-ABC123",
  "status": "CREATED",
  "amount": 99.99,
  "currency": "USD"
}
```

#### Initiate Payment

```http
POST /v1/payments/{txnReference}/initiate
Content-Type: application/json

{
  "successUrl": "https://yoursite.com/success",
  "cancelUrl": "https://yoursite.com/cancel",
  "customerEmail": "customer@example.com"
}
```

**Response:**
```json
{
  "redirectUrl": "https://checkout.stripe.com/pay/cs_test_...",
  "sessionId": "cs_test_...",
  "status": "PENDING"
}
```

#### Webhook Notification (Internal)

```http
POST /v1/payments/notifications
Content-Type: application/json

{
  "txnReference": "TXN-20250101-ABC123",
  "status": "COMPLETED",
  "providerReference": "pi_..."
}
```

### Stripe Provider Service (Port 8081)

#### Create Payment Session

```http
POST /payments
Content-Type: application/json

{
  "amount": 9999,
  "currency": "usd",
  "successUrl": "https://yoursite.com/success",
  "cancelUrl": "https://yoursite.com/cancel",
  "customerEmail": "customer@example.com",
  "txnReference": "TXN-20250101-ABC123"
}
```

#### Get Payment Session

```http
GET /payments/{sessionId}
```

#### Expire Payment Session

```http
POST /payments/{sessionId}/expire
```

#### Stripe Webhook

```http
POST /v1/stripe/webhook
Stripe-Signature: t=...,v1=...
Content-Type: application/json

{
  "type": "checkout.session.completed",
  "data": {
    "object": {
      "id": "cs_test_...",
      "payment_status": "paid"
    }
  }
}
```

## 🔌 Frontend Integration

### JavaScript Example

```javascript
// Payment Processing Class
class PaymentProcessor {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
    }

    // Step 1: Create Transaction
    async createTransaction(paymentData) {
        const response = await fetch(`${this.baseUrl}/v1/payments`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(paymentData)
        });

        if (!response.ok) {
            throw new Error('Failed to create transaction');
        }

        return await response.json();
    }

    // Step 2: Initiate Payment
    async initiatePayment(txnReference, paymentDetails) {
        const response = await fetch(
            `${this.baseUrl}/v1/payments/${txnReference}/initiate`,
            {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(paymentDetails)
            }
        );

        if (!response.ok) {
            throw new Error('Failed to initiate payment');
        }

        return await response.json();
    }

    // Complete Payment Flow
    async processPayment(paymentData, paymentDetails) {
        try {
            // Create transaction
            const transaction = await this.createTransaction(paymentData);
            console.log('Transaction created:', transaction.txnReference);

            // Initiate payment
            const paymentResponse = await this.initiatePayment(
                transaction.txnReference,
                paymentDetails
            );
            console.log('Payment initiated:', paymentResponse.sessionId);

            // Redirect to Stripe
            window.location.href = paymentResponse.redirectUrl;
        } catch (error) {
            console.error('Payment processing failed:', error);
            throw error;
        }
    }
}

// Usage Example
const processor = new PaymentProcessor('http://localhost:8080');

// Payment data
const paymentData = {
    userId: 123,
    paymentMethodId: 1,
    providerId: 1,
    paymentTypeId: 1,
    amount: 99.99,
    currency: 'USD',
    merchantTransactionReference: 'ORDER-12345'
};

// Payment details
const paymentDetails = {
    successUrl: 'https://yoursite.com/success',
    cancelUrl: 'https://yoursite.com/cancel',
    customerEmail: 'customer@example.com'
};

// Process payment
processor.processPayment(paymentData, paymentDetails);
```

### React Example

```jsx
import { useState } from 'react';

function PaymentForm() {
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);

    const handlePayment = async (e) => {
        e.preventDefault();
        setLoading(true);
        setError(null);

        try {
            const processor = new PaymentProcessor('http://localhost:8080');
            
            const paymentData = {
                userId: 123,
                paymentMethodId: 1,
                providerId: 1,
                paymentTypeId: 1,
                amount: 99.99,
                currency: 'USD',
                merchantTransactionReference: 'ORDER-12345'
            };

            const paymentDetails = {
                successUrl: window.location.origin + '/success',
                cancelUrl: window.location.origin + '/cancel',
                customerEmail: 'customer@example.com'
            };

            await processor.processPayment(paymentData, paymentDetails);
        } catch (err) {
            setError(err.message);
            setLoading(false);
        }
    };

    return (
        <form onSubmit={handlePayment}>
            <button type="submit" disabled={loading}>
                {loading ? 'Processing...' : 'Pay $99.99'}
            </button>
            {error && <div className="error">{error}</div>}
        </form>
    );
}
```

### Success/Cancel Pages

```html
<!-- success.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Payment Successful</title>
</head>
<body>
    <h1>Payment Successful!</h1>
    <p>Your payment has been processed successfully.</p>
    <a href="/">Return to Home</a>
</body>
</html>

<!-- cancel.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Payment Cancelled</title>
</head>
<body>
    <h1>Payment Cancelled</h1>
    <p>Your payment was cancelled. You can try again.</p>
    <a href="/">Return to Home</a>
</body>
</html>
```

## 🧪 Testing

### Run Unit Tests

```bash
# Payment Processing Service
cd payment-processing-service
mvn test

# Stripe Provider Service
cd stripe-provider-expire-session
mvn test
```

### Run Integration Tests

```bash
mvn verify
```

### Manual Testing with cURL

**Create Transaction:**
```bash
curl -X POST http://localhost:8080/v1/payments \
  -H "Content-Type: application/json" \
  -d '{
    "userId": 123,
    "paymentMethodId": 1,
    "providerId": 1,
    "paymentTypeId": 1,
    "amount": 99.99,
    "currency": "USD",
    "merchantTransactionReference": "ORDER-12345"
  }'
```

**Initiate Payment:**
```bash
curl -X POST http://localhost:8080/v1/payments/TXN-20250101-ABC123/initiate \
  -H "Content-Type: application/json" \
  -d '{
    "successUrl": "https://yoursite.com/success",
    "cancelUrl": "https://yoursite.com/cancel",
    "customerEmail": "customer@example.com"
  }'
```

## 🚢 Deployment

### Environment Variables

**Production Configuration:**

```bash
# Payment Processing Service
export SPRING_PROFILES_ACTIVE=prod
export DB_URL=jdbc:mysql://production-db:3306/payments
export DB_USERNAME=prod_user
export DB_PASSWORD=prod_password
export STRIPE_PROVIDER_URL=https://stripe-provider.yourcompany.com/payments

# Stripe Provider Service
export STRIPE_API_KEY=sk_live_your_production_key
export STRIPE_WEBHOOK_SECRET=whsec_your_production_secret
export PROCESSING_SERVICE_URL=https://payment-service.yourcompany.com/v1/payments/notifications
```

### Docker Deployment

**Payment Processing Service Dockerfile:**

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/payment-processing-service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Stripe Provider Service Dockerfile:**

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/stripe-provider-service.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build and Run:**

```bash
# Build images
docker build -t payment-processing-service:latest ./payment-processing-service
docker build -t stripe-provider-service:latest ./stripe-provider-expire-session

# Run containers
docker run -d -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_URL=jdbc:mysql://db:3306/payments \
  payment-processing-service:latest

docker run -d -p 8081:8081 \
  -e STRIPE_API_KEY=sk_live_... \
  stripe-provider-service:latest
```

### Docker Compose

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: payments
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  payment-service:
    build: ./payment-processing-service
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_URL: jdbc:mysql://mysql:3306/payments
      DB_USERNAME: root
      DB_PASSWORD: rootpassword
    depends_on:
      - mysql

  stripe-provider:
    build: ./stripe-provider-expire-session
    ports:
      - "8081:8081"
    environment:
      STRIPE_API_KEY: ${STRIPE_API_KEY}
      STRIPE_WEBHOOK_SECRET: ${STRIPE_WEBHOOK_SECRET}
    depends_on:
      - payment-service

volumes:
  mysql_data:
```

## 🔒 Security

### Best Practices

1. **API Keys Protection**
   - Store Stripe API keys in environment variables
   - Never commit keys to version control
   - Use different keys for development and production

2. **Webhook Security**
   - Validate Stripe webhook signatures
   - Use HTTPS for webhook endpoints
   - Implement request throttling

3. **Database Security**
   - Use strong passwords
   - Limit database user permissions
   - Enable SSL for database connections

4. **Network Security**
   - Use HTTPS in production
   - Implement rate limiting
   - Add authentication/authorization to endpoints

5. **Data Protection**
   - Never log sensitive payment information
   - Implement PCI DSS compliance measures
   - Encrypt sensitive data at rest

### Webhook Signature Validation

The system automatically validates Stripe webhook signatures:

```java
// Webhook validation (handled automatically)
String payload = request.getBody();
String sigHeader = request.getHeader("Stripe-Signature");
Event event = Webhook.constructEvent(payload, sigHeader, webhookSecret);
```

## 📊 Monitoring and Logging

### Health Checks

- Payment Processing Service: `http://localhost:8080/actuator/health`
- Stripe Provider Service: `http://localhost:8081/actuator/health`

### Logging Levels

Configure in `application.properties`:

```properties
# Root level
logging.level.root=INFO

# Application level
logging.level.com.hulkhiretech.payments=DEBUG

# SQL queries
logging.level.org.springframework.jdbc=DEBUG
```

### Monitoring Transactions

Query transaction status:

```sql
-- Check transaction status
SELECT * FROM Transaction 
WHERE txnReference = 'TXN-20250101-ABC123';

-- Failed transactions
SELECT * FROM Transaction 
WHERE txnStatusId = 3 
ORDER BY creationDate DESC;

-- Recent transactions
SELECT * FROM Transaction 
WHERE creationDate >= DATE_SUB(NOW(), INTERVAL 1 DAY);
```

## 📝 Transaction Status Codes

| Status ID | Status | Description |
|-----------|--------|-------------|
| 1 | CREATED | Transaction created, not yet initiated |
| 2 | PENDING | Payment initiated, awaiting completion |
| 3 | COMPLETED | Payment successfully completed |
| 4 | FAILED | Payment failed |
| 5 | CANCELLED | Payment cancelled by user |
| 6 | EXPIRED | Payment session expired |

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License.

## 📞 Support

For issues and questions:
- Create an issue in the repository
- Email: support@hulkhiretech.com

## 🔗 Related Documentation

- [Stripe API Documentation](https://stripe.com/docs/api)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [MySQL Documentation](https://dev.mysql.com/doc/)

---

**Built with ❤️ by Somashekhar**