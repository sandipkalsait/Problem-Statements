# Problem Statement: LegalLink – Legal Case Management System API

## Scenario

LegalLink is an advanced platform designed to streamline legal case management for law firms and corporate legal departments. The system provides a secure, scalable RESTful API backend built with Spring Boot that enables lawyers to manage case files, track court dates, collaborate on documents, and communicate with clients. The platform supports distinct role-based access for attorneys, paralegals, and administrators, ensuring confidentiality, compliance, and efficiency throughout the legal process.

## Business Requirements

### Attorneys
- Create, update, and archive case files.
- Schedule and manage court dates and client meetings.
- Collaborate with paralegals on document preparation and case strategies.

### Paralegals
- Assist in document management, case research, and scheduling.
- Maintain digital case files and update case status.

### Administrators
- Oversee user management, maintain audit logs, and ensure system compliance.
- Configure system settings and generate legal process analytics.

## Security
- Enforce robust JWT-based authentication and strict role-based access control to protect sensitive legal data and maintain client confidentiality.

## Database Schema and Default Data

### User
| Field    | Type         | Description                        |
|----------|--------------|------------------------------------|
| userId   | Integer      | Unique identifier for the user     |
| username | String       | Login username                     |
| password | String       | Hashed password                    |
| roles    | List<Role>   | Assigned roles (ATTORNEY, PARALEGAL, ADMIN) |

### Role
| Field    | Type         | Description                        |
|----------|--------------|------------------------------------|
| roleId   | Integer      | Unique role identifier             |
| roleName | String       | Possible values: ATTORNEY, PARALEGAL, ADMIN |

### CaseFile
| Field      | Type         | Description                        |
|------------|--------------|------------------------------------|
| caseId     | Integer      | Unique case file identifier        |
| title      | String       | Title of the case                  |
| description| String       | Detailed summary of the case       |
| status     | String       | Case status (OPEN, IN_PROGRESS, CLOSED, ARCHIVED) |
| createdBy  | User         | Attorney who created the case      |
| createdAt  | DateTime     | Date and time when the case was created |

### CourtDate
| Field        | Type         | Description                        |
|--------------|--------------|------------------------------------|
| courtDateId  | Integer      | Unique identifier for the court date|
| case         | CaseFile     | Associated case file               |
| date         | DateTime     | Scheduled court appearance date and time |
| location     | String       | Court location                     |
| description  | String       | Brief description of the hearing purpose |

### Document
| Field        | Type         | Description                        |
|--------------|--------------|------------------------------------|
| documentId   | Integer      | Unique document identifier         |
| case         | CaseFile     | Associated case file               |
| title        | String       | Document title                     |
| fileUrl      | String       | URL link to the digital document   |
| uploadedAt   | DateTime     | Timestamp when the document was uploaded |

### ClientCommunication
| Field          | Type         | Description                        |
|----------------|--------------|------------------------------------|
| messageId      | Integer      | Unique message identifier          |
| case           | CaseFile     | Associated case file               |
| sender         | User         | User who sent the message          |
| messageContent | String       | Content of the message             |
| sentAt         | DateTime     | Timestamp when the message was sent |

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication is required for all secure endpoints.
- Role-based access control:
    - **Attorneys**: Create and manage case files, schedule court dates, and collaborate via document uploads and messaging.
    - **Paralegals**: Update case files, assist in document management, and maintain schedules.
    - **Administrators**: Perform full system management including user accounts, audit logs, and compliance monitoring.
- Return `401 Unauthorized` if JWT is missing or invalid.
- Return `403 Forbidden` if a user attempts to access endpoints without the necessary permissions.

### Public Endpoints (No Authentication Required)

#### Login – `/api/public/login` – POST
**Request:**
```json
{
    "username": "attorney_john",
    "password": "secure_pass"
}
```
**Response (200 OK):**
```json
{
    "token": "jwt_token_here"
}
```

#### Case File Search – `/api/public/cases/search` – GET
Returns a list of public case summaries based on query parameters (e.g., status, title).

### Authenticated Endpoints (JWT Required)

#### For Attorneys (`/api/auth/attorney`)

##### Create Case File – `/api/auth/attorney/case` – POST
**Request:**
```json
{
    "title": "Contract Dispute with XYZ Corp",
    "description": "Dispute arising from breach of contract...",
    "status": "OPEN"
}
```
**Response (201 Created):** Returns newly created case file details.

##### Schedule Court Date – `/api/auth/attorney/court-date` – POST
**Request:**
```json
{
    "caseId": 101,
    "date": "2025-09-15T10:00:00",
    "location": "Downtown Court",
    "description": "Pre-trial hearing"
}
```
**Response (201 Created):** Returns court date details.

##### Upload Document – `/api/auth/attorney/document` – POST
**Request:**
```json
{
    "caseId": 101,
    "title": "Evidence Document",
    "fileUrl": "https://example.com/docs/evidence.pdf"
}
```
**Response (201 Created):** Returns document details.

##### Send Client Message – `/api/auth/attorney/message` – POST
**Request:**
```json
{
    "caseId": 101,
    "messageContent": "Please review the attached contract amendments."
}
```
**Response (201 Created):** Confirmation of message sent.

#### For Paralegals (`/api/auth/paralegal`)

##### Update Case File – `/api/auth/paralegal/case/{caseId}` – PUT
**Request:**
```json
{
    "description": "Updated case description...",
    "status": "IN_PROGRESS"
}
```
**Response (200 OK):** Returns updated case file details.

##### Manage Documents – `/api/auth/paralegal/document` – GET/PUT
**Request (GET):**
```json
{
    "caseId": 101
}
```
**Response (200 OK):**
```json
[
    {
        "documentId": 1,
        "title": "Initial Contract",
        "fileUrl": "https://example.com/docs/initial_contract.pdf",
        "uploadedAt": "2025-01-01T10:00:00"
    }
]
```
**Request (PUT):**
```json
{
    "documentId": 1,
    "title": "Updated Contract",
    "fileUrl": "https://example.com/docs/updated_contract.pdf"
}
```
**Response (200 OK):** Returns updated document details.

##### View Scheduled Court Dates – `/api/auth/paralegal/court-dates` – GET
**Response (200 OK):**
```json
[
    {
        "courtDateId": 1,
        "caseId": 101,
        "date": "2025-09-15T10:00:00",
        "location": "Downtown Court",
        "description": "Pre-trial hearing"
    }
]
```

#### For Administrators (`/api/auth/admin`)

##### User Management – `/api/auth/admin/users` – CRUD operations
**Request (POST):**
```json
{
    "username": "new_user",
    "password": "new_password",
    "roles": ["PARALEGAL"]
}
```
**Response (201 Created):** Returns newly created user details.

##### Audit Logs – `/api/auth/admin/logs` – GET
**Response (200 OK):**
```json
[
    {
        "logId": 1,
        "action": "CREATE_CASE",
        "userId": 1,
        "timestamp": "2025-01-01T10:00:00",
        "details": "Case ID 101 created by user ID 1"
    }
]
```

##### System Configuration – `/api/auth/admin/config` – PUT
**Request:**
```json
{
    "settingName": "maxLoginAttempts",
    "value": 5
}
```
**Response (200 OK):** Returns updated configuration details.

## Implementation Priorities

### Security
- Enforce robust JWT authentication and role-based access control using Spring Security.
- Maintain comprehensive audit logs to ensure compliance and traceability of legal actions.

### Scalability
- Utilize Spring Data JPA with PostgreSQL to manage high volumes of case files, documents, and communications.
- Optimize performance through effective indexing and caching of frequently accessed case data.

### Reliability & Validation
- Implement rigorous data validation to prevent submission of incomplete or incorrect case information.
- Ensure atomic transaction management for operations such as case file creation and document uploads.

### RESTful API Best Practices
- Use standardized HTTP status codes and detailed error messages for clarity.
- Implement pagination and filtering for endpoints that return large datasets (e.g., case lists, audit logs).

## Testing & Deployment

### Unit Testing
Leverage JUnit and Mockito to test individual components.

### Integration Testing
Conduct end-to-end tests covering case management, document handling, and client communications.

### CI/CD Pipeline
Automate deployment to facilitate rapid updates and minimize system downtime.