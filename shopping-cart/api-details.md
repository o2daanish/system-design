## API Details for Shared Shopping Cart System

1. API Gateway
```
Base URL: https://api.example.com
```
2. Authentication API
- Endpoint: /auth/login
- Method: POST
- Request Body:
```
{
  "username": "user@example.com",
  "password": "password123"
}

```
  Response:
```
{
  "token": "jwt.token.here",
  "userId": "userId_value"
}
```
3. Cart APIs
3.1 Create Cart
- Endpoint: /carts      
- Method: POST      
- Request Body:      
```
{
  "userId": "userId_value",  // Unique identifier for the user creating the cart
  "cartName": "Family Shopping",  // Optional Name of the cart
  "isShared": true,  // Optional Indicates if the cart is shared
  "sharedWith": [    // Optional List of user IDs with whom the cart is shared
    "userId_1",
    "userId_2"
  ],
  "creationDate": "1727102098", // EPOCH Time
  "currency": "USD"
}
```
- Response:
```
{
  "cartId": "cartId_value",
  "message": "Cart created successfully.",
  "cartDetails": {
    "userId": "userId_value",
    "cartName": "Family Shopping",
    "isShared": true,
    "sharedWith": [
      "userId_1",
      "userId_2"
    ],
    "creationDate": "2024-09-23T10:00:00Z",
    "currency": "USD"
  }
}
````
3.2 Add Multiple Items to Cart
- Endpoint: /carts/{cartId}/items
- Method: POST
- Request Body:
```
{
  "items": [
    {
      "itemId": "item123",
      "itemName": "Wireless Mouse",
      "quantity": 1,
      "price": 29.99
    },
    {
      "itemId": "item456",
      "itemName": "Mechanical Keyboard",
      "quantity": 1,
      "price": 89.99
    }
  ]
}
```
- Response:
```
{
  "message": "Items added successfully.",
  "updatedCart": {
    "cartId": "cartId_value",
    "items": [
      {
        "itemId": "item123",
        "quantity": 1,
        "price": 29.99
      },
      {
        "itemId": "item456",
        "quantity": 1,
        "price": 89.99
      }
    ]
  }
}
```
3.3 Update Item in Cart
- Endpoint: /carts/{cartId}/items/{itemId}
- Method: PUT
- Request Body:
```
{
  "quantity": 2
}
```
- Response:
```
{
  "message": "Item updated successfully.",
  "updatedItem": {
    "itemId": "item123",
    "quantity": 2
  }
}
```
3.4 Remove Item from Cart
- Endpoint: /carts/{cartId}/items/{itemId}
- Method: DELETE
- Response:
```
{
  "message": "Item removed successfully."
}
```
3.5 Get Cart Details
- Endpoint: /carts/{cartId}
- Method: GET
- Response:
```
{
  "cartId": "cartId_value",
  "items": [
    {
      "itemId": "item123",
      "itemName": "Wireless Mouse",
      "quantity": 1,
      "price": 29.99
    },
    {
      "itemId": "item456",
      "itemName": "Mechanical Keyboard",
      "quantity": 1,
      "price": 89.99
    }
  ],
  "totalPrice": 119.98
}
```
3.6 Clear Cart
- Endpoint: /carts/{cartId}/clear
- Method: DELETE
- Response:
```
{
  "message": "Cart cleared successfully."
}
```
3.7 Share Cart with Users
- Endpoint: /carts/{cartId}/share
- Method: POST
- Request Body:
```
{
  "sharedWith": [
    "userId_1",
    "userId_2"
  ]
}
```
- Response:
```
{
  "message": "Cart shared successfully.",
  "updatedSharedWith": [
    "userId_1",
    "userId_2"
  ]
}
```
4. Real-Time Notifications API (Optional)
- Endpoint: /notifications
- Method: GET
- Response: (WebSocket connection for real-time notifications)
- Example Notification:
```
{
  "event": "ITEM_ADDED",
  "cartId": "cartId_value",
  "item": {
    "itemId": "item123",
    "itemName": "Wireless Mouse",
    "quantity": 1
  },
  "updatedBy": "UserA_ID"
}
```

# Further Enhancemnets For More Robust API
1. Error Handling:
Include error response examples for each API endpoint, detailing common errors (e.g., item not found, insufficient stock, unauthorized access).
```{
  "error": "ITEM_NOT_FOUND",
  "message": "The item with ID item123 does not exist."
}
```
2. Validation:
Specify validation rules for inputs (e.g., required fields, data types).Ensure APIs return meaningful error messages for invalid input.
Pagination for Cart Items:
If carts can contain many items, consider implementing pagination in the Get Cart Details API.
```
{
  "cartId": "cartId_value",
  "items": [ /* paginated items */ ],
  "totalItems": 100,
  "currentPage": 1,
  "totalPages": 10
}
```
3. Rate Limiting and Throttling:
Consider mentioning rate limits for APIs to prevent abuse (especially important for shared carts).

4. Security Considerations:
Document security measures (e.g., JWT for authentication) to protect endpoints.Specify CORS policy for frontend integration.
5. Versioning:
Consider adding versioning to the API endpoints for future enhancements without breaking existing clients.
Example: /v1/carts






