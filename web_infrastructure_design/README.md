# Web Infrastructure with HAProxy Cluster and Role Separation

## Purpose
The goal of this setup is to deploy the website **www.foobar.com** with an infrastructure that prioritizes **high availability**, **scalability**, and **clear separation of duties**. By assigning each role to its own server and clustering the entry point, the system becomes more reliable and easier to maintain.

---

## Components of the Infrastructure
The design is composed of:

- **Two HAProxy load balancers** working together in a cluster (active/passive) to guarantee availability.
- **One web server (Nginx)** responsible for static content delivery and request forwarding.
- **One application server** (running Node.js, Gunicorn, or PHP-FPM) where business logic is executed.
- **One dedicated database server** (MySQL/PostgreSQL) to store persistent data.
- **One additional server** reserved for redundancy, monitoring, or as an extra web node.

All client traffic is encrypted and terminated at the load balancer level using SSL certificates.

---

## Architecture Diagram (Text Representation)

```
          Internet (HTTPS)
                 |
       +----------------------+
       | HAProxy #1 (active) |
       +----------------------+
                 |
        [ Cluster / VIP ]
                 |
       +----------------------+
       | HAProxy #2 (backup) |
       +----------------------+
                 |
         Traffic Distribution
                 |
          +---------------+
          |               |
   +---------------+ +---------------+
   |  Web Server   | |  Extra Server |
   |   (Nginx)     | |  (Optional)   |
   +---------------+ +---------------+
                 |
         Application Server
            (Business Logic)
                 |
          Database Server
              (SQL DB)
```

---

## Role of Each Element

### 1. Load Balancer Cluster (HAProxy)
- **Reason**: Avoid a single point of failure at the entry.  
- Two HAProxy instances share a **virtual IP address**. If the active one fails, the backup instantly takes over.  
- Also responsible for distributing requests between web servers or app servers as needed.  

### 2. Web Server (Nginx)
- **Reason**: Offload static files (HTML, CSS, JS, images) and protect the application server.  
- Can cache frequent responses and terminate SSL if not done at HAProxy.  
- Provides flexibility: scaling horizontally is as simple as adding more web nodes behind the load balancer.  

### 3. Application Server
- **Reason**: Execute all dynamic operations.  
- Processes requests, handles authentication, runs API endpoints, and talks to the database.  
- Keeping it separate improves performance and simplifies debugging.  

### 4. Database Server
- **Reason**: Store persistent information in a centralized, secure location.  
- Only reachable by the application server (not exposed to the internet).  
- Isolation improves data integrity and reduces risk of unauthorized access.  

### 5. Additional Server
- **Reason**: Provide room for expansion.  
- Can be configured as:
  - A **second web server** to share traffic,  
  - A **monitoring/logging server** for observability,  
  - Or a **bastion host** to manage secure SSH access.  

---

## Web Server vs Application Server: Key Difference
- A **web server** (like Nginx/Apache) is optimized for connection management and static content delivery.  
- An **application server** (like Gunicorn, Node.js, or uWSGI) is built to run business logic, process user input, and interact with the database.  
- Separating them allows static requests to be handled quickly while dynamic ones are passed only where needed.

---

## Why This Design?
- **High availability**: Load balancer cluster removes downtime risk.  
- **Scalability**: Each role (web, app, DB) can be scaled independently.  
- **Security**: Database is isolated, application logic is protected, and web servers shield the backend.  
- **Maintainability**: Each server focuses on a single role, reducing complexity when troubleshooting or upgrading.

---

## Quick Defense Summary
> “Our architecture uses two HAProxy load balancers in a cluster to guarantee availability. The web server handles static content and forwards dynamic requests to the application server, which processes logic and communicates with the database. Each component is isolated for performance, scalability, and security. An additional server is included to allow redundancy or monitoring. The main distinction is that the web server focuses on serving content and managing connections, while the application server executes the business logic.”
