# Book Reader Web Application - Comprehensive Project Guide

This guide provides a complete roadmap for developing a Book Reader Web Application for comics, manga, and graphic novels from scratch to deployment. It covers every aspect of the project, including planning, architecture, implementation, testing, and deployment.

## 1. Project Overview

### Purpose
To create a web platform where users can read, upload, and monetize various types of comics and graphic novels from around the world, including Manga, Manhwa, Webtoons, American Comics, and other regional formats.

### Business Model
- Only 2 types of users.
- One type of user is admin.
- One type of user is user.  
- Every user can upload content.
- Users need to purchase credits to read content.
- Content uploaders earn a share of the credits spent on their content.
- Users can exchange credits to real money back.
- Users can define credit cost when upload the book.
- Users can create whishlist which the user wants to read or buy.
- Users can add books to readlist which the user has read.
- Users can add books to Currently Reading which the user is actively reading.
- Users can add books to bookmarks which the user loves. 
- Users can provide likes, rating(max 5 stars) or feedback(comments) for every books.
- Admin get will 10% fee when users exchange credits to real money back. (Example - Users have 100 credits, if he exchanges that credits to real money back, he will get real money for 90 credits)
- Admin can define (real money) how much for 1 credits.
- Admin management every things and moderation capabilities.

### Target Audience
- Comic/manga readers worldwide
- Content creators looking to monetize their work
- Publishers seeking digital distribution channels

## 2. Requirements Gathering

### Functional Requirements

#### User Management
- Registration and authentication
- User profiles and preferences
- Reading history and bookmarks, readlist, whishlish and currently reading.

#### Content Management
- Multi-format comic catalog
- Metadata support (author, publisher, etc.)
- Upload system for all users
- Support for various file formats

#### Reading Experience
- Format-specific reading modes (page-by-page, continuous scroll)
- Reading direction controls (LTR, RTL, vertical)
- View customization (zoom, brightness)
- Progress tracking

#### Monetization
- Credit system for pay-per-read
- Revenue sharing for content uploaders
- Transaction management

#### Social Features
- Comments and discussions
- Ratings and reviews
- Content recommendations

#### Administration
- Content moderation and approval
- User management
- System configuration
- Analytics

### Non-Functional Requirements
- Responsive design for all devices
- Fast loading times for images
- Secure payment processing
- Scalable architecture
- Content delivery optimization
- Offline reading capability

## 3. Technology Stack Selection

### Backend
- **Framework**: Spring Boot (Java)
- **Database**: PostgreSQL
- **Authentication**: Spring Security with JWT
- **ORM**: JPA/Hibernate with Spring Data
- **File Storage**: Amazon S3 (or similar)
- **Cache**: Redis (or similar)
- **Search**: Elasticsearch (optional for advanced search)

### Frontend
- **Framework**: Next.js 14+ with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State Management**: Zustand
- **Data Fetching**: React Query/Axios
- **Authentication**: NextAuth.js

### DevOps & Infrastructure
- **Containerization**: Docker
- **CI/CD**: GitHub Actions
- **Hosting**: AWS or similar cloud platform
- **CDN**: Cloudflare or similar

## 4. Database Design

### Core Tables Design

1. **Users Schema**
   - users (account information)
   - user_preferences (reading settings)
   - user_credits (credit balance)
   - user_subscriptions (subscription details)

2. **Content Schema**
   - books (comics/manga metadata)
   - chapters (organization of content)
   - pages (individual page data)
   - categories (content classification)
   - tags (additional metadata)

3. **Interaction Schema**
   - reading_progress (user reading positions)
   - user_library (saved content)
   - ratings_reviews (user feedback)
   - comments (discussions)

4. **Monetization Schema**
   - credit_packages (purchasable credit bundles)
   - subscription_plans (subscription options)
   - transactions (payment records)
   - content_purchases (access records)
   - creator_earnings (revenue distribution)

# Database Schema and API Architecture for Book Reader Web Application

## Database Schema

### 1. Users Schema

```sql
-- Users Table
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    profile_image_url VARCHAR(255),
    bio TEXT,
    role VARCHAR(20) NOT NULL DEFAULT 'USER', -- USER, CREATOR, ADMIN
    is_verified BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User Preferences Table
CREATE TABLE user_preferences (
    preference_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    reading_direction VARCHAR(10) DEFAULT 'LTR', -- LTR, RTL, VERTICAL
    theme VARCHAR(10) DEFAULT 'LIGHT', -- LIGHT, DARK
    font_size INTEGER DEFAULT 16,
    language VARCHAR(10) DEFAULT 'en',
    notifications_enabled BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User Credits Table
CREATE TABLE user_credits (
    credit_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    balance DECIMAL(10, 2) DEFAULT 0.00,
    lifetime_earned DECIMAL(10, 2) DEFAULT 0.00,
    lifetime_spent DECIMAL(10, 2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User Subscriptions Table
CREATE TABLE user_subscriptions (
    subscription_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    plan_id INTEGER REFERENCES subscription_plans(plan_id),
    status VARCHAR(20) NOT NULL, -- ACTIVE, CANCELLED, EXPIRED
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP NOT NULL,
    auto_renew BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 2. Content Schema

```sql
-- Books Table
CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    cover_image_url VARCHAR(255),
    author_id INTEGER REFERENCES users(user_id),
    publisher VARCHAR(100),
    publication_date DATE,
    book_type VARCHAR(50) NOT NULL, -- Manga, Manhwa, etc.
    status VARCHAR(20) NOT NULL, -- ONGOING, COMPLETED, HIATUS
    credit_cost DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    is_approved BOOLEAN DEFAULT FALSE,
    is_featured BOOLEAN DEFAULT FALSE,
    view_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categories Table
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Book Categories (Many-to-Many)
CREATE TABLE book_categories (
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(category_id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, category_id)
);

-- Tags Table
CREATE TABLE tags (
    tag_id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Book Tags (Many-to-Many)
CREATE TABLE book_tags (
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(tag_id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, tag_id)
);

-- Chapters Table
CREATE TABLE chapters (
    chapter_id SERIAL PRIMARY KEY,
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    chapter_number DECIMAL(8, 2) NOT NULL, -- Allows for 1.5, 2.5 etc.
    description TEXT,
    credit_cost DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    is_free BOOLEAN DEFAULT FALSE,
    is_approved BOOLEAN DEFAULT FALSE,
    view_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Pages Table
CREATE TABLE pages (
    page_id SERIAL PRIMARY KEY,
    chapter_id INTEGER REFERENCES chapters(chapter_id) ON DELETE CASCADE,
    page_number INTEGER NOT NULL,
    image_url VARCHAR(255) NOT NULL,
    thumbnail_url VARCHAR(255),
    width INTEGER,
    height INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. Interaction Schema

```sql
-- Reading Progress Table
CREATE TABLE reading_progress (
    progress_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    chapter_id INTEGER REFERENCES chapters(chapter_id) ON DELETE SET NULL,
    page_number INTEGER DEFAULT 1,
    percentage_complete DECIMAL(5, 2) DEFAULT 0.00,
    last_read_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, book_id)
);

-- User Library Table
CREATE TABLE user_library (
    library_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    library_type VARCHAR(20) NOT NULL, -- WISHLIST, READLIST, CURRENTLY_READING, BOOKMARK
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, book_id, library_type)
);

-- Ratings and Reviews Table
CREATE TABLE ratings_reviews (
    review_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    review_text TEXT,
    is_approved BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, book_id)
);

-- Comments Table
CREATE TABLE comments (
    comment_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    chapter_id INTEGER REFERENCES chapters(chapter_id) ON DELETE CASCADE,
    parent_comment_id INTEGER REFERENCES comments(comment_id) ON DELETE CASCADE,
    comment_text TEXT NOT NULL,
    is_approved BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Likes Table
CREATE TABLE likes (
    like_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (user_id, book_id)
);
```

### 4. Monetization Schema

```sql
-- Credit Packages Table
CREATE TABLE credit_packages (
    package_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    credit_amount INTEGER NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Subscription Plans Table
CREATE TABLE subscription_plans (
    plan_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    duration_days INTEGER NOT NULL,
    credits_per_month INTEGER DEFAULT 0,
    benefits TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Transactions Table
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE SET NULL,
    transaction_type VARCHAR(20) NOT NULL, -- CREDIT_PURCHASE, SUBSCRIPTION, CREDIT_EXCHANGE
    amount DECIMAL(10, 2) NOT NULL,
    credits_amount INTEGER,
    currency VARCHAR(3) DEFAULT 'USD',
    payment_method VARCHAR(50),
    payment_status VARCHAR(20) NOT NULL, -- PENDING, COMPLETED, FAILED, REFUNDED
    transaction_reference VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Content Purchases Table
CREATE TABLE content_purchases (
    purchase_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE SET NULL,
    chapter_id INTEGER REFERENCES chapters(chapter_id) ON DELETE SET NULL,
    credits_spent DECIMAL(10, 2) NOT NULL,
    purchase_type VARCHAR(20) NOT NULL, -- BOOK, CHAPTER
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Creator Earnings Table
CREATE TABLE creator_earnings (
    earning_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(book_id) ON DELETE SET NULL,
    chapter_id INTEGER REFERENCES chapters(chapter_id) ON DELETE SET NULL,
    purchase_id INTEGER REFERENCES content_purchases(purchase_id) ON DELETE SET NULL,
    credits_earned DECIMAL(10, 2) NOT NULL,
    is_paid BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Credit Exchange Requests Table
CREATE TABLE credit_exchange_requests (
    request_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id) ON DELETE CASCADE,
    credits_amount DECIMAL(10, 2) NOT NULL,
    money_amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL, -- PENDING, APPROVED, REJECTED, COMPLETED
    payment_method VARCHAR(50),
    payment_details TEXT,
    admin_notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## API Architecture

### 1. Authentication and User Management

```
/api/v1/auth
  POST /register                 - Register a new user
  POST /login                    - Login and get JWT token
  POST /refresh-token            - Refresh JWT token
  POST /forgot-password          - Request password reset
  POST /reset-password           - Reset password with token
  GET /verify-email/{token}      - Verify email address

/api/v1/users
  GET /me                        - Get current user profile
  PUT /me                        - Update current user profile
  GET /{userId}                  - Get user profile by ID
  PUT /preferences               - Update user preferences
  GET /preferences               - Get user preferences
  GET /statistics                - Get user reading statistics
```

### 2. Content Management

```
/api/v1/books
  GET /                          - List all books (with filters)
  POST /                         - Create a new book
  GET /{bookId}                  - Get book details
  PUT /{bookId}                  - Update book details
  DELETE /{bookId}               - Delete a book
  GET /featured                  - Get featured books
  GET /popular                   - Get popular books
  GET /recent                    - Get recently added books
  GET /types                     - Get all book types
  GET /search                    - Search books

/api/v1/books/{bookId}/chapters
  GET /                          - List all chapters for a book
  POST /                         - Add a new chapter
  GET /{chapterId}               - Get chapter details
  PUT /{chapterId}               - Update chapter details
  DELETE /{chapterId}            - Delete a chapter

/api/v1/chapters/{chapterId}/pages
  GET /                          - List all pages for a chapter
  POST /                         - Add a new page
  POST /batch                    - Add multiple pages at once
  PUT /{pageId}                  - Update page details
  DELETE /{pageId}               - Delete a page
  PUT /reorder                   - Reorder pages

/api/v1/categories
  GET /                          - List all categories
  POST /                         - Create a new category (admin)
  PUT /{categoryId}              - Update a category (admin)
  DELETE /{categoryId}           - Delete a category (admin)

/api/v1/tags
  GET /                          - List all tags
  POST /                         - Create a new tag (admin)
  DELETE /{tagId}                - Delete a tag (admin)
```

### 3. Reading Experience

```
/api/v1/reader
  GET /books/{bookId}/chapters/{chapterId}  - Get chapter content for reading
  POST /progress                            - Update reading progress
  GET /progress/books/{bookId}              - Get reading progress for a book
  GET /progress                             - Get all reading progress
```

### 4. User Library

```
/api/v1/library
  GET /wishlist                  - Get user's wishlist
  POST /wishlist/{bookId}        - Add book to wishlist
  DELETE /wishlist/{bookId}      - Remove book from wishlist
  
  GET /readlist                  - Get user's read list
  POST /readlist/{bookId}        - Add book to read list
  DELETE /readlist/{bookId}      - Remove book from read list
  
  GET /currently-reading         - Get user's currently reading list
  POST /currently-reading/{bookId} - Add book to currently reading
  DELETE /currently-reading/{bookId} - Remove book from currently reading
  
  GET /bookmarks                 - Get user's bookmarks
  POST /bookmarks/{bookId}       - Add book to bookmarks
  DELETE /bookmarks/{bookId}     - Remove book from bookmarks
```

### 5. Social Features

```
/api/v1/books/{bookId}/ratings
  GET /                          - Get all ratings for a book
  POST /                         - Rate a book
  PUT /                          - Update rating
  DELETE /                       - Delete rating

/api/v1/books/{bookId}/comments
  GET /                          - Get all comments for a book
  POST /                         - Add a comment
  PUT /{commentId}               - Update a comment
  DELETE /{commentId}            - Delete a comment
  POST /{commentId}/replies      - Reply to a comment

/api/v1/chapters/{chapterId}/comments
  GET /                          - Get all comments for a chapter
  POST /                         - Add a comment
  
/api/v1/books/{bookId}/likes
  POST /                         - Like a book
  DELETE /                       - Unlike a book
  GET /count                     - Get like count
```

### 6. Monetization

```
/api/v1/credits
  GET /balance                   - Get user credit balance
  GET /history                   - Get credit transaction history
  
/api/v1/credits/packages
  GET /                          - List all credit packages
  POST /purchase                 - Purchase credits
  
/api/v1/subscriptions
  GET /plans                     - List all subscription plans
  GET /my-subscription           - Get user's current subscription
  POST /subscribe                - Subscribe to a plan
  PUT /cancel                    - Cancel subscription
  
/api/v1/purchases
  GET /                          - Get purchase history
  POST /book/{bookId}            - Purchase a book
  POST /chapter/{chapterId}      - Purchase a chapter
  GET /check/{contentType}/{contentId} - Check if content is purchased
  
/api/v1/earnings
  GET /                          - Get creator earnings
  GET /statistics                - Get earnings statistics
  
/api/v1/exchange
  POST /request                  - Request credit exchange
  GET /requests                  - Get exchange request history
```

### 7. Administration

```
/api/v1/admin/users
  GET /                          - List all users
  PUT /{userId}/status           - Update user status
  PUT /{userId}/role             - Update user role
  
/api/v1/admin/content
  GET /pending                   - Get pending content for approval
  PUT /books/{bookId}/approve    - Approve a book
  PUT /books/{bookId}/reject     - Reject a book
  PUT /chapters/{chapterId}/approve - Approve a chapter
  PUT /chapters/{chapterId}/reject  - Reject a chapter
  
/api/v1/admin/reports
  GET /                          - Get all reports
  PUT /{reportId}/resolve        - Resolve a report
  
/api/v1/admin/settings
  GET /                          - Get system settings
  PUT /                          - Update system settings
  PUT /credit-value              - Update credit value
  
/api/v1/admin/statistics
  GET /users                     - Get user statistics
  GET /content                   - Get content statistics
  GET /revenue                   - Get revenue statistics
  
/api/v1/admin/exchange-requests
  GET /                          - Get all exchange requests
  PUT /{requestId}/approve       - Approve exchange request
  PUT /{requestId}/reject        - Reject exchange request
  PUT /{requestId}/complete      - Mark exchange request as completed
```

## Spring Boot Project Structure

```
com.bookreader
├── config
│   ├── SecurityConfig.java
│   ├── JwtConfig.java
│   ├── AmazonS3Config.java
│   └── RedisConfig.java
├── controller
│   ├── AuthController.java
│   ├── UserController.java
│   ├── BookController.java
│   ├── ChapterController.java
│   ├── ReaderController.java
│   ├── LibraryController.java
│   ├── SocialController.java
│   ├── CreditController.java
│   ├── SubscriptionController.java
│   └── AdminController.java
├── dto
│   ├── request
│   │   ├── UserRegistrationRequest.java
│   │   ├── LoginRequest.java
│   │   ├── BookCreateRequest.java
│   │   └── ...
│   └── response
│       ├── UserResponse.java
│       ├── BookResponse.java
│       ├── ChapterResponse.java
│       └── ...
├── entity
│   ├── User.java
│   ├── UserPreference.java
│   ├── UserCredit.java
│   ├── Book.java
│   ├── Chapter.java
│   ├── Page.java
│   └── ...
├── repository
│   ├── UserRepository.java
│   ├── BookRepository.java
│   ├── ChapterRepository.java
│   └── ...
├── service
│   ├── AuthService.java
│   ├── UserService.java
│   ├── BookService.java
│   ├── ChapterService.java
│   ├── FileStorageService.java
│   ├── CreditService.java
│   └── ...
├── exception
│   ├── GlobalExceptionHandler.java
│   ├── ResourceNotFoundException.java
│   ├── UnauthorizedException.java
│   └── ...
└── util
    ├── JwtUtil.java
    ├── FileUtil.java
    └── ...
```



