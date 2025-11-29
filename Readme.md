### File: README.md

Creating a comprehensive README file for the payment processing system

```markdown

# Payment Processing System

A comprehensive payment processing system with microservices architecture that handles payment transactions, integrates with Stripe as a payment provider, and manages payment status updates through webhooks.

## Project Structure

The system consists of two main microservices:

### 1. Payment Processing Service (`payment-processing-service`)
```
payment-processing-service/
├── src/
│   ├── main/
│   │   ├── java/com/hulkhiretech/payments/
│   │   │   ├── config/              # Application configuration
│   │   │   ├── constant/            # Enums and constants
│   │   │   ├── controller/          # REST API controllers
│   │   │   ├── dao/                 # Data access layer
│   │   │   ├── dto/                 # Data transfer objects
│   │   │   ├── entity/              # Database entities
│   │   │   ├── exception/           # Custom exceptions and handlers
│   │   │   ├── http/                # HTTP client utilities
│   │   │   ├── pojo/                # Plain Java objects
│   │   │   ├── service/             # Business logic layer
│   │   │   ├── stripeprovider/     # Stripe provider integration models
│   │   │   ├── util/                # Utility classes
│   │   │   └── PaymentProcessingServiceApplication.java
│   │   └── resources/               # Configuration files
│   └── test/                        # Unit and integration tests
└── pom.xml                          # Maven dependencies
```

### 2. Stripe Provider Service (`stripe-provider-expire-session`)
```
stripe-provider-expire-session/
├── src/
│   ├── main/
│   │   ├── java/com/hulkhiretech/payments/
│   │   │   ├── config/              # Application configuration
│   │   │   ├── constant/            # Enums and constants
│   │   │   ├── controller/          # REST API controllers
│   │   │   ├── exception/           # Custom exceptions and handlers
│   │   │   ├── http/                # HTTP client utilities
│   │   │   ├── interfaces/          # Service interfaces
│   │   │   ├── pojo/                # Plain Java objects
│   │   │   ├── processingservice/  # Processing service integration models
│   │   │   ├── service/             # Business logic layer
│   │   │   ├── stripe/              # Stripe response models
│   │   │   ├── util/                # Utility classes
│   │   │   └── StripeProviderServiceApplication.java
│   │   └── resources/               # Configuration files
│   └── test/                        # Unit and integration tests
└── pom.xml                          # Maven dependencies
```

## How It Works

### Payment Flow

1. **Create Transaction**: Frontend sends payment details to Payment Processing Service
2. **Initiate Payment**: System creates a payment session with Stripe Provider Service
3. **Redirect to Stripe**: User is redirected to Stripe checkout page
4. **Webhook Notification**: Stripe sends payment status updates via webhooks
5. **Status Update**: System updates transaction status in database

### Key Components

- **Payment Processing Service**: Core service that manages payment transactions and database operations
- **Stripe Provider Service**: Handles all communication with Stripe API
- **Webhook Processing**: Asynchronous handling of payment status updates from Stripe
- **Database Layer**: MySQL database with JDBC template for data persistence

## Technologies Used

### Backend
- **Java 21**: Primary programming language
- **Spring Boot 3.4+**: Framework for building RESTful services
- **Maven**: Build automation and dependency management
- **MySQL 8+**: Relational database for data storage
- **JDBC Template**: Database access layer
- **ModelMapper**: Object mapping between layers
- **Lombok**: Reducing boilerplate code
- **JUnit 5 & Mockito**: Testing framework
- **RestClient**: HTTP client for external API calls
- **Jackson**: JSON processing

### Payment Integration
- **Stripe API**: Payment processing provider
- **Stripe Java SDK**: Webhook signature validation

## Setup Instructions

### Prerequisites
- Java 21 JDK
- Maven 3.6+
- MySQL 8+
- Stripe account with API keys

### Database Setup
1. Create MySQL database:
```sql
CREATE DATABASE payments;
```

2. Create required tables:
```sql
CREATE TABLE payments.Transaction (
    id INT AUTO_INCREMENT PRIMARY KEY,
    userId INT,
    paymentMethodId INT,
    providerId INT,
    paymentTypeId INT,
    txnStatusId INT,
    amount DECIMAL(10,2),
    currency VARCHAR(3),
    merchantTransactionReference VARCHAR(255),
    txnReference VARCHAR(255),
    providerReference VARCHAR(255),
    errorCode VARCHAR(50),
    errorMessage VARCHAR(255),
    creationDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    retryCount INT DEFAULT 0
);
```

### Environment Configuration

#### Payment Processing Service
Create `src/main/resources/application-local.properties`:
```properties
# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/payments
spring.datasource.username=your_username
spring.datasource.password=your_password

# Stripe Provider Configuration
stripe-provider.create.payment.url=http://localhost:8081/payments

# Logging
logging.level.com.hulkhiretech.payments=DEBUG
```

#### Stripe Provider Service
Create `src/main/resources/application-local.properties`:
```properties
# Stripe Configuration
stripe.api.key=your_stripe_secret_key
stripe.create.session.url=https://api.stripe.com/v1/checkout/sessions
stripe.get.session.url=https://api.stripe.com/v1/checkout/sessions/{id}
stripe.expire.session.url=https://api.stripe.com/v1/checkout/sessions/{id}/expire
stripe.webhook.secret=your_webhook_secret

# Processing Service Notification URL
processing.notification.url=http://localhost:8080/v1/payments/notifications

# Logging
logging.level.com.hulkhiretech.payments=DEBUG
```

### Building and Running

1. **Payment Processing Service**:
```bash
cd payment-processing-service
mvn clean install
mvn spring-boot:run -Dspring.profiles.active=local
```

2. **Stripe Provider Service**:
```bash
cd stripe-provider-expire-session
mvn clean install
mvn spring-boot:run -Dspring.profiles.active=local
```

## API Endpoints

### Payment Processing Service
- `POST /v1/payments` - Create a new transaction
- `POST /v1/payments/{txnReference}/initiate` - Initiate payment with Stripe
- `POST /v1/payments/notifications` - Receive payment status notifications

### Stripe Provider Service
- `POST /payments` - Create Stripe checkout session
- `GET /payments/{id}` - Get payment session details
- `POST /payments/{id}/expire` - Expire payment session
- `POST /v1/stripe/webhook` - Receive Stripe webhooks

## Frontend Integration

### 1. Create Payment Transaction
```javascript
// Step 1: Create transaction
const createTransaction = async (paymentData) => {
  const response = await fetch('http://localhost:8080/v1/payments', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(paymentData),
  });
  return await response.json();
};

// Step 2: Initiate payment
const initiatePayment = async (txnReference, paymentDetails) => {
  const response = await fetch(`http://localhost:8080/v1/payments/${txnReference}/initiate`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(paymentDetails),
  });
  return await response.json();
};
```

### 2. Payment Flow Implementation
```javascript
// Complete payment flow
const processPayment = async (paymentData, paymentDetails) => {
  try {
    // Create transaction
    const transaction = await createTransaction(paymentData);
    
    // Initiate payment
    const paymentResponse = await initiatePayment(
      transaction.txnReference, 
      paymentDetails
    );
    
    // Redirect to Stripe
    window.location.href = paymentResponse.redirectUrl;
  } catch (error) {
    console.error('Payment initiation failed:', error);
  }
};
```

### 3. Webhook Handling
The system automatically handles payment status updates from Stripe through webhooks. No frontend implementation is required for this.

## Testing

### Unit Tests
Run unit tests with Maven:
```bash
mvn test
```

### Integration Tests
Run integration tests:
```bash
mvn verify
```

## Deployment

### Production Configuration
Set environment variables:
```bash
export SPRING_PROFILES_ACTIVE=prod
export DB_URL=jdbc:mysql://production-db:3306/payments
export DB_USERNAME=prod_user
export DB_PASSWORD=prod_password
export STRIPE_API_KEY=your_production_stripe_key
```

### Docker Deployment (Optional)
Create Docker images for both services:
```bash
# Payment Processing Service
cd payment-processing-service
docker build -t payment-processing-service .

# Stripe Provider Service
cd stripe-provider-expire-session
docker build -t stripe-provider-service .
```

## Monitoring and Logging

- All services use structured logging with log levels
- Monitor transaction statuses through database queries
- Webhook delivery failures are logged for troubleshooting
- Health checks available at `/actuator/health` endpoints

## Security Considerations

- Stripe webhook signatures are validated to prevent tampering
- Database credentials should be stored securely (e.g., environment variables)
- HTTPS should be used in production environments
- API endpoints should be protected with authentication where appropriate

```