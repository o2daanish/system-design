# Data Flow in Distributed Shopping Cart System
## 1. Adding an Item to the Cart
### Step-by-Step Data Flow:
#### 1. User Action:
- A user adds an item to their shopping cart via the UI.
- The client sends an addItemToCart request to the API Gateway.
#### 2. API Gateway:
- The API Gateway authenticates the user and forwards the request to the Cart Service.
#### 3. Cart Service:
- The Cart Service checks if the cart is shared with other users.
- It then checks the availability of the item by calling the Item Service.
- Upon receiving confirmation of item availability, the Cart Service updates the cart in Cassandra.
- A CartItemAdded event is published to Kafka for further processing.
#### 4. Kafka:
- The CartItemAdded event contains details about the cart and the items added. This event is consumed by:
  - Real-Time Sync Service to notify all shared users of the cart.
  - Analytics Service (optional) to log metrics and user behavior.
#### 5. Real-Time Sync Service:
- The Real-Time Sync Service receives the event and pushes a WebSocket message to all active users sharing the cart.
- The UI of these users receives the update in real-time, showing the updated cart.
#### 6. Cassandra Update:
- The cart is persistently updated in Cassandra, including the new item information.

## 2. Removing an Item from the Cart
### Step-by-Step Data Flow:
#### 2.1 User Action:
- The user initiates the removal of an item from the cart.
- The client sends a removeItemFromCart request to the API Gateway.
#### 2.2 API Gateway:
- The API Gateway authenticates the user and forwards the request to the Cart Service.
#### 2.3 Cart Service:
- The Cart Service validates the request (i.e., checks if the item exists in the cart).
- The cart is updated in Cassandra by removing the specified item.
- A CartItemRemoved event is published to Kafka.
#### 2.3 Kafka:
- The CartItemRemoved event is consumed by:
  - Real-Time Sync Service to notify all shared users of the cart.
  - Inventory Service (optional) to update item stock in the system.
  - Analytics Service to log changes and perform analysis.
#### 2.4 Real-Time Sync Service:
- Upon consuming the event, the Real-Time Sync Service sends real-time notifications to all active users through WebSocket, informing them that an item was removed.
#### 2.5 Cassandra Update:
- The cart is persistently updated in Cassandra, reflecting the removal of the item.

## 3. Sharing the Cart with Other Users
### Step-by-Step Data Flow:
#### 3.1 User Action:
- The user initiates a cart sharing operation, selecting multiple users to share the cart with.
- The client sends a shareCart request to the API Gateway.
#### 3.2 API Gateway:
- The API Gateway authenticates the request and forwards it to the Cart Service.
#### 3.3 Cart Service:
- The Cart Service adds the selected users to the list of shared users for the cart.
- It updates the Cassandra database with the new list of shared users.
- A CartShared event is published to Kafka.
#### 3.4 Kafka:
- The CartShared event is consumed by:
  - Real-Time Sync Service to notify the newly added users that they have access to the shared cart.
  - Notification Service (optional) to send push/email notifications to the added users.
#### 3.5 Real-Time Sync Service:
- The Real-Time Sync Service opens WebSocket connections for the newly added users and updates the UI in real-time.

## 4. Modifying a Shared Cart
### Step-by-Step Data Flow:
#### 4.1 User Action:
One of the shared users modifies the cart (e.g., adds or removes items).
The client sends a **modifyCart** request to the API Gateway.
#### 4.2 API Gateway:
The API Gateway authenticates the user and forwards the request to the Cart Service.
#### 4.3 Cart Service:
- The Cart Service checks if the cart is shared and if the user has access to modify the cart.
- The cart is updated in Cassandra (e.g., new items added or existing items removed).
- A **CartModified** event is published to Kafka.
#### 4.4 Kafka:
- The **CartModified** event is consumed by:
  - Real-Time Sync Service to notify all other shared users in real-time about the changes.
  - Inventory Service (optional) to adjust stock levels if items were added or removed.
#### 4.5 Real-Time Sync Service:
- Upon consuming the **CartModified** event, the Real-Time Sync Service sends WebSocket notifications to all active users sharing the cart.
- The UI of these users is updated in real-time to reflect the changes.

## 5. Checkout Process
### Step-by-Step Data Flow:
#### 5.1 User Action:
- One of the shared users proceeds to checkout.
- The client sends a checkoutCart request to the API Gateway.
#### 5.2 API Gateway:
- The API Gateway authenticates the user and forwards the request to the Cart Service.
#### 5.3 Cart Service:
- The Cart Service checks the validity of the cart (i.e., ensures all items are available and in stock).
- A CartCheckedOut event is published to Kafka.
- The cart status is updated in Cassandra to "CHECKED_OUT".
- The checkout request is forwarded to the Order Service (assumed to be handled separately).
#### 5.4 Kafka:
- The CartCheckedOut event is consumed by:
  - Real-Time Sync Service to notify all shared users that the cart has been checked out.
  - Inventory Service to lock or reduce the stock of items in the cart.
#### 5.5 Real-Time Sync Service:
- The Real-Time Sync Service sends WebSocket messages to notify all active shared users that the cart has been checked out.

## Fault Tolerance in Data Flow
1. Kafka ensures that messages/events such as **CartItemAdded**, **CartModified**, etc., are processed reliably.
2. In case a service fails, Kafka will retry delivering the event.
   - Cassandra provides high availability and data replication across nodes. Any modifications to the cart are reflected in real-time 
     across multiple replicas.
3. API Gateway employs circuit breakers and load balancers to ensure fault isolation in the event of microservice failures.

