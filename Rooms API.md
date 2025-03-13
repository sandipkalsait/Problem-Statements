# Problem Statement: Loge – Smart Room Management and Security System API

## Scenario:
Lodge is a state-of-the-art smart room management and security platform that integrates IoT devices, sensors, smart cameras, and automated alarms into a unified management system. Built with Spring Boot, the RESTful API backend empowers room owners to control and monitor their room environments remotely, enables service technicians to diagnose issues in real time, and provides administrators with comprehensive oversight for system health and user management.

## Business Requirements:
### Room Owners:
- Remotely manage smart devices (lights, thermostats, locks, cameras, etc.).
- View real-time security camera feeds and receive instant alerts for anomalies.
- Schedule automated routines and monitor energy consumption.

### Service Technicians:
- Access diagnostic data from smart devices to troubleshoot issues.
- Schedule and perform maintenance on malfunctioning equipment.
- Receive alerts on critical device failures requiring on-site intervention.

### Administrators:
- Oversee user accounts, device registrations, and system configurations.
- Monitor audit logs, analyze system performance metrics, and ensure compliance.

### Security:
- Utilize JWT-based authentication and enforce strict role-based access control.
- Ensure data integrity and privacy for all smart room interactions.

## Database Schema and Default Data
### User
| Field   | Type       | Description                        |
|---------|------------|------------------------------------|
| userId  | Integer    | Unique identifier for the user     |
| username| String     | Login username                     |
| password| String     | Hashed password                    |
| roles   | List<Role> | Assigned roles (OWNER, TECHNICIAN, ADMIN) |

### Role
| Field   | Type    | Description                        |
|---------|---------|------------------------------------|
| roleId  | Integer | Unique role identifier             |
| roleName| String  | Possible values: OWNER, TECHNICIAN, ADMIN |

### Device
| Field   | Type    | Description                        |
|---------|---------|------------------------------------|
| deviceId| Integer | Unique device identifier           |
| name    | String  | Device name (e.g., "Room Thermostat") |
| type    | String  | Device type (e.g., LIGHT, THERMOSTAT, LOCK, CAMERA) |
| status  | String  | Operational status (ONLINE, OFFLINE, MALFUNCTIONING) |
| location| String  | Physical location within the room  |
| owner   | User    | Associated room owner              |

### DeviceStatus
| Field     | Type    | Description                        |
|-----------|---------|------------------------------------|
| statusId  | Integer | Unique status record identifier    |
| device    | Device  | Associated device                  |
| timestamp | DateTime| Time of the status update          |
| statusInfo| String  | Detailed status message (e.g., "Battery Low") |

### Alert
| Field     | Type    | Description                        |
|-----------|---------|------------------------------------|
| alertId   | Integer | Unique alert identifier            |
| device    | Device  | Device generating the alert        |
| alertType | String  | Type of alert (SECURITY, MAINTENANCE, ENERGY) |
| message   | String  | Alert message describing the event |
| issuedAt  | DateTime| Timestamp when the alert was issued|

### MaintenanceRequest
| Field              | Type    | Description                        |
|--------------------|---------|------------------------------------|
| requestId          | Integer | Unique maintenance request identifier |
| device             | Device  | Device requiring maintenance       |
| technician         | User    | Technician assigned to the request (if applicable) |
| requestDescription | String  | Description of the issue           |
| status             | String  | Request status (PENDING, IN_PROGRESS, COMPLETED) |
| requestedAt        | DateTime| Timestamp when the maintenance was requested |

## API Requirements and Endpoints
### Security & Authentication
- **JWT Authentication:** Mandatory for all protected endpoints.
- **Role-based Access Control:**
    - **Room Owners:** Manage devices, view alerts, schedule routines, and monitor energy usage.
    - **Technicians:** Access device diagnostics, update maintenance statuses, and receive alerts on critical issues.
    - **Administrators:** Execute full system management, including user and device administration, audit logging, and configuration adjustments.
- Return `401 Unauthorized` if the JWT is missing or invalid.
- Return `403 Forbidden` if a user lacks the required privileges.

### Public Endpoints (No Authentication Required)
- **Login – `/api/public/login` – POST**
    - **Request:**
        ```json
        {
            "username": "owner_jane",
            "password": "secure_pass"
        }
        ```
    - **Response (200 OK):**
        ```json
        {
            "token": "jwt_token_here"
        }
        ```

- **Device Discovery – `/api/public/devices/search?location=Room%20A` – GET**
    - Returns a list of publicly discoverable devices based on location.

### Authenticated Endpoints (JWT Required)
#### For Room Owners (`/api/auth/owner`)
- **Manage Devices – `/api/auth/owner/device` – CRUD operations**
    - **POST:** Register a new smart device.
    - **PUT:** Update device settings or location.
    - **GET:** Retrieve a list of devices owned by the user.

- **View Device Status & Alerts – `/api/auth/owner/device/{deviceId}/status` – GET**
    - Retrieves real-time status updates and alerts for a specific device.

- **Schedule Automation Routine – `/api/auth/owner/automation` – POST**
    - **Request:**
        ```json
        {
            "deviceId": 101,
            "action": "TURN_ON",
            "scheduledTime": "2025-05-01T07:00:00"
        }
        ```
    - **Response (201 Created):** Returns confirmation of scheduled action.

#### For Technicians (`/api/auth/technician`)
- **Access Device Diagnostics – `/api/auth/technician/device/{deviceId}/diagnostics` – GET**
    - Retrieves detailed diagnostic data for troubleshooting.

- **Update Maintenance Request Status – `/api/auth/technician/maintenance/{requestId}` – PUT**
    - **Request:**
        ```json
        {
            "status": "COMPLETED"
        }
        ```
    - **Response (200 OK):** Confirmation of maintenance update.

- **Log Maintenance Activity – `/api/auth/technician/maintenance/log` – POST**
    - **Request:**
        ```json
        {
            "deviceId": 101,
            "requestDescription": "Replaced faulty sensor",
            "status": "COMPLETED"
        }
        ```
    - **Response (201 Created):** Logs maintenance activity details.

#### For Administrators (`/api/auth/admin`)
- **User & Device Management – `/api/auth/admin/users` and `/api/auth/admin/devices` – CRUD operations**
    - Manage user accounts and oversee device registrations.

- **Audit Logs – `/api/auth/admin/logs` – GET**
    - Retrieve comprehensive logs of device interactions, user actions, and alert events.

- **System Configuration – `/api/auth/admin/config` – PUT**
    - Update platform settings, integration parameters, and security policies.

## Implementation Priorities
### Security:
- Implement robust JWT authentication and strict role-based access control using Spring Security.
- Ensure comprehensive audit logging for all critical operations to support compliance and traceability.

### Scalability:
- Leverage Spring Data JPA with PostgreSQL to manage large volumes of device data, alerts, and user interactions.
- Optimize system performance through efficient indexing, caching strategies, and load balancing.

### Reliability & Validation:
- Enforce rigorous data validation to prevent malformed requests and ensure data integrity.
- Ensure atomic transaction management for operations such as device updates and maintenance requests.

### RESTful API Best Practices:
- Utilize standardized HTTP status codes and clear, descriptive error messages.
- Implement pagination and filtering on endpoints that return extensive datasets (e.g., device lists, audit logs).

## Testing & Deployment
- **Unit Testing:** Use JUnit and Mockito to validate individual API components.
- **Integration Testing:** Conduct end-to-end tests covering device registration, status monitoring, and maintenance workflows.
- **CI/CD Pipeline:** Automate deployment processes to ensure rapid, reliable updates and minimize downtime.