# Smart Society API

## Problem Statement: Smart Society Connect – Urban Infrastructure & Public Services Management API

### Scenario:
Smart Society Connect is a forward-thinking urban management platform engineered to streamline public services across metropolitan areas. The system integrates transportation management, public safety, waste collection, and event coordination into a unified, secure, and scalable RESTful API backend built with Spring Boot. The platform is designed to empower city officials, service providers, and administrators with real-time insights and efficient control over urban infrastructure.

### Business Requirements:
#### City Officials:
- Monitor and manage public service requests and incident reports.
- Oversee transportation schedules and public event approvals.
- Access comprehensive analytics and generate performance reports.

#### Service Providers:
- Update service status and manage daily operations (e.g., waste collection, transit services).
- Coordinate schedule adjustments and address incident escalations.

#### Administrators:
- Conduct full system oversight including user management, audit logging, and configuration adjustments.
- Ensure system compliance with municipal regulations and data security protocols.

### Security:
- Implement robust JWT-based authentication and strict role-based access control to safeguard all endpoints.

## Database Schema and Default Data

### User
| Field   | Type          | Description                          |
|---------|---------------|--------------------------------------|
| userId  | Integer       | Unique identifier for the user       |
| username| String        | Login username                       |
| password| String        | Hashed password                      |
| roles   | List<Role>    | Assigned roles (CITY_OFFICIAL, SERVICE_PROVIDER, ADMIN) |

### Role
| Field   | Type          | Description                          |
|---------|---------------|--------------------------------------|
| roleId  | Integer       | Unique role identifier               |
| roleName| String        | Possible values: CITY_OFFICIAL, SERVICE_PROVIDER, ADMIN |

### Service
| Field   | Type          | Description                          |
|---------|---------------|--------------------------------------|
| serviceId | Integer     | Unique identifier for the public service |
| name      | String      | Name of the service (e.g., Waste Collection, Transit) |
| description | String    | Detailed description of the service  |
| status    | String      | Operational status (ACTIVE, SCHEDULED, UNDER_MAINTENANCE) |
| provider  | User        | Service provider responsible for the service |

### IncidentReport
| Field     | Type        | Description                          |
|-----------|-------------|--------------------------------------|
| reportId  | Integer     | Unique incident report identifier    |
| reportedBy| User        | User (usually a city official) who filed the report |
| service   | Service     | Associated service affected by the incident |
| description | String    | Detailed account of the incident     |
| location  | String      | Geographic location of the incident  |
| status    | String      | Report status (OPEN, IN_PROGRESS, RESOLVED, CLOSED) |
| reportedAt| DateTime    | Timestamp when the incident was reported |

### TransportationSchedule
| Field       | Type      | Description                          |
|-------------|-----------|--------------------------------------|
| scheduleId  | Integer   | Unique transportation schedule identifier |
| service     | Service   | Associated transit service           |
| departureTime | DateTime| Scheduled departure time             |
| arrivalTime | DateTime  | Scheduled arrival time               |
| route       | String    | Description of the transit route     |
| status      | String    | Schedule status (ON_TIME, DELAYED, CANCELLED) |

### PublicEvent
| Field     | Type        | Description                          |
|-----------|-------------|--------------------------------------|
| eventId   | Integer     | Unique public event identifier       |
| name      | String      | Event name                           |
| location  | String      | Venue or area where the event will be held |
| startTime | DateTime    | Event start time                     |
| endTime   | DateTime    | Event end time                       |
| description | String    | Brief overview of the event          |
| status    | String      | Event status (PLANNED, ONGOING, COMPLETED, CANCELLED) |

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication is mandatory for all secure endpoints.
- Role-based access control:
    - City Officials: Manage incident reports, public events, and transportation schedules.
    - Service Providers: Update and manage service statuses and daily operations.
    - Administrators: Oversee full system operations, user management, and audit logs.
- Return `401 Unauthorized` if JWT is missing or invalid.
- Return `403 Forbidden` if a user attempts to access endpoints without proper permissions.

### Public Endpoints (No Authentication Required)
#### Login – `/api/public/login` – POST
**Request:**
```json
{
        "username": "official_jane",
        "password": "secure_pass"
}
```
**Response (200 OK):**
```json
{
        "token": "jwt_token_here"
}
```

#### Browse Public Events – `/api/public/events` – GET
Returns a list of approved public events.

### Authenticated Endpoints (JWT Required)
#### For City Officials (`/api/auth/official`)
- **Manage Incident Reports** – `/api/auth/official/incidents` – GET/POST/PUT
    - **GET:** Retrieve incident reports.
    - **POST:** File a new incident report.
    - **PUT:** Update report status.

- **Oversee Transportation Schedules** – `/api/auth/official/transportation` – GET
    - Retrieves transit schedules and statuses.

- **Approve Public Events** – `/api/auth/official/events` – PUT
    - Update event status to APPROVED or REJECTED.

#### For Service Providers (`/api/auth/provider`)
- **Update Service Status** – `/api/auth/provider/service/{serviceId}` – PUT
    - Modify service details and operational status.

- **Report Operational Metrics** – `/api/auth/provider/metrics` – POST
    - Submit daily performance data and incident resolutions.

#### For Administrators (`/api/auth/admin`)
- **User Management** – `/api/auth/admin/users` – CRUD operations
    - Manage accounts and assign roles.

- **Audit Logs** – `/api/auth/admin/logs` – GET
    - Retrieve comprehensive logs of system activities.

- **System Configuration** – `/api/auth/admin/config` – PUT
    - Update platform settings and manage integration configurations.

## Implementation Priorities

### Security:
- Enforce stringent JWT authentication and precise role-based access control using Spring Security.
- Implement comprehensive audit logging to ensure regulatory compliance and operational transparency.

### Scalability:
- Utilize Spring Data JPA with PostgreSQL to handle high volumes of service and incident data.
- Optimize performance through efficient database indexing and caching strategies.

### Reliability & Validation:
- Implement rigorous data validation to safeguard against malformed requests.
- Ensure atomic transaction management for operations such as service updates and incident resolution.

### RESTful API Best Practices:
- Use standardized HTTP status codes and clear error messaging.
- Incorporate pagination and filtering for endpoints returning large datasets (e.g., incident reports, event lists).

## Testing & Deployment
- **Unit Testing:** Leverage JUnit and Mockito to validate API components.
- **Integration Testing:** Conduct end-to-end tests covering incident management, service updates, and event approvals.
- **CI/CD Pipeline:** Automate deployment processes to minimize downtime and streamline updates.

## Sample JSON Examples for APIs

### Login
**Request:**
```json
{
        "username": "official_jane",
        "password": "secure_pass"
}
```
**Response:**
```json
{
        "token": "jwt_token_here"
}
```

### File an Incident Report (City Official)
**Request:**
```json
{
        "serviceId": 102,
        "description": "Overflowing trash bins near Central Park.",
        "location": "Central Park, Zone 3"
}
```
**Response:**
```json
{
        "reportId": 5001,
        "status": "OPEN",
        "reportedAt": "2025-03-13T10:00:00"
}
```

### Update Service Status (Service Provider)
**Request:**
```json
{
        "status": "UNDER_MAINTENANCE"
}
```
**Response:**
```json
{
        "serviceId": 102,
        "status": "UNDER_MAINTENANCE"
}
```

### Approve a Public Event (City Official)
**Request:**
```json
{
        "status": "APPROVED"
}
```
**Response:**
```json
{
        "eventId": 3001,
        "status": "APPROVED"
}
```