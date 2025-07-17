# Voting App Requirements

## Project Overview
Build a real-time collaborative voting/polling application showcasing Aiven Postgres + Redis with Vercel deployment. Target completion: 4 hours.

## Architecture Stack
- **Frontend**: Next.js 14 (App Router) + TypeScript + Tailwind CSS
- **Backend**: Next.js API routes + tRPC (optional) or REST
- **Database**: Aiven PostgreSQL
- **Cache/Real-time**: Aiven Redis
- **Deployment**: Vercel
- **Development**: Claude Code for rapid development

## Core Features

### 1. Poll Management
- Create new polls with title, description, and multiple options
- Set poll expiration dates (optional)
- Poll categories/tags
- View poll details and results

### 2. Voting System
- One vote per user per poll (IP-based or session-based)
- Real-time vote counting
- Vote validation and duplicate prevention
- Anonymous voting (no user accounts required)

### 3. Real-Time Updates
- Live vote count updates across all connected clients
- Real-time result percentages
- Live participant count
- Animated progress bars

### 4. Analytics & Insights
- Vote distribution charts
- Voting timeline (votes over time)
- Geographic distribution (based on IP)
- Most popular polls

## Technical Requirements

### Database Schema (PostgreSQL)

```sql
-- Polls table
CREATE TABLE polls (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    creator_ip INET,
    is_active BOOLEAN DEFAULT TRUE,
    category VARCHAR(100),
    total_votes INTEGER DEFAULT 0
);

-- Poll options
CREATE TABLE poll_options (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    poll_id UUID REFERENCES polls(id) ON DELETE CASCADE,
    option_text VARCHAR(255) NOT NULL,
    vote_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Vote tracking (prevent duplicates)
CREATE TABLE votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    poll_id UUID REFERENCES polls(id) ON DELETE CASCADE,
    option_id UUID REFERENCES poll_options(id) ON DELETE CASCADE,
    voter_ip INET,
    voter_session VARCHAR(255),
    voted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(poll_id, voter_ip),
    UNIQUE(poll_id, voter_session)
);

-- Indexes for performance
CREATE INDEX idx_polls_active ON polls(is_active, created_at);
CREATE INDEX idx_votes_poll_ip ON votes(poll_id, voter_ip);
CREATE INDEX idx_poll_options_poll_id ON poll_options(poll_id);
```

### Redis Usage Patterns

```javascript
// Vote counting (Redis Counters)
INCR poll:{poll_id}:votes
INCR poll:{poll_id}:option:{option_id}:votes

// Real-time pub/sub
PUBLISH poll:{poll_id}:updates "{vote_count: 45, option_id: 'abc'}"

// Duplicate prevention (Redis Sets)
SADD poll:{poll_id}:voters {voter_ip}
SISMEMBER poll:{poll_id}:voters {voter_ip}

// Rate limiting (Redis with TTL)
SETEX ratelimit:{voter_ip} 60 1

// Session management
SETEX session:{session_id} 3600 {poll_id}

// Popular polls cache
ZADD trending_polls {vote_count} {poll_id}
```

### API Endpoints

```typescript
// REST API structure
POST /api/polls              // Create new poll
GET /api/polls               // List polls (paginated)
GET /api/polls/[id]          // Get poll details
POST /api/polls/[id]/vote    // Submit vote
GET /api/polls/[id]/results  // Get current results
DELETE /api/polls/[id]       // Delete poll (creator only)

// WebSocket/SSE for real-time updates
GET /api/polls/[id]/stream   // Real-time updates stream
```

## Environment Setup

### Required Environment Variables

```env
# Aiven PostgreSQL
POSTGRES_URL=postgres://user:password@host:port/dbname?sslmode=require

# Aiven Redis
REDIS_URL=redis://user:password@host:port
REDIS_TLS_URL=rediss://user:password@host:port

# Application
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:3000
NODE_ENV=development

# Optional: IP Geolocation
IPGEOLOCATION_API_KEY=your-api-key
```

### Package Dependencies

```json
{
  "dependencies": {
    "next": "14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "tailwindcss": "^3.3.0",
    "pg": "^8.11.0",
    "redis": "^4.6.0",
    "uuid": "^9.0.0",
    "zod": "^3.22.0",
    "recharts": "^2.8.0",
    "socket.io": "^4.7.0",
    "socket.io-client": "^4.7.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@types/react": "^18.2.0",
    "@types/pg": "^8.10.0",
    "@types/uuid": "^9.0.0",
    "autoprefixer": "^10.4.0",
    "postcss": "^8.4.0"
  }
}
```

## UI/UX Requirements

### Key Pages
1. **Home Page** - Browse active polls, create new poll
2. **Poll Detail** - Vote interface with real-time results
3. **Create Poll** - Form to create new polls
4. **Results Page** - Detailed analytics and charts

### Design System
- **Colors**: Modern gradient backgrounds, Aiven brand colors
- **Typography**: Clean, readable fonts (Inter or similar)
- **Components**: Animated progress bars, real-time counters, hover effects
- **Mobile-first**: Responsive design for all screen sizes

### Interactive Elements
- Real-time vote count animations
- Progress bar fills with smooth transitions
- Live participant counters
- Confetti/celebration effects for poll completion
- Share buttons for social media

## Development Workflow

### Phase 1: Setup (30 minutes)
1. Create Next.js project with TypeScript
2. Set up Aiven PostgreSQL and Redis services
3. Configure environment variables
4. Set up database schema

### Phase 2: Core Backend (90 minutes)
1. Database connection setup
2. Redis client configuration
3. API routes for poll CRUD operations
4. Vote submission and validation logic
5. Real-time pub/sub implementation

### Phase 3: Frontend (120 minutes)
1. Create poll form and listing
2. Vote interface with real-time updates
3. Results visualization with charts
4. Responsive design implementation
5. WebSocket/SSE integration

### Phase 4: Polish & Deploy (30 minutes)
1. Error handling and validation
2. Loading states and animations
3. Vercel deployment configuration
4. Environment variable setup
5. Final testing

## Testing Strategy

### Manual Testing Checklist
- [ ] Create poll with multiple options
- [ ] Vote from multiple browser sessions
- [ ] Verify duplicate vote prevention
- [ ] Test real-time updates across browsers
- [ ] Check mobile responsiveness
- [ ] Verify poll expiration functionality
- [ ] Test error handling (network issues, invalid data)

### Performance Targets
- **Page load**: < 2 seconds
- **Vote submission**: < 500ms
- **Real-time updates**: < 100ms latency
- **Concurrent users**: Support 100+ simultaneous voters

## Deployment Configuration

### Vercel Settings
```json
{
  "build": {
    "env": {
      "POSTGRES_URL": "@postgres-url",
      "REDIS_URL": "@redis-url"
    }
  },
  "functions": {
    "app/api/**": {
      "maxDuration": 30
    }
  }
}
```

### Aiven Service Configuration
- **PostgreSQL**: Basic plan, connection pooling enabled
- **Redis**: Basic plan, persistence enabled
- **Networking**: VPC peering (optional for demo)
- **Monitoring**: Enable service logs and metrics

## Success Metrics

### Technical Metrics
- Zero downtime during demo
- Sub-second response times
- No vote counting errors
- Smooth real-time updates

### Marketing Metrics
- Visually impressive for social media
- Clear value proposition demonstration
- Shareable architecture diagram
- Reusable code patterns

## Content Marketing Deliverables

### Blog Post Topics
1. "Building Real-Time Apps with Aiven Redis + PostgreSQL"
2. "Why Redis Beats Database Pub/Sub for Real-Time Features"
3. "Deploy Full-Stack Apps to Vercel with Aiven Services"

### Social Media Content
- Architecture diagram
- Real-time voting demo video
- Performance comparison screenshots
- Code snippets highlighting Redis patterns

### Technical Documentation
- Complete setup guide
- Redis patterns explanation
- PostgreSQL schema design
- Vercel deployment tutorial

## Additional Features (If Time Permits)

### Advanced Analytics
- Vote patterns over time
- Geographic heat maps
- Device/browser analytics
- Popular poll categories

### Enhanced UX
- Poll templates
- Custom poll URLs
- Embed codes for external sites
- Dark/light mode toggle

### Performance Optimizations
- Connection pooling
- Redis cluster setup
- CDN integration
- Database query optimization

## Troubleshooting Guide

### Common Issues
1. **Redis connection timeout**: Check TLS configuration
2. **Database connection pool exhaustion**: Implement connection limits
3. **Real-time updates not working**: Verify WebSocket configuration
4. **Vote counting inconsistencies**: Check Redis atomic operations
5. **Deployment issues**: Verify environment variables in Vercel

### Monitoring Setup
- Aiven service health dashboards
- Vercel function logs
- Redis performance metrics
- PostgreSQL query performance

---

**Estimated Total Time: 4 hours**
**Difficulty Level: Intermediate**
**Primary Value: Showcases enterprise-grade real-time architecture patterns**