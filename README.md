/task1.md
1. Identified Logical Contradictions and Deficiencies
1.1 Price Fixation Conflict

Requirement 7 states that the product price is fixed at the moment of adding to the cart and does not change.

Requirement 13 states that if the product price changes in the catalog, the system must automatically update it in all users' carts.

Issue:
These requirements contradict each other. A product price cannot simultaneously remain fixed and be dynamically updated.

1.2 Quantity Reduction Conflict

Requirement 2 states that the quantity of a product cannot be reduced below 1, and a separate button must be used for removal.

Requirement 9 states that if the quantity is reduced to 0, the product is removed from the cart.

Issue:
If quantity cannot be reduced below 1, then reducing it to 0 should not be possible. This creates a logical inconsistency.

1.3 Redundant Requirement

Requirement 5 states: “Products in the cart may be different.”

Issue:
This statement is redundant and does not provide any meaningful functional requirement, since a cart naturally supports multiple products.

1.4 Advertisement Scheduling Ambiguity

Requirement 10 allows advertisements in the cart.

Requirement 11 states that advertisements must be shown every weekday in the morning and evening.

Issues Identified:

Time zone is not specified.

The definition of “morning” and “evening” is unclear.

It is not defined whether ads should only appear during those periods or be refreshed at those times.

No information about personalization or targeting logic.

1.5 Generic Error Message

Requirement 6 specifies a single message: “Cart limit exceeded.”

Issue:
There are multiple limits (per-product limit, total quantity limit, number of distinct products). A generic message does not provide sufficient feedback to the user and negatively impacts user experience.

1.6 Missing Edge Case Scenarios

The specification does not clarify:

Whether the cart is available for guest users.

How long cart data is stored.

What happens if a product becomes unavailable after being added to the cart.

Whether cart limits are configurable.

How the system behaves across multiple user sessions or devices.

2. Revised and Logically Consistent Version of the Requirements
Shopping Cart Functionality – Corrected Version

A user can add from 1 to 10 units of a single product to the cart.

The cart may contain a maximum of 5 distinct products.

The total quantity of all products in the cart cannot exceed 20 units.

If any cart limit is exceeded, the system must display a specific error message indicating which limit was violated.

A user can change the quantity of a product in the cart to a minimum value of 1.

A dedicated “Remove” button must be provided to delete a product from the cart.

The product price is fixed at the moment of addition to the cart and remains unchanged until checkout confirmation.

The cart page must display:

Product name

Quantity

Price per unit

Total price per product

Total cart amount

If a product becomes unavailable or out of stock, the user must be notified before checkout and prevented from completing the order until the issue is resolved.

Advertisements may be displayed in a separate section of the cart page.

Advertisement scheduling rules (time zone, display periods, personalization logic) must be defined in a separate marketing specification.

3. Clarifying Questions to the Product Owner

Should the cart functionality be available to guest users or only authenticated users?

How long should cart data be stored (session-based or persistent storage)?

Should the product price remain fixed or be updated dynamically before checkout?

What should happen if a product goes out of stock after being added to the cart?

Should advertisements be personalized based on user behavior?

Which time zone defines advertisement display periods?

Should cart limits (5 products, 20 total items) be configurable via admin settings?

Should discount codes or promotions override the fixed price rule?

How should the system handle cart synchronization across multiple devices?

Is there a cart expiration policy?



/task2.md
Task 2 – REST API Design
Feature: Partner Stores Screen

Project: Online Store “Petrushka Green”

1. API Overview

When a user navigates to the “Partner Stores” screen in the mobile application, the client application must request a list of partner stores from the backend.

The API should follow REST principles and return structured JSON data suitable for rendering the screen according to the provided mockups.

2. REST API Request Example
Endpoint
GET /api/v1/partner-stores

Description

Returns a list of active partner stores available for display in the mobile application.

Optional Query Parameters (if needed)
Parameter	Type	Description
city	string	Filters stores by city
page	int	Page number for pagination
size	int	Number of records per page

Example with query parameters:

GET /api/v1/partner-stores?city=moscow&page=1&size=10

3. Example JSON Response
{
  "timestamp": "2026-02-16T10:30:00Z",
  "status": "success",
  "data": {
    "stores": [
      {
        "id": 101,
        "name": "Green Market",
        "description": "Fresh vegetables and organic products",
        "logoUrl": "https://cdn.petrushka.ru/logos/green-market.png",
        "bannerImageUrl": "https://cdn.petrushka.ru/banners/green-market.jpg",
        "externalUrl": "https://greenmarket.ru",
        "isActive": true
      },
      {
        "id": 102,
        "name": "Eco Farm",
        "description": "Natural dairy and farm products",
        "logoUrl": "https://cdn.petrushka.ru/logos/eco-farm.png",
        "bannerImageUrl": "https://cdn.petrushka.ru/banners/eco-farm.jpg",
        "externalUrl": "https://ecofarm.ru",
        "isActive": true
      }
    ],
    "pagination": {
      "page": 1,
      "size": 10,
      "totalElements": 25,
      "totalPages": 3
    }
  }
}

4. Field Description
Field	Description
id	Unique identifier of the partner store
name	Store name displayed on UI
description	Short description shown on store card
logoUrl	URL of store logo
bannerImageUrl	Optional promotional banner
externalUrl	URL to external partner website
isActive	Indicates whether the store is currently active
pagination	Pagination metadata
5. Behavior on Click

When the user taps on a store card:

The mobile application retrieves the externalUrl field.

The application opens the external link in:

In-app browser (recommended), or

Device default browser.

6. Error Response Example
500 Internal Server Error
{
  "timestamp": "2026-02-16T10:35:00Z",
  "status": "error",
  "errorCode": "INTERNAL_SERVER_ERROR",
  "message": "Unable to retrieve partner stores. Please try again later."
}

7. Design Considerations

API follows REST principles (resource-based endpoint).

Versioning included (v1).

Supports pagination for scalability.

Returns only active stores.

External link is explicitly provided to support navigation outside the app.

Response structured for future extensibility.

/task3.md
Task 3 – High-Level Architecture for Push Notifications
Feature: Push Notification System

Project: Online Store “Petrushka Green”
Architecture Assumption: Microservices-based backend

1. Objective

Design a scalable and reliable push notification system for the mobile application that supports multiple notification types, including:

Cart inactivity reminders

Order cancellation notifications

Promotional campaigns

System announcements

The system must be flexible, event-driven, and scalable.

2. High-Level Architecture Overview

The push notification system should follow an event-driven microservices architecture.

Main Components:

Mobile Application (iOS / Android)

API Gateway

Backend Microservices:

Cart Service

Order Service

Promotion Service

Event Broker (Kafka or RabbitMQ)

Notification Service

Push Provider:

Firebase Cloud Messaging (FCM) for Android

Apple Push Notification Service (APNs) for iOS

Database (for storing notification logs and user device tokens)

3. Architecture Flow Description
Step 1 – Event Generation

A business event occurs:

Cart inactive for X hours

Order status changed to “Cancelled”

Marketing campaign triggered

The responsible microservice (e.g., Cart Service, Order Service) publishes an event to the Message Broker.

Example event:

CartInactiveEvent
OrderCancelledEvent
PromotionCampaignEvent

Step 2 – Message Broker

A message broker (Kafka / RabbitMQ):

Decouples services

Ensures asynchronous processing

Improves scalability

Supports retry mechanisms

Step 3 – Notification Service

The Notification Service:

Subscribes to relevant events

Retrieves user device tokens

Applies notification templates

Personalizes message content

Sends push request to push provider

It is a dedicated microservice responsible only for notifications.

Step 4 – Push Provider

Notification Service communicates with:

Firebase Cloud Messaging (FCM) – Android

Apple Push Notification Service (APNs) – iOS

Push provider delivers notification to the user's device.

Step 5 – Mobile Application

The mobile app:

Receives push

Displays notification

Optionally opens deep link inside app

4. Architecture Diagram (Block Representation)

You can include this in README as text or draw a diagram:

[Mobile App]
      |
      v
[API Gateway]
      |
      v
------------------------------
|   Microservices Layer      |
|  - Cart Service            |
|  - Order Service           |
|  - Promotion Service       |
------------------------------
      |
      v
[Message Broker (Kafka)]
      |
      v
[Notification Service]
      |
      v
[FCM / APNs]
      |
      v
[User Device]

5. Data Storage Components
Device Token Storage

A database table:

Field	Description
user_id	User identifier
device_token	Push token
platform	iOS / Android
updated_at	Timestamp
Notification Log Table

Stores:

Notification type

Delivery status

Retry attempts

Timestamp

Useful for analytics and troubleshooting.

6. Key Architectural Decisions
6.1 Event-Driven Approach

Ensures loose coupling between business services and notification logic.

6.2 Dedicated Notification Service

Improves maintainability and scalability.

6.3 Asynchronous Processing

Prevents blocking main business operations.

6.4 Scalability

Broker allows horizontal scaling.

Notification service can scale independently.

6.5 Reliability

Retry mechanism for failed deliveries.

Dead-letter queue for failed messages.

7. Non-Functional Considerations

High availability

Fault tolerance

Horizontal scalability

Secure token storage

GDPR compliance for marketing pushes

Rate limiting for bulk campaigns

8. Optional Enhancements

User notification preferences

Push scheduling system

A/B testing for marketing pushes

Analytics service integration

Why This Architecture Is Appropriate

Aligns with microservices architecture

Scalable and event-driven

Supports multiple notification types

Easy to extend

Production-ready approach

/tTask 4 – Interview Preparation Overview
1. Types of Requirements in System Analysis
1.1 Requirements Received from Business

A System Analyst typically receives:

Business Requirements – High-level goals (e.g., increase sales, improve user retention)

Stakeholder Requirements – Expectations from product owner or marketing team

Regulatory Requirements – Legal or compliance constraints

1.2 Requirements Delivered to Developers

The System Analyst translates business needs into:

Functional Requirements – What the system must do

Non-Functional Requirements – Performance, scalability, security

Technical Specifications – API contracts, data models

User Stories / Use Cases

2. ER Diagram (Entity-Relationship Diagram)

An ER diagram is a graphical representation of:

Entities (e.g., User, Order, Product)

Attributes (e.g., user_id, price)

Relationships (e.g., User places Order)

Purpose:

Visualize database structure

Define relationships

Support database design

Example (simplified):

User (1) —— (M) Order
Order (1) —— (M) OrderItem
OrderItem (M) —— (1) Product

3. Frontend vs Backend
Frontend	Backend
User interface	Business logic
Runs in browser/mobile	Runs on server
React, Flutter	Java, Node.js
UI rendering	API, DB, authentication

Frontend = Presentation Layer
Backend = Logic + Data Layer

4. HTTP vs REST API
HTTP

Protocol used for communication over the web

Defines methods: GET, POST, PUT, DELETE

REST API

Architectural style

Uses HTTP methods

Resource-based endpoints

Stateless communication

Example:

GET /api/v1/orders/10


REST uses HTTP, but HTTP itself is just a protocol.

5. Microservices vs Monolithic Architecture
Monolithic

Single deployable application

Shared database

Easier initial development

Harder to scale independently

Microservices

Multiple independent services

Each service handles specific domain

Independent deployment

Better scalability

Push notification system in Task 3 follows microservice approach.

6. How the Internet Works (High-Level)

When a user types a website address in the browser:

DNS resolves domain to IP address

Browser sends HTTP request to server

Server processes request

Server returns HTTP response

Browser renders HTML/CSS/JS

7. SQL Basics

Common operations:

SELECT – retrieve data

INSERT – add data

UPDATE – modify data

DELETE – remove data

Example:

SELECT * FROM orders WHERE user_id = 10;


Important concepts:

Primary Key

Foreign Key

Index

JOIN

8. REST vs SOAP (Optional Knowledge)
REST	SOAP
Lightweight	Heavy protocol
JSON	XML
Faster	More structured
Common in modern systems	Used in legacy enterprise systems
9. Designing a Simple Database

Example tables for e-commerce:

users

products

orders

order_items

cart

cart_items

Relationships:

One user → many orders

One order → many order items

10. Kafka vs RabbitMQ
Kafka	RabbitMQ
Distributed streaming platform	Message broker
High throughput	Reliable queue system
Log-based storage	Queue-based storage
Best for event streaming	Best for task queues

For large-scale event-driven systems → Kafka preferred.
For simpler queue-based messaging → RabbitMQ sufficient.

11. Backend for Frontend (BFF) Pattern

BFF is a pattern where:

Separate backend layer is created specifically for a frontend application (mobile/web).

Tailors API responses to UI needs.

Reduces over-fetching and under-fetching.

Useful when:

Mobile app and web app have different requirements.

Conclusion

The proposed solutions demonstrate understanding of:

Requirements analysis

API design principles

Microservices architecture

Event-driven systems

Database fundamentals

REST communication

The system design aligns with scalable, production-ready backend architecture practices.ask4.md

