# API Development Guide: Vercel to Aiven Integration

Quick reference for building secure Express.js APIs with Aiven PostgreSQL deployment on Vercel.

## Core Setup (10 minutes)

### 1. Create Express API
```javascript
// server.js
const express = require('express');
const cors = require('cors');
const v2momRoutes = require('./routes/v2mom');

const app = express();
app.use(cors());
app.use(express.json());
app.use('/api/v2mom', v2momRoutes);

module.exports = app;
```

### 2. Database Connection
```javascript
// routes/v2mom.js
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  ssl: { rejectUnauthorized: false }  // Aiven requirement
});
```

### 3. Environment Variables
```env
DB_HOST=pg-xyz123-myapp.e.aivencloud.com
DB_PORT=25464
DB_NAME=defaultdb
DB_USER=avnadmin
DB_PASSWORD=AVNS_[your_aiven_password]
```

## Vercel Configuration

### vercel.json
```json
{
  "functions": {
    "server.js": {
      "runtime": "@vercel/node"
    }
  },
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/server.js"
    }
  ]
}
```

## DO THIS ✅

- **Use environment variables** from first commit
- **Set `ssl: { rejectUnauthorized: false }`** for Aiven
- **Test locally first** with `npm run dev`
- **Add environment variables in Vercel dashboard** before deployment
- **Use GitHub integration** for automatic deployments
- **Include CORS** for frontend connectivity

## DON'T DO THIS ❌

- **Hard-code database credentials** in code
- **Skip SSL configuration** (connections will fail)
- **Deploy without testing locally** 
- **Forget to set Vercel environment variables**
- **Use connection string parsing** (causes corruption issues)
- **Skip error handling** in database operations

## Essential CRUD Pattern

```javascript
// GET all
router.get('/', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM v2mom ORDER BY created_at DESC');
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// POST create
router.post('/', async (req, res) => {
  try {
    const { vision, values, methods, obstacles, measures } = req.body;
    const result = await pool.query(
      'INSERT INTO v2mom (vision, values, methods, obstacles, measures) VALUES ($1, $2, $3, $4, $5) RETURNING *',
      [vision, values, methods, obstacles, measures]
    );
    res.status(201).json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## Debugging Common Issues

### SSL Certificate Errors
- **Problem**: "self-signed certificate in certificate chain"
- **Solution**: Use `ssl: { rejectUnauthorized: false }`

### Connection Failures
- **Problem**: Can't connect to Aiven
- **Solution**: Verify environment variables are set in Vercel dashboard

### CORS Errors
- **Problem**: Frontend can't fetch from API
- **Solution**: Ensure `app.use(cors())` in server.js

## FUTURE CONSIDERATIONS

### Security Enhancements
- **Authentication**: Add JWT tokens for user sessions
- **Rate Limiting**: Implement request throttling
- **Input Validation**: Use Joi or similar for request validation
- **SSL Certificates**: Configure proper SSL certificates for production

### Performance Optimizations
- **Connection Pooling**: Tune pool settings for high traffic
- **Caching**: Add Redis for frequently accessed data
- **Database Indexing**: Optimize queries with proper indexes

### Monitoring & Logging
- **Error Tracking**: Integrate Sentry or similar
- **Performance Monitoring**: Add APM tools
- **Query Logging**: Log slow database queries

This setup provides a production-ready foundation that can be enhanced based on specific application needs.