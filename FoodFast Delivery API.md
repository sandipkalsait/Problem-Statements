# Problem Statement: FoodFast Delivery API

FoodFast Delivery is a disruptive startup in the food delivery industry. They are transitioning from traditional channels to an innovative online platform. As part of the initial MVP, you are tasked with designing and implementing a distributed RESTful API backend using Spring Boot. The API must be robust, secure, and scalable, with a special emphasis on security via JWT authentication.

## Business Context

FoodFast Delivery aims to streamline the food ordering process by connecting customers with local restaurants. The platform should enable customers to search for restaurants and menus, place orders, and track their order history. Simultaneously, restaurant owners require a dedicated interface to manage their menus and process incoming orders.

## Database Models and Default Data

The database models are pre-defined and initialized with default data. You will work with the following entities:

### User

- `userId`: Integer
- `username`: String
- `password`: String
- `roles`: Role

### Role

- `roleId`: Integer
- `roleName`: String (Possible values: CUSTOMER, RESTAURANT_OWNER)

### Restaurant

- `restaurantId`: Integer
- `name`: String
- `address`: String
- `owner`: User (only users with the RESTAURANT_OWNER role)

### MenuItem

- `menuItemId`: Integer
- `name`: String
- `price`: Double
- `category`: String (e.g., "Pizza", "Sushi", "Burgers")
- `restaurant`: Restaurant

### Order

- `orderId`: Integer
- `customer`: User
- `restaurant`: Restaurant
- `totalAmount`: Double
- `status`: String (PENDING, PREPARING, DELIVERED, CANCELLED)
- `orderItems`: List of OrderItem

### OrderItem

- `orderItemId`: Integer
- `order`: Order
- `menuItem`: MenuItem
- `quantity`: Integer

## API Requirements and Endpoints

### Security & Authentication

**JWT Security:**

Every API endpoint must enforce security via JWT tokens. The JWT is issued upon successful authentication and must be included in the header (key: JWT) for all protected endpoints. If the JWT is missing or invalid, return HTTP 401. Additionally, if an endpoint is accessed by a user with an incorrect role, return HTTP 403.

#### /api/public/login – POST

**Description:** Accepts JSON containing a username and password, validates credentials, and returns a JWT token.

**Example Request:**

```json
{ "username": "johnDoe", "password": "pass_word" }
```

**Success Response:** HTTP 200 with JWT token in the response body.

**Failure Response:** HTTP 401 for invalid credentials.

### Public Endpoints

#### /api/public/restaurant/search – GET

**Description:** Accepts a query parameter keyword and returns a list of restaurants matching the keyword in their name or address.

**Example:** `/api/public/restaurant/search?keyword="Pizza"`

**Success Response:** HTTP 200 with a JSON array of restaurant objects.

### Authenticated Endpoints

**Access Control:**

Endpoints prefixed with `/api/auth/customer` are for users with the CUSTOMER role.  
Endpoints prefixed with `/api/auth/restaurant` are for users with the RESTAURANT_OWNER role.

#### For Customers (/api/auth/customer):

**Retrieve Order History**

**Endpoint:** `/api/auth/customer/order` – GET

**Description:** Returns the logged-in customer’s order history.

**Success Response:** HTTP 200 with a list of order details.

**Place a New Order**

**Endpoint:** `/api/auth/customer/order` – POST

**Description:** Accepts an order JSON with the selected restaurant, menu items, and quantities. Calculates the total amount and persists the order.

**Example Request:**

```json
{
    "restaurantId": 5,
    "orderItems": [
        { "menuItemId": 12, "quantity": 2 },
        { "menuItemId": 15, "quantity": 1 }
    ]
}
```

**Success Response:** HTTP 201 (Created) along with the order details.

**Edge Case:** Return HTTP 400 for invalid menu items or restaurant mismatches.

**Cancel an Order**

**Endpoint:** `/api/auth/customer/order/{orderId}` – DELETE

**Description:** Allows a customer to cancel an order provided it is still in a PENDING status.

**Success Response:** HTTP 200 if cancellation is successful.

**Failure Response:** HTTP 404 if the order is not found or cannot be canceled.

#### For Restaurant Owners (/api/auth/restaurant):

**Manage Menu Items**

**Retrieve Menu Items**

**Endpoint:** `/api/auth/restaurant/menu` – GET

**Description:** Returns the menu items associated with the restaurant owned by the logged-in user.

**Success Response:** HTTP 200 with a JSON array of menu items.

**Add a New Menu Item**

**Endpoint:** `/api/auth/restaurant/menu` – POST

**Description:** Accepts JSON for a new menu item to be added under the restaurant.

**Example Request:**

```json
{
    "name": "Margherita Pizza",
    "price": 12.99,
    "category": "Pizza"
}
```

**Success Response:** HTTP 201 with a Location header pointing to the created resource URI.

**Update a Menu Item**

**Endpoint:** `/api/auth/restaurant/menu/{menuItemId}` – PUT

**Description:** Accepts a JSON body with updated details for an existing menu item.

**Success Response:** HTTP 200 if the update is successful.

**Delete a Menu Item**

**Endpoint:** `/api/auth/restaurant/menu/{menuItemId}` – DELETE

**Description:** Deletes the specified menu item from the restaurant’s menu.

**Success Response:** HTTP 200 if deletion is successful; return HTTP 404 if the item is not found.

**Manage Incoming Orders**

**Retrieve Orders**

**Endpoint:** `/api/auth/restaurant/order` – GET

**Description:** Returns all orders placed with the restaurant.

**Success Response:** HTTP 200 with a list of orders.

**Update Order Status**

**Endpoint:** `/api/auth/restaurant/order/{orderId}` – PUT

**Description:** Accepts a JSON body to update the order status (e.g., from PENDING to PREPARING or DELIVERED).

**Example Request:**

```json
{ "status": "PREPARING" }
```

**Success Response:** HTTP 200 upon successful update.

## Implementation Priorities:

- **Security:** Implement JWT authentication rigorously across all endpoints. Every secured endpoint must validate the JWT token and ensure that only users with the appropriate roles can access them.
- **Robustness:** Implement thorough validation and error handling. Edge cases such as unauthorized access, invalid order details, or duplicate menu entries should be clearly managed.
- **Scalability:** Architect the API for high-load scenarios and future feature extensions.
- **Compliance:** Adhere to RESTful best practices and coding standards.

## Testing and Validation:

Thoroughly test each endpoint using provided test cases and ensure your implementation meets all functional and non-functional requirements. Validate the responses with the appropriate HTTP status codes and error handling as specified.
