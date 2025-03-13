# Problem Statement: TravelMate – Global Travel Booking & Itinerary Management System

## Scenario
TravelMate is a next-generation travel booking and itinerary management platform designed for global travelers. The system enables seamless booking of flights, hotels, and rental cars, as well as curated travel packages, all through a secure, scalable RESTful API backend built with Spring Boot. The platform supports differentiated access for travelers, travel agents, and administrators to ensure precision in bookings and operational control.

## Business Requirements
### Travelers
- Can search for and book flights, hotels, rental cars, and bundled travel packages
- View booking history
- Manage itineraries

### Travel Agents
- Can manage customer bookings
- Offer personalized itinerary recommendations
- Handle booking modifications

### Administrators
- Maintain comprehensive system control
- Manage users and bookings
- Monitor transaction logs

### Security
- Enforce JWT-based authentication
- Strict role-based access controls for data integrity and secure transactions

## Database Schema and Default Data

### User
| Field   | Type        | Description                      |
|---------|-------------|----------------------------------|
| userId  | Integer     | Unique identifier for the user   |
| username| String      | Login username                   |
| password| String      | Hashed password                  |
| roles   | List<Role>  | Assigned roles (TRAVELER, AGENT, ADMIN) |

### Role
| Field   | Type    | Description                          |
|---------|---------|--------------------------------------|
| roleId  | Integer | Unique role identifier               |
| roleName| String  | Possible values: TRAVELER, AGENT, ADMIN |

### Flight
| Field         | Type    | Description                      |
|---------------|---------|----------------------------------|
| flightId      | Integer | Unique flight identifier         |
| airline       | String  | Airline name                     |
| departure     | String  | Departure airport code           |
| arrival       | String  | Arrival airport code             |
| price         | Double  | Ticket cost                      |
| departureTime | DateTime| Scheduled departure time         |

### Hotel
| Field         | Type    | Description                      |
|---------------|---------|----------------------------------|
| hotelId       | Integer | Unique hotel identifier          |
| name          | String  | Hotel name                       |
| location      | String  | City or area                     |
| pricePerNight | Double  | Cost per night                   |
| availability  | Boolean | Room availability status         |

### RentalCar
| Field         | Type    | Description                      |
|---------------|---------|----------------------------------|
| carId         | Integer | Unique rental car identifier     |
| company       | String  | Rental company name              |
| model         | String  | Car model                        |
| pricePerDay   | Double  | Rental cost per day              |
| availability  | Boolean | Availability status              |

### Booking
| Field         | Type    | Description                      |
|---------------|---------|----------------------------------|
| bookingId     | Integer | Unique booking identifier        |
| traveler      | User    | Customer who made the booking    |
| bookingType   | String  | Type: FLIGHT, HOTEL, CAR, PACKAGE|
| details       | String  | JSON with booking details (IDs, dates, etc.) |
| totalAmount   | Double  | Total cost of the booking        |
| status        | String  | Booking status (CONFIRMED, CANCELLED, PENDING) |

### Itinerary
| Field            | Type    | Description                      |
|------------------|---------|----------------------------------|
| itineraryId      | Integer | Unique itinerary identifier      |
| booking          | Booking | Associated booking details       |
| itineraryDetails | String  | Consolidated travel itinerary information |

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication is mandatory for all secure endpoints.
- Role-based access control:
    - **Travelers**: Can search and book travel services, view booking history, and manage itineraries.
    - **Travel Agents**: Can manage bookings, provide personalized recommendations, and process modifications.
    - **Administrators**: Have full system management privileges including user and transaction oversight.
- Return `401 Unauthorized` if JWT is missing or invalid.
- Return `403 Forbidden` when access is attempted without sufficient privileges.

### Public Endpoints (No Authentication Required)
- **Login** – `/api/public/login` – POST
    - **Request:**
        ```json
        {
            "username": "traveler1",
            "password": "secure_pass"
        }
        ```
    - **Response (200 OK):**
        ```json
        {
            "token": "jwt_token_here"
        }
        ```
- **Search Flights/Hotels/Cars** – `/api/public/search` – GET
    - Accepts query parameters (e.g., destination, dates) and returns matching travel options.

### Authenticated Endpoints (JWT Required)
#### For Travelers (`/api/auth/traveler`)
- **Book a Flight/Hotel/Car** – `/api/auth/traveler/booking` – POST
    - **Request:**
        ```json
        {
            "bookingType": "FLIGHT",
            "details": { "flightId": 101, "departureDate": "2025-06-15T09:00:00" }
        }
        ```
    - **Response (201 Created):** Returns booking confirmation with booking details.
- **View Booking History** – `/api/auth/traveler/bookings` – GET
    - Retrieves a list of past and upcoming bookings.
- **Manage Itinerary** – `/api/auth/traveler/itinerary` – GET/PUT
    - Allows travelers to view or update their consolidated travel itinerary.

#### For Travel Agents (`/api/auth/agent`)
- **Manage Customer Bookings** – `/api/auth/agent/bookings` – GET/PUT
    - Retrieve and update booking details for assigned customers.
- **Personalized Itinerary Recommendations** – `/api/auth/agent/recommendations` – GET
    - Generate tailored travel itineraries based on customer preferences.

#### For Administrators (`/api/auth/admin`)
- **User Management** – `/api/auth/admin/users` – CRUD operations
    - Manage traveler, agent, and admin accounts.
- **Transaction Audit Logs** – `/api/auth/admin/logs` – GET
    - Retrieve comprehensive logs of bookings, cancellations, and payment transactions.

## Implementation Priorities
### Security
- Enforce robust JWT authentication and precise role-based authorization via Spring Security.
- Implement comprehensive audit logging and transaction monitoring to ensure compliance and data integrity.

### Scalability
- Leverage Spring Data JPA with PostgreSQL to support high-volume booking transactions.
- Optimize database performance through strategic indexing and query tuning.

### Reliability & Validation
- Implement strict data validation to prevent malformed requests.
- Ensure atomic transaction management to maintain data consistency during simultaneous bookings.

### RESTful API Best Practices
- Use standardized HTTP status codes and detailed error messaging.
- Integrate pagination and filtering on endpoints returning large datasets.

## Testing & Deployment
- **Unit Testing**: Use JUnit and Mockito to validate individual service components.
- **Integration Testing**: Conduct comprehensive end-to-end tests of booking flows.
- **CI/CD Pipeline**: Establish automated deployment processes to streamline updates and minimize downtime.

## Sample JSON Examples for APIs

### Login
**Request:**
```json
{
    "username": "traveler1",
    "password": "secure_pass"
}
```
**Response:**
```json
{
    "token": "jwt_token_here"
}
```

### Book a Flight
**Request:**
```json
{
    "bookingType": "FLIGHT",
    "details": {
        "flightId": 101,
        "departureDate": "2025-06-15T09:00:00"
    }
}
```
**Response:**
```json
{
    "bookingId": 5501,
    "traveler": { "userId": 210, "username": "traveler1" },
    "bookingType": "FLIGHT",
    "totalAmount": 350.00,
    "status": "CONFIRMED"
}
```

### View Booking History (Traveler)
**Response:**
```json
[
    {
        "bookingId": 5501,
        "bookingType": "FLIGHT",
        "totalAmount": 350.00,
        "status": "CONFIRMED",
        "details": { "flightId": 101, "departureDate": "2025-06-15T09:00:00" }
    },
    {
        "bookingId": 5502,
        "bookingType": "HOTEL",
        "totalAmount": 450.00,
        "status": "CONFIRMED",
        "details": { "hotelId": 202, "checkIn": "2025-06-15", "checkOut": "2025-06-20" }
    }
]
```