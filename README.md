# MEAN Stack Application — Dockerized with Nginx Reverse Proxy & CI/CD

This project is a Docker-based MEAN stack setup where:

- **Angular frontend** is served by Nginx  
- **Node.js backend** exposes REST APIs  
- **MongoDB** stores application data  
- **Nginx reverse proxy** handles all incoming traffic  
- **GitHub Actions CI/CD** builds & pushes Docker images to Docker Hub and deploys on a self-hosted runner  

Only **port 80** is exposed to the outside world.

---

## ⚙️ Architecture Flow

### **User Request → Reverse Proxy → Frontend/Backend → MongoDB → Back to User**

```
User Browser (http://<VM_IP>/)
        │
        ▼
NGINX Reverse Proxy (public port 80)
 ├── "/"      → frontend:80  (Angular served by Nginx)
 └── "/api/*" → backend:3000 (Node.js API)
        │
        ▼
Frontend container (Nginx → Angular static files)
Angular runs in browser → calls /api/tutorials
        │
        ▼
Reverse Proxy forwards /api → backend:3000
        │
        ▼
Backend container (Node.js + Express)
Connects to MongoDB: mongodb://mongodb:27017/dd_db
        │
        ▼
MongoDB container returns JSON
        │
        ▼
Backend → Reverse Proxy → Angular Browser UI updates
```

---

## What Each Part Does

### **Frontend**
- Angular app is built using Node (Stage 1).
- Served by **Nginx** inside the frontend container.
- Internal port: **80**.

### **Backend**
- Node.js + Express.
- Exposes REST APIs such as `/api/tutorials`.
- Reads DB config from:

```
process.env.MONGO_URL
# fallback
mongodb://mongodb:27017/dd_db
```

### **Database**
- MongoDB container.
- Stores tutorials data.
- Runs on port **27017** internally.

### **Reverse Proxy**
- Only container exposed to the VM:
  ```
  VM Port 80 → Reverse Proxy (80)
  ```
- Routes:
  - `/` → frontend
  - `/api` → backend

---

## 🔧 Why Certain Changes Were Made

### ✔️ Backend DB config change  
Because containers communicate using service names (not localhost), backend now uses:

```
mongodb://mongodb:27017/dd_db
```

This is correct for Docker Compose networking.

---

### ✔️ Angular base URL changed to `/api/...`  
Angular runs **in the browser**, so API calls automatically go to:

```
/api/tutorials
```

Reverse proxy sees `/api` and forwards to backend.  
No CORS issues.  
No IP address hard-coding.

---

## 🐳 Dockerfile Summary

### **Frontend Dockerfile**
- **Stage 1**: Build Angular app using `node:18-alpine`
- Installs dependencies with `npm ci`
- Runs `npm run build --prod`
- Output stored in `/app/dist/angular-15-crud`

- **Stage 2**: Serve Angular using `nginx:alpine`
- Copies build output to `/usr/share/nginx/html`
- Exposes port **80**

### **Backend Dockerfile**
- Uses `node:18-alpine`
- Installs dependencies (`npm ci` if lockfile exists)
- Copies backend code
- Exposes port **3000**
- Runs `node server.js`

---

## 🐳 docker-compose.yml Summary

- Services:
  - **nginx** → Reverse proxy (only public port)
  - **frontend** → Angular build served by Nginx
  - **backend** → Node.js API
  - **mongodb** → Database
- Network: `mean-network`
- Volume: `mongo-data` (stores MongoDB data)

Routing inside Docker network:
```
frontend → backend → mongodb
```

---

## 🌐 nginx.conf Summary

```
/      → frontend:80
/api   → backend:3000
```

Acts as a single entry point for the entire application.

---

## 🔁 CI/CD Pipeline (GitHub Actions)

### **Build job**
- Trigger: push to `main`
- Builds frontend & backend Docker images
- Tags with short commit SHA + `latest`
- Pushes images to Docker Hub

### **Deploy job**
- Runs on **self-hosted runner**
- Pulls latest images from Docker Hub
- Creates `docker-compose.override.yml`
- Executes:
  ```
  docker compose up -d --remove-orphans
  ```
- Automatically updates the application on the server.

---

## 🔧 How Angular, Backend, and DB Communicate

### **1. Angular loads UI**
User opens:
```
http://<_IP>/
```
Reverse proxy forwards UI request → `frontend:80`  
Nginx in frontend container serves Angular static files.

### **2. Angular calls API**
Angular sends:
```
/api/tutorials
```

### **3. Reverse Proxy logic**
If path starts with `/api`, forward to:
```
backend:3000
```

### **4. Backend connects to MongoDB**
Backend uses:
```
mongodb://mongodb:27017/dd_db
```

### **5. Response flows back**
MongoDB → Backend → Reverse Proxy → Angular → UI updates

---

## ✅ Final Summary

Your application flow:

**User → Reverse Proxy → Frontend**  
When Angular needs data:  
**/api → Reverse Proxy → Backend → MongoDB → Backend → Reverse Proxy → Angular UI**

Everything runs inside containers.  
All services communicate using Docker Compose service names.  
Only **port 80** is exposed.  
CI/CD fully automates build → push → deploy.

---

## 📘 Applications Used

### 🟢 Backend
| Application    | Purpose                         | Runs Inside              |
| -------------- | ------------------------------- | ------------------------ |
| Node.js        | Runs server.js (REST API)       | backend container        |
| Express.js     | Handles API routes              | backend container        |
| MongoDB        | Stores data                     | mongodb container        |

### 🔵 Frontend
| Application         | Purpose                            | Runs Inside            |
| ------------------- | ---------------------------------- | ---------------------- |
| Angular             | Builds UI                           | frontend container     |
| Nginx (static)      | Serves Angular build                | frontend container     |
| Nginx (proxy)       | Main entry point for whole app      | nginx container        |

---

