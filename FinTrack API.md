# Problem Statement: FinTrack – Personal Finance Management System

## Scenario
FinTrack is an innovative fintech platform that empowers users to manage their finances with precision. The system enables users to track income, expenses, budgets, and investments through a secure, scalable, and efficient RESTful API backend developed in Spring Boot. The solution is designed to support robust role-based access for end-users, financial advisors, and administrators, ensuring data integrity and uncompromising security.

## Business Requirements
- **End-Users** can view account details, add transactions, set budgets, and generate insightful financial reports.
- **Financial Advisors** have access to aggregated financial data, enabling them to analyze spending trends and provide tailored advice.
- **Administrators** hold full system control, including user management, audit log review, and configuration adjustments.

## Security
- Leverage JWT-based authentication to protect endpoints and enforce rigorous role-based access controls.

## Database Schema and Default Data

### User
| Field   | Type          | Description                       |
|---------|---------------|-----------------------------------|
| userId  | Integer       | Unique identifier for the user    |
| username| String        | Login username                    |
| password| String        | Hashed password                   |
| roles   | List<Role>    | Assigned roles (USER, ADVISOR, ADMIN) |

### Role
| Field   | Type          | Description                       |
|---------|---------------|-----------------------------------|
| roleId  | Integer       | Unique role identifier            |
| roleName| String        | Possible values: USER, ADVISOR, ADMIN |

### Account
| Field      | Type    | Description                       |
|------------|---------|-----------------------------------|
| accountId  | Integer | Unique account identifier         |
| user       | User    | Associated user                   |
| accountType| String  | E.g., SAVINGS, CHECKING, INVESTMENT |
| balance    | Double  | Current balance of the account    |

### Transaction
| Field          | Type    | Description                       |
|----------------|---------|-----------------------------------|
| transactionId  | Integer | Unique transaction identifier     |
| account        | Account | Associated account                |
| amount         | Double  | Transaction amount (positive for deposits, negative for withdrawals) |
| transactionDate| Date    | Date and time of the transaction  |
| description    | String  | Transaction memo or description   |

### Budget
| Field   | Type    | Description                       |
|---------|---------|-----------------------------------|
| budgetId| Integer | Unique budget identifier          |
| user    | User    | Associated user                   |
| category| String  | Budget category (e.g., Food, Entertainment) |
| limit   | Double  | Spending limit for the category   |
| period  | String  | Budget period (e.g., MONTHLY, YEARLY) |

## API Requirements and Endpoints

### Security & Authentication
- JWT Authentication is mandatory for all protected endpoints.
- Role-based access control:
    - End-Users can manage accounts, transactions, and budgets.
    - Financial Advisors can access aggregated financial summaries and spending trends.
    - Administrators have full operational control including user management.
- Return `401 Unauthorized` if JWT is missing or invalid.
- Return `403 Forbidden` when access is attempted with an insufficient role.

### Public Endpoints (No Authentication Required)
- **Login** – `/api/public/login` – POST
    - **Request:**
        ```json
        {
            "username": "jane_doe",
            "password": "strong_pass"
        }
        ```
    - **Response (Success, 200):**
        ```json
        {
            "token": "jwt_token_here"
        }
        ```
    - Invalid credentials yield a `401 Unauthorized`.

- **Register** – `/api/public/register` – POST
    - **Request:**
        ```json
        {
            "username": "jane_doe",
            "password": "strong_pass",
            "email": "jane@example.com"
        }
        ```
    - **Response (Success, 201):**
        ```json
        {
            "userId": 101,
            "username": "jane_doe"
        }
        ```

### Authenticated Endpoints (JWT Required)

#### For End-Users (`/api/auth/user`)
- **View Account Details** – `/api/auth/user/accounts` – GET
    - Returns the list of accounts associated with the logged-in user.

- **Add Transaction** – `/api/auth/user/transaction` – POST
    - **Request:**
        ```json
        {
            "accountId": 202,
            "amount": -50.0,
            "description": "Grocery shopping"
        }
        ```
    - **Response (201 Created):** Provides transaction details and updates the account balance.

- **Set Budget** – `/api/auth/user/budget` – POST
    - **Request:**
        ```json
        {
            "category": "Food",
            "limit": 300.0,
            "period": "MONTHLY"
        }
        ```
    - **Response (201 Created):** Returns the newly set budget details.

- **Generate Financial Report** – `/api/auth/user/report` – GET
    - Provides summarized insights and trends based on user financial data.

#### For Financial Advisors (`/api/auth/advisor`)
- **View User Financial Summaries** – `/api/auth/advisor/users` – GET
    - Retrieves aggregated financial data for multiple users.

- **Analyze Spending Trends** – `/api/auth/advisor/trends` – GET
    - Offers detailed analytics on user spending patterns.

#### For Administrators (`/api/auth/admin`)
    "username": "jane_doe",
    "password": "strong_pass"
}
Response (Success, 200):

json
Copy
Edit
{
    "token": "jwt_token_here"
}
Add Transaction
Request:

json
Copy
Edit
{
    "accountId": 202,
    "amount": -50.0,
    "description": "Grocery shopping"
}
Response (201 Created):

json
Copy
Edit
{
    "transactionId": 305,
    "account": {
        "accountId": 202,
        "accountType": "CHECKING",
        "balance": 950.0
    },
    "amount": -50.0,
    "transactionDate": "2025-03-13T10:15:30",
    "description": "Grocery shopping"
}
Set Budget
Request:

json
Copy
Edit
{
    "category": "Food",
    "limit": 300.0,
    "period": "MONTHLY"
}
Response (201 Created):

json
Copy
Edit
{
    "budgetId": 407,
    "user": {
        "userId": 101,
        "username": "jane_doe"
    },
    "category": "Food",
    "limit": 300.0,
    "period": "MONTHLY"
}