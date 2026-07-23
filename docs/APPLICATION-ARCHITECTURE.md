# 🏗️ ShopNow Application Architecture

Comprehensive guide to understanding the ShopNow e-commerce application architecture, deployment patterns, and technology stack.

## 🎯 Overview

ShopNow is a **full-stack MERN application** designed to demonstrate real-world Kubernetes deployment patterns. The architecture follows modern microservices principles with containerized components.

## 📊 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Internet/Users                           │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                 Kubernetes Cluster                         │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Frontend   │  │    Admin    │  │   Backend   │        │
│  │   (React)   │  │   (React)   │  │ (Node.js)   │        │
│  │             │  │             │  │             │        │
│  │   nginx     │  │   nginx     │  │  Express    │        │
│  │   :3000     │  │   :3001     │  │   :5000     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│         │                 │                 │              │
│         └─────────────────┼─────────────────┘              │
│                           │                                │
│                  ┌────────▼────────┐                       │
│                  │    MongoDB      │                       │
│                  │   (Database)    │                       │
│                  │     :27017      │                       │
│                  └─────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

## 🧩 Component Architecture

### 1. Frontend Application (Customer Interface)

**Technology Stack:**
- **Framework**: React 18 (Client-side JavaScript)
- **Build Tool**: Create React App
- **Runtime**: nginx (Static file server + Reverse proxy)
- **Port**: 3000

**Architecture Pattern:**
```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend Container                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Multi-Stage Docker Build                   ││
│  │                                                         ││
│  │  Stage 1: Builder (node:18-alpine)                     ││
│  │  ├── npm ci (install dependencies)                     ││
│  │  ├── npm run build (compile React → static files)     ││
│  │  └── Output: /app/build/ (HTML, CSS, JS bundles)      ││
│  │                                                         ││
│  │  Stage 2: Runtime (nginx:alpine)                       ││
│  │  ├── Copy static files from Stage 1                   ││
│  │  ├── Configure nginx for SPA routing                  ││
│  │  └── Proxy /api/* requests to backend                 ││
│  └─────────────────────────────────────────────────────────┘│
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                 nginx Configuration                     ││
│  │                                                         ││
│  │  server {                                               ││
│  │    listen 80;                                           ││
│  │    root /usr/share/nginx/html;                          ││
│  │                                                         ││
│  │    # Serve React static files                           ││
│  │    location / {                                         ││
│  │      try_files $uri $uri/ /index.html;                 ││
│  │    }                                                    ││
│  │                                                         ││
│  │    # Proxy API calls to backend                         ││
│  │    location /api/ {                                     ││
│  │      proxy_pass http://backend:5000/api/;              ││
│  │    }                                                    ││
│  │  }                                                      ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Execution Flow:**
1. **Build Time**: Node.js compiles React code into static files
2. **Runtime**: nginx serves static files to browsers
3. **Client-Side**: JavaScript executes in user's browser
4. **API Calls**: nginx proxies requests to backend service

### 2. Admin Dashboard (Administrative Interface)

**Technology Stack:**
- **Framework**: React 18 (Client-side JavaScript)
- **Build Tool**: Create React App
- **Runtime**: nginx (Static file server + Reverse proxy)
- **Port**: 3001

**Architecture Pattern:**
```
┌─────────────────────────────────────────────────────────────┐
│                     Admin Container                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Multi-Stage Docker Build                   ││
│  │                                                         ││
│  │  Stage 1: Builder (node:18-alpine)                     ││
│  │  ├── npm ci (install dependencies)                     ││
│  │  ├── npm run build (compile React → static files)     ││
│  │  └── Output: /app/build/ (HTML, CSS, JS bundles)      ││
│  │                                                         ││
│  │  Stage 2: Runtime (nginx:alpine)                       ││
│  │  ├── Copy static files from Stage 1                   ││
│  │  ├── Configure nginx for admin SPA                    ││
│  │  └── Proxy /api/* requests to backend                 ││
│  └─────────────────────────────────────────────────────────┘│
│                                                             │
│  Features:                                                  │
│  ├── Product Management (CRUD)                             │
│  ├── Order Processing                                      │
│  ├── User Management                                       │
│  ├── Analytics Dashboard                                   │
│  └── System Configuration                                  │
└─────────────────────────────────────────────────────────────┘
```

### 3. Backend API (Business Logic)

**Technology Stack:**
- **Runtime**: Node.js 18
- **Framework**: Express.js
- **Database ODM**: Mongoose
- **Authentication**: JWT
- **Port**: 5000

**Architecture Pattern:**
```
┌─────────────────────────────────────────────────────────────┐
│                    Backend Container                        │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                 Express.js Server                       ││
│  │                                                         ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │                API Routes                           │││
│  │  │                                                     │││
│  │  │  /api/products    ← Product CRUD operations         │││
│  │  │  /api/users       ← User authentication             │││
│  │  │  /api/orders      ← Order management                │││
│  │  │  /api/cart        ← Shopping cart operations        │││
│  │  │  /api/admin       ← Admin-only endpoints            │││
│  │  │  /health          ← Health check endpoint           │││
│  │  └─────────────────────────────────────────────────────┘││
│  │                                                         ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │              Middleware Stack                       │││
│  │  │                                                     │││
│  │  │  ├── CORS (Cross-origin requests)                  │││
│  │  │  ├── Body Parser (JSON parsing)                    │││
│  │  │  ├── JWT Authentication                             │││
│  │  │  ├── Request Validation                             │││
│  │  │  └── Error Handling                                 │││
│  │  └─────────────────────────────────────────────────────┘││
│  │                                                         ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │              Database Layer                         │││
│  │  │                                                     │││
│  │  │  ├── Mongoose ODM                                   │││
│  │  │  ├── Connection Management                          │││
│  │  │  ├── Schema Definitions                             │││
│  │  │  └── Query Optimization                             │││
│  │  └─────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 4. MongoDB Database (Data Persistence)

**Technology Stack:**
- **Database**: MongoDB 5.0
- **Deployment**: StatefulSet (for data persistence)
- **Storage**: Persistent Volumes
- **Port**: 27017

**Architecture Pattern:**
```
┌─────────────────────────────────────────────────────────────┐
│                   MongoDB Container                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                 Database Schema                         ││
│  │                                                         ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │                Collections                          │││
│  │  │                                                     │││
│  │  │  ├── products     ← Product catalog data            │││
│  │  │  ├── users        ← User accounts and profiles      │││
│  │  │  ├── orders       ← Order history and status       │││
│  │  │  ├── cart         ← Shopping cart items             │││
│  │  │  └── sessions     ← User session data               │││
│  │  └─────────────────────────────────────────────────────┘││
│  │                                                         ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │              Persistent Storage                     │││
│  │  │                                                     ││
│  │  │  ├── PersistentVolume (10Gi default)               │││
│  │  │  ├── Data Directory: /data/db                      │││
│  │  │  ├── Backup Strategy                               │││
│  │  │  └── Replication (for HA)                          │││
│  │  └─────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## 🔄 Request Flow Architecture

### Customer Shopping Flow
```
1. User visits http://frontend-service:3000
   ↓
2. nginx serves React app (index.html + JS bundles)
   ↓
3. React app loads in browser (CLIENT-SIDE)
   ↓
4. User browses products → React makes API call
   ↓
5. Browser: GET /api/products
   ↓
6. nginx proxy: forwards to http://backend:5000/api/products
   ↓
7. Express.js: handles request, queries MongoDB
   ↓
8. MongoDB: returns product data
   ↓
9. Express.js: sends JSON response
   ↓
10. nginx: forwards response to browser
    ↓
11. React: updates UI with product data (CLIENT-SIDE)
```

### Admin Management Flow
```
1. Admin visits http://admin-service:3001
   ↓
2. nginx serves React admin app
   ↓
3. Admin app loads in browser (CLIENT-SIDE)
   ↓
4. Admin logs in → React makes authentication API call
   ↓
5. Browser: POST /api/admin/login
   ↓
6. nginx proxy: forwards to http://backend:5000/api/admin/login
   ↓
7. Express.js: validates credentials, returns JWT token
   ↓
8. React: stores token, makes authenticated requests
   ↓
9. Admin manages products → CRUD operations via API
   ↓
10. All changes persist to MongoDB
```

## 🐳 Container Architecture

### Multi-Stage Build Pattern (Frontend & Admin)

**Why Multi-Stage Builds?**
- **Smaller Images**: Production image doesn't include Node.js build tools
- **Security**: Fewer attack vectors in production image
- **Performance**: nginx is optimized for serving static files
- **Efficiency**: Faster container startup and lower resource usage

**Build Process:**
```dockerfile
# Stage 1: Build Stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --no-audit --prefer-offline
COPY . .
RUN npm run build  # Creates /app/build with static files

# Stage 2: Runtime Stage  
FROM nginx:alpine
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Result:**
- **Build Stage**: ~500MB (includes Node.js, npm, build tools)
- **Runtime Stage**: ~50MB (only nginx + static files)

### Single-Stage Build Pattern (Backend)

**Why Single-Stage for Backend?**
- **Runtime Requirement**: Node.js needed to execute JavaScript
- **Dynamic Content**: API responses generated at runtime
- **Process Management**: Express server must stay running

**Build Process:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 5000
CMD ["node", "server.js"]
```

## ☸️ Kubernetes Deployment Architecture

### Service Communication
```
┌─────────────────────────────────────────────────────────────┐
│                 Kubernetes Namespace: shopnow-demo         │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  frontend   │    │    admin    │    │   backend   │     │
│  │   Service   │    │   Service   │    │   Service   │     │
│  │             │    │             │    │             │     │
│  │ ClusterIP   │    │ ClusterIP   │    │ ClusterIP   │     │
│  │ :3000       │    │ :3001       │    │ :5000       │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│         │                   │                   │           │
│         ▼                   ▼                   ▼           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  frontend   │    │    admin    │    │   backend   │     │
│  │ Deployment  │    │ Deployment  │    │ Deployment  │     │
│  │             │    │             │    │             │     │
│  │ nginx pods  │    │ nginx pods  │    │ node.js pods│     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                               │             │
│                                               ▼             │
│                                    ┌─────────────┐         │
│                                    │    mongo    │         │
│                                    │   Service   │         │
│                                    │             │         │
│                                    │ Headless    │         │
│                                    │ :27017      │         │
│                                    └─────────────┘         │
│                                               │             │
│                                               ▼             │
│                                    ┌─────────────┐         │
│                                    │    mongo    │         │
│                                    │ StatefulSet │         │
│                                    │             │         │
│                                    │ mongodb pod │         │
│                                    │ + PVC       │         │
│                                    └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### Resource Allocation
```yaml
# Typical resource allocation
Resources:
  frontend:
    requests: { cpu: 50m, memory: 128Mi }
    limits:   { cpu: 200m, memory: 256Mi }
  
  admin:
    requests: { cpu: 50m, memory: 128Mi }
    limits:   { cpu: 200m, memory: 256Mi }
  
  backend:
    requests: { cpu: 100m, memory: 200Mi }
    limits:   { cpu: 500m, memory: 512Mi }
  
  mongodb:
    requests: { cpu: 250m, memory: 512Mi }
    limits:   { cpu: 1000m, memory: 2Gi }
```

## 🔐 Security Architecture

### Network Security
- **Namespace Isolation**: All components in `shopnow-demo` namespace
- **Service-to-Service**: Internal ClusterIP communication
- **API Gateway Pattern**: nginx proxies external requests
- **No Direct Database Access**: Only backend can access MongoDB

### Application Security
- **JWT Authentication**: Stateless token-based auth
- **CORS Configuration**: Controlled cross-origin requests
- **Input Validation**: Request validation middleware
- **Security Headers**: nginx adds security headers

## 🎓 Key Learning Points

### Architecture Concepts
1. **Microservices**: Each component has a single responsibility
2. **Containerization**: Consistent deployment across environments
3. **Service Mesh**: Internal service communication patterns
4. **Data Persistence**: StatefulSet vs Deployment patterns
5. **Load Balancing**: Service discovery and traffic distribution

### React + nginx Pattern
1. **Build-Time Compilation**: React code becomes static files
2. **Runtime Serving**: nginx serves static files efficiently
3. **API Proxying**: nginx forwards API calls to backend
4. **SPA Routing**: nginx handles client-side routing
5. **Production Optimization**: Multi-stage builds for efficiency

### Kubernetes Patterns
1. **Service Discovery**: DNS-based service communication
2. **Configuration Management**: ConfigMaps and Secrets
3. **Storage Patterns**: Persistent volumes for databases
4. **Scaling Patterns**: Horizontal Pod Autoscaler
5. **Health Monitoring**: Liveness and readiness probes

## 🔍 Common Questions

### Q: Why use nginx for React apps instead of Node.js?
**A**: nginx is optimized for serving static files and uses fewer resources than Node.js. The React app is compiled to static files, so no JavaScript runtime is needed in production.

### Q: How do the React apps communicate with the backend?
**A**: The React apps make HTTP requests to `/api/*` endpoints, which nginx proxies to the backend service using Kubernetes service discovery.

### Q: Why use StatefulSet for MongoDB instead of Deployment?
**A**: StatefulSets provide stable network identities and ordered deployment, which is important for databases that need persistent storage and consistent hostnames.

### Q: How does service discovery work between components?
**A**: Kubernetes provides DNS-based service discovery. Services can communicate using service names (e.g., `http://backend:5000`) within the same namespace.

---

This architecture demonstrates modern cloud-native application patterns while remaining simple enough for learning Kubernetes fundamentals. Each component serves a specific purpose and can be scaled independently based on demand.