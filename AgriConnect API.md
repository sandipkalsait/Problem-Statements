# Problem Statement: AgriConnect – Smart Agriculture Management System

## Scenario:
AgriConnect is a cutting-edge agriculture management platform designed to empower farmers, agronomists, and agri-businesses with real-time data and streamlined operations. The system provides a robust, scalable RESTful API backend built with Spring Boot to manage farm operations, crop monitoring, weather analytics, and supply chain logistics. It ensures secure access via role-based controls to facilitate data-driven decision-making across the agricultural value chain.

## Business Requirements:
- **Farmers**: Can manage farm profiles, monitor crop growth, schedule irrigation, and receive real-time weather alerts.
- **Agronomists**: Access detailed field analytics, provide crop health insights, and recommend precision farming techniques.
- **Agri-Businesses**: Track crop yield forecasts, manage supply chain logistics, and optimize market distribution.
- **Security**: Implement JWT-based authentication to protect all endpoints and enforce strict role-based access control.

## Database Schema and Default Data

### User
| Field   | Type        | Description                           |
|---------|-------------|---------------------------------------|
| userId  | Integer     | Unique identifier for the user        |
| username| String      | Login username                        |
| password| String      | Hashed password                       |
| roles   | List<Role>  | Assigned roles (FARMER, AGRONOMIST, BUSINESS) |

### Role
| Field   | Type        | Description                           |
|---------|-------------|---------------------------------------|
| roleId  | Integer     | Unique role identifier                |
| roleName| String      | Possible values: FARMER, AGRONOMIST, BUSINESS |

### Farm
| Field   | Type        | Description                           |
|---------|-------------|---------------------------------------|
| farmId  | Integer     | Unique farm identifier                |
| name    | String      | Name of the farm                      |
| location| String      | Geographic location (address, region) |
| size    | Double      | Area of the farm (acres/hectares)     |
| owner   | User        | Associated farmer                     |

### Crop
| Field          | Type        | Description                           |
|----------------|-------------|---------------------------------------|
| cropId         | Integer     | Unique crop identifier                |
| farm           | Farm        | Associated farm                       |
| type           | String      | Crop type (e.g., Wheat, Corn, Soy)    |
| plantingDate   | Date        | Date when the crop was planted        |
| expectedHarvest| Date        | Anticipated harvest date              |
| status         | String      | Growth status (PLANTED, GROWING, READY_FOR_HARVEST) |

### WeatherAlert
| Field   | Type        | Description                           |
|---------|-------------|---------------------------------------|
| alertId | Integer     | Unique weather alert identifier       |
| location| String      | Location of the alert                 |
| alertType| String     | Type (RAIN, DROUGHT, FROST, STORM)    |
| severity| String      | Severity level (LOW, MEDIUM, HIGH)    |
| issuedAt| DateTime    | Timestamp of the alert                |

### SupplyChain
| Field           | Type        | Description                           |
|-----------------|-------------|---------------------------------------|
| shipmentId      | Integer     | Unique shipment identifier            |
| crop            | Crop        | Associated crop shipment              |
| business        | User        | Agri-business managing the shipment   |
| status          | String      | Shipment status (PENDING, IN_TRANSIT, DELIVERED) |
| expectedDelivery| DateTime    | Estimated delivery time               |

## API Requirements and Endpoints

### Security & Authentication
- **JWT Authentication**: Mandatory for all protected endpoints.
- **Role-based access control**:
    - Farmers: Manage farm details, monitor crops, and receive weather alerts.
    - Agronomists: Access field analytics, provide crop health insights, and recommend interventions.
    - Agri-Businesses: Manage crop shipments and track yield forecasts.
- Return `401 Unauthorized` if the JWT is missing or invalid.
- Return `403 Forbidden` when a user lacks the required role for an endpoint.

### Public Endpoints (No Authentication Required)
- **Login** – `/api/public/login` – POST
    - **Request**:
        ```json
        {
            "username": "farmer_john",
            "password": "secure_pass"
        }
        ```
    - **Response (200 OK)**:
        ```json
        {
            "token": "jwt_token_here"
        }
        ```
- **Browse Weather Alerts** – `/api/public/weather/alerts?location=RegionX` – GET
    - Returns current weather alerts for a specified location.

### Authenticated Endpoints (JWT Required)

#### For Farmers (`/api/auth/farmer`)
- **Manage Farm Profile** – `/api/auth/farmer/farm` – CRUD operations
    - **POST**: Create a new farm profile.
    - **PUT**: Update farm details.
- **Monitor Crop Status** – `/api/auth/farmer/crops` – GET
    - Retrieves crop details and growth status for the farmer’s fields.
- **Receive Weather Alerts** – `/api/auth/farmer/weather` – GET
    - Get personalized weather alerts based on farm location.

#### For Agronomists (`/api/auth/agronomist`)
- **View Field Analytics** – `/api/auth/agronomist/fields` – GET
    - Retrieves detailed analytics on crop health and soil conditions.
- **Submit Crop Recommendations** – `/api/auth/agronomist/recommendations` – POST
    - **Request**:
        ```json
        {
            "cropId": 205,
            "recommendation": "Increase irrigation frequency due to low moisture levels."
        }
        ```
    - **Response (200 OK)**: Confirmation of the recommendation.

#### For Agri-Businesses (`/api/auth/business`)
- **Manage Crop Shipments** – `/api/auth/business/shipments` – CRUD operations
    - **POST**: Initiate a new crop shipment.
    - **PUT**: Update shipment status.
- **Track Yield Forecasts** – `/api/auth/business/yields` – GET
    - Provides forecasts and analytics on crop yields.

## Implementation Priorities

### Security:
- Enforce robust JWT authentication and strict role-based access using Spring Security.
- Implement comprehensive audit logging for all critical operations.

### Scalability:
- Utilize Spring Data JPA with PostgreSQL to efficiently manage high volumes of farm and crop data.
- Optimize database queries and index critical tables for rapid retrieval.

### Reliability & Validation:
- Integrate rigorous data validation to prevent malformed or erroneous requests.
- Ensure atomic transaction management across multi-step operations (e.g., crop monitoring and shipment tracking).

### RESTful API Best Practices:
- Use standardized HTTP status codes for error handling and success responses.
- Implement pagination and filtering on endpoints returning large datasets (e.g., crop lists, weather alerts).

## Sample JSON Examples for APIs

### Login
- **Request**:
    ```json
    {
        "username": "farmer_john",
        "password": "secure_pass"
    }
    ```
- **Response**:
    ```json
    {
        "token": "jwt_token_here"
    }
    ```

### Create Farm Profile (Farmer)
- **Request**:
    ```json
    {
        "name": "Green Acres",
        "location": "RegionX",
        "size": 150.5
    }
    ```
- **Response**:
    ```json
    {
        "farmId": 101,
        "name": "Green Acres",
        "location": "RegionX",
        "size": 150.5
    }
    ```

### View Crop Status (Farmer)
- **Response**:
    ```json
    [
        {
            "cropId": 205,
            "type": "Corn",
            "plantingDate": "2025-03-15",
            "expectedHarvest": "2025-09-15",
            "status": "GROWING"
        },
        {
            "cropId": 206,
            "type": "Wheat",
            "plantingDate": "2025-02-10",
            "expectedHarvest": "2025-07-10",
            "status": "READY_FOR_HARVEST"
        }
    ]
    ```

### Initiate Crop Shipment (Agri-Business)
- **Request**:
    ```json
    {
        "cropId": 205,
        "expectedDelivery": "2025-10-01T08:00:00"
    }
    ```
- **Response**:
    ```json
    {
        "shipmentId": 3101,
        "status": "PENDING"
    }
    ```