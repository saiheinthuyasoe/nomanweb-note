# Book Reader Web Application - Comprehensive Project Guide

This guide provides a complete roadmap for developing a Book Reader Web Application for comics, manga, and graphic novels from scratch to deployment. It covers every aspect of the project, including planning, architecture, implementation, testing, and deployment.

## 1. Project Overview

### Purpose
To create a web platform where users can read, upload, and monetize various types of comics and graphic novels from around the world, including Manga, Manhwa, Webtoons, American Comics, and other regional formats.

### Business Model
- Every user can upload content
- Users need to purchase credits or subscriptions to read content
- Content uploaders earn a share of the credits spent on their content
- Admin management and moderation capabilities

### Target Audience
- Comic/manga readers worldwide
- Content creators looking to monetize their work
- Publishers seeking digital distribution channels

## 2. Requirements Gathering

### Functional Requirements

#### User Management
- Registration and authentication
- User profiles and preferences
- Reading history and bookmarks

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
- Subscription plans (weekly, monthly, yearly)
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

### Database Schema Implementation

```sql
-- Users and Authentication
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    is_admin BOOLEAN NOT NULL DEFAULT FALSE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    avatar_url VARCHAR(255)
);

-- User Preferences
CREATE TABLE user_preferences (
    user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    reading_direction VARCHAR(20) NOT NULL DEFAULT 'LTR',
    theme VARCHAR(20) NOT NULL DEFAULT 'light',
    font_size INTEGER NOT NULL DEFAULT 12,
    auto_bookmark BOOLEAN NOT NULL DEFAULT TRUE,
    notifications_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Credit System
CREATE TABLE credit_packages (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    credits INTEGER NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE user_credits (
    user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    balance INTEGER NOT NULL DEFAULT 0,
    total_purchased INTEGER NOT NULL DEFAULT 0,
    total_spent INTEGER NOT NULL DEFAULT 0,
    total_earned INTEGER NOT NULL DEFAULT 0,
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Subscription System
CREATE TABLE subscription_plans (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    duration VARCHAR(20) NOT NULL,  -- 'weekly', 'monthly', 'yearly'
    price DECIMAL(10, 2) NOT NULL,
    description TEXT,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE user_subscriptions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    plan_id INTEGER NOT NULL REFERENCES subscription_plans(id),
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP NOT NULL,
    auto_renew BOOLEAN NOT NULL DEFAULT TRUE,
    status VARCHAR(20) NOT NULL, -- 'active', 'expired', 'cancelled'
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Content Catalog
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_id INTEGER REFERENCES categories(id),
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    cover_url VARCHAR(255),
    author VARCHAR(255),
    artist VARCHAR(255),
    publisher VARCHAR(255),
    publication_year INTEGER,
    language VARCHAR(50),
    status VARCHAR(50) NOT NULL, -- 'ongoing', 'completed', 'hiatus'
    format_type VARCHAR(50) NOT NULL, -- 'manga', 'manhwa', 'webtoon', 'comic', etc.
    reading_direction VARCHAR(20) NOT NULL DEFAULT 'LTR', -- 'LTR', 'RTL', 'vertical'
    is_mature BOOLEAN NOT NULL DEFAULT FALSE,
    uploader_id INTEGER NOT NULL REFERENCES users(id),
    credit_cost INTEGER NOT NULL DEFAULT 1,
    view_count INTEGER NOT NULL DEFAULT 0,
    rating DECIMAL(3, 2),
    review_count INTEGER NOT NULL DEFAULT 0,
    is_featured BOOLEAN NOT NULL DEFAULT FALSE,
    is_approved BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE chapters (
    id SERIAL PRIMARY KEY,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    chapter_number DECIMAL(8, 2) NOT NULL,
    pages_count INTEGER NOT NULL DEFAULT 0,
    credit_cost INTEGER,  -- If NULL, uses book's cost
    is_free BOOLEAN NOT NULL DEFAULT FALSE,
    view_count INTEGER NOT NULL DEFAULT 0,
    is_published BOOLEAN NOT NULL DEFAULT FALSE,
    publish_date TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE (book_id, chapter_number)
);

CREATE TABLE pages (
    id SERIAL PRIMARY KEY,
    chapter_id INTEGER NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    page_number INTEGER NOT NULL,
    image_url VARCHAR(255) NOT NULL,
    width INTEGER,
    height INTEGER,
    file_size INTEGER,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE (chapter_id, page_number)
);

-- User Interactions
CREATE TABLE reading_progress (
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    chapter_id INTEGER NOT NULL REFERENCES chapters(id) ON DELETE CASCADE,
    page_number INTEGER NOT NULL,
    last_read_at TIMESTAMP NOT NULL DEFAULT NOW(),
    PRIMARY KEY (user_id, book_id)
);

CREATE TABLE content_purchases (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    chapter_id INTEGER REFERENCES chapters(id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(id) ON DELETE CASCADE, -- For whole book purchases
    credits_spent INTEGER NOT NULL,
    purchased_at TIMESTAMP NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMP, -- NULL means permanent access
    CHECK (chapter_id IS NOT NULL OR book_id IS NOT NULL)
);

-- Creator Earnings
CREATE TABLE creator_earnings (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    book_id INTEGER REFERENCES books(id) ON DELETE CASCADE,
    chapter_id INTEGER REFERENCES chapters(id) ON DELETE CASCADE,
    credits_earned INTEGER NOT NULL,
    earned_from_user_id INTEGER REFERENCES users(id),
    earned_at TIMESTAMP NOT NULL DEFAULT NOW(),
    CHECK (book_id IS NOT NULL OR chapter_id IS NOT NULL)
);
```

## 5. API Architecture

### Authentication API
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Authenticate user
- `POST /api/auth/refresh` - Refresh authentication token
- `POST /api/auth/logout` - Log out user

### User API
- `GET /api/users/profile` - Get user profile
- `PUT /api/users/profile` - Update user profile
- `GET /api/users/preferences` - Get reading preferences
- `PUT /api/users/preferences` - Update reading preferences
- `GET /api/users/library` - Get saved books
- `GET /api/users/history` - Get reading history
- `GET /api/users/uploads` - Get uploaded content

### Content API
- `GET /api/books` - Browse books with filters
- `GET /api/books/{id}` - Get book details
- `GET /api/books/{id}/chapters` - Get book chapters
- `GET /api/chapters/{id}` - Get chapter details
- `GET /api/chapters/{id}/pages` - Get chapter pages
- `GET /api/categories` - Get categories list

### Upload API
- `POST /api/uploads/books` - Create new book
- `PUT /api/uploads/books/{id}` - Update book details
- `POST /api/uploads/books/{id}/cover` - Upload cover image
- `POST /api/uploads/books/{id}/chapters` - Create new chapter
- `POST /api/uploads/chapters/{id}/pages` - Upload pages

### Monetization API
- `GET /api/credits/packages` - Get credit packages
- `POST /api/credits/purchase` - Purchase credits
- `GET /api/credits/balance` - Get user credit balance
- `GET /api/subscriptions/plans` - Get subscription plans
- `POST /api/subscriptions/subscribe` - Subscribe to a plan
- `GET /api/earnings` - Get creator earnings
- `POST /api/purchases/chapter/{id}` - Purchase chapter access

### Admin API
- `GET /api/admin/users` - Manage users
- `GET /api/admin/books/pending` - Review content
- `PUT /api/admin/books/{id}/approve` - Approve content
- `GET /api/admin/stats` - View platform statistics

