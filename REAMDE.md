Here’s the updated **README.md** with Docker and Docker Compose details, reflecting the backend and frontend structures shown in the screenshots.

---

# 🌐 Adaptive Point App
A full-stack RESTful web application built with **JAX-RS**, **Hibernate ORM**, **Angular**, and **JWT-based authentication**. The application is containerized using **Docker** and managed via **Docker Compose**, making it easy to deploy and run locally or in production.

---

## ✨ **Features**
- 🔐 **Secure Authentication:** Uses JSON Web Tokens (JWT) for secure user authentication and session management.
- 🔄 **RESTful API:** Built with JAX-RS to handle backend requests for authentication and point validation.
- 🛠️ **Database Persistence:** All point-check results and user credentials are stored in a **PostgreSQL** database using **Hibernate ORM**.
- 🌐 **Dockerized Deployment:** Easily deploy and run the app using **Docker** and **Docker Compose**.
- 📊 **Interactive Frontend:** Built with Angular, featuring dynamic validation, responsive design, and real-time updates.
- 📱 **Adaptive Layout:** Fully responsive interface with three display modes:
    - **Desktop:** ≥ 1046px
    - **Tablet:** 869px to 1046px
    - **Mobile:** < 869px

---

## 📁 **Project Structure**
### **Backend (JAX-RS, Hibernate)**
```
/src
    └── main
        └── java
            └── by
                └── yaroslavesev
                    └── backend
                        ├── config          # Configuration (DB, CORS, JWT Filters)
                        ├── controllers     # RESTful API endpoints
                        ├── DTO             # Data Transfer Objects
                        ├── models          # Hibernate entity models
                        ├── repositories    # Data access layers
                        ├── services        # Business logic
                        └── utils           # Utility classes (e.g., token handling)
```

### **Frontend (Angular)**
```
/app
    ├── components
    │   ├── login                 # Login page components
    │   └── main                  # Main application page components
    ├── guards                    # Route guards for authentication
    ├── interceptors              # Interceptors for attaching JWT tokens to requests
    ├── services                  # Services for authentication and point management
    └── assets                    # Static assets (images)
```

---

## 🔧 **Technologies Used**
- **Backend:**
    - JAX-RS (REST API)
    - Hibernate ORM (Database access)
    - PostgreSQL (Database)
    - JWT (Authentication)
    - WildFly (Application server)

- **Frontend:**
    - Angular 2+
    - Responsive HTML/CSS (Three modes: desktop, tablet, mobile)

- **Containerization:**
    - Docker and Docker Compose

---

## 🚀 **Getting Started with Docker Compose**

### **1. Clone the repository**
```bash
git clone https://github.com/yourusername/adaptive-point-app.git
cd adaptive-point-app
```

### **2. Build and run the containers**
```bash
docker-compose up --build
```

This will:
- Build the backend using WildFly and deploy it on port **8080**.
- Build the frontend using Nginx and serve it on port **4200**.
- Start a PostgreSQL database on port **5432**.

---

## 🛠️ **Docker Configuration**
### **Backend (Dockerfile.backend)**
The backend is built using **WildFly** and the application is deployed as a WAR file.

```dockerfile
FROM jboss/wildfly:34.0.1.Final

# Copy the built WAR file
COPY build/libs/Backend-1.0-SNAPSHOT.war /opt/jboss/wildfly/standalone/deployments/backend.war

# Install PostgreSQL driver
ADD https://jdbc.postgresql.org/download/postgresql-42.6.0.jar /opt/jboss/wildfly/standalone/deployments/

# Configure database connection and data source
RUN /opt/jboss/wildfly/bin/add-user.sh admin admin --silent
RUN /opt/jboss/wildfly/bin/standalone.sh -c standalone.xml --server-config=standalone.xml & sleep 10 && \
    /opt/jboss/wildfly/bin/jboss-cli.sh --connect --command="data-source add --name=PostgresDS --driver-name=postgresql-42.6.0.jar --connection-url=jdbc:postgresql://db:5432/appdb --jndi-name=java:/PostgresDS --user-name=postgres --password=root --use-ccm=true --driver-class=org.postgresql.Driver --valid-connection-checker-class-name=org.jboss.jca.adapters.jdbc.extensions.postgresql.PostgreSQLValidConnectionChecker --exception-sorter-class-name=org.jboss.jca.adapters.jdbc.extensions.postgresql.PostgreSQLExceptionSorter" && \
    /opt/jboss/wildfly/bin/jboss-cli.sh --connect --command="reload"

EXPOSE 8080

CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0"]
```

### **Frontend (Dockerfile.frontend)**
The frontend is built using Node.js and served using **Nginx**.

```dockerfile
# Build stage
FROM node:16 AS build-stage

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Build the Angular project
COPY . .
RUN npm run build --prod

# Serve using Nginx
FROM nginx:alpine
COPY --from=build-stage /app/dist/frontend /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### **Docker Compose Configuration (docker-compose.yml)**
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    container_name: postgres_container
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: root
      POSTGRES_DB: appdb
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  backend:
    build:
      context: .
      dockerfile: Dockerfile.backend
    container_name: wildfly_container
    restart: always
    depends_on:
      - db
    ports:
      - "8080:8080"

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.frontend
    container_name: frontend_container
    restart: always
    ports:
      - "4200:80"

volumes:
  db_data:
```

---

## 🛡️ **Security with JWT**
- Upon successful login, a JWT token is generated and sent to the client.
- The token is stored locally in the browser and attached to every API request.
- The backend verifies the token to authorize access to protected resources.

---

## 📊 **REST API Endpoints**

### **Authentication**
| Method | Endpoint           | Description               | Body (JSON)                      |
|--------|--------------------|---------------------------|----------------------------------|
| POST   | `/api/auth/login`  | Authenticate user and generate JWT token | `{ "username": "user", "password": "pass" }` |
| POST   | `/api/auth/register` | Register a new user       | `{ "username": "user", "password": "pass" }` |

### **Point Operations**
| Method | Endpoint                    | Description                             | Body (JSON)                          |
|--------|-----------------------------|-----------------------------------------|--------------------------------------|
| GET    | `/api/points`               | Get all points for the authenticated user | -                                    |
| POST   | `/api/points`               | Check and save a point                  | `{ "x": 1, "y": 2, "radius": 3 }`    |

---

## 📚 **Further Reading**
- [JAX-RS Documentation](https://jakarta.ee/specifications/restful-ws/)
- [Hibernate ORM](https://hibernate.org/)
- [JSON Web Tokens](https://jwt.io/)
- [Angular](https://angular.io/)
- [Docker Documentation](https://docs.docker.com/)

---

**Happy coding!** 🎉