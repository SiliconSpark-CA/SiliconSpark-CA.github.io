

# Camping Spot Reservation Application - Architecture Document  
**For AI Programming Agents**  
*Backend: Hono.dev | Frontend: Vue.js | Database: PostgreSQL*

---

## 1. System Overview
This architecture outlines a full-stack web application for camping spot reservations with:
- **API-first backend** using Hono.dev for lightweight, performant services
- **Component-based frontend** using Vue.js for responsive UI
- **Relational data storage** using PostgreSQL for structured data integrity
- **Cloud deployment** strategy for scalability and maintainability

---

## 2. Architecture Diagram
```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Vue.js     │◄───►│  Hono.dev    │◄───►│  PostgreSQL  │
│  Frontend    │     │   Backend    │     │   Database   │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                     │
       │                    │                     │
       ▼                    ▼                     ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Browser    │     │ Cloudflare   │     │   Database   │
│  (Mobile/    │     │   Workers/   │     │   Mgmt       │
│   Desktop)   │     │   Node.js    │     │   (Optional) │
└──────────────┘     └──────────────┘     └──────────────┘
```

---

## 3. Backend Architecture (Hono.dev)

### 3.1 Structure
```
backend/
├── src/
│   ├── controllers/       # Request handlers
│   │   ├── auth.ts
│   │   ├── camping.ts
│   │   └── booking.ts
│   ├── middleware/        # Custom middleware
│   │   ├── auth.ts
│   │   └── validation.ts
│   ├── routes/            # Route definitions
│   │   ├── auth.ts
│   │   ├── camping.ts
│   │   └── booking.ts
│   ├── services/          # Business logic
│   │   ├── auth.ts
│   │   ├── camping.ts
│   │   └── payment.ts
│   ├── utils/             # Helper functions
│   │   ├── db.ts          # Database connection
│   │   └── errors.ts      # Error handling
│   └── app.ts             # Hono app initialization
├── tests/                 # Test files
└── package.json
```

### 3.2 Key Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/camping-spots` | List all camping spots with filters |
| GET | `/api/camping-spots/:id` | Get details for a specific spot |
| GET | `/api/camping-spots/availability` | Check availability for dates |
| POST | `/api/bookings` | Create a new reservation |
| GET | `/api/bookings/:id` | Get booking details |
| PUT | `/api/bookings/:id` | Modify existing booking |
| DELETE | `/api/bookings/:id` | Cancel a booking |
| POST | `/api/auth/register` | User registration |
| POST | `/api/auth/login` | User authentication |
| GET | `/api/users/me` | Get current user profile |

### 3.3 Implementation Guidance
- Use **Hono's built-in middleware** for CORS, logging, and error handling
- Implement **JWT authentication** with `@hono/jwt` middleware
- Use **Zod** for request validation in middleware
- Create **modular route handlers** in separate files
- Implement **database connection pooling** with `pg` package
- Use **environment variables** for configuration

---

## 4. Frontend Architecture (Vue.js)

### 4.1 Structure
```
frontend/
├── public/               # Static assets
├── src/
│   ├── assets/           # Images, styles
│   ├── components/       # Reusable components
│   │   ├── ui/           # Base UI components
│   │   ├── camping/      # Camping-related components
│   │   └── booking/      # Booking-related components
│   ├── composables/      # Reusable logic
│   │   ├── useAuth.ts    # Authentication logic
│   │   └── useApi.ts     # API calls
│   ├── router/           # Vue Router configuration
│   ├── stores/           # Pinia state management
│   │   ├── auth.ts       # Auth state
│   │   └── camping.ts    # Camping data state
│   ├── views/            # Page components
│   │   ├── Home.vue
│   │   ├── CampingSpots.vue
│   │   ├── Booking.vue
│   │   └── Profile.vue
│   ├── App.vue           # Root component
│   └── main.ts           # App entry point
├── tests/                # Test files
└── package.json
```

### 4.2 Key Components
| Component | Purpose |
|-----------|---------|
| `CampingSpotCard` | Display individual camping spot info |
| `AvailabilityCalendar` | Show available dates visually |
| `BookingForm` | Handle reservation creation |
| `SearchFilters` | Filter camping spots by criteria |
| `UserDashboard` | Show user's bookings |
| `ReviewForm` | Submit reviews after stay |

### 4.3 Implementation Guidance
- Use **Vue 3 Composition API** with `<script setup>` syntax
- Implement **Pinia** for state management
- Use **Vue Router** for navigation and route guards
- Create **composables** for reusable logic (API calls, auth)
- Use **Vite** for fast development and building
- Implement **responsive design** with Tailwind CSS
- Use **VueUse** for common utilities (debounce, localStorage)

---

## 5. Database Schema (PostgreSQL)

### 5.1 Core Tables
```sql
-- Users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Camping spots table
CREATE TABLE camping_spots (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  location VARCHAR(255) NOT NULL,
  price_per_night DECIMAL(10,2) NOT NULL,
  max_guests INTEGER NOT NULL,
  amenities TEXT[], -- Array of amenities
  images TEXT[], -- Array of image URLs
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Availability table
CREATE TABLE availability (
  id SERIAL PRIMARY KEY,
  camping_spot_id INTEGER REFERENCES camping_spots(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  available BOOLEAN DEFAULT true,
  UNIQUE(camping_spot_id, date)
);

-- Bookings table
CREATE TABLE bookings (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  camping_spot_id INTEGER REFERENCES camping_spots(id) ON DELETE CASCADE,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  total_price DECIMAL(10,2) NOT NULL,
  status VARCHAR(50) DEFAULT 'confirmed', -- confirmed, cancelled, completed
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Reviews table
CREATE TABLE reviews (
  id SERIAL PRIMARY KEY,
  booking_id INTEGER REFERENCES bookings(id) ON DELETE CASCADE,
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  comment TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 5.2 Implementation Guidance
- Use **UUIDs** instead of SERIAL if distributed systems needed
- Create **indexes** on frequently queried columns (email, camping_spot_id)
- Implement **foreign key constraints** for data integrity
- Use **JSONB** instead of TEXT[] for more complex amenities if needed
- Add **triggers** for maintaining data consistency (e.g., update availability on booking)
- Create **materialized views** for complex reporting needs

---

## 6. Authentication & Security

### 6.1 Authentication Flow
1. User registers with email/password
2. Backend hashes password with bcrypt
3. On login, backend verifies credentials and issues JWT
4. Frontend stores JWT in localStorage/httpOnly cookie
5. Subsequent requests include JWT in Authorization header

### 6.2 Security Measures
- **Password hashing** with bcrypt (salt rounds: 12)
- **JWT expiration** (15 minutes access, 7 days refresh)
- **HTTPS** for all communications
- **CORS** configured for specific domains
- **Rate limiting** on authentication endpoints
- **Input validation** on all API endpoints
- **SQL injection prevention** with parameterized queries
- **Role-based access control** for admin functions

---

## 7. External Integrations

### 7.1 Payment Processing
- **Stripe** integration for payment processing
- Implement webhook handlers for payment events
- Store payment intent IDs, not full payment details

### 7.2 Email Notifications
- **Resend** or **SendGrid** for transactional emails
- Templates for:
  - Booking confirmation
  - Cancellation notice
  - Pre-arrival reminder
  - Review request

### 7.3 Additional Services
- **Mapbox** or **Google Maps** for location visualization
- **Weather API** for weather forecasts
- **Cloudinary** for image storage and optimization

---

## 8. Deployment Strategy

### 8.1 Backend Deployment
- **Platform**: Cloudflare Workers or Deno Deploy for Hono
- **Environment Variables**: Securely store database credentials, API keys
- **CI/CD**: Automated deployment from GitHub/GitLab
- **Monitoring**: Cloudflare analytics or custom monitoring

### 8.2 Frontend Deployment
- **Platform**: Vercel or Netlify for Vue.js
- **Build Process**: Vite build optimized for production
- **CDN**: Asset delivery via edge network
- **Environment**: API endpoint configuration

### 8.3 Database Deployment
- **Service**: ElephantSQL, Supabase, or AWS RDS
- **Backups**: Automated daily backups
- **Migrations**: Version-controlled schema changes

---

## 9. Development Workflow

### 9.1 Local Development
1. **Database**: Local PostgreSQL with Docker
2. **Backend**: `npm run dev` for Hono hot reload
3. **Frontend**: `npm run dev` for Vite dev server
4. **API Proxy**: Configure Vite to proxy API requests

### 9.2 Testing Strategy
- **Backend**: Jest/ Vitest for unit tests, Supertest for integration
- **Frontend**: Vitest for unit tests, Cypress for E2E
- **Database**: Testcontainers for isolated test DB

### 9.3 Code Quality
- **Linting**: ESLint with shared config
- **Formatting**: Prettier with consistent rules
- **TypeScript**: Strict mode for both frontend and backend
- **Pre-commit hooks**: Husky for linting and testing

---

## 10. Scalability Considerations

### 10.1 Database Optimization
- **Connection pooling**: PgBouncer or built-in pooling
- **Read replicas**: For reporting and analytics
- **Caching**: Redis for frequently accessed data
- **Query optimization**: Regular analysis and indexing

### 10.2 Backend Scaling
- **Serverless**: Auto-scaling with Cloudflare Workers
- **Microservices**: Separate services for payments, notifications
- **Queue system**: For background jobs (email sending)

### 10.3 Frontend Optimization
- **Code splitting**: Route-based and component-based
- **Lazy loading**: Images and non-critical components
- **Caching**: Service worker for offline functionality
- **CDN**: Global asset distribution

---

**AI Agent Implementation Notes:**
1. Start with core features (auth, camping spots, bookings) before adding optional features
2. Implement backend first with API testing, then build frontend
3. Use TypeScript throughout for type safety
4. Prioritize security from day one
5. Build modular components for easy extension
6. Implement comprehensive error handling and logging
7. Follow RESTful conventions for API design
8. Create a responsive mobile-first UI
9. Add comprehensive testing at all levels
10. Document all APIs with OpenAPI/Swagger specification

