# Production Deployment Guide - SooqBot

## Overview
This guide covers deploying your AI-powered Arabic e-commerce platform to various production environments.

## Option 1: Replit Deployments (Easiest)

### Prerequisites
- Active Replit account
- OpenAI API key configured in secrets

### Steps
1. **Configure Environment Variables**
   ```bash
   # In Replit Secrets tab, add:
   OPENAI_API_KEY=your_openai_key_here
   NODE_ENV=production
   ```

2. **Deploy**
   - Click "Deploy" button in Replit
   - Choose "Autoscale Deployment"
   - Your app will be available at `https://your-repl-name.your-username.repl.co`

3. **Custom Domain** (Optional)
   - Add your domain in deployment settings
   - Update DNS to point to Replit's servers

### Production Configuration
```typescript
// server/index.ts - Already configured
const PORT = process.env.PORT || 3001;
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

## Option 2: Vercel (Frontend) + Railway (Backend)

### Frontend Deployment (Vercel)

1. **Install Vercel CLI**
   ```bash
   npm install -g vercel
   ```

2. **Configure Build**
   ```json
   // vercel.json
   {
     "buildCommand": "npm run build",
     "outputDirectory": "dist/public",
     "devCommand": "npm run dev",
     "installCommand": "npm install"
   }
   ```

3. **Deploy**
   ```bash
   vercel --prod
   ```

### Backend Deployment (Railway)

1. **Connect Repository**
   - Go to railway.app
   - Connect your GitHub repository
   - Choose the backend folder

2. **Environment Variables**
   ```bash
   NODE_ENV=production
   DATABASE_URL=your_postgresql_url
   OPENAI_API_KEY=your_openai_key
   PORT=8080
   ```

3. **Deploy Configuration**
   ```json
   // railway.toml
   [build]
   builder = "nixpacks"
   buildCommand = "npm run build"

   [deploy]
   startCommand = "npm start"
   restartPolicyType = "always"
   ```

## Option 3: Docker Deployment

### Dockerfile
```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./package.json

EXPOSE 3001
CMD ["node", "dist/index.js"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/sooqbot
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=sooqbot
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Deploy Commands
```bash
# Build and start
docker-compose up --build -d

# View logs
docker-compose logs -f app

# Scale application
docker-compose up --scale app=3
```

## Option 4: AWS Deployment

### AWS App Runner
1. **Create apprunner.yaml**
   ```yaml
   version: 1.0
   runtime: nodejs18
   build:
     commands:
       build:
         - npm ci
         - npm run build
   run:
     runtime-version: 18
     command: node dist/index.js
     network:
       port: 3001
       env: PORT
     env:
       - name: NODE_ENV
         value: production
   ```

2. **Deploy via AWS Console**
   - Connect GitHub repository
   - Configure environment variables
   - Deploy automatically on push

### AWS ECS (Advanced)
```json
// task-definition.json
{
  "family": "sooqbot",
  "networkMode": "awsvpc",
  "requiresAttributes": [
    {
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
    }
  ],
  "containerDefinitions": [
    {
      "name": "sooqbot-app",
      "image": "your-ecr-repo/sooqbot:latest",
      "memory": 512,
      "cpu": 256,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3001,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:name"
        }
      ]
    }
  ]
}
```

## Database Setup

### Option 1: Neon (Recommended)
```bash
# Already configured in your project
# Just update DATABASE_URL in production environment
DATABASE_URL=postgresql://username:password@ep-xxx.neon.tech/sooqbot?sslmode=require
```

### Option 2: Supabase
```bash
# Create project at supabase.com
# Get connection string
DATABASE_URL=postgresql://postgres:password@db.xxx.supabase.co:5432/postgres
```

### Option 3: AWS RDS
```bash
# Create PostgreSQL instance
# Update connection string
DATABASE_URL=postgresql://username:password@rds-instance.region.rds.amazonaws.com:5432/sooqbot
```

## Environment Variables Setup

### Production Environment Variables
```bash
# Required
NODE_ENV=production
DATABASE_URL=your_postgresql_connection_string
OPENAI_API_KEY=your_openai_api_key

# Optional
PORT=3001
CORS_ORIGIN=https://yourdomain.com
SESSION_SECRET=your_session_secret_here
```

### Secrets Management
```bash
# For AWS
aws secretsmanager create-secret --name sooqbot/prod --secret-string '{"DATABASE_URL":"...","OPENAI_API_KEY":"..."}'

# For Azure
az keyvault secret set --vault-name sooqbot-vault --name database-url --value "..."

# For Google Cloud
gcloud secrets create database-url --data-file=-
```

## SSL Certificate Setup

### Let's Encrypt (Free)
```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d yourdomain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### Cloudflare (Recommended)
1. Add your domain to Cloudflare
2. Update nameservers
3. Enable "Full (strict)" SSL mode
4. Turn on "Always Use HTTPS"

## Performance Optimization

### 1. CDN Configuration
```javascript
// Add to your HTML
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="preconnect" href="https://fonts.gstatic.com">
<link rel="dns-prefetch" href="//api.openai.com">
```

### 2. Database Connection Pooling
```typescript
// server/db.ts - Already configured
export const pool = new Pool({ 
  connectionString: process.env.DATABASE_URL,
  max: 20, // Maximum connections
  min: 5,  // Minimum connections
  acquireTimeoutMillis: 60000,
  idleTimeoutMillis: 600000
});
```

### 3. Caching Strategy
```typescript
// Add Redis for caching
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Cache store data
const cacheStore = async (storeId: number, data: Store) => {
  await redis.setex(`store:${storeId}`, 3600, JSON.stringify(data));
};
```

## Monitoring and Logging

### 1. Application Monitoring
```typescript
// server/index.ts
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Log requests
app.use((req, res, next) => {
  logger.info(`${req.method} ${req.path}`, { 
    ip: req.ip, 
    userAgent: req.get('User-Agent') 
  });
  next();
});
```

### 2. Database Monitoring
```sql
-- Monitor slow queries
SELECT query, mean_time, calls, rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Monitor connections
SELECT count(*) as active_connections
FROM pg_stat_activity
WHERE state = 'active';
```

### 3. Health Check Endpoint
```typescript
// server/routes.ts - Add health check
app.get('/health', async (req, res) => {
  try {
    // Check database connection
    await db.select().from(stores).limit(1);
    
    // Check OpenAI API
    const openaiStatus = process.env.OPENAI_API_KEY ? 'configured' : 'missing';
    
    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      database: 'connected',
      openai: openaiStatus,
      version: process.env.npm_package_version || '1.0.0'
    });
  } catch (error) {
    res.status(500).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});
```

## Security Checklist

### 1. Environment Security
- [ ] Environment variables secured
- [ ] Database credentials rotated
- [ ] API keys restricted to necessary scopes
- [ ] HTTPS enabled everywhere
- [ ] CORS properly configured

### 2. Application Security
```typescript
// server/index.ts - Security headers
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';

app.use(helmet());

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // Limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);
```

### 3. Database Security
```sql
-- Create read-only user for analytics
CREATE ROLE analytics_user WITH LOGIN PASSWORD 'secure_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analytics_user;

-- Create application user with limited permissions
CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON stores, products, orders TO app_user;
```

## Backup Strategy

### 1. Database Backups
```bash
# Daily automated backup
#!/bin/bash
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql
aws s3 cp backup_$(date +%Y%m%d).sql s3://your-backup-bucket/

# Retention policy (keep 30 days)
find /backups -name "backup_*.sql" -mtime +30 -delete
```

### 2. Application Backups
```bash
# Backup application files
tar -czf app_backup_$(date +%Y%m%d).tar.gz /path/to/app
aws s3 cp app_backup_$(date +%Y%m%d).tar.gz s3://your-backup-bucket/
```

## Domain and DNS Setup

### 1. Domain Configuration
```bash
# A Records
yourdomain.com     IN A    your_server_ip
www.yourdomain.com IN A    your_server_ip

# CNAME for API
api.yourdomain.com IN CNAME your-backend-url.com
```

### 2. Subdomain Setup
```nginx
# nginx configuration
server {
    listen 80;
    server_name api.yourdomain.com;
    
    location / {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## Cost Optimization

### 1. Resource Planning
- **Small Scale**: 1 server, shared database ($20-50/month)
- **Medium Scale**: Load balancer + 2 servers + managed DB ($100-200/month)  
- **Large Scale**: Auto-scaling + CDN + multiple regions ($500+/month)

### 2. Database Optimization
```sql
-- Index optimization
CREATE INDEX idx_products_store_active ON products(store_id, is_active);
CREATE INDEX idx_orders_store_date ON orders(store_id, created_at);

-- Partition large tables
CREATE TABLE orders_2025 PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

## Deployment Checklist

### Pre-Deployment
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] SSL certificates installed
- [ ] Monitoring setup
- [ ] Backup strategy implemented

### Post-Deployment
- [ ] Health checks passing
- [ ] Performance metrics baseline
- [ ] Error monitoring active
- [ ] User acceptance testing
- [ ] Documentation updated

Your SooqBot platform is now ready for production deployment with enterprise-level reliability and scalability.