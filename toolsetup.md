# V2MOM System Setup Guide - Aiven + Vercel + Claude Code Workflow

This guide provides step-by-step instructions to replicate the complete V2MOM system setup using Aiven PostgreSQL, Vercel deployment, and Claude Code CLI for development.

## Prerequisites & Environment Setup

### Required Tools Installation

1. **Node.js & npm**
   ```bash
   # Check if installed
   node --version
   npm --version
   
   # Install if needed (recommended: LTS version)
   # Download from https://nodejs.org or use homebrew:
   brew install node
   ```

2. **Git**
   ```bash
   # Check if installed
   git --version
   
   # Install if needed
   brew install git
   ```

3. **Claude Code CLI**
   ```bash
   # Install Claude Code (if not already installed)
   # Follow instructions at https://docs.anthropic.com/claude-code
   ```

4. **GitHub CLI (Optional but Recommended)**
   ```bash
   # Install GitHub CLI
   brew install gh
   
   # Authenticate
   gh auth login
   ```

5. **Vercel CLI (Optional - can use web interface)**
   ```bash
   # Install Vercel CLI
   npm install -g vercel
   ```

6. **PostgreSQL Client (for database verification)**
   ```bash
   # Install PostgreSQL client tools
   brew install postgresql@15
   ```

## Database Setup - Aiven PostgreSQL

### 1. Create Aiven Account & Database

1. **Sign up for Aiven**
   - Go to https://aiven.io
   - Create account or log in
   - Navigate to "Create Service"

2. **Create PostgreSQL Service**
   ```
   Service Type: PostgreSQL
   Cloud Provider: (your preference - AWS/GCP/Azure)
   Region: (choose closest to your users)
   Service Plan: (Hobbyist plan sufficient for development)
   Service Name: goals-db (or your preference)
   ```

3. **Configure Service Settings**
   ```
   PostgreSQL Version: 15 or later
   Connection Pooling: PgBouncer (recommended)
   IP Whitelist: Leave open for development (0.0.0.0/0)
   ```

4. **Record Connection Details**
   After service creation, save these details:
   ```
   Service URI: postgres://username:password@host:port/database
   Host: pg-xxxxxxxx-goals.x.aivencloud.com
   Port: 25464
   Database: defaultdb
   Username: avnadmin
   Password: (generated password)
   ```

### 2. Create Database Schema

1. **Connect via PostgreSQL Client**
   ```bash
   /opt/homebrew/opt/postgresql@15/bin/psql "postgres://avnadmin:PASSWORD@HOST:25464/defaultdb?sslmode=require"
   ```

2. **Create V2MOM Table**
   ```sql
   -- Execute the database-setup.sql script
   -- Or run manually:
   
   CREATE TABLE v2mom (
       id SERIAL PRIMARY KEY,
       name VARCHAR(255) NOT NULL,
       title VARCHAR(255),
       manager VARCHAR(255),
       function VARCHAR(255),
       vision TEXT,
       values TEXT,
       method1 TEXT,
       obstacles1 TEXT,
       measures1 TEXT,
       method2 TEXT,
       obstacles2 TEXT,
       measures2 TEXT,
       method3 TEXT,
       obstacles3 TEXT,
       measures3 TEXT,
       q1attainment DECIMAL(5,2),
       q2attainment DECIMAL(5,2),
       q3attainment DECIMAL(5,2),
       q4attainment DECIMAL(5,2),
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   
   -- Create indexes
   CREATE INDEX idx_v2mom_name ON v2mom(name);
   CREATE INDEX idx_v2mom_manager ON v2mom(manager);
   
   -- Create update trigger
   CREATE OR REPLACE FUNCTION update_updated_at_column()
   RETURNS TRIGGER AS $$
   BEGIN
       NEW.updated_at = CURRENT_TIMESTAMP;
       RETURN NEW;
   END;
   $$ language 'plpgsql';
   
   CREATE TRIGGER update_v2mom_updated_at 
   BEFORE UPDATE ON v2mom 
   FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
   ```

3. **Verify Table Creation**
   ```sql
   \dt
   \d v2mom
   SELECT COUNT(*) FROM v2mom;
   ```

## API Service Setup

### 1. Initialize Node.js Project

```bash
# Create project directory
mkdir v2mom-api
cd v2mom-api

# Initialize npm project
npm init -y

# Install dependencies
npm install express pg dotenv cors

# Install development dependencies
npm install -D nodemon
```

### 2. Create Project Structure

```bash
# Create directories and files
mkdir routes
touch server.js
touch .env
touch .env.example
touch .gitignore
touch vercel.json
touch README.md
touch API-DOCUMENTATION.md
```

### 3. Configure Environment Variables

**Create `.env` file for local development:**
```bash
DB_HOST=your-host.aivencloud.com
DB_PORT=25464
DB_NAME=defaultdb
DB_USER=avnadmin
DB_PASSWORD=your-password
PORT=3000
```

**Create `.env.example` file:**
```bash
DB_HOST=your-host.aivencloud.com
DB_PORT=25464
DB_NAME=defaultdb
DB_USER=avnadmin
DB_PASSWORD=your-password
PORT=3000
```

### 4. Create API Code

**Create `routes/v2mom.js`:**
```javascript
const express = require('express');
const { Pool } = require('pg');
const router = express.Router();

// Secure configuration using environment variables
const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  ssl: {
    rejectUnauthorized: false
  }
});

// CRUD routes implementation...
```

**Create `server.js`:**
```javascript
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const v2momRoutes = require('./routes/v2mom');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());

// Routes
app.use('/api/v2mom', v2momRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'OK', timestamp: new Date().toISOString() });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### 5. Configure Vercel Deployment

**Create `vercel.json`:**
```json
{
  "version": 2,
  "builds": [
    {
      "src": "server.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/server.js"
    }
  ]
}
```

**Update `package.json`:**
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "build": "echo 'No build step required'"
  },
  "engines": {
    "node": "18.x"
  }
}
```

### 6. Deploy API to Vercel

#### Option 1: Using GitHub + Vercel (Recommended)

```bash
# Initialize git repository
git init
git add .
git commit -m "Initial API setup"

# Create GitHub repository using GitHub CLI
gh repo create v2mom-api --private --description "V2MOM API Service"

# Push to GitHub
git remote add origin https://github.com/yourusername/v2mom-api.git
git push -u origin main
```

**In Vercel Dashboard:**
1. Go to vercel.com and sign in
2. Click "Add New" → "Project"
3. Import from GitHub → select `v2mom-api`
4. Deploy with default settings
5. Add Environment Variables:
   - `DB_HOST`: your-host.aivencloud.com
   - `DB_PORT`: 25464
   - `DB_NAME`: defaultdb
   - `DB_USER`: avnadmin
   - `DB_PASSWORD`: your-generated-password

#### Option 2: Using Vercel CLI

```bash
# Login to Vercel
vercel login

# Deploy
vercel

# Set environment variables
vercel env add DB_HOST
vercel env add DB_PORT
vercel env add DB_NAME
vercel env add DB_USER
vercel env add DB_PASSWORD
```

### 7. Test API Deployment

```bash
# Test health endpoint
curl https://your-api.vercel.app/health

# Test V2MOM endpoints
curl https://your-api.vercel.app/api/v2mom
```

## Frontend Application Setup

### 1. Create Next.js Project

```bash
# Create Next.js project
mkdir v2mom-data-loader
cd v2mom-data-loader

# Initialize with required dependencies
npm init -y
npm install next@15.3.5 react@18.3.1 react-dom@18.3.1
npm install react-markdown typescript @types/node @types/react @types/react-dom
npm install -D eslint eslint-config-next tailwindcss autoprefixer postcss
```

### 2. Configure Next.js

**Create `next.config.js`:**
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // No experimental config needed for Next.js 15
}

module.exports = nextConfig
```

**Create `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{"name": "next"}],
    "baseUrl": ".",
    "paths": {"@/*": ["./src/*"]}
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 3. Setup Tailwind CSS

**Create `tailwind.config.js`:**
```javascript
module.exports = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

**Create `postcss.config.js`:**
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### 4. Create Project Structure

```bash
mkdir -p src/app src/components src/lib src/types
touch src/app/globals.css
touch src/app/layout.tsx
touch src/app/page.tsx
```

### 5. Implement Core Components

Create the following components with full functionality:

- `src/types/v2mom.ts` - TypeScript interfaces
- `src/lib/api.ts` - API client class
- `src/lib/markdown.ts` - Markdown parsing utilities
- `src/lib/simulation.ts` - AI data generation
- `src/components/FormMode.tsx` - Form interface
- `src/components/MarkdownMode.tsx` - Markdown editor
- `src/components/SimulateMode.tsx` - AI generation interface
- `src/components/RecordsList.tsx` - Data viewing
- `src/components/Settings.tsx` - API configuration
- `src/app/page.tsx` - Main application
- `src/app/layout.tsx` - App layout

### 6. Deploy Frontend to Vercel

```bash
# Initialize git repository
git init
git add .
git commit -m "Initial V2MOM Data Loader setup"

# Create GitHub repository
gh repo create v2mom-data-loader --private --description "Frontend app for managing V2MOM records"

# Push to GitHub
git remote add origin https://github.com/yourusername/v2mom-data-loader.git
git push -u origin main
```

**In Vercel Dashboard:**
1. Click "Add New" → "Project"
2. Import from GitHub → select `v2mom-data-loader`
3. Deploy with default settings (Vercel auto-detects Next.js)

## Claude Code Integration

### 1. Using Claude Code for Development

**Start Claude Code session:**
```bash
# Navigate to your project directory
cd /path/to/your/project

# Start Claude Code
claude-code
```

**Common Claude Code workflows:**
```bash
# File operations
Read package.json
Edit src/components/FormMode.tsx
Write src/new-component.tsx

# Search operations
Grep "API_BASE" --type js
Glob "**/*.tsx"

# Development tasks
Bash "npm run dev"
Bash "npm install react-query"

# Git operations
Bash "git status"
Bash "git add . && git commit -m 'Add new feature'"
```

### 2. Debugging with Claude Code

**For SSL certificate issues:**
```bash
# Check logs
Bash "vercel logs --follow"

# Test database connection
Bash "psql 'your-connection-string' -c 'SELECT 1'"

# Debug API endpoints
Bash "curl -v https://your-api.vercel.app/health"
```

**For frontend issues:**
```bash
# Check build errors
Bash "npm run build"

# Run development server
Bash "npm run dev"

# Check TypeScript errors
Bash "npx tsc --noEmit"
```

## Environment Configuration

### Development Environment

**Local development setup:**
```bash
# API service
cd v2mom-api
npm install
npm run dev  # Runs on http://localhost:3000

# Frontend (separate terminal)
cd v2mom-data-loader
npm install
npm run dev  # Runs on http://localhost:3001
```

### Production Environment

**Vercel Environment Variables:**

For API service:
- `DB_HOST`: Aiven PostgreSQL host
- `DB_PORT`: Database port (typically 25464)
- `DB_NAME`: Database name (defaultdb)
- `DB_USER`: Database username (avnadmin)
- `DB_PASSWORD`: Database password
- `NODE_ENV`: production (automatically set by Vercel)

For Frontend:
- No additional environment variables needed (API URL is configurable in UI)

### Local Development with Remote Database

**Connect to Aiven from local development:**
```bash
# Use same environment variables in local .env file
# Ensure all DB_* variables are properly set
# SSL certificate bypass already configured in code
npm run dev
```

## Troubleshooting Common Issues

### SSL Certificate Problems

**Symptoms:** `SELF_SIGNED_CERT_IN_CHAIN` errors

**Solution:** Use environment variable configuration with SSL bypass:
```javascript
const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  ssl: {
    rejectUnauthorized: false
  }
});
```

### Database Connection Issues

**Symptoms:** Database connection timeouts or authentication errors

**Troubleshooting steps:**
1. Verify Aiven service is running
2. Check connection string format
3. Confirm IP whitelist settings
4. Test connection with psql client

### Vercel Deployment Issues

**Common problems and solutions:**
1. **Build failures:** Check Node.js version in package.json
2. **Environment variables:** Ensure all required vars are set
3. **Function timeouts:** Optimize database queries
4. **CORS errors:** Verify CORS configuration in API

### Frontend API Connection

**Symptoms:** "Failed to fetch records" errors

**Troubleshooting:**
1. Check API URL configuration in frontend
2. Verify API is deployed and responding
3. Check browser console for CORS errors
4. Test API endpoints with curl

## Testing & Validation

### API Testing

```bash
# Health check
curl https://your-api.vercel.app/health

# Get all records
curl https://your-api.vercel.app/api/v2mom

# Create record
curl -X POST https://your-api.vercel.app/api/v2mom \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","title":"Test Title"}'
```

### Database Validation

```sql
-- Connect to database
psql "your-connection-string"

-- Check table structure
\d v2mom

-- Verify data
SELECT COUNT(*) FROM v2mom;
SELECT name, title, created_at FROM v2mom LIMIT 5;
```

### Frontend Testing

1. **Manual testing:** Navigate through all four modes
2. **Create records:** Test Form, Markdown, and Simulate modes
3. **Data operations:** Edit and delete existing records
4. **API configuration:** Test with different API endpoints

## Deployment Verification

### Complete End-to-End Test

1. **Create test record via frontend**
2. **Verify in database:** Check record appears in PostgreSQL
3. **Test all CRUD operations:** Create, read, update, delete
4. **Test across modes:** Ensure all four frontend modes work
5. **Test simulation:** Generate records for different job titles

### Performance Validation

1. **API response times:** Should be < 2 seconds
2. **Frontend load times:** Should be < 3 seconds
3. **Database queries:** Monitor for slow queries
4. **Error handling:** Verify graceful error responses

This setup guide provides a complete workflow for replicating the V2MOM system using the Aiven + Vercel + Claude Code technology stack.