# Voting App Architecture Diagrams

## High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Browser  │    │   User Browser  │    │   User Browser  │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │ React App   │ │    │ │ React App   │ │    │ │ React App   │ │
│ │ WebSocket   │ │    │ │ WebSocket   │ │    │ │ WebSocket   │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │     Vercel Platform      │
                    │                          │
                    │  ┌─────────────────────┐ │
                    │  │   Next.js API       │ │
                    │  │   - /api/polls      │ │
                    │  │   - /api/vote       │ │
                    │  │   - WebSocket       │ │
                    │  └─────────────────────┘ │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │    Aiven Services        │
                    │                          │
                    │  ┌─────────┐ ┌─────────┐ │
                    │  │ PostgreSQL│ │  Redis  │ │
                    │  │   (Data)  │ │(Real-Time)│ │
                    │  │ ┌───────┐ │ │ ┌───────┐ │ │
                    │  │ │ Polls │ │ │ │ Cache │ │ │
                    │  │ │ Votes │ │ │ │ Pub/Sub│ │ │
                    │  │ └───────┘ │ │ └───────┘ │ │
                    │  └─────────┘ └─────────┘ │
                    └─────────────────────────────┘
```

## Data Flow: Vote Submission

```
User Vote Submission Flow:
═══════════════════════════

1. User clicks vote
   │
   ▼
┌─────────────────────────┐
│  Frontend validates     │
│  and sends POST request │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐
│   API Route Handler     │
│   /api/polls/[id]/vote  │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐
│    Vote Validator       │
│  - Check IP duplicate   │
│  - Validate poll active │
└─────────────┬───────────┘
              │
              ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│        Redis            │     │      PostgreSQL         │
│  Check: SISMEMBER       │────▶│  Insert vote record     │
│  poll:123:voters IP     │     │  Update option count    │
└─────────────┬───────────┘     └─────────────┬───────────┘
              │                               │
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│        Redis            │     │    Success Response     │
│  SADD poll:123:voters   │────▶│  Return updated counts  │
│  INCR poll:123:votes    │     │  Trigger real-time      │
│  PUBLISH updates        │     │  broadcast              │
└─────────────────────────┘     └─────────────────────────┘
              │
              ▼
┌─────────────────────────┐
│   All Connected Clients │
│   Receive live updates  │
└─────────────────────────┘
```

## Redis Usage Patterns

```
Redis Data Structures & Operations:
══════════════════════════════════

Vote Counting:
┌─────────────────────────────────┐
│  Key Pattern: poll:123:votes    │
│  Operation: INCR                │
│  Value: 42                      │
└─────────────────────────────────┘

Option Counting:
┌─────────────────────────────────┐
│  Key: poll:123:option:abc:votes │
│  Operation: INCR                │
│  Value: 15                      │
└─────────────────────────────────┘

Duplicate Prevention:
┌─────────────────────────────────┐
│  Key: poll:123:voters           │
│  Operation: SADD, SISMEMBER     │
│  Value: {192.168.1.1, ...}     │
└─────────────────────────────────┘

Real-Time Updates:
┌─────────────────────────────────┐
│  Channel: poll:123:updates      │
│  Operation: PUBLISH/SUBSCRIBE   │
│  Payload: {"votes": 42, "opt"}  │
└─────────────────────────────────┘

Rate Limiting:
┌─────────────────────────────────┐
│  Key: ratelimit:192.168.1.1     │
│  Operation: SETEX (60 seconds)  │
│  Value: 1                       │
└─────────────────────────────────┘

Trending Polls:
┌─────────────────────────────────┐
│  Key: trending_polls            │
│  Operation: ZADD, ZREVRANGE     │
│  Value: {score: poll_id}        │
└─────────────────────────────────┘
```

## Database Schema

```
PostgreSQL Tables:
═════════════════

┌─────────────────────────────────┐
│              polls              │
├─────────────────────────────────┤
│ id          | UUID (PK)         │
│ title       | VARCHAR(255)      │
│ description | TEXT              │
│ created_at  | TIMESTAMP         │
│ expires_at  | TIMESTAMP         │
│ creator_ip  | INET              │
│ is_active   | BOOLEAN           │
│ category    | VARCHAR(100)      │
│ total_votes | INTEGER           │
└─────────────────────────────────┘
              │
              │ (1 to many)
              ▼
┌─────────────────────────────────┐
│           poll_options          │
├─────────────────────────────────┤
│ id          | UUID (PK)         │
│ poll_id     | UUID (FK)         │
│ option_text | VARCHAR(255)      │
│ vote_count  | INTEGER           │
│ created_at  | TIMESTAMP         │
└─────────────────────────────────┘
              │
              │ (1 to many)
              ▼
┌─────────────────────────────────┐
│             votes               │
├─────────────────────────────────┤
│ id            | UUID (PK)       │
│ poll_id       | UUID (FK)       │
│ option_id     | UUID (FK)       │
│ voter_ip      | INET            │
│ voter_session | VARCHAR(255)    │
│ voted_at      | TIMESTAMP       │
│ UNIQUE(poll_id, voter_ip)       │
└─────────────────────────────────┘

Indexes:
- idx_polls_active ON polls(is_active, created_at)
- idx_votes_poll_ip ON votes(poll_id, voter_ip)
- idx_poll_options_poll_id ON poll_options(poll_id)
```

## Real-Time Communication

```
WebSocket Message Flow:
══════════════════════

Browser A          Server           Browser B          Browser C
    │                 │                 │                 │
    │─── Vote ────────▶│                 │                 │
    │                 │                 │                 │
    │                 │── Update ──────▶│                 │
    │                 │                 │                 │
    │                 │── Update ──────────────────────▶│
    │                 │                 │                 │
    │◄─── Confirm ────│                 │                 │
    │                 │                 │                 │
    │                 │                 │◄─── Refresh ────│
    │                 │                 │                 │
    │                 │─── Current ────▶│                 │
    │                 │                 │                 │

Message Types:
┌─────────────────────────────────┐
│ vote_submitted                  │
│ {                               │
│   poll_id: "123",              │
│   option_id: "abc",            │
│   new_count: 42,               │
│   total_votes: 156             │
│ }                               │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│ poll_updated                    │
│ {                               │
│   poll_id: "123",              │
│   options: [                   │
│     {id: "abc", count: 42},    │
│     {id: "def", count: 38}     │
│   ],                           │
│   total_votes: 80              │
│ }                               │
└─────────────────────────────────┘
```

## API Structure

```
REST API Endpoints:
══════════════════

POST /api/polls
┌─────────────────────────────────┐
│ Create new poll                 │
│ Body: {title, description, ...} │
│ Returns: {id, title, options}   │
└─────────────────────────────────┘

GET /api/polls
┌─────────────────────────────────┐
│ List all active polls           │
│ Query: ?page=1&limit=20         │
│ Returns: {polls[], pagination}  │
└─────────────────────────────────┘

GET /api/polls/[id]
┌─────────────────────────────────┐
│ Get poll details                │
│ Returns: {poll, options, stats} │
└─────────────────────────────────┘

POST /api/polls/[id]/vote
┌─────────────────────────────────┐
│ Submit vote                     │
│ Body: {option_id}               │
│ Returns: {success, new_counts}  │
└─────────────────────────────────┘

GET /api/polls/[id]/stream
┌─────────────────────────────────┐
│ WebSocket/SSE for live updates  │
│ Returns: Real-time vote stream  │
└─────────────────────────────────┘
```

## Performance Architecture

```
Caching Strategy:
════════════════

┌─────────────────────────────────┐
│           Request               │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│       Redis Cache               │
│   ┌─────────────────────────┐   │
│   │ Check for cached data   │   │
│   └─────────────────────────┘   │
└─────────────┬───────────────────┘
              │
       ┌──────┴──────┐
       │             │
   HIT │             │ MISS
       ▼             ▼
┌─────────────┐ ┌─────────────┐
│Return Cache │ │Query Postgres│
│    Data     │ │Store in Cache│
└─────────────┘ └─────────────┘

Connection Pooling:
┌─────────────────────────────────┐
│        API Requests             │
│    ┌─────┐ ┌─────┐ ┌─────┐     │
│    │ Req │ │ Req │ │ Req │     │
│    │  1  │ │  2  │ │  3  │     │
│    └─────┘ └─────┘ └─────┘     │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│      Connection Pool            │
│  ┌─────┐ ┌─────┐ ┌─────┐       │
│  │ PG  │ │ PG  │ │ PG  │       │
│  │ Conn│ │ Conn│ │ Conn│       │
│  └─────┘ └─────┘ └─────┘       │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│      Aiven PostgreSQL           │
└─────────────────────────────────┘
```

## Deployment Flow

```
Development to Production:
═════════════════════════

Local Development:
┌─────────────────────────────────┐
│  Next.js Dev Server             │
│  ├── Hot reloading              │
│  ├── Local API routes           │
│  └── WebSocket dev server       │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│    Aiven Services               │
│  ├── PostgreSQL (Dev)           │
│  └── Redis (Dev)                │
└─────────────┬───────────────────┘
              │
              ▼ (git push)
┌─────────────────────────────────┐
│      Vercel Build               │
│  ├── Build Next.js app          │
│  ├── Deploy to edge functions   │
│  └── Set environment variables  │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│   Production Deployment         │
│  ├── Global CDN                 │
│  ├── Serverless functions       │
│  └── WebSocket support          │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│    Aiven Services               │
│  ├── PostgreSQL (Prod)          │
│  └── Redis (Prod)               │
└─────────────────────────────────┘
```

## Security Model

```
Security Layers:
═══════════════

┌─────────────────────────────────┐
│          User Request           │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│        Rate Limiting            │
│  ┌─────────────────────────┐   │
│  │ Max 10 votes per minute │   │
│  │ Per IP address          │   │
│  └─────────────────────────┘   │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│       Input Validation          │
│  ┌─────────────────────────┐   │
│  │ Sanitize all inputs     │   │
│  │ Validate against schema │   │
│  └─────────────────────────┘   │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│    Duplicate Prevention         │
│  ┌─────────────────────────┐   │
│  │ Check IP in Redis set   │   │
│  │ One vote per poll/IP    │   │
│  └─────────────────────────┘   │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│      SQL Injection Prevention   │
│  ┌─────────────────────────┐   │
│  │ Parameterized queries   │   │
│  │ ORM/Query builder       │   │
│  └─────────────────────────┘   │
└─────────────┬───────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│        Process Request          │
└─────────────────────────────────┘
```

## Why This Architecture?

```
Technology Decision Matrix:
═══════════════════════════

Need: Real-time updates
├── Option 1: Database polling ❌
│   └── Too slow, high load
├── Option 2: Database LISTEN/NOTIFY ⚠️
│   └── Limited scalability
└── Option 3: Redis Pub/Sub ✅
    └── Sub-millisecond, scales well

Need: Data persistence
├── Option 1: File storage ❌
│   └── No ACID, hard to query
├── Option 2: NoSQL ⚠️
│   └── No complex queries
└── Option 3: PostgreSQL ✅
    └── ACID, rich queries, reliable

Need: Global deployment
├── Option 1: Traditional server ❌
│   └── Single region, manual scaling
├── Option 2: Container orchestration ⚠️
│   └── Complex setup, over-engineered
└── Option 3: Vercel serverless ✅
    └── Global edge, auto-scaling, simple

Result: Perfect stack for real-time voting! 🎯
```