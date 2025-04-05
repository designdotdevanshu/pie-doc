# Pie: A Scalable and Modern Multi-Vendor eCommerce Platform

Here's a breakdown of 10 key tasks for the development roadmap:

## Task 1: User Authentication & Authorization System

Implement the core user system with role-based access control for Customers, Sellers, and Admins. This includes:

- Setting up Supabase Auth integration
- Creating user registration and login flows
- Implementing role-based middleware and guards
- Building profile management for different user types

## Task 2: Product Catalog Management

Build the product management system allowing sellers to:

- Create, read, update, and delete products
- Manage product categories (with hierarchy support)
- Handle product images (both main and additional)
- Set up product variants (colors and sizes)
- Implement product tagging system

## Task 3: Shop Management System

Create the shop management functionality:

- Shop creation and verification workflow
- Shop profile and settings management
- Shop metrics and analytics dashboard
- Address management for shops

## Task 4: Shopping Cart & Checkout Flow

Develop the core shopping experience:

- Cart creation and management
- Add/remove products from cart
- Apply coupon codes
- Calculate totals, taxes, and shipping
- Checkout process with address selection

## Task 5: Order Processing System

Implement the order lifecycle management:

- Order creation from cart
- Order status updates and tracking
- Order history for customers
- Order management for sellers
- Order notifications

## Task 6: Payment Integration

Set up the payment processing system:

- Razorpay integration
- Payment status tracking
- Transaction recording
- Receipt generation
- Refund processing

## Task 7: Admin Dashboard & Management Tools

Build the administrative interface:

- Seller verification system
- Platform metrics and reporting
- Content management
- User management (ban/unban)
- Support system

## Task 8: API Documentation & Testing

Create comprehensive documentation and testing:

- Swagger API documentation
- Integration tests for core flows
- Unit tests for critical business logic
- Performance testing

## Task 9: Deployment & DevOps Setup

Set up your deployment pipeline:

- Docker containerization
- GitHub Actions CI/CD
- AWS infrastructure provisioning
- Nginx configuration
- Database migration strategy

## Task 10: Caching & Performance Optimization

Implement caching and optimize performance:

- Redis caching for product catalog
- Query optimization for complex joins
- Image optimization and CDN setup
- API response time improvements

## Project Overview

Welcome to Pie, an innovative multi-vendor eCommerce platform designed to simplify business and management operations. Firstly I will go with MVP which will meet the minimum requirements to run our initial business, which means I am prioritising the user fetching feature and the manager roll feature that will be developed first. Here I am planning version (v1) requirements.

## System Design

The system should meet the following requirements:

### Functional requirements

I will design the system for five diffrent role Customer, Seller, Admin

**Customers**

- Buy, wishlist, or report product
- Checkout cart
- Take AI guidance to shop (V2)
- View,search and filter products
- Make and Track Order
- Read blogs (V2)
- Contact with supports or chat with seller (V2)
- Take the help of map to navigate towards shop (V2)

**Seller**

- Remove reported item
- Create discount, Offer
- Create Product
- Edit product(availability and others)
- Accept order
- Update tracking
- Chat with customer (V2)

**Admin**

- Verify and edit,ban seller
- Add, change the content of the application
- Manage the sales and Seller of the software
- Business supervision
- Assist seller to get a better user experience
- Collect feedback from seller

### Non-Functional requirements

- High reliability.
- High availability with minimal latency.
- The system should be scalable and efficient (V2)

### Extended requirements

- Customers can rate the product, shop after it's completed.
- Payment processing.
- Metrics and analytics (V2).

## Estimation and Constraints

Let's start with the estimation and constraints.

### Traffic

Let us assume we have 10K daily active users (DAU). If on average each user performs 10 actions (such as request a check available product, addtoCart, checkout, etc.) we will have to handle 100K requests daily.

$$
10,000 \times 10 \space actions = 100,000 \space /day
$$

**What would be Requests Per Second (RPS) for our system?**

100000 requests per day translate into 1.2 requests per second.

$$
\frac{100,000 \space}{(24 \space hrs \times 3600 \space seconds)} = \sim 1.2 \space requests/second
$$

### Storage

If we assume each user comsume on average is 400 bytes, we will require about 0.04 GB of database storage every day.

$$
100,000 \space \times 400 \space bytes = \sim 0.04 \space GB/day
$$

And for 10 years, we will require about 146 GB of storage.

$$
0.04 \space GB \times 10 \space years \times 365 \space days = \sim 146 \space GB
$$

### Bandwidth

As our system is handling 0.04 GB of ingress every day, we will require a minimum bandwidth of around 0.46 KB per second.

$$
\frac{0.04 \space GB}{(24 \space hrs \times 3600 \space seconds)} = \sim 0.46 \space KB/second
$$

### High-level estimate

Here is our high-level estimate:

| Type                      | Estimate   |
| ------------------------- | ---------- |
| Daily active users (DAU)  | 10,000     |
| Requests per second (RPS) | 1.2/s      |
| Storage (per day)         | ~0.04 GB   |
| Storage (10 years)        | ~146 GB    |
| Bandwidth                 | ~0.46 KB/s |

## Data Model Design

This is the general data model which reflects our requirements.

![ERD](../figures/erd.svg)

We have the following tables:

### User

Stores core user data.

- `id` (UUID, Primary Key)
- `email` (String, Unique)
- `fullName` (String)
- `role` (ENUM: Customer, Seller, Admin)

### Customer

Holds customer-specific details.

- `id` (UUID, Primary Key)
- `userId` (UUID, Foreign Key → User.id)
- `dateOfBirth` (Date)
- `gender` (ENUM: Male, Female, Other)
- `referralCode` (String)
- `loyaltyPoints` (Integer)

### Seller

Stores seller-specific details.

- `id` (UUID, Primary Key)
- `userId` (UUID, Foreign Key → User.id, Unique)
- `businessName` (String)
- `taxId` (String, Unique)
- `rating` (Float)
- `totalSales` (Integer)

### Contact

Manages multiple contact numbers for a user.

- `id` (UUID, Primary Key)
- `userId` (UUID, Foreign Key → User.id)
- `type` (ENUM: Mobile, Work, Home, Emergency)
- `number` (String)

### Address

Stores multiple addresses for customers, sellers, and shops.

- `id` (UUID, Primary Key)
- `customerId` (UUID, Foreign Key → Customer.id, Nullable)
- `sellerId` (UUID, Foreign Key → Seller.id, Nullable)
- `shopId` (UUID, Foreign Key → Shop.id, Nullable)
- `type` (ENUM: Home, Work, Billing, Shipping)
- `street` (String)
- `city` (String)
- `state` (String)
- `country` (String)
- `postalCode` (String)

### Shop

Stores shop details.

- `id` (UUID, Primary Key)
- `shopName` (String)
- `sellerId` (UUID, Foreign Key → Seller.id)
- `rating` (Float, Default: 0.0)
- `isVerified` (Boolean, Default: False)

### Product

Stores product information.

- `id` (UUID, Primary Key)
- `productName` (String)
- `mainImage` (String)
- `additionalImages` (String[])
- `categoryId` (UUID, Foreign Key → Category.id)
- `shortDescription` (String)
- `productDescription` (String)
- `price` (Decimal(10,2))
- `discountPrice` (Decimal(10,2))
- `discountPercentage` (Float)
- `inStock` (Boolean)
- `colorId` (UUID, Foreign Key → Color.id, Nullable)
- `sizeId` (UUID, Foreign Key → Size.id, Nullable)

### **Image**

Manages all images dynamically for products, shops, categories, and more.

- `id` (UUID, Primary Key)
- `imageUrl` (String, Not Null)
- `type` (ENUM: `Product`, `Shop`, `Category`, `Other`)
- `referenceId` (UUID, Foreign Key → **Depends on `type`**)
- `createdAt` (Timestamp, Default: `NOW()`)
- `updatedAt` (Timestamp, Auto-update on change)

### Category

Manages product categories, supporting hierarchy.

- `id` (UUID, Primary Key)
- `categoryName` (String, Unique)
- `imageUrl` (String)
- `parentCategoryId` (UUID, Foreign Key → Category.id, Nullable)

### Orders

Tracks all customer orders.

- `id` (UUID, Primary Key)
- `userId` (UUID, Foreign Key → User.id)
- `subTotal` (Decimal(10,2))
- `shippingCharge` (Decimal(10,2))
- `shopId` (UUID, Foreign Key → Shop.id)
- `tax` (Decimal(10,2))
- `total` (Decimal(10,2))
- `paidAmount` (Decimal(10,2))
- `orderStatus` ENUM could include:
  - `Pending` → Order placed but not processed
  - `Processing` → Being packed/shipped
  - `Shipped` → Dispatched but not delivered
  - `Delivered` → Successfully received
  - `Canceled` → Canceled by user/admin
- `txnId` (String)
- `couponId` (UUID, Foreign Key → Coupon.id, Nullable)
- `isPaid` (Boolean, Default: False)
- **Timestamps:**
  - `orderPlacedAt`
  - `paymentAcceptedAt`
  - `deliveredToCourierAt`
  - `beingDelivered`
  - `delivered`
  - `canceledAt`
- `createdBy` (UUID, Foreign Key → User.id)
- `updatedBy` (UUID, Foreign Key → User.id)
- `orderNotes` (Text, Nullable)

### What kind of database should we use?

We are developing a monolithic application, and we will use a single [PostgreSQL](https://www.postgresql.org) database to store all of our data. While our data model is quite relational, keeping everything in one database is suitable for our current architecture. This allows us to leverage PostgreSQL's powerful relational capabilities, including complex queries, transactions, and referential integrity, which are critical for our application.

Despite using a monolithic architecture, we can still ensure scalability and maintainability by organizing the database schema in a way that logically separates concerns. We will define clear ownership of tables and ensure that the application components interact with the database in a modular and efficient way. This approach enables us to take full advantage of PostgreSQL's features while avoiding the complexity of managing multiple databases.

## High-level design

Now let us do a high-level design of our system.

We will be using Monolith architecure while we are building the MVP. According to demand we will scale it horizontally and If it reach to limitations we will consider to build to microservice.

## How is the service expected to work?

Our service operates as follows:

### **1️⃣ Customer Experience**

- Customers **browse, search, and filter** products.
- They **add products to their cart, apply coupons, and choose a payment method.**
- Customers can **wishlist items** for later purchases.
- Orders can be **tracked** in real time, with notifications.
- Customers can **request returns/refunds** based on eligibility.
- They can **contact support** for order issues, disputes, or product inquiries.

### **2️⃣ Seller Responsibilities**

- Sellers **list products, update stock, and offer discounts.**
- They must **accept, reject, or fulfill orders** within time limits.
- Sellers **update tracking details** and ensure smooth delivery.
- They can **edit or remove their products** if necessary.
- Sellers can **respond to customer inquiries** regarding their products.

### **3️⃣ Admin Supervision**

- Admins **oversee the platform**, ensuring smooth operations.
- They **verify sellers**, approve shops, or **ban fraudulent accounts.**
- Admins **manage platform content**, including promotions & banners.
- They **resolve escalated disputes** between customers and sellers.
- Admins track **business analytics & performance metrics.**

### Payments

Handling payments at scale is challenging, to simplify our system we can use a third-party payment processor like [Razorpay](https://razorpay.com/), [Stripe](https://stripe.com) or [SSLCommerz](https://www.paypal.com). Once the payment is complete, the payment processor will redirect the user back to our application and we can set up a [webhook](https://en.wikipedia.org/wiki/Webhook) to capture all the payment-related data.

### Notifications

We will send notifications through our server in future.

### Identify and resolve bottlenecks

Let us identify and resolve bottlenecks such as single points of failure in our design:

- "What if one of our services crashes?"
- "How can we reduce the load on our database?"

To make our system more resilient we can do the following: (V2)

- Implement load balancer to distribute traffic across multiple instances or containers.
- Use a queue system (e.g., RabbitMQ or Kafka) to offload heavy tasks like email sending, Invoice generation etc.
- Implement logging to identify the problem.
- Using caching to reduce load on our database.

## Technology Stack

- **Backend**: Typescript, Node.js, Express.js, Docker, Nginx, Prisma (Supabase/Neon DB/Prisma DB, PostgreSQL), Redis, Razorpay, Socket.io, zod, bcrypt, JWT, etc

- **Containerization**:
  - **Docker** for creating, managing, and deploying containers.
  - **Docker Hub** for publishing and accessing the container images.
- **Deployment**:
  - **AWS** for hosting the application.
  - **Nginx** for reverse proxying and load balancing.
  - **PM2** for process management and monitoring.
- **Version Control**: GitHub for version control and collaboration.
- **CI/CD**: GitHub Actions for continuous integration and deployment.
- **Documentation**: Swagger for API documentation.
- **Email Service**: Email.js for sending emails.
- **Payment Gateway**: Razorpay for handling payments.
- **Storage**: AWS S3 for storing images and other assets.
- **Analytics**: Google Analytics for tracking user behavior and application performance.
