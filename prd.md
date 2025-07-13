# V2MOM Management System - Product Requirements Document

## Overview

A complete V2MOM (Vision, Values, Methods, Obstacles, Measures) goal-setting framework management system consisting of a REST API backend and a Next.js frontend application. Built for high-growth SaaS companies to manage employee goal-setting across all organizational functions.

## Product Architecture

### System Components

1. **API Service** (`v2mom-api`)
   - Express.js REST API
   - PostgreSQL database (Aiven)
   - Deployed on Vercel

2. **Frontend Application** (`v2mom-data-loader`)
   - Next.js with TypeScript
   - Multiple interaction modes
   - Deployed on Vercel

### Technology Stack

- **Backend:** Node.js, Express.js, PostgreSQL (pg driver)
- **Frontend:** Next.js 15, React 18, TypeScript, Tailwind CSS
- **Database:** Aiven PostgreSQL (managed cloud database)
- **Deployment:** Vercel (both API and frontend)
- **Development:** Claude Code CLI

## API Service Specifications

### Database Schema

**Table: `v2mom`**
```sql
- id (SERIAL PRIMARY KEY)
- name (VARCHAR(255), NOT NULL)
- title (VARCHAR(255))
- manager (VARCHAR(255))
- function (VARCHAR(255))
- vision (TEXT)
- values (TEXT)
- method1 (TEXT)
- obstacles1 (TEXT)
- measures1 (TEXT)
- method2 (TEXT)
- obstacles2 (TEXT)
- measures2 (TEXT)
- method3 (TEXT)
- obstacles3 (TEXT)
- measures3 (TEXT)
- q1attainment (DECIMAL(5,2))
- q2attainment (DECIMAL(5,2))
- q3attainment (DECIMAL(5,2))
- q4attainment (DECIMAL(5,2))
- created_at (TIMESTAMP DEFAULT CURRENT_TIMESTAMP)
- updated_at (TIMESTAMP DEFAULT CURRENT_TIMESTAMP)
```

**Indexes:**
- Primary key on `id`
- Index on `name` for faster searches
- Index on `manager` for organizational queries

**Triggers:**
- Auto-update `updated_at` timestamp on record modification

### API Endpoints

**Base URL:** `https://v2mom-api.vercel.app`

#### Core CRUD Operations

1. **GET /api/v2mom**
   - Returns all V2MOM records
   - Ordered by creation date (newest first)
   - No pagination (suitable for moderate datasets)

2. **GET /api/v2mom/:id**
   - Returns specific V2MOM record by ID
   - 404 if record not found

3. **GET /api/v2mom/manager/:manager**
   - Returns all records for a specific manager
   - URL-encoded manager name parameter

4. **POST /api/v2mom**
   - Creates new V2MOM record
   - Requires `name` field (all others optional)
   - Returns created record with generated ID and timestamps

5. **PUT /api/v2mom/:id**
   - Updates existing V2MOM record
   - Updates `updated_at` timestamp automatically
   - 404 if record not found

6. **DELETE /api/v2mom/:id**
   - Deletes V2MOM record
   - 404 if record not found

#### Utility Endpoints

1. **GET /health**
   - Health check endpoint
   - Returns status and timestamp

2. **GET /**
   - Root endpoint with API information
   - Lists available endpoints

### Data Validation

- **Required Fields:** `name` only
- **Field Lengths:** Name/title/manager limited to 255 characters
- **Numeric Fields:** Quarterly attainment percentages (0-100, 2 decimal places)
- **Error Handling:** Comprehensive error responses with appropriate HTTP status codes

### Security Features

- **Environment Variables:** Database credentials securely managed through environment variables
- **CORS Enabled:** Allows frontend access
- **SSL Connections:** Secure database connections (with certificate bypass for Aiven compatibility)
- **Input Validation:** Basic validation on required fields
- **Error Sanitization:** Database errors are caught and sanitized

## Frontend Application Specifications

### User Interface Modes

1. **View Mode**
   - Browse all existing V2MOM records
   - Expandable details for each record
   - Edit/Delete actions for each record
   - Refresh functionality

2. **Form Mode**
   - Traditional form interface
   - All 19 V2MOM fields organized in sections:
     - Basic Information (name, title, manager, function)
     - Vision & Values
     - Method 1-3 (each with method, obstacles, measures)
     - Quarterly Attainment (Q1-Q4 percentages)
   - Form validation (name required)
   - Create new or edit existing records

3. **Markdown Mode**
   - Edit V2MOM data in markdown format
   - Live preview functionality
   - Template-based structure
   - Markdown-to-object parsing
   - Full editing capabilities

4. **Simulate Mode**
   - AI-powered data generation
   - Job title input field
   - Function-specific content generation
   - SaaS-focused templates for multiple departments:
     - GTM, Sales, Marketing
     - Product, Engineering
     - CEO Office, COO
     - Finance, People & Culture, Legal, Data & Analytics
   - Leadership role detection
   - Edit/preview before saving

### Navigation & UX

- **Tab-based Navigation:** Four primary modes
- **Responsive Design:** Works on desktop and mobile
- **Minimalist Styling:** Clean, professional interface using Tailwind CSS
- **Loading States:** Visual feedback during operations
- **Error Handling:** User-friendly error messages

### Configuration Features

- **API Endpoint Configuration:** Configurable base URL with default
- **Environment Detection:** Different behavior for development vs production
- **Settings Panel:** Collapsible API configuration interface

### Data Management

- **Full CRUD Operations:** Create, read, update, delete records
- **Real-time Updates:** Automatic refresh after operations
- **Data Validation:** Client-side validation before API calls
- **Error Recovery:** Graceful handling of API failures

## Functional Requirements

### Core User Journeys

1. **New User Onboarding**
   - User accesses application
   - Views sample data (if available)
   - Creates first V2MOM using preferred mode (Form/Markdown/Simulate)
   - Saves to database

2. **Regular Data Entry**
   - User selects appropriate input mode
   - Enters V2MOM data for team members
   - Reviews and edits as needed
   - Saves multiple records

3. **Data Management**
   - User views existing records
   - Edits records using any mode
   - Deletes outdated records
   - Manages team data by manager

4. **Bulk Data Generation (Testing)**
   - User generates multiple test records using Simulate mode
   - Uses various job titles across different functions
   - Populates database for testing/demonstration

### Performance Requirements

- **API Response Time:** < 2 seconds for CRUD operations
- **Frontend Load Time:** < 3 seconds initial load
- **Database Queries:** Optimized with appropriate indexes
- **Concurrent Users:** Supports multiple simultaneous users

### Compatibility Requirements

- **Browsers:** Modern browsers (Chrome, Firefox, Safari, Edge)
- **Devices:** Desktop and mobile responsive
- **Database:** PostgreSQL 12+ compatibility
- **Node.js:** Version 18.x runtime environment

## Non-Functional Requirements

### Scalability

- **Database:** Handles thousands of V2MOM records
- **API:** Stateless design for horizontal scaling
- **Frontend:** Client-side rendering for performance

### Reliability

- **Uptime:** 99.9% availability target
- **Data Integrity:** ACID transactions for database operations
- **Error Recovery:** Graceful degradation on failures

### Security

- **Environment Configuration:** Database credentials managed through secure environment variables
- **Data Transmission:** HTTPS for all communications
- **Database Connections:** SSL encrypted (with Aiven compatibility)
- **Input Validation:** Protection against basic injection attacks
- **API Access:** Currently open (no authentication - see security considerations)

### Maintainability

- **Code Organization:** Modular architecture with separation of concerns
- **Documentation:** Comprehensive API documentation
- **Version Control:** Git-based workflow with descriptive commits
- **Deployment:** Automated deployment via Vercel integration

## Success Metrics

### User Engagement

- **Record Creation Rate:** Number of V2MOMs created per user session
- **Mode Usage:** Distribution of usage across Form/Markdown/Simulate modes
- **Session Duration:** Time spent in application
- **Return Usage:** User retention and repeat usage

### System Performance

- **API Success Rate:** Percentage of successful API calls
- **Error Rate:** Frequency and types of errors
- **Response Times:** Average API response times
- **Database Performance:** Query execution times

### Business Value

- **Data Quality:** Completeness of V2MOM records
- **User Adoption:** Number of active users
- **Use Case Coverage:** Coverage across different organizational functions
- **Feedback Quality:** User satisfaction with generated content

## Future Enhancements

### Phase 2 Features

1. **User Authentication & Authorization**
   - User accounts and login system
   - Role-based access control
   - Manager/employee permissions

2. **Advanced Data Management**
   - Bulk import/export functionality
   - Data archiving and historical tracking
   - Advanced filtering and search

3. **Collaboration Features**
   - Comments and feedback on V2MOMs
   - Approval workflows
   - Real-time collaboration

4. **Analytics & Reporting**
   - Dashboard with key metrics
   - Progress tracking and analytics
   - Goal achievement reporting

5. **Integration Capabilities**
   - HRIS system integration
   - Slack/Teams notifications
   - Calendar integration for reviews

### Technical Improvements

1. **Enhanced Security**
   - API authentication (JWT tokens)
   - Enhanced input sanitization and validation
   - Rate limiting and abuse prevention

2. **Performance Optimization**
   - Database query optimization
   - Caching layer implementation
   - API pagination for large datasets

3. **Developer Experience**
   - Automated testing suite
   - CI/CD pipeline enhancements
   - Monitoring and alerting

This PRD serves as the foundation for the V2MOM Management System, providing clear specifications for both current functionality and future development priorities.