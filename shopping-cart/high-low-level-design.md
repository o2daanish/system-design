# Low Level Design
1. Component Breakdown and Class Design
1.1 API Gateway  
- Responsibilities:
  - Authenticate user requests.
  - Route requests to the appropriate microservice.
  - Apply rate-limiting, caching, and circuit-breaking strategies.
- Class Diagram:
```
class APIGateway {
    - routeRequest(request): Response
    - authenticate(request): boolean
    - rateLimit(request): void
}
```
## 1.2 Cart Service
- Responsibilities:
  - Handle cart CRUD operations.
  - Manage cart sharing and item operations.
  - Interact with Kafka for event-driven communication.
- Class Diagram:
```
class CartService {
    - createCart(userId: String, items: List<ItemRequest>): CartResponse
    - addItemToCart(cartId: String, itemId: String, quantity: int, userId: String): CartResponse
    - removeItemFromCart(cartId: String, itemId: String, userId: String): CartResponse
    - shareCart(cartId: String, sharedUserIds: List<String>): void
    - getCart(cartId: String): CartResponse
    - checkoutCart(cartId: String, userId: String): CheckoutResponse
    - publishEvent(event: CartEvent): void
}
```
- Data Model (using Java-like structure):
```
class Cart {
    String cartId;
    List<Item> items;
    List<String> sharedUserIds;
    Date createdAt;
    Date updatedAt;
    String status; // (e.g., "ACTIVE", "CHECKED_OUT")
}

class Item {
    String itemId;
    int quantity;
    double price;
}

class CartResponse {
    String cartId;
    List<Item> items;
    List<String> sharedUserIds;
    Date createdAt;
    Date updatedAt;
    String status;
}
```
- Flow for Adding Item:
  - addItemToCart API will first check the item availability in the Item Service.
  - If available, it updates the cart in Cassandra and publishes an event to Kafka (CartItemAdded).
  - Real-Time Sync Service listens to the event and updates all shared users.
    
## 1.3 Real-Time Sync Service
- Responsibilities:
  - Subscribe to Kafka topics (CartItemAdded, CartModified, CartShared).
  - Push real-time updates to shared users using WebSockets.
- Class Diagram:
```
class RealTimeSyncService {
    - onCartModified(event: CartEvent): void
    - notifyUsers(cartId: String, updateMessage: String): void
}
```
- WebSocket Connection:
```
class WebSocketManager {
    - openConnection(userId: String): void
    - sendUpdate(userId: String, updateMessage: String): void
    - closeConnection(userId: String): void
}
```
- Flow:
  - The RealTimeSyncService receives events from Kafka whenever a modification occurs in the cart.
  - It triggers the WebSocketManager to notify the active users sharing the cart about the change in real-time.

# 1.4 Item Service
- Responsibilities:
  - Fetch item details such as price, availability, and stock.
  - Cache frequently accessed items using Redis.
- Class Diagram:
```
class ItemService {
    - getItemDetails(itemId: String): Item
    - updateItemStock(itemId: String, quantity: int): void
}
```
Flow:
Before adding or updating an item in the cart, the CartService will call getItemDetails to validate the availability of the item.
The ItemService retrieves the item from Redis (cache) or from the persistent store.
# 1.5 User Service
- Responsibilities:
  - Manage user data and cart-sharing permissions.
- Class Diagram:
```
class UserService {
    - getUserDetails(userId: String): User
    - canUserAccessCart(userId: String, cartId: String): boolean
}
```
- Data Model:
```
class User {
    String userId;
    String username;
    List<String> cartIds;
}
```
# Database Schema Design (Cassandra)
- Cart Table
```
CREATE TABLE carts (
  cart_id UUID PRIMARY KEY,
  user_ids LIST<TEXT>,
  items MAP<TEXT, INT>, -- item_id to quantity
  status TEXT,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```
- Item Table
```
CREATE TABLE items (
  item_id UUID PRIMARY KEY,
  name TEXT,
  price DOUBLE,
  stock INT
);
```
- User_Cart_Sharing Table
```
CREATE TABLE user_cart_sharing (
  user_id TEXT,
  cart_id TEXT,
  PRIMARY KEY (user_id, cart_id)
);
```
# Event Handling (Kafka)
- Kafka Topics:
  - CartCreated
  - CartItemAdded
  - CartModified
  - CartShared
  - CartCheckout
- Kafka Flow:
```
Each service publishes events to Kafka whenever an action is performed on the cart. For example, when an item is added, a CartItemAdded event is published with details of the cart,
and the Real-Time Sync Service consumes this event to notify users.
```
# Real-Time Sync
```
WebSocketManager handles the establishment of WebSocket connections and pushing updates. When an event like CartItemAdded is consumed,
RealTimeSyncService calls WebSocketManager.sendUpdate to broadcast updates to all active users connected to the shared cart.
```
# Error Handling and Fault Tolerance
- Retry Mechanism:
Kafka consumers will retry processing the events in case of failures.
- Circuit Breaker:
A circuit breaker can be implemented at the API Gateway level to ensure that a failure in one service doesn't bring down the entire system.
- Data Consistency:
Cassandra offers eventual consistency; however, microservices will apply optimistic locking to manage concurrent writes, ensuring consistency between operations.

