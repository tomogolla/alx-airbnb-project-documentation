# Requirements specifications for backend features
## **User Authentication**
## Backend Feature: **User Authentication**

### **1. Objective**

Enable users (Guests or Hosts) to securely register, log in, and manage their profile using JWT-based authentication and optional OAuth integration.

---

### **2. Functional Requirements**

| Feature                  | Description                                                                   |
| ------------------------ | ----------------------------------------------------------------------------- |
| User Registration        | Allow new users to register with a role (guest or host), email, and password. |
| User Login               | Allow registered users to log in using email/password.                        |
| OAuth Login (optional)   | Enable social login via Google/Facebook.                                      |
| Profile Update           | Allow authenticated users to update their profile info.                       |
| Token Refresh (optional) | Support refresh token flow for longer sessions.                               |

---

### **3. API Endpoints**

#### **1. POST `/api/auth/register`**

* **Description:** Register a new user.
* **Input (JSON):**

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "role": "guest",
  "full_name": "John Doe"
}
```

* **Validation Rules:**

  * `email` must be unique and valid format.
  * `password` must be ≥ 8 characters, include numbers and symbols.
  * `role` must be either `guest` or `host`.

* **Output (Success):**

```json
{
  "message": "Registration successful",
  "token": "<JWT_ACCESS_TOKEN>"
}
```

* **Error Codes:**

  * `400`: Validation errors
  * `409`: Email already exists

---

#### **2. POST `/api/auth/login`**

* **Description:** Login an existing user.
* **Input (JSON):**

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

* **Output (Success):**

```json
{
  "token": "<JWT_ACCESS_TOKEN>",
  "user": {
    "id": 1,
    "email": "user@example.com",
    "role": "host",
    "full_name": "John Doe"
  }
}
```

* **Error Codes:**

  * `401`: Invalid credentials

---

#### **3. GET `/api/users/me`**

* **Description:** Get authenticated user’s profile.
* **Authorization:** Bearer Token (JWT)
* **Output:**

```json
{
  "id": 1,
  "email": "user@example.com",
  "role": "guest",
  "full_name": "John Doe",
  "created_at": "2024-01-01T12:00:00Z"
}
```

---

#### **4. PUT `/api/users/me`**

* **Description:** Update user profile.
* **Authorization:** Bearer Token
* **Input (JSON):**

```json
{
  "full_name": "Johnathan Doe",
  "phone": "+254712345678",
  "profile_photo": "https://cdn.com/photo.jpg"
}
```

* **Validation:**

  * Optional fields; photo must be a valid image URL.

---

### **4. Validation Rules Summary**

* Email must be unique and properly formatted.
* Password must meet complexity requirements.
* All endpoints requiring authentication must validate JWT and user role.
* Rate limiting on login to prevent brute-force attacks.

---

### **5. Performance Criteria**

* Login and registration requests should complete in under **300ms**.
* JWTs should be issued and verified in under **50ms**.
* Rate limiting: max 5 login attempts/minute per IP.

---

### **6. Security Considerations**

* Passwords hashed using bcrypt or Argon2.
* JWTs signed with a secure secret and short expiration (e.g., 15 min).
* HTTPS enforced for all endpoints.
* Optional 2FA integration for higher security.


---
---
## **Property Management**

### **1. Objective**

Enable **hosts** to create, manage, and delete their property listings with structured information such as title, description, price, location, and availability.

---

### **2. Functional Requirements**

| Feature         | Description                                                                 |
| --------------- | --------------------------------------------------------------------------- |
| Add Property    | Allow a host to create a new property listing.                              |
| Edit Property   | Allow a host to update the details of their own listings.                   |
| Delete Property | Allow a host to delete their property listings.                             |
| View Listings   | Allow hosts to view their own properties and guests to browse all listings. |

---

### **3. API Endpoints**

#### **1. POST `/api/properties`**

* **Description:** Create a new property listing.
* **Authorization:** Required (host only)
* **Input (JSON):**

```json
{
  "title": "Modern Loft in Nairobi",
  "description": "Spacious loft in Kilimani with Wi-Fi and balcony.",
  "price_per_night": 60.0,
  "location": "Nairobi, Kenya",
  "max_guests": 4,
  "amenities": ["Wi-Fi", "Balcony", "Parking"],
  "availability": {
    "start_date": "2025-07-01",
    "end_date": "2025-08-31"
  },
  "images": [
    "https://cdn.site.com/loft1.jpg",
    "https://cdn.site.com/loft2.jpg"
  ]
}
```

* **Validation Rules:**

  * Required: `title`, `description`, `price_per_night`, `location`, `max_guests`, `availability`
  * Price must be a positive number
  * Images must be valid URLs

* **Output (Success):**

```json
{
  "id": 101,
  "message": "Property created successfully"
}
```

* **Error Codes:**

  * `400`: Validation errors
  * `403`: If user is not a host

---

#### **2. PUT `/api/properties/:id`**

* **Description:** Update an existing property listing.
* **Authorization:** Required (must be the property owner)
* **Input (JSON):** Same as `POST`, but partial updates allowed.
* **Output (Success):**

```json
{
  "message": "Property updated successfully"
}
```

* **Error Codes:**

  * `403`: Unauthorized (not owner)
  * `404`: Property not found

---

#### **3. DELETE `/api/properties/:id`**

* **Description:** Delete a property.
* **Authorization:** Required (must be owner)
* **Output:**

```json
{
  "message": "Property deleted"
}
```

---

#### **4. GET `/api/properties`**

* **Description:** Retrieve all available property listings.

* **Query Parameters:**

  * `location`, `price_min`, `price_max`, `amenities`, `guests`, `page`, `limit`

* **Output (Success):**

```json
[
  {
    "id": 101,
    "title": "Modern Loft in Nairobi",
    "price_per_night": 60,
    "location": "Nairobi, Kenya",
    "max_guests": 4,
    "host_id": 9,
    "rating": 4.8,
    "thumbnail": "https://cdn.site.com/loft1.jpg"
  }
]
```

---

#### **5. GET `/api/properties/me`**

* **Description:** Get properties owned by the authenticated host.
* **Authorization:** Required (host)
* **Output:** Same as above, filtered by owner.

---

### **4. Validation Rules Summary**

* All string fields must be trimmed and non-empty.
* Price must be numeric and > 0.
* Amenities must be a valid list.
* Dates must be ISO 8601 format.

---

### **5. Performance Criteria**

* Listing creation/update should complete in under **500ms**.
* Support **pagination** and **filtering** on public listing queries.
* Queries should use **indexed fields** for filtering (e.g., location, price).

---

### **6. Security Considerations**

* Only the listing owner (host) can update or delete a property.
* All listing media must be sanitized URLs or uploaded via a secure uploader.
* Role-based access control (RBAC): Only hosts can create or manage listings.

---
---
## : Booking System

### 1. Objective

Enable guests to book available properties, manage their bookings (e.g., cancel), and allow hosts to view incoming booking requests. The system must ensure bookings are valid (no double-booking) and track booking statuses throughout the process.

---

### 2. Functional Requirements

| Feature                | Description                                                                                   |
| ---------------------- | --------------------------------------------------------------------------------------------- |
| Create Booking         | Guests can book a property for specific available dates.                                      |
| Cancel Booking         | Guests and hosts can cancel bookings before or under specific conditions.                     |
| View Bookings          | Guests can view their bookings; hosts can view bookings for their properties.                 |
| Booking Status         | System must manage and expose the current status: pending, confirmed, canceled, or completed. |
| Prevent Double Booking | System must validate date availability before confirming.                                     |

---

### 3. API Endpoints

#### 1. POST `/api/bookings`

* Description: Create a new booking for a property.
* Authorization: Required (guest only)
* Input (JSON):

```json
{
  "property_id": 101,
  "start_date": "2025-08-01",
  "end_date": "2025-08-05",
  "guests": 2
}
```

* Validation Rules:

  * Dates must be valid and within the property’s availability window.
  * Property must not already be booked for any overlapping dates.
  * Guest count must not exceed the property’s max\_guests value.
* Output (Success):

```json
{
  "booking_id": 3001,
  "message": "Booking request submitted",
  "status": "pending"
}
```

* Error Codes:

  * 400: Invalid input or unavailable dates
  * 403: If user is not a guest
  * 409: Overlapping booking already exists

---

#### 2. GET `/api/bookings/me`

* Description: Get all bookings made by the authenticated guest.
* Authorization: Required
* Output:

```json
[
  {
    "booking_id": 3001,
    "property": "Modern Loft in Nairobi",
    "start_date": "2025-08-01",
    "end_date": "2025-08-05",
    "status": "confirmed"
  }
]
```

---

#### 3. GET `/api/bookings/property/:property_id`

* Description: Get all bookings for a property (host only).
* Authorization: Required (host must own the property)
* Output: Same structure as above.

---

#### 4. PUT `/api/bookings/:id/cancel`

* Description: Cancel a booking by guest or host.
* Authorization: Required (guest who booked or host who owns the property)
* Output:

```json
{
  "message": "Booking canceled",
  "status": "canceled"
}
```

* Error Codes:

  * 403: Unauthorized
  * 404: Booking not found
  * 409: Cancellation not allowed based on timing/policy

---

### 4. Validation Rules Summary

* Dates must be ISO 8601 formatted (`YYYY-MM-DD`) and not in the past.
* End date must be after start date.
* Guest must not exceed the max allowed capacity.
* No overlapping confirmed bookings for the same property and date range.

---

### 5. Performance Criteria

* Booking creation request must complete in under 400 milliseconds.
* The system should handle at least 100 concurrent booking requests with transactional integrity.
* Booking availability checks must use optimized queries with indexes on date fields and property ID.

---

### 6. Security and Data Integrity

* Access control must ensure:

  * Only the booking owner or property host can view/cancel the booking.
  * Only authenticated users can access the endpoint.
* Booking creation must use database-level locking or atomic operations to prevent race conditions and double-bookings.
* Booking records must be immutable after a stay is completed (except by admin).

---



