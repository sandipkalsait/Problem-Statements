# Problem Statement: HRLink – Employee Management and Payroll System

## Scenario

HRLink is a modern, comprehensive Employee Management and Payroll System designed to streamline HR operations for organizations. The system provides a secure, scalable RESTful API backend built with Spring Boot to manage employee records, leave applications, timesheets, and payroll processing. It supports role-based access for employees, HR managers, and administrators, ensuring seamless management of human resources and payroll processes.

## Business Requirements

### Employees can:
- View and update their personal profiles.
- Submit leave requests and track approval status.
- Record daily work hours via timesheets.
- Access digital payslips and payroll history.

### HR Managers can:
- Manage employee records and update employment details.
- Approve or reject leave requests.
- Review submitted timesheets and generate payroll reports.

### Administrators have full control over the system:
- Manage all user accounts and roles.
- Configure system settings and maintain audit logs.
- Oversee HR operations and monitor system performance.

## Security

- Enforce JWT-based authentication to protect all endpoints and ensure data confidentiality.

## Database Schema and Default Data

### User
| Field   | Type        | Description                      |
|---------|-------------|----------------------------------|
| userId  | Integer     | Unique identifier for the user   |
| username| String      | Login username                   |
| password| String      | Hashed password                  |
| roles   | List<Role>  | Assigned roles (EMPLOYEE, HR_MANAGER, ADMIN) |

### Role
| Field   | Type    | Description                          |
|---------|---------|--------------------------------------|
| roleId  | Integer | Unique role identifier               |
| roleName| String  | Possible values: EMPLOYEE, HR_MANAGER, ADMIN |

### EmployeeProfile
| Field       | Type    | Description                          |
|-------------|---------|--------------------------------------|
| employeeId  | Integer | Unique employee profile identifier   |
| user        | User    | Associated user account              |
| fullName    | String  | Employee’s full name                 |
| department  | String  | Department (e.g., IT, Finance, HR)   |
| designation | String  | Job title (e.g., Software Engineer, Manager) |
| salary      | Double  | Base salary of the employee          |
| joiningDate | Date    | Date of joining the organization     |

### LeaveRequest
| Field          | Type    | Description                          |
|----------------|---------|--------------------------------------|
| leaveRequestId | Integer | Unique leave request identifier      |
| employee       | User    | Employee submitting the leave request|
| startDate      | Date    | Start date of leave                  |
| endDate        | Date    | End date of leave                    |
| leaveType      | String  | Type of leave (e.g., CASUAL, SICK, EARNED) |
| status         | String  | Request status (PENDING, APPROVED, REJECTED) |

### Timesheet
| Field       | Type    | Description                          |
|-------------|---------|--------------------------------------|
| timesheetId | Integer | Unique timesheet identifier          |
| employee    | User    | Employee who recorded the timesheet  |
| date        | Date    | Date of work                         |
| hoursWorked | Float   | Number of hours worked               |
| description | String  | Brief work summary                   |

### Payroll
| Field       | Type    | Description                          |
|-------------|---------|--------------------------------------|
| payrollId   | Integer | Unique payroll record identifier     |
| employee    | User    | Associated employee                  |
| period      | String  | Payroll period (e.g., “2025-03”)     |
| baseSalary  | Double  | Base salary for the period           |
| deductions  | Double  | Deductions (tax, insurance, etc.)    |
| bonus       | Double  | Additional bonus if applicable       |
| netPay      | Double  | Calculated net pay after deductions  |
| paymentDate | Date    | Date when payment was processed      |

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication is mandatory for all private endpoints.
- Role-based access control:
    - Employees: Manage personal profiles, submit leave requests, record timesheets, and access payslips.
    - HR Managers: Manage employee profiles, review and approve leave requests, validate timesheets, and generate payroll reports.
    - Administrators: Full CRUD operations on all users, system configurations, and audit logs.
- Return `401 Unauthorized` if the JWT is missing or invalid.
- Return `403 Forbidden` when an endpoint is accessed with insufficient privileges.

### Public Endpoints (No Authentication Required)

#### Login – `/api/public/login` – POST
**Request:**
```json
{
        "username": "employee_user",
        "password": "secure_pass"
}
```
**Response (200 OK):**
```json
{
        "token": "jwt_token_here"
}
```

### Authenticated Endpoints (JWT Required)

#### For Employees (`/api/auth/employee`)

##### View/Update Profile – `/api/auth/employee/profile` – GET/PUT
Retrieve or update personal employee details.

##### Submit Leave Request – `/api/auth/employee/leave` – POST
**Request:**
```json
{
        "startDate": "2025-04-15",
        "endDate": "2025-04-20",
        "leaveType": "CASUAL"
}
```
**Response (201 Created):**
```json
{
        "leaveRequestId": 3001,
        "status": "PENDING"
}
```

##### Record Timesheet – `/api/auth/employee/timesheet` – POST
**Request:**
```json
{
        "date": "2025-03-13",
        "hoursWorked": 8,
        "description": "Completed project module development."
}
```
**Response (201 Created):**
```json
{
        "timesheetId": 4001,
        "date": "2025-03-13",
        "hoursWorked": 8
}
```

##### View Payroll History – `/api/auth/employee/payroll` – GET
Provides digital payslips and payroll history.

#### For HR Managers (`/api/auth/hr`)

##### Manage Employee Profiles – `/api/auth/hr/employees` – CRUD operations
Create, update, or view employee profiles.

##### Approve/Reject Leave Requests – `/api/auth/hr/leave/{leaveRequestId}` – PUT
**Request:**
```json
{
        "status": "APPROVED"
}
```
**Response (200 OK):**
```json
{
        "leaveRequestId": 3001,
        "status": "APPROVED"
}
```

##### Validate Timesheets – `/api/auth/hr/timesheets` – GET
Retrieve submitted timesheets for approval and record validation.

##### Generate Payroll Reports – `/api/auth/hr/payroll` – POST
Process payroll for a given period.

#### For Administrators (`/api/auth/admin`)

##### User Management – `/api/auth/admin/users` – CRUD operations
Manage all system users and assign roles.

##### Audit Logs – `/api/auth/admin/logs` – GET
Retrieve comprehensive logs of HR operations and system activities.

##### System Configuration – `/api/auth/admin/config` – PUT
Update system settings and payroll configurations.

## Implementation Priorities

### Security:
- Enforce strict JWT authentication and role-based access control using Spring Security.
- Maintain comprehensive audit logs for all critical operations to ensure compliance.

### Scalability:
- Utilize Spring Data JPA with PostgreSQL to efficiently manage high volumes of employee data and transactions.
- Optimize database performance with strategic indexing and query tuning.

### Reliability & Validation:
- Implement rigorous data validation to prevent malformed requests.
- Ensure atomic transaction management for operations such as payroll processing and leave approvals.

### RESTful API Best Practices:
- Use standardized HTTP status codes and detailed error messages.
- Incorporate pagination and filtering for endpoints returning large datasets (e.g., employee lists, audit logs).

## Testing & Deployment

### Unit Testing:
- Leverage JUnit and Mockito to test individual components.

### Integration Testing:
- Conduct end-to-end tests for critical HR workflows.

### CI/CD Pipeline:
- Automate deployment processes to ensure rapid and reliable updates.

## Sample JSON Examples for APIs

### Login
**Request:**
```json
{
        "username": "employee_user",
        "password": "secure_pass"
}
```
**Response:**
```json
{
        "token": "jwt_token_here"
}
```

### Submit Leave Request (Employee)
**Request:**
```json
{
        "startDate": "2025-04-15",
        "endDate": "2025-04-20",
        "leaveType": "CASUAL"
}
```
**Response:**
```json
{
        "leaveRequestId": 3001,
        "status": "PENDING"
}
```

### Record Timesheet (Employee)
**Request:**
```json
{
        "date": "2025-03-13",
        "hoursWorked": 8,
        "description": "Completed module development for project X."
}
```
**Response:**
```json
{
        "timesheetId": 4001,
        "date": "2025-03-13",
        "hoursWorked": 8
}
```

### Approve Leave Request (HR Manager)
**Request:**
```json
{
        "status": "APPROVED"
}
```
**Response:**
```json
{
        "leaveRequestId": 3001,
        "status": "APPROVED"
}
```
