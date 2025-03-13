# Problem Statement: EduSync – AI-Powered Learning Management System (LMS)

## Scenario:
EduSync is an AI-powered Learning Management System (LMS) designed to enhance online education by providing seamless interactions between students, instructors, and administrators. The platform facilitates course management, automated grading, real-time progress tracking, and personalized learning recommendations.

## Business Requirements:
- **Students**: Browse courses, enroll, track progress, submit assignments, and receive AI-generated learning recommendations.
- **Instructors**: Create and manage courses, upload study materials, grade assignments, and monitor student performance.
- **Administrators**: Oversee user management, manage course approvals, and monitor platform analytics.
- **AI-powered Learning Suggestions**: An AI module analyzes student progress and recommends additional resources or personalized learning paths.

## Database Schema

### User
| Field   | Type     | Description           |
|---------|----------|-----------------------|
| userId  | Integer  | Unique user identifier|
| username| String   | Login username        |
| password| String   | Hashed password       |
| roles   | List<Role>| Roles: STUDENT, INSTRUCTOR, ADMIN |

### Role
| Field   | Type     | Description           |
|---------|----------|-----------------------|
| roleId  | Integer  | Unique role identifier|
| roleName| String   | Possible values: STUDENT, INSTRUCTOR, ADMIN |

### Course
| Field      | Type    | Description                   |
|------------|---------|-------------------------------|
| courseId   | Integer | Unique course identifier      |
| title      | String  | Course title                  |
| description| String  | Brief description of the course|
| instructor | User    | Instructor assigned to the course|
| price      | Double  | Course fee (if applicable)    |
| status     | String  | Course status (DRAFT, PUBLISHED, ARCHIVED) |

### Enrollment
| Field       | Type    | Description                   |
|-------------|---------|-------------------------------|
| enrollmentId| Integer | Unique enrollment ID          |
| student     | User    | Student enrolled              |
| course      | Course  | Associated course             |
| enrolledOn  | DateTime| Enrollment timestamp          |
| progress    | Double  | Percentage of course completion|

### Assignment
| Field       | Type    | Description                   |
|-------------|---------|-------------------------------|
| assignmentId| Integer | Unique assignment ID          |
| course      | Course  | Associated course             |
| title       | String  | Assignment title              |
| description | String  | Assignment details            |
| dueDate     | DateTime| Submission deadline           |

### Submission
| Field       | Type    | Description                   |
|-------------|---------|-------------------------------|
| submissionId| Integer | Unique submission ID          |
| student     | User    | Student who submitted         |
| assignment  | Assignment| Related assignment          |
| submittedOn | DateTime| Submission timestamp          |
| grade       | Double  | Score assigned by instructor  |
| feedback    | String  | Instructor feedback           |

### AI Recommendation
| Field           | Type    | Description                   |
|-----------------|---------|-------------------------------|
| recommendationId| Integer | Unique ID for AI-generated recommendation|
| student         | User    | Student receiving recommendation|
| course          | Course  | Suggested course              |
| reason          | String  | Explanation for the recommendation|

## API Requirements and Endpoints

### Security & Authentication
- JWT-based authentication for all protected endpoints.
- Role-based access control:
    - **Students**: Enroll in courses, track progress, submit assignments, and receive AI recommendations.
    - **Instructors**: Manage courses, grade assignments, and track student performance.
    - **Administrators**: Full control over the platform, including course approvals and user management.

### Public Endpoints (No Authentication Required)
- **Login** – `/api/public/login` – POST
    - **Request:**
        ```json
        {
            "username": "student_user",
            "password": "secure_password"
        }
        ```
    - **Response:**
        ```json
        {
            "token": "jwt_token_here"
        }
        ```
- **Browse Available Courses** – `/api/public/courses` – GET
    - Returns a list of published courses.

### Authenticated Endpoints (JWT Required)

#### For Students (`/api/auth/student`)
- **Enroll in Course** – `/api/auth/student/enroll` – POST
    - **Request:**
        ```json
        {
            "courseId": 101
        }
        ```
    - **Response:**
        ```json
        {
            "enrollmentId": 567,
            "courseId": 101,
            "status": "ENROLLED"
        }
        ```
- **Track Course Progress** – `/api/auth/student/progress` – GET
    - Retrieves a list of enrolled courses with completion status.
- **Submit Assignment** – `/api/auth/student/assignments/submit` – POST
    - **Request:**
        ```json
        {
            "assignmentId": 303,
            "fileUrl": "https://example.com/submission.pdf"
        }
        ```
    - **Response:**
        ```json
        {
            "submissionId": 890,
            "status": "SUBMITTED"
        }
        ```
- **Get AI Learning Recommendations** – `/api/auth/student/recommendations` – GET
    - Retrieves AI-generated course suggestions.

#### For Instructors (`/api/auth/instructor`)
- **Create New Course** – `/api/auth/instructor/course` – POST
    - **Request:**
        ```json
        {
            "title": "Advanced Java Programming",
            "description": "In-depth Java course for professionals",
            "price": 99.99,
            "status": "DRAFT"
        }
        ```
    - **Response:**
        ```json
        {
            "courseId": 200,
            "status": "CREATED"
        }
        ```
- **Publish Course** – `/api/auth/instructor/course/{courseId}/publish` – PUT
    - Updates course status to PUBLISHED.
- **Review Student Assignments** – `/api/auth/instructor/assignments/review` – GET
    - Retrieves submitted assignments for grading.
- **Grade Assignment** – `/api/auth/instructor/assignments/grade` – PUT
    - **Request:**
        ```json
        {
            "submissionId": 890,
            "grade": 85,
            "feedback": "Great work!"
        }
        ```
    - **Response:**
        ```json
        {
            "status": "GRADED"
        }
        ```

#### For Administrators (`/api/auth/admin`)
- **User Management** – `/api/auth/admin/users` – CRUD operations.
- **Course Approval** – `/api/auth/admin/courses/approve` – PUT
    - Approves courses submitted by instructors.
- **Platform Analytics** – `/api/auth/admin/analytics` – GET
    - Retrieves system-wide statistics.

## Implementation Priorities

### Security
- Strict JWT authentication and role-based access control using Spring Security.
- Audit logging to track key user activities.

### Scalability
- Spring Boot + PostgreSQL for handling high traffic and concurrent users.
- AI-based learning recommendations using machine learning models trained on student progress.

### Reliability & Validation
- Automated testing (JUnit, Mockito) to ensure stability.
- CI/CD pipelines for rapid deployment.

### RESTful API Best Practices
- Pagination and filtering for scalable course retrieval.
- Standard HTTP status codes for predictable error handling.

## Sample JSON Examples for APIs

### Login
**Request:**
```json
{
    "username": "student_user",
    "password": "secure_password"
}
```
**Response:**
```json
{
    "token": "jwt_token_here"
}
```

### Enroll in Course
**Request:**
```json
{
    "courseId": 101
}
```
**Response:**
```json
{
    "enrollmentId": 567,
    "status": "ENROLLED"
}
```

### Submit Assignment
**Request:**
```json
{
    "assignmentId": 303,
    "fileUrl": "https://example.com/submission.pdf"
}
```
**Response:**
```json
{
    "submissionId": 890,
    "status": "SUBMITTED"
}
```

### Get AI Learning Recommendations
**Response:**
```json
[
    {
        "recommendationId": 120,
        "course": {
            "courseId": 150,
            "title": "Machine Learning Basics"
        },
        "reason": "Based on your performance in Python Programming"
    }
]
```

