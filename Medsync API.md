# Problem Statement: MedSync – Online Pharmacy Management System

## Scenario:
MedSync is a growing online pharmacy that allows customers to purchase medicines, schedule refills, and consult pharmacists. The company wants to build a secure, scalable, and efficient RESTful API backend using Spring Boot. The system should support role-based access for customers, pharmacists, and admins, ensuring strict control over who can access and modify data.

## Business Requirements:
- **Customers** can browse medicines, place orders, track order history, and schedule refills.
- **Pharmacists** can manage inventory, validate prescriptions, and approve/refuse orders based on stock availability.
- **Admins** have full control over the system, including user management and audit logs.
- **Security**: Use JWT-based authentication to protect all endpoints, ensuring proper role-based access control.

## Database Schema and Default Data

### User
| Field   | Type          | Description                  |
|---------|---------------|------------------------------|
| userId  | Integer       | Unique ID of the user        |
| username| String        | Username for login           |
| password| String        | Hashed password              |
| roles   | List<Role>    | Assigned roles (CUSTOMER, PHARMACIST, ADMIN) |

### Role
| Field   | Type          | Description                  |
|---------|---------------|------------------------------|
| roleId  | Integer       | Unique ID for role           |
| roleName| String        | Possible values: CUSTOMER, PHARMACIST, ADMIN |

### Medicine
| Field               | Type    | Description                  |
|---------------------|---------|------------------------------|
| medicineId          | Integer | Unique medicine identifier   |
| name                | String  | Name of the medicine         |
| category            | String  | E.g., "Pain Relief", "Antibiotic", "Vitamins" |
| price               | Double  | Cost per unit                |
| stock               | Integer | Available quantity           |
| prescriptionRequired| Boolean | Whether a doctor’s prescription is needed |

### Prescription
| Field          | Type        | Description                  |
|----------------|-------------|------------------------------|
| prescriptionId | Integer     | Unique ID for prescription   |
| customer       | User        | Associated customer          |
| medicines      | List<Medicine> | Prescribed medicines     |
| doctorName     | String      | Doctor’s name who issued the prescription |
| validUntil     | Date        | Expiry date of prescription  |

### Order
| Field      | Type        | Description                  |
|------------|-------------|------------------------------|
| orderId    | Integer     | Unique order ID              |
| customer   | User        | Customer who placed the order|
| medicines  | List<OrderItem> | Medicines in the order   |
| totalAmount| Double      | Total cost of the order      |
| status     | String      | Order status (PLACED, APPROVED, SHIPPED, DELIVERED, CANCELLED) |

### OrderItem
| Field       | Type        | Description                  |
|-------------|-------------|------------------------------|
| orderItemId | Integer     | Unique ID for order item     |
| order       | Order       | Associated order             |
| medicine    | Medicine    | Medicine details             |
| quantity    | Integer     | Quantity ordered             |

## API Requirements and Endpoints

### Security & Authentication
- JWT Authentication is required for all private endpoints.
- Role-based access control:
    - Customers can browse medicines, place orders, and track order status.
    - Pharmacists can manage inventory, approve/cancel orders, and validate prescriptions.
    - Admins can perform all operations, including user management.
- If JWT is missing, return 401 Unauthorized.
- If an endpoint is accessed with an incorrect role, return 403 Forbidden.

### Public Endpoints (No Authentication Required)
- **Login** – `/api/public/login` – POST
    - **Request:**
        ```json
        { "username": "john_doe", "password": "secure_pass" }
        ```
    - **Response (Success, 200):**
        ```json
        { "token": "jwt_token_here" }
        ```
    - Failure (401 Unauthorized) for incorrect credentials.
- **Browse Medicines** – `/api/public/medicine/search?keyword=antibiotic` – GET
    - Returns a list of medicines matching the keyword in name or category.

### Authenticated Endpoints (JWT Required)

#### For Customers (`/api/auth/customer`)
- **View Available Medicines** – `/api/auth/customer/medicine` – GET
    - Returns all medicines available for purchase.
- **Place an Order** – `/api/auth/customer/order` – POST
    - **Request:**
        ```json
        {
            "medicines": [
                { "medicineId": 5, "quantity": 2 },
                { "medicineId": 12, "quantity": 1 }
            ]
        }
        ```
    - **Response (201 Created):** Order details with total price.
    - If a prescription is required, return 400 Bad Request prompting prescription upload.
- **Upload Prescription** – `/api/auth/customer/prescription` – POST
    - Customers upload prescriptions for pharmacist approval.
- **View Order History** – `/api/auth/customer/order/history` – GET
    - Returns past and active orders.

#### For Pharmacists (`/api/auth/pharmacist`)
- **Manage Inventory** – `/api/auth/pharmacist/medicine`
    - **GET** – Retrieve stock levels.
    - **POST** – Add a new medicine.
    - **PUT** – Update medicine stock.
    - **DELETE** – Remove a medicine from inventory.
- **Approve or Reject Orders** – `/api/auth/pharmacist/order/{orderId}` – PUT
    - Approves or cancels orders based on stock levels.
    - If a prescription is invalid, return 403 Forbidden.

#### For Admins (`/api/auth/admin`)
- **User Management** – `/api/auth/admin/users`
    - **GET** – View all users.
    - **POST** – Create a new user (CUSTOMER, PHARMACIST, ADMIN).
    - **PUT** – Update user roles.
    - **DELETE** – Remove a user.
- **Audit Logs** – `/api/auth/admin/logs` – GET
    - Returns all system activities and changes made by users.

## Implementation Priorities

### Security:
- Enforce JWT authentication and role-based authorization.
- Use Spring Security to manage access control.

### Scalability:
- Use Spring Data JPA with PostgreSQL for database interactions.
- Optimize query performance using indexing.

### Reliability & Validation:
- Implement data validation to avoid malformed requests.
- Ensure transaction management for atomic operations.

### RESTful API Best Practices:
- Use appropriate HTTP status codes and error handling.
- Implement pagination for large result sets.

## Testing & Deployment
- Unit tests using JUnit and Mockito.
- Integration tests for API flows.
- CI/CD pipeline for automated deployment.

## Sample JSON Examples for APIs

### Login
**Request:**
```json
{
    "username": "john_doe",
    "password": "secure_pass"
}
```
**Response (Success, 200):**
```json
{
    "token": "jwt_token_here"
}
```

### Place an Order
**Request:**
```json
{
    "medicines": [
        { "medicineId": 5, "quantity": 2 },
        { "medicineId": 12, "quantity": 1 }
    ]
}
```
**Response (201 Created):**
```json
{
    "orderId": 123,
    "customer": {
        "userId": 1,
        "username": "john_doe"
    },
    "medicines": [
        { "medicineId": 5, "name": "Paracetamol", "quantity": 2, "price": 5.0 },
        { "medicineId": 12, "name": "Amoxicillin", "quantity": 1, "price": 10.0 }
    ],
    "totalAmount": 20.0,
    "status": "PLACED"
}
```

### Upload Prescription
**Request:**
```json
{
    "customerId": 1,
    "prescription": {
        "doctorName": "Dr. Smith",
        "medicines": [
            { "medicineId": 5, "name": "Paracetamol" },
            { "medicineId": 12, "name": "Amoxicillin" }
        ],
        "validUntil": "2023-12-31"
    }
}
```
**Response (201 Created):**
```json
{
    "prescriptionId": 456,
    "customer": {
        "userId": 1,
        "username": "john_doe"
    },
    "doctorName": "Dr. Smith",
    "medicines": [
        { "medicineId": 5, "name": "Paracetamol" },
        { "medicineId": 12, "name": "Amoxicillin" }
    ],
    "validUntil": "2023-12-31"
}
```

### View Order History
**Response (200 OK):**
```json
[
    {
        "orderId": 123,
        "medicines": [
            { "medicineId": 5, "name": "Paracetamol", "quantity": 2, "price": 5.0 },
            { "medicineId": 12, "name": "Amoxicillin", "quantity": 1, "price": 10.0 }
        ],
        "totalAmount": 20.0,
        "status": "DELIVERED"
    },
    {
        "orderId": 124,
        "medicines": [
            { "medicineId": 7, "name": "Ibuprofen", "quantity": 1, "price": 8.0 }
        ],
        "totalAmount": 8.0,
        "status": "CANCELLED"
    }
]
```
