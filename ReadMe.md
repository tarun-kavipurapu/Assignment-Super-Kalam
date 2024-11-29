### DB Architecture

For this system, a combination of **RDBMS** (PostgreSQL) and **NoSQL** (MongoDB) is used to efficiently manage both structured and unstructured data:

- **PostgreSQL** (Relational Database) is used for transactional data, ensuring **ACID** (Atomicity, Consistency, Isolation, Durability) properties where required. This includes data such as user information, events, bookings, seats, payments, etc., which need to maintain strict integrity and consistency.
  
- **MongoDB** (NoSQL Database) is used for handling unstructured or semi-structured data, such as reviews, comments, and other extra metadata that may not need ACID compliance but still needs to be efficiently queried. MongoDB will store additional information that can be linked to relational data via a common ID.

- Both databases will be **connected via a common ID** to enable seamless integration and data retrieval across both systems.

[Link to DB Schema](https://dbdiagram.io/d/67499a5de9daa85aca20c151)

---

### **API Endpoints**

---

### **Feed System**

#### 1. **Post Location**
   - **Endpoint**: `POST /api/location`
   - **Description**: Allows the user to post their current location to the server.
   - **Request Body**:
     ```json
     {
       "latitude": 40.7128,
       "longitude": -74.0060
     }
     ```

#### 2. **Get Events by Location**
   - **Endpoint**: `GET /api/events/by-location`
   - **Description**: Fetch all events based on the user's location (city name or coordinates).
   - **Query Params**:
     - `city_name` (optional) - Name of the city.
     - `latitude` (optional) - Latitude of the user.
     - `longitude` (optional) - Longitude of the user.
     - `radius` (optional) - Search radius in kilometers (default is 50 km).
   - **Response**:
     ```json
     [
       {
         "event_id": 1,
         "title": "Concert",
         "event_type": "music",
         "city": "New York",
         "start_date": "2024-12-05",
         "end_date": "2024-12-05",
         "venue": "Stadium ABC",
         "description": "A live music concert"
       }
     ]
     ```
#### 2.  **Search Events**
   - **Endpoint**: `GET /api/events/search`
   - **Description**: Search for events based on a query. This query can include event title, description, city, and event type. The search can be flexible and support partial matches.

#### 3. **Get All Events**
   - **Endpoint**: `GET /api/events`
   - **Description**: Fetch all events. Optionally, filter by city, event type, or date range.
   - **Query Params**:
     - `city_id` (optional)
     - `event_type` (optional, `movie`, `sport`, `standup`)


#### 4. **Get Event by ID**
   - **Endpoint**: `GET /api/events/{event_id}`
   - **Description**: Fetch detailed information for a specific event.

#### 5. **Get Events by City**
   - **Endpoint**: `GET /api/cities/{city_id}/events`
   - **Description**: Fetch all events in a specific city.

---

### **Booking System**

#### 1. **Get User Bookings**
   - **Endpoint**: `GET /api/users/{user_id}/bookings`
   - **Description**: Fetch all bookings for a user.

#### 2. **Cancel Booking**
   - **Endpoint**: `DELETE /api/bookings/{booking_id}`
   - **Description**: Cancel a specific booking.

#### 3. **Get Available Seats**
   - **Endpoint**: `GET /api/event-schedules/{schedule_id}/seats`
   - **Description**: Fetch all available seats for a specific event schedule.
   - **Response**:
     ```json
     [
       {
         "seat_id": 1,
         "status": "available",
         "row_number": "A",
         "seat_number": 1,
         "locked_at": null
       },
       {
         "seat_id": 2,
         "status": "locked",
         "row_number": "A",
         "seat_number": 2,
         "locked_at": "2024-11-29T10:00:00"
       }
     ]
     ```
     -  conditional check for locked_at and locked status

 - Please Note that the Lock Seat and Book seat has been merged into a single api call
#### 4. **Book Seats**
   - **Endpoint**: `POST /api/bookings/{booking_id}/seats`
   - **Description**: Assign specific seats to a booking. Updates the lock status and lock time for the seats.
   - **Request Body**:
     ```json
     {
       "seat_ids": [1, 2, 3],
       "status": "locked",  // Or "booked"
       "locked_at": "2024-11-29T10:30:00"
     }
     ```

---

### **Payment System**

#### 1. **Initiate Payment**
   - **Endpoint**: `POST /api/payments/initiate`
   - **Description**: Initiates a payment process for booking.
   - **Request Body**:
     ```json
     {
       "booking_id": 12345,
       "amount": 150.00
     }
     ```

#### 2. **Confirm Payment**
   - **Endpoint**: `POST /api/payments/confirm`
   - **Description**: Confirms the payment status after payment is processed.
   - **Request Body**:
     ```json
     {
       "payment_id": 98765,
       "status": "completed"
     }
     ```

#### 3. **Get Payment Details**
   - **Endpoint**: `GET /api/payments/{payment_id}`
   - **Description**: Fetch the payment status and details.

---

### **Notification System**

#### 1. **Get User Notifications**
   - **Endpoint**: `GET /api/users/{user_id}/notifications`
   - **Description**: Get a list of notifications for a user.

#### 2. **Mark Notification as Read**
   - **Endpoint**: `PUT /api/users/{user_id}/notifications/{notification_id}/read`
   - **Description**: Mark a specific notification as read.


#### 3. **Send Push Notification**
   - **Endpoint**: `POST /api/notifications/push`
   - **Description**: Send a push notification to users (admin access required).
   - we can use this to send push notification of booking sucess or failure or event_schedule notification   
  - Notification Can also be stored in database but did not have time to implement this
---

### **Concurrency and Lock Status**

- The **Lock Status** mechanism is used to handle concurrency for seat bookings. When a user selects a seat, it is marked as **locked**, and the **locked_at** timestamp is recorded. This prevents other users from booking the same seat at the same time.
  
- **Concurrency control**: If a seat is marked as **locked**, any subsequent booking attempts for that seat will be prevented, ensuring that the seat is only **booked** once the payment is successfully completed. If the lock expires (after a certain timeout or payment failure), the seat becomes available again.

- The **status** of seats can be:
  - `available`: The seat is not booked or locked.
  - `locked`: The seat is temporarily reserved for a user (pending confirmation).
  - `booked`: The seat is confirmed as part of the user's booking.

- Payment Status can also be Maintained the similar way by maintaining the Simmilar way with payment_status
  

