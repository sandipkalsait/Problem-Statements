# Problem Statement: FleetSync – Smart Fleet Management System

## Scenario:
FleetSync is a next-generation fleet management system designed for logistics companies to track and optimize vehicle usage in real-time. The company needs a RESTful API backend using Spring Boot to manage vehicles, drivers, trips, and maintenance schedules. The system should ensure secure role-based access using JWT authentication and support real-time tracking, vehicle assignment, and automated alerts.

## Business Requirements
- **Fleet Managers** can register vehicles, assign drivers, schedule maintenance, and view analytics.
- **Drivers** can view their assigned vehicles, update trip statuses, and report issues.
- **Admins** have full system control, including user management and audit logs.
- **Security**: All endpoints must be secured using JWT authentication with role-based access control.

## Database Schema and Default Data

### User
| Field   | Type        | Description           |
|---------|-------------|-----------------------|
| userId  | Integer     | Unique user ID        |
| username| String      | Username for login    |
| password| String      | Hashed password       |
| roles   | List<Role>  | Assigned roles (DRIVER, MANAGER, ADMIN) |

### Role
| Field   | Type    | Description                        |
|---------|---------|------------------------------------|
| roleId  | Integer | Unique ID for role                 |
| roleName| String  | Possible values: DRIVER, MANAGER, ADMIN |

### Vehicle
| Field         | Type    | Description                        |
|---------------|---------|------------------------------------|
| vehicleId     | Integer | Unique vehicle ID                  |
| licensePlate  | String  | Vehicle registration number        |
| model         | String  | Vehicle model                      |
| status        | String  | ACTIVE, UNDER_MAINTENANCE, IN_USE  |
| assignedDriver| User    | (Nullable) Currently assigned driver |

### Trip
| Field          | Type      | Description                        |
|----------------|-----------|------------------------------------|
| tripId         | Integer   | Unique trip ID                     |
| vehicle        | Vehicle   | Assigned vehicle                   |
| driver         | User      | Assigned driver                    |
| startTime      | Timestamp | Trip start time                    |
| endTime        | Timestamp | Trip end time                      |
| status         | String    | ONGOING, COMPLETED, CANCELLED      |
| distanceCovered| Double    | Distance traveled (km)             |

### MaintenanceSchedule
| Field         | Type    | Description                        |
|---------------|---------|------------------------------------|
| maintenanceId | Integer | Unique ID for maintenance          |
| vehicle       | Vehicle | Vehicle under maintenance          |
| dateScheduled | Date    | Planned maintenance date           |
| status        | String  | SCHEDULED, COMPLETED               |

## API Requirements and Endpoints

### Security & Authentication
- JWT Authentication is required for all private endpoints.
- Role-based access control:
    - Drivers can only view their assigned vehicles and update trip statuses.
    - Fleet Managers can manage vehicles, assign trips, and schedule maintenance.
    - Admins have full system control.
- If JWT is missing, return 401 Unauthorized.
- If an endpoint is accessed with an incorrect role, return 403 Forbidden.

### Public Endpoints (No Authentication Required)

#### Login – `/api/public/login` – POST
**Request:**
```json
{
    "username": "fleet_manager",
    "password": "secure_pass"
}
```
**Response (Success, 200):**
```json
{
    "token": "jwt_token_here"
}
```
**Failure (401 Unauthorized) for incorrect credentials.**

### Authenticated Endpoints (JWT Required)

#### For Drivers (`/api/auth/driver`)

##### View Assigned Vehicle – `/api/auth/driver/vehicle` – GET
Returns details of the vehicle assigned to the logged-in driver.
**Response (200 OK):**
```json
{
    "vehicleId": 2,
    "licensePlate": "ABC123",
    "model": "Toyota Prius",
    "status": "IN_USE",
    "assignedDriver": {
        "userId": 5,
        "username": "driver1"
    }
}
```

##### Start Trip – `/api/auth/driver/trip/start` – POST
**Request:**
```json
{
    "vehicleId": 2,
    "startTime": "2025-03-12T08:00:00Z"
}
```
**Response (201 Created):**
```json
{
    "tripId": 10,
    "vehicle": {
        "vehicleId": 2,
        "licensePlate": "ABC123",
        "model": "Toyota Prius"
    },
    "driver": {
        "userId": 5,
        "username": "driver1"
    },
    "startTime": "2025-03-12T08:00:00Z",
    "status": "ONGOING"
}
```

##### Update Trip Status – `/api/auth/driver/trip/{tripId}/update` – PUT
Allows drivers to update a trip's distance and end time.
**Request:**
```json
{
    "endTime": "2025-03-12T12:30:00Z",
    "distanceCovered": 250.0
}
```
**Response (200 OK):**
```json
{
    "tripId": 10,
    "endTime": "2025-03-12T12:30:00Z",
    "distanceCovered": 250.0,
    "status": "COMPLETED"
}
```

##### Report Issue – `/api/auth/driver/issue` – POST
Drivers can report vehicle issues, which fleet managers will review.
**Request:**
```json
{
    "vehicleId": 2,
    "issueDescription": "Engine noise detected"
}
```
**Response (201 Created):**
```json
{
    "issueId": 15,
    "vehicleId": 2,
    "issueDescription": "Engine noise detected",
    "reportedBy": {
        "userId": 5,
        "username": "driver1"
    },
    "status": "REPORTED"
}
```

#### For Fleet Managers (`/api/auth/manager`)

##### Manage Vehicles – `/api/auth/manager/vehicle`
- **GET** – View all vehicles.
**Response (200 OK):**
```json
[
    {
        "vehicleId": 1,
        "licensePlate": "XYZ789",
        "model": "Ford Transit",
        "status": "ACTIVE"
    },
    {
        "vehicleId": 2,
        "licensePlate": "ABC123",
        "model": "Toyota Prius",
        "status": "IN_USE"
    }
]
```
- **POST** – Add a new vehicle.
**Request:**
```json
{
    "licensePlate": "DEF456",
    "model": "Honda Civic",
    "status": "ACTIVE"
}
```
**Response (201 Created):**
```json
{
    "vehicleId": 3,
    "licensePlate": "DEF456",
    "model": "Honda Civic",
    "status": "ACTIVE"
}
```
- **PUT** – Update vehicle details.
**Request:**
```json
{
    "vehicleId": 2,
    "licensePlate": "ABC123",
    "model": "Toyota Prius",
    "status": "UNDER_MAINTENANCE"
}
```
**Response (200 OK):**
```json
{
    "vehicleId": 2,
    "licensePlate": "ABC123",
    "model": "Toyota Prius",
    "status": "UNDER_MAINTENANCE"
}
```
- **DELETE** – Remove a vehicle.
**Response (200 OK):**
```json
{
    "message": "Vehicle removed successfully"
}
```

##### Assign Vehicle to Driver – `/api/auth/manager/vehicle/{vehicleId}/assign` – PUT
**Request:**
```json
{
    "driverId": 7
}
```
**Response (200 OK):**
```json
{
    "vehicleId": 2,
    "assignedDriver": {
        "userId": 7,
        "username": "driver2"
    }
}
```

##### View and Approve Trips – `/api/auth/manager/trip`
- **GET** – View all trips.
**Response (200 OK):**
```json
[
    {
        "tripId": 10,
        "vehicle": {
            "vehicleId": 2,
            "licensePlate": "ABC123",
            "model": "Toyota Prius"
        },
        "driver": {
            "userId": 5,
            "username": "driver1"
        },
        "startTime": "2025-03-12T08:00:00Z",
        "endTime": "2025-03-12T12:30:00Z",
        "status": "COMPLETED",
        "distanceCovered": 250.0
    }
]
```
- **PUT** – Approve completed trips.
**Request:**
```json
{
    "tripId": 10,
    "status": "APPROVED"
}
```
**Response (200 OK):**
```json
{
    "tripId": 10,
    "status": "APPROVED"
}
```

##### Schedule Maintenance – `/api/auth/manager/maintenance` – POST
**Request:**
```json
{
    "vehicleId": 3,
    "dateScheduled": "2025-03-20"
}
```
**Response (201 Created):**
```json
{
    "maintenanceId": 5,
    "vehicle": {
        "vehicleId": 3,
        "licensePlate": "DEF456",
        "model": "Honda Civic"
    },
    "dateScheduled": "2025-03-20",
    "status": "SCHEDULED"
}
```

#### For Admins (`/api/auth/admin`)

##### User Management – `/api/auth/admin/users`
- **GET** – View all users.
**Response (200 OK):**
```json
[
    {
        "userId": 1,
        "username": "admin",
        "roles": ["ADMIN"]
    },
    {
        "userId": 2,
        "username": "fleet_manager",
        "roles": ["MANAGER"]
    }
]
```
- **POST** – Create a new user.
**Request:**
```json
{
    "username": "new_user",
    "password": "new_password",
    "roles": ["DRIVER"]
}
```
**Response (201 Created):**
```json
{
    "userId": 3,
    "username": "new_user",
    "roles": ["DRIVER"]
}
```
- **PUT** – Update user roles.
**Request:**
```json
{
    "userId": 3,
    "roles": ["MANAGER"]
}
```
**Response (200 OK):**
```json
{
    "userId": 3,
    "username": "new_user",
    "roles": ["MANAGER"]
}
```
- **DELETE** – Remove a user.
**Response (200 OK):**
```json
{
    "message": "User removed successfully"
}
```

##### Audit Logs – `/api/auth/admin/logs` – GET
Returns all system activities.
**Response (200 OK):**
```json
[
    {
        "logId": 1,
        "action": "USER_LOGIN",
        "timestamp": "2025-03-12T08:00:00Z",
        "details": "User 'fleet_manager' logged in"
    },
    {
        "logId": 2,
        "action": "VEHICLE_ASSIGNED",
        "timestamp": "2025-03-12T09:00:00Z",
        "details": "Vehicle 'ABC123' assigned to driver 'driver1'"
    }
]
```

## Implementation Priorities

### Security:
- JWT authentication for role-based access.
- Spring Security to enforce role checks.

### Scalability:
- Spring Data JPA with PostgreSQL.
- Optimized queries with indexing.

### Reliability & Validation:
- Strict data validation.
- Transaction management for atomic operations.

### RESTful API Best Practices:
- Proper HTTP status codes and error handling.
- Pagination for large datasets.
