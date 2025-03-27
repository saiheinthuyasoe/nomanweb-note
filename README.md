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
- **File Storage**: Cloudflare R2 (or similar)
- **Cache**: Redis Cloud (or similar)
- **Search**: OpenSearch (self-hosted)

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
- **Hosting**: Render (Backend), Supabase(Database) and Vercel (Frontend)
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
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    profile_image_url VARCHAR(255),
    bio TEXT,
    role VARCHAR(20) NOT NULL DEFAULT 'USER', -- 'USER' or 'ADMIN'
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

-- User Preferences Table
CREATE TABLE user_preferences (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    reading_direction VARCHAR(20) DEFAULT 'LTR', -- 'LTR', 'RTL', 'VERTICAL'
    theme VARCHAR(20) DEFAULT 'LIGHT', -- 'LIGHT', 'DARK', 'SEPIA'
    font_size INTEGER DEFAULT 100, -- percentage
    enable_notifications BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id)
);

-- User Credits Table
CREATE TABLE user_credits (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    balance DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    lifetime_earned DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    lifetime_spent DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id)
);
```

### 2. Content Schema

```sql
-- Categories Table
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    parent_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Tags Table
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Books Table
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    cover_image_url VARCHAR(255),
    author VARCHAR(100),
    publisher VARCHAR(100),
    publication_year INTEGER,
    language VARCHAR(50),
    book_type VARCHAR(100) NOT NULL, -- Manga, Manhwa, Webtoon, etc.
    reading_direction VARCHAR(20) DEFAULT 'LTR', -- 'LTR', 'RTL', 'VERTICAL'
    credit_cost DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    uploader_id INTEGER NOT NULL REFERENCES users(id),
    is_approved BOOLEAN DEFAULT FALSE,
    total_views INTEGER DEFAULT 0,
    total_likes INTEGER DEFAULT 0,
    average_rating DECIMAL(3, 2) DEFAULT 0.00,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Book Categories Relationship
CREATE TABLE book_categories (
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    category_id INTEGER NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, category_id)
);

-- Book Tags Relationship
CREATE TABLE book_tags (
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    tag_id INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (book_id, tag_id)
);

-- Chapters Table
CREATE TABLE chapters (
    id SERIAL PRIMARY KEY,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    chapter_number INTEGER NOT NULL,
    credit_cost DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    is_free BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(book_id, chapter_number)
);

-- Pages Table
CREATE TABLE pages (
    id SERIAL PRIMARY KEY,
    chapter_id INTEGER NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    page_number INTEGER NOT NULL,
    image_url VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(chapter_id, page_number)
);
```

### 3. Interaction Schema

```sql
-- Reading Progress Table
CREATE TABLE reading_progress (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    chapter_id INTEGER NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    page_number INTEGER NOT NULL,
    last_read_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, book_id)
);

-- User Library Tables
CREATE TABLE user_wishlists (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    added_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, book_id)
);

CREATE TABLE user_readlists (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    added_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, book_id)
);

CREATE TABLE user_currently_reading (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    added_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, book_id)
);

CREATE TABLE user_bookmarks (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    added_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, book_id)
);

-- Ratings and Reviews Table
CREATE TABLE ratings_reviews (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    review TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, book_id)
);

-- Book Likes Table
CREATE TABLE book_likes (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, book_id)
);

-- Comments Table
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    chapter_id INTEGER REFERENCES chapters(id) ON DELETE CASCADE,
    parent_comment_id INTEGER REFERENCES comments(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 4. Monetization Schema

```sql
-- Credit Packages Table
CREATE TABLE credit_packages (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    credit_amount INTEGER NOT NULL,
    price_usd DECIMAL(10, 2) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Credit Value Configuration
CREATE TABLE credit_value_config (
    id SERIAL PRIMARY KEY,
    credit_to_usd_ratio DECIMAL(10, 6) NOT NULL, -- How much 1 credit is worth in USD
    admin_fee_percentage DECIMAL(5, 2) NOT NULL DEFAULT 10.00, -- Default 10%
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Transactions Table
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    transaction_type VARCHAR(50) NOT NULL, -- 'PURCHASE_CREDITS', 'CONTENT_PURCHASE', 'CREDIT_WITHDRAWAL'
    amount DECIMAL(10, 2) NOT NULL,
    credits_amount INTEGER,
    status VARCHAR(20) NOT NULL, -- 'PENDING', 'COMPLETED', 'FAILED', 'REFUNDED'
    payment_method VARCHAR(50),
    payment_reference VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Content Purchases Table
CREATE TABLE content_purchases (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(id) ON DELETE SET NULL,
    chapter_id INTEGER REFERENCES chapters(id) ON DELETE SET NULL,
    credits_spent DECIMAL(10, 2) NOT NULL,
    transaction_id INTEGER REFERENCES transactions(id),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Creator Earnings Table
CREATE TABLE creator_earnings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(id) ON DELETE SET NULL,
    chapter_id INTEGER REFERENCES chapters(id) ON DELETE SET NULL,
    purchase_id INTEGER REFERENCES content_purchases(id) ON DELETE CASCADE,
    credits_earned DECIMAL(10, 2) NOT NULL,
    is_paid BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Withdrawal Requests Table
CREATE TABLE withdrawal_requests (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    credits_amount DECIMAL(10, 2) NOT NULL,
    usd_amount DECIMAL(10, 2) NOT NULL,
    admin_fee_amount DECIMAL(10, 2) NOT NULL,
    payment_method VARCHAR(50) NOT NULL,
    payment_details JSONB NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING', -- 'PENDING', 'APPROVED', 'REJECTED', 'COMPLETED'
    transaction_id INTEGER REFERENCES transactions(id),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## API Architecture

### 1. Authentication and User Management

```
/api/auth
  POST /register - Register a new user
  POST /login - Authenticate user and get JWT token
  POST /refresh-token - Refresh JWT token
  POST /logout - Logout user
  GET /me - Get current user information
  PUT /me - Update current user information
  PUT /me/password - Update password
  POST /forgot-password - Request password reset
  POST /reset-password - Reset password with token

/api/users
  GET / - Get all users (admin only)
  GET /{id} - Get user by ID
  PUT /{id} - Update user (admin only)
  DELETE /{id} - Delete user (admin only)
  GET /{id}/profile - Get user public profile
  GET /{id}/uploads - Get books uploaded by user
  GET /me/preferences - Get user preferences
  PUT /me/preferences - Update user preferences
  GET /me/credits - Get user credit balance
```

### 2. Content Management

```
/api/books
  GET / - Get all books (with filtering, pagination, sorting)
  POST / - Create a new book
  GET /{id} - Get book details
  PUT /{id} - Update book
  DELETE /{id} - Delete book
  GET /{id}/chapters - Get all chapters for a book
  POST /{id}/chapters - Add a new chapter to a book
  GET /types - Get all book types
  GET /popular - Get popular books
  GET /recent - Get recently added books
  GET /search - Search books by title, author, tags, etc.

/api/chapters
  GET /{id} - Get chapter details
  PUT /{id} - Update chapter
  DELETE /{id} - Delete chapter
  GET /{id}/pages - Get all pages for a chapter
  POST /{id}/pages - Add pages to a chapter
  PUT /{id}/pages/reorder - Reorder pages in a chapter

/api/pages
  GET /{id} - Get page details
  PUT /{id} - Update page
  DELETE /{id} - Delete page

/api/categories
  GET / - Get all categories
  POST / - Create a new category (admin only)
  PUT /{id} - Update category (admin only)
  DELETE /{id} - Delete category (admin only)
  GET /{id}/books - Get books in a category

/api/tags
  GET / - Get all tags
  POST / - Create a new tag
  DELETE /{id} - Delete tag (admin only)
  GET /{id}/books - Get books with a specific tag
```

### 3. User Interaction

```
/api/library
  GET /wishlist - Get user's wishlist
  POST /wishlist/{bookId} - Add book to wishlist
  DELETE /wishlist/{bookId} - Remove book from wishlist
  
  GET /readlist - Get user's readlist
  POST /readlist/{bookId} - Add book to readlist
  DELETE /readlist/{bookId} - Remove book from readlist
  
  GET /currently-reading - Get user's currently reading list
  POST /currently-reading/{bookId} - Add book to currently reading
  DELETE /currently-reading/{bookId} - Remove book from currently reading
  
  GET /bookmarks - Get user's bookmarks
  POST /bookmarks/{bookId} - Add book to bookmarks
  DELETE /bookmarks/{bookId} - Remove book from bookmarks

/api/reading
  GET /progress/{bookId} - Get reading progress for a book
  POST /progress - Update reading progress
  DELETE /progress/{bookId} - Reset reading progress for a book

/api/ratings
  GET /books/{bookId} - Get all ratings for a book
  POST /books/{bookId} - Rate a book
  PUT /books/{bookId} - Update book rating
  DELETE /books/{bookId} - Remove book rating

/api/comments
  GET /books/{bookId} - Get all comments for a book
  POST /books/{bookId} - Add a comment to a book
  PUT /{id} - Update a comment
  DELETE /{id} - Delete a comment
  POST /{id}/replies - Reply to a comment
```

### 4. Monetization

```
/api/credits
  GET /packages - Get available credit packages
  POST /purchase - Purchase credits
  GET /value - Get current credit value configuration
  PUT /value - Update credit value (admin only)

/api/purchases
  GET / - Get user's purchase history
  POST /books/{bookId} - Purchase a book
  POST /chapters/{chapterId} - Purchase a chapter
  GET /books/{bookId}/status - Check if user has purchased a book
  GET /chapters/{chapterId}/status - Check if user has purchased a chapter

/api/earnings
  GET / - Get creator's earnings
  GET /books/{bookId} - Get earnings for a specific book
  GET /summary - Get earnings summary

/api/withdrawals
  GET / - Get withdrawal history
  POST / - Create withdrawal request
  GET /{id} - Get withdrawal request details
  PUT /{id}/cancel - Cancel withdrawal request (if pending)
```

### 5. Admin APIs

```
/api/admin
  GET /dashboard - Get admin dashboard statistics
  
  GET /books/pending - Get books pending approval
  PUT /books/{id}/approve - Approve a book
  PUT /books/{id}/reject - Reject a book
  
  GET /withdrawals/pending - Get pending withdrawal requests
  PUT /withdrawals/{id}/approve - Approve withdrawal request
  PUT /withdrawals/{id}/reject - Reject withdrawal request
  
  GET /users/statistics - Get user statistics
  GET /content/statistics - Get content statistics
  GET /financial/statistics - Get financial statistics
  
  GET /logs - Get system logs
  GET /configuration - Get system configuration
  PUT /configuration - Update system configuration
```

## API Implementation Considerations

1. **Security**:
   - Implement JWT-based authentication
   - Role-based access control (RBAC)
   - Input validation and sanitization
   - CSRF protection
   - Rate limiting

2. **Performance**:
   - Implement caching for frequently accessed data
   - Pagination for large result sets
   - Optimize database queries
   - Use content delivery networks (CDNs) for images

3. **Error Handling**:
   - Consistent error response format
   - Appropriate HTTP status codes
   - Detailed error messages (for development)
   - Logging for debugging

4. **Documentation**:
   - Use Swagger/OpenAPI for API documentation
   - Include request/response examples
   - Document authentication requirements

This database schema and API architecture provide a solid foundation for your Book Reader web application. The design supports all the requirements mentioned in your project description, including user management, content management, reading experience, monetization, and social features.



