# Problem Statement: ShopSync – Omnichannel E-commerce Platform API

## Scenario
ShopSync is an innovative e-commerce platform designed to deliver a seamless shopping experience across web and mobile channels. The API, built with Spring Boot, manages product listings, user orders, payments, and shipping logistics. It caters to a diverse user base—including customers, sellers, and administrators—by enforcing strict role-based access controls and ensuring high performance and data integrity throughout the system.

## Business Requirements

### Customers
- Browse and search products by category, price, and ratings.
- Place orders, track order status, and submit product reviews.
- Manage user profiles and view order history.

### Sellers
- List new products, update inventory and pricing, and manage order fulfillment.
- Access sales analytics and respond to customer reviews.

### Administrators
- Oversee platform operations, manage users and sellers, and maintain audit logs.
- Monitor system performance and handle dispute resolution.

### Security
- Enforce JWT-based authentication to protect endpoints.
- Implement robust role-based access control to segregate customer, seller, and admin functionalities.

## Database Schema and Default Data

### User
| Field   | Type        | Description                        |
|---------|-------------|------------------------------------|
| userId  | Integer     | Unique identifier for the user     |
| username| String      | Login username                     |
| password| String      | Hashed password                    |
| roles   | List<Role>  | Assigned roles (CUSTOMER, SELLER, ADMIN) |

### Role
| Field   | Type    | Description                            |
|---------|---------|----------------------------------------|
| roleId  | Integer | Unique role identifier                 |
| roleName| String  | Possible values: CUSTOMER, SELLER, ADMIN |

### Category
| Field      | Type    | Description                            |
|------------|---------|----------------------------------------|
| categoryId | Integer | Unique category identifier             |
| name       | String  | Name of the category (e.g., Electronics, Apparel) |
| description| String  | Brief description of the category      |

### Product
| Field      | Type    | Description                            |
|------------|---------|----------------------------------------|
| productId  | Integer | Unique product identifier              |
| name       | String  | Product name                           |
| description| String  | Detailed product description           |
| price      | Double  | Cost of the product                    |
| stock      | Integer | Available quantity                     |
| category   | Category| Associated product category            |
| seller     | User    | Seller who listed the product          |

### Order
| Field      | Type    | Description                            |
|------------|---------|----------------------------------------|
| orderId    | Integer | Unique order identifier                |
| customer   | User    | Customer who placed the order          |
| orderDate  | DateTime| Date and time when the order was placed|
| status     | String  | Order status (PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED) |
| totalAmount| Double  | Total cost of the order                |

### OrderItem
| Field      | Type    | Description                            |
|------------|---------|----------------------------------------|
| orderItemId| Integer | Unique order item identifier           |
| order      | Order   | Associated order                       |
| product    | Product | Product details                        |
| quantity   | Integer | Quantity ordered                       |
| unitPrice  | Double  | Price per unit at the time of order    |

### Payment
| Field      | Type    | Description                            |
|------------|---------|----------------------------------------|
| paymentId  | Integer | Unique payment identifier              |
| order      | Order   | Associated order                       |
| amount     | Double  | Payment amount                         |
| paymentDate| DateTime| Date and time of payment               |
| method     | String  | Payment method (e.g., Credit Card, PayPal) |

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication is required for all secure endpoints.
- Role-based access control:
    - Customers: Browse products, place orders, view order history, and submit reviews.
    - Sellers: List products, update inventory, and manage order fulfillment.
    - Administrators: Perform full system management including user and product moderation.
- Return `401 Unauthorized` if the JWT is missing or invalid.
- Return `403 Forbidden` if a user does not have the necessary permissions.

### Public Endpoints (No Authentication Required)

#### Login – `/api/public/login` – POST
**Request:**
```json
{
        "username": "shopper123",
        "password": "secure_pass"
}
```
**Response (200 OK):**
```json
{
        "token": "jwt_token_here"
}
```

#### Browse Products – `/api/public/products` – GET
Returns a list of products with filtering options (e.g., by category or price range).

### Authenticated Endpoints (JWT Required)

#### For Customers (`/api/auth/customer`)

##### View Profile – `/api/auth/customer/profile` – GET/PUT
Retrieve or update customer profile details.

##### Place Order – `/api/auth/customer/order` – POST
**Request:**
```json
{
        "orderItems": [
                { "productId": 101, "quantity": 2 },
                { "productId": 205, "quantity": 1 }
        ]
}
```
**Response (201 Created):**
```json
{
        "orderId": 5501,
        "customer": { "userId": 310, "username": "shopper123" },
        "totalAmount": 299.97,
        "status": "PENDING"
}
```

##### View Order History – `/api/auth/customer/orders` – GET
Retrieve a list of past and active orders.

##### Submit Product Review – `/api/auth/customer/review` – POST
**Request:**
```json
{
        "productId": 101,
        "rating": 4,
        "comment": "Great product and fast delivery!"
}
```
**Response (201 Created):**
```json
{
        "reviewId": 1001,
        "productId": 101,
        "rating": 4,
        "comment": "Great product and fast delivery!"
}
```

#### For Sellers (`/api/auth/seller`)

##### List New Product – `/api/auth/seller/product` – POST
**Request:**
```json
{
        "name": "Wireless Headphones",
        "description": "High quality sound with noise cancellation",
        "price": 99.99,
        "stock": 50,
        "categoryId": 3
}
```
**Response (201 Created):**
```json
{
        "productId": 101,
        "name": "Wireless Headphones",
        "price": 99.99,
        "stock": 50,
        "status": "ACTIVE"
}
```

##### Update Product Inventory – `/api/auth/seller/product/{productId}` – PUT
Update product details such as stock levels or pricing.

##### Manage Orders – `/api/auth/seller/orders` – GET
Retrieve orders for products sold by the seller.

#### For Administrators (`/api/auth/admin`)

##### User Management – `/api/auth/admin/users` – CRUD operations
Manage customer and seller accounts.

##### Product Moderation – `/api/auth/admin/products` – GET/PUT/DELETE
Review and moderate product listings.

##### Audit Logs & Analytics – `/api/auth/admin/logs` – GET
Retrieve system logs and generate sales performance reports.

## Implementation Priorities

### Security
- Implement robust JWT authentication and strict role-based access controls using Spring Security.
- Maintain detailed audit logs to ensure transparency and compliance.

### Scalability
- Utilize Spring Data JPA with PostgreSQL to manage large volumes of product and transaction data.
- Optimize API performance with effective indexing and caching strategies.

### Reliability & Validation
- Enforce comprehensive data validation to prevent malformed requests.
- Ensure atomic transactions for order processing and payment operations to maintain data integrity.

### RESTful API Best Practices
- Utilize standardized HTTP status codes and clear error messaging.
- Integrate pagination and filtering on endpoints returning extensive datasets (e.g., product lists, audit logs).

## Testing & Deployment

### Unit Testing
Employ JUnit and Mockito for rigorous testing of individual components.

### Integration Testing
Conduct end-to-end tests covering customer checkout, seller product updates, and admin moderation flows.

### CI/CD Pipeline
Establish automated deployment pipelines to ensure rapid, reliable updates and minimize downtime.

## Sample JSON Examples for APIs

### Login
**Request:**
```json
{
        "username": "shopper123",
        "password": "secure_pass"
}
```
**Response:**
```json
{
        "token": "jwt_token_here"
}
```

### Place Order (Customer)
**Request:**
```json
{
        "orderItems": [
                { "productId": 101, "quantity": 2 },
                { "productId": 205, "quantity": 1 }
        ]
}
```
**Response:**
```json
{
        "orderId": 5501,
        "customer": { "userId": 310, "username": "shopper123" },
        "totalAmount": 299.97,
        "status": "PENDING"
}
```

### List New Product (Seller)
**Request:**
```json
{
        "name": "Wireless Headphones",
        "description": "High quality sound with noise cancellation",
        "price": 99.99,
        "stock": 50,
        "categoryId": 3
}
```
**Response:**
```json
{
        "productId": 101,
        "name": "Wireless Headphones",
        "price": 99.99,
        "stock": 50,
        "status": "ACTIVE"
}
```