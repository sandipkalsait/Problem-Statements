# Problem Statement: LogiChain – Supply Chain Management & Tracking System API

## Scenario
Logi-Chain is a comprehensive supply chain management platform that streamlines the flow of goods from suppliers to retailers. The system offers real-time tracking of shipments, inventory management, and order processing through a secure, scalable RESTful API backend built with Spring Boot. It supports role-based access for suppliers, transporters, retailers, and administrators, ensuring transparency, efficiency, and integrity throughout the supply chain.

## Business Requirements
- **Suppliers**: Manage product shipments, update inventory levels, and coordinate raw material orders.
- **Transporters**: Update shipment statuses, manage delivery routes, and monitor transit conditions.
- **Retailers**: Place orders, track order fulfillment, and manage inventory replenishment.
- **Administrators**: Oversee system operations, manage user accounts, audit activities, and generate performance analytics.
- **Security**: Enforce JWT-based authentication to protect endpoints and implement strict role-based access control.

## Database Schema and Default Data

### User
| Field    | Type          | Description                          |
|----------|---------------|--------------------------------------|
| userId   | Integer       | Unique identifier for the user       |
| username | String        | Login username                       |
| password | String        | Hashed password                      |
| roles    | List<Role>    | Assigned roles (SUPPLIER, TRANSPORTER, RETAILER, ADMIN) |

### Role
| Field    | Type    | Description                              |
|----------|---------|------------------------------------------|
| roleId   | Integer | Unique role identifier                   |
| roleName | String  | Possible values: SUPPLIER, TRANSPORTER, RETAILER, ADMIN |

### Shipment
| Field            | Type    | Description                              |
|------------------|---------|------------------------------------------|
| shipmentId       | Integer | Unique shipment identifier               |
| supplier         | User    | Supplier initiating the shipment         |
| transporter      | User    | Assigned transporter for the shipment    |
| origin           | String  | Shipping origin location                 |
| destination      | String  | Delivery destination                     |
| status           | String  | Shipment status (PENDING, IN_TRANSIT, DELIVERED, DELAYED) |
| dispatchedAt     | DateTime| Timestamp when shipment was dispatched   |
| expectedDelivery | DateTime| Estimated delivery time                  |

### Inventory
| Field        | Type    | Description                              |
|--------------|---------|------------------------------------------|
| inventoryId  | Integer | Unique inventory record identifier       |
| supplier     | User    | Supplier associated with the inventory record |
| productName  | String  | Name of the product                      |
| quantity     | Integer | Available quantity                       |
| lastUpdated  | DateTime| Timestamp when inventory was last updated|

### Order
| Field       | Type    | Description                              |
|-------------|---------|------------------------------------------|
| orderId     | Integer | Unique order identifier                  |
| retailer    | User    | Retailer placing the order               |
| shipment    | Shipment| Associated shipment details              |
| orderDate   | DateTime| Date and time when the order was placed  |
| totalAmount | Double  | Total order cost                         |
| status      | String  | Order status (PLACED, CONFIRMED, SHIPPED, COMPLETED) |

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication is required for all secured endpoints.
- Role-based access control:
    - **Suppliers**: Manage inventory records, initiate shipments, and update product details.
    - **Transporters**: Update shipment statuses, manage delivery routes, and log transit events.
    - **Retailers**: Place orders, track shipments, and manage received inventory.
    - **Administrators**: Perform full system management including user administration, audit logging, and analytics.
- Return `401 Unauthorized` if the JWT is missing or invalid.
- Return `403 Forbidden` if a user accesses an endpoint without the required role.

### Public Endpoints (No Authentication Required)

#### Login – `/api/public/login` – POST
**Request:**
```json
{
    "username": "retailer_jane",
    "password": "secure_pass"
}
```
**Response (200 OK):**
```json
{
    "token": "jwt_token_here"
}
```

#### Browse Available Shipments – `/api/public/shipments` – GET
Returns a list of shipments with basic status and tracking details.

### Authenticated Endpoints (JWT Required)

#### For Suppliers (`/api/auth/supplier`)

##### Manage Inventory – `/api/auth/supplier/inventory` – CRUD operations
- **POST**: Add new inventory records.
- **PUT**: Update inventory quantities.
- **GET**: Retrieve current inventory data.

##### Initiate Shipment – `/api/auth/supplier/shipment` – POST
**Request:**
```json
{
    "origin": "Factory A",
    "destination": "Distribution Center B",
    "expectedDelivery": "2025-08-15T12:00:00"
}
```
**Response (201 Created):**
```json
{
    "shipmentId": 123,
    "origin": "Factory A",
    "destination": "Distribution Center B",
    "expectedDelivery": "2025-08-15T12:00:00",
    "status": "PENDING"
}
```

#### For Transporters (`/api/auth/transporter`)

##### Update Shipment Status – `/api/auth/transporter/shipment/{shipmentId}` – PUT
**Request:**
```json
{
    "status": "IN_TRANSIT"
}
```
**Response (200 OK):**
```json
{
    "shipmentId": 123,
    "status": "IN_TRANSIT"
}
```

##### Log Transit Events – `/api/auth/transporter/shipments/log` – POST
**Request:**
```json
{
    "shipmentId": 102,
    "event": "Reached checkpoint at Highway 50",
    "timestamp": "2025-08-10T09:30:00"
}
```
**Response (200 OK):**
```json
{
    "message": "Transit event logged successfully"
}
```

#### For Retailers (`/api/auth/retailer`)

##### Place Order – `/api/auth/retailer/order` – POST
**Request:**
```json
{
    "shipmentId": 102,
    "totalAmount": 1500.00
}
```
**Response (201 Created):**
```json
{
    "orderId": 456,
    "status": "PLACED"
}
```

##### View Order History – `/api/auth/retailer/orders` – GET
Retrieves a list of past and current orders with shipment tracking.

#### For Administrators (`/api/auth/admin`)

##### User Management – `/api/auth/admin/users` – CRUD operations
Manage supplier, transporter, and retailer accounts.

##### Audit Logs – `/api/auth/admin/logs` – GET
Retrieve detailed logs of user activities and shipment updates.

##### System Analytics – `/api/auth/admin/analytics` – GET
Generate reports on shipment performance, order fulfillment, and inventory turnover.

## Implementation Priorities

### Security
- Enforce robust JWT authentication and role-based access control using Spring Security.
- Implement comprehensive audit logging for all critical operations to ensure system transparency and compliance.

### Scalability
- Utilize Spring Data JPA with PostgreSQL to manage large volumes of inventory, shipment, and order data.
- Optimize API performance with effective indexing, caching strategies, and load balancing.

### Reliability & Validation
- Implement rigorous data validation to prevent malformed requests.
- Ensure atomic transaction management across operations such as order processing and shipment updates.

### RESTful API Best Practices
- Use standardized HTTP status codes and clear error messaging.
- Integrate pagination and filtering on endpoints that return large datasets (e.g., shipment lists, audit logs).

## Testing & Deployment
- **Unit Testing**: Use JUnit and Mockito for rigorous component testing.
- **Integration Testing**: Conduct end-to-end tests covering inventory updates, shipment processing, and order placements.
- **CI/CD Pipeline**: Automate deployment processes to ensure rapid, reliable updates and minimal downtime.