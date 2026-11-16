<div align="center">

# 🎟️ EventHub

### Full-Stack Event Management Platform

A production-style MERN application with role-based access control, real-time notifications, Docker containerization and a GitHub Actions CI pipeline.

[![CI](https://github.com/swayammankar1-hub/FSD-Lab-Event-Management/actions/workflows/ci.yml/badge.svg)](https://github.com/swayammankar1-hub/FSD-Lab-Event-Management/actions)
![Node](https://img.shields.io/badge/Node.js-20-339933?logo=node.js&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?logo=mongodb&logoColor=white)
![Socket.IO](https://img.shields.io/badge/Socket.IO-Realtime-010101?logo=socket.io&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)

</div>

---

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Tech Stack](#tech-stack)
4. [Architecture](#architecture)
5. [Project Structure](#project-structure)
6. [Getting Started](#getting-started)
   - [Prerequisites](#prerequisites)
   - [Environment Variables](#environment-variables)
   - [Option A: Docker (Recommended)](#option-a-docker-recommended)
   - [Option B: Manual Setup](#option-b-manual-setup)
   - [Creating an Admin Account](#creating-an-admin-account)
7. [Application Routes](#application-routes)
8. [API Reference](#api-reference)
9. [Real-Time Events](#real-time-events-socketio)
10. [Data Models](#data-models)
11. [Security](#security)
12. [CI/CD Pipeline](#cicd-pipeline)
13. [Deployment](#deployment)
14. [Documentation](#documentation)
15. [Author](#author)

---

## Overview

**EventHub** lets organizations publish events and lets attendees discover and book them. Admins manage the full event lifecycle, and every change is pushed to connected users instantly through WebSockets, so nobody has to refresh the page to see what's new.

The project was built for the **Full Stack Development Lab** and covers the full path from application code to a deployable, containerized system with automated checks.

## Key Features

### 🔐 Authentication & Authorization
- Registration and login with **bcrypt** password hashing.
- Stateless sessions using **JSON Web Tokens (JWT)**.
- Two roles: **`user`** and **`admin`**, enforced on the server by middleware and on the client by protected routes.
- Session recovery on page reload and automatic redirect for unauthorized access.

### 📅 Event Discovery
- Browse all upcoming events on a public page.
- Keyword search across title and description.
- Filter by **location** and **date range**.

### 🧑‍💼 Admin Dashboard
- Create, edit and cancel events with server-side validation.
- View all attendee registrations.
- Cancelling an event automatically removes its registrations (cascade delete).

### 🎫 Registrations
- One-click registration for an event.
- Duplicate bookings for the same event are blocked by a unique database index.
- Personal **My Registrations** page with cancellation.

### 🔔 Real-Time Notifications
- Socket.IO broadcasts whenever an event is **created**, **updated** or **deleted**.
- Notification bell in the navbar with an unread badge, a scrollable dropdown, timestamps, connection status and a clear action.

### 🚢 DevOps
- Multi-stage **Docker** builds for client and server.
- **Docker Compose** orchestration with an Nginx-served frontend.
- **GitHub Actions** pipeline for backend checks, frontend lint and production build.
- Render blueprint (`render.yaml`) and Vercel rewrites (`vercel.json`) for cloud deployment.

## Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | React 19, Vite, Tailwind CSS, React Router v7, `socket.io-client` |
| **Backend** | Node.js 20, Express, Socket.IO, `jsonwebtoken`, `bcryptjs`, `cors`, `dotenv` |
| **Database** | MongoDB (Atlas) with Mongoose ODM |
| **Containers** | Docker, Docker Compose, Alpine Linux, Nginx |
| **CI/CD** | GitHub Actions |
| **Hosting** | Vercel (frontend), Render (backend) |

## Architecture

```text
┌──────────────────────┐        HTTPS / REST         ┌──────────────────────────┐
│   React SPA (Vite)   │ ──────────────────────────▶ │  Express API  (Node 20)  │
│  Tailwind · Router   │                             │  JWT auth · RBAC         │
│                      │ ◀────── WebSocket ───────── │  Socket.IO server        │
└──────────────────────┘     (event:* broadcasts)    └────────────┬─────────────┘
                                                                  │ Mongoose
                                                                  ▼
                                                      ┌──────────────────────┐
                                                      │   MongoDB Atlas      │
                                                      │ Users · Events ·     │
                                                      │ Registrations        │
                                                      └──────────────────────┘
```

In Docker, Nginx serves the built SPA, falls back to `index.html` for client-side routes, and proxies `/api/` and `/socket.io/` to the backend container.

## Project Structure

```text
FSD-Lab-Event-Management/
├── .github/workflows/ci.yml          # CI pipeline
├── docs/                             # Lab reports (Experiments 7 and 8)
├── docker-compose.yml                # Frontend + backend orchestration
├── render.yaml                       # Render blueprint
└── Event-Management-system/
    ├── client/                       # React frontend
    │   ├── src/
    │   │   ├── components/           # Navbar, NotificationBell, EventCard, Modal, Button, ProtectedRoute, Loading
    │   │   ├── pages/                # Home, Events, Login, Register, MyRegistrations, AdminDashboard
    │   │   ├── context/              # AuthContext
    │   │   ├── hooks/                # useAuth
    │   │   ├── services/api.js       # API service layer
    │   │   ├── App.jsx               # Route definitions
    │   │   └── main.jsx              # Entry point
    │   ├── Dockerfile                # Multi-stage build, served by Nginx
    │   ├── nginx.conf                # SPA fallback and API/WebSocket proxy
    │   └── vercel.json               # Vercel rewrites
    └── server/                       # Express backend
        ├── config/db.js              # MongoDB connection
        ├── controllers/              # auth, event, registration logic
        ├── middleware/               # protect, adminOnly, error handlers
        ├── models/                   # User, Event, Registration
        ├── routes/                   # auth, event, registration routes
        ├── utils/generateToken.js    # JWT helper
        ├── Dockerfile
        └── server.js                 # Express + Socket.IO entry point
```

## Getting Started

### Prerequisites

- **Node.js 20+** and npm
- A **MongoDB** connection string ([MongoDB Atlas](https://www.mongodb.com/atlas) free tier works well)
- **Docker Desktop** (only for the Docker option)

### Environment Variables

**Server**: create `Event-Management-system/server/.env`

| Variable | Required | Description | Example |
| :--- | :---: | :--- | :--- |
| `MONGODB_URI` | ✅ | MongoDB connection string | `mongodb+srv://user:pass@cluster.mongodb.net/eventhub` |
| `JWT_SECRET` | ✅ | Secret used to sign tokens (the server refuses to start without it) | a long random string |
| `JWT_EXPIRES_IN` | | Token lifetime | `1d` |
| `PORT` | | API port (default `5000`) | `5000` |
| `CLIENT_ORIGIN` | | Allowed CORS origin for the frontend | `http://localhost:5173` |

**Client**: create `Event-Management-system/client/.env` (optional for local development)

| Variable | Description | Default |
| :--- | :--- | :--- |
| `VITE_API_URL` | Backend base URL. When unset, requests go to the same origin and are proxied. | none |
| `VITE_SOCKET_URL` | Socket.IO server URL | `http://localhost:5000` |

> ⚠️ Never commit `.env` files. They are already listed in `.gitignore`.

### Option A: Docker (Recommended)

Create the server `.env` first (see above), then from the repository root:

```bash
docker compose up -d --build     # build and start both services
docker compose logs -f           # follow logs
docker compose down              # stop and remove containers
```

| Service | URL |
| :--- | :--- |
| Frontend | http://localhost:5173 |
| Backend API | http://localhost:5000 |
| Health check | http://localhost:5000/api/health |

### Option B: Manual Setup

```bash
# 1. Clone
git clone https://github.com/swayammankar1-hub/FSD-Lab-Event-Management.git
cd FSD-Lab-Event-Management/Event-Management-system
```

**Backend** (terminal 1)

```bash
cd server
npm install
npm run dev          # nodemon on http://localhost:5000
```

**Frontend** (terminal 2)

```bash
cd client
npm install
npm run dev          # Vite on http://localhost:5173
```

| Script | Location | Purpose |
| :--- | :--- | :--- |
| `npm run dev` | server | Start API with auto-reload |
| `npm start` | server | Start API (production) |
| `npm run dev` | client | Start Vite dev server |
| `npm run build` | client | Production build |
| `npm run lint` | client | Run ESLint |

### Creating an Admin Account

For security, public registration **always** creates a `user`; a role is never accepted from request data. To create an admin:

1. Register a normal account through the app.
2. In MongoDB (Atlas UI or `mongosh`), change that user's role:

```js
db.users.updateOne({ email: "you@example.com" }, { $set: { role: "admin" } })
```

3. Log out and back in. The **Admin** dashboard is now available.

## Application Routes

| Path | Page | Access |
| :--- | :--- | :--- |
| `/` | Home | Public |
| `/events` | Event discovery with search and filters | Public |
| `/login` | Login | Public |
| `/register` | Sign up | Public |
| `/my-registrations` | Your bookings | Logged-in users |
| `/admin` | Admin dashboard | Admins only |

## API Reference

Base path: `/api`. Protected routes expect the header `Authorization: Bearer <token>`.

### Auth

| Method | Endpoint | Description | Access |
| :--- | :--- | :--- | :--- |
| `POST` | `/auth/register` | Create an account | Public |
| `POST` | `/auth/login` | Log in and receive a JWT | Public |
| `GET` | `/auth/me` | Return the current user | Authenticated |

### Events

| Method | Endpoint | Description | Access |
| :--- | :--- | :--- | :--- |
| `GET` | `/events` | List events. Query: `search`, `location`, `dateFrom`, `dateTo` | Public |
| `GET` | `/events/:id` | Get one event | Public |
| `POST` | `/events` | Create an event | Admin |
| `PUT` | `/events/:id` | Update an event | Admin |
| `DELETE` | `/events/:id` | Delete an event and its registrations | Admin |

### Registrations

| Method | Endpoint | Description | Access |
| :--- | :--- | :--- | :--- |
| `POST` | `/registrations` | Register for an event | User |
| `GET` | `/registrations/me` | List your registrations | User |
| `DELETE` | `/registrations/:id` | Cancel a registration | User |
| `GET` | `/registrations` | List all registrations | Admin |

### System

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Returns `{ "status": "ok" }` |

**Example**

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"you@example.com","password":"your-password"}'
```

## Real-Time Events (Socket.IO)

| Event | Direction | Triggered when | Payload |
| :--- | :--- | :--- | :--- |
| `event:created` | Server → Clients | An admin creates an event | `{ message, event: { _id, title, date, location }, createdAt }` |
| `event:updated` | Server → Clients | An admin edits an event | `{ message, event: { _id, title, date, location }, createdAt }` |
| `event:deleted` | Server → Clients | An admin deletes an event | `{ message, event: { _id, title }, createdAt }` |

## Data Models

| Model | Fields | Notes |
| :--- | :--- | :--- |
| **User** | `name`, `email` (unique), `password` (hashed, not returned by default), `role` (`user` \| `admin`) | Timestamps enabled |
| **Event** | `title`, `description`, `date`, `time` (`HH:MM`), `location`, `createdBy` → User | Indexed on date, location and a text index on title/description |
| **Registration** | `user` → User, `event` → Event | Unique compound index on `(user, event)` prevents duplicate bookings |

## Security

- Passwords hashed with bcrypt; the password field is excluded from queries by default.
- JWT verification middleware (`protect`) and role middleware (`adminOnly`) guard sensitive routes.
- Roles are never read from registration requests.
- CORS restricted to the configured `CLIENT_ORIGIN`.
- Request body size limited to 100 KB.
- User-supplied search and location text is regex-escaped before querying.
- The server refuses to start when `JWT_SECRET` is missing.

## CI/CD Pipeline

Defined in [`.github/workflows/ci.yml`](.github/workflows/ci.yml); runs on every push and pull request to `main`.

| Job | Steps |
| :--- | :--- |
| **Backend Code Verification** | Node 20 setup → `npm ci` → `node -c server.js` syntax check |
| **Frontend Lint & Build** | Node 20 setup → `npm ci` → `npm run lint` → `npm run build` |

## Deployment

| Component | Platform | Configuration |
| :--- | :--- | :--- |
| **Backend** | Render (Docker / Node web service) | [`render.yaml`](render.yaml). Set `MONGODB_URI`, `JWT_SECRET` and `CLIENT_ORIGIN` in the dashboard. |
| **Frontend** | Vercel | Root directory `Event-Management-system/client`. Set `VITE_API_URL` and `VITE_SOCKET_URL` to your backend URL. |

> After deploying the backend, update the placeholder `https://YOUR-BACKEND.onrender.com` in [`client/vercel.json`](Event-Management-system/client/vercel.json) so API calls are proxied to your own server, and set `CLIENT_ORIGIN` on the backend to your Vercel URL.

Live links (add yours after deploying):

- Frontend: `https://YOUR-FRONTEND.vercel.app`
- Backend: `https://YOUR-BACKEND.onrender.com`

## Documentation

Detailed lab reports live in [`docs/`](docs/README.md):

- [Experiment 7: Real-Time Communication with Socket.IO](docs/EXPERIMENT_7_SOCKET_IO_REALTIME.md)
- [Experiment 8: Containerization, CI/CD and Deployment](docs/EXPERIMENT_8_CONTAINERIZATION_AND_DEPLOYMENT.md)

## Author

**Swayam Mankar**
Full Stack Development Lab (FSD Lab)

[GitHub](https://github.com/swayammankar1-hub) · [LinkedIn](https://linkedin.com/in/swayam-mankar-989179320)
