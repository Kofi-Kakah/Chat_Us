<p align="center"> 💬 Chat_Us </p>

---

<p align="center">
  A production-style real-time chat API for a one-to-one messaging application.
</p>

<p align="center">
  Authentication → security → contacts → real-time chat → presence → image sharing
</p>

<p align="center">
  Built with Node.js, Express 5, Socket.IO 4, MongoDB, JWT, and HTTP-only cookies.
</p>

![Node.js](https://img.shields.io/badge/Node.js-22%2B-339933?style=flat&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-5-000000?style=flat&logo=express&logoColor=white)
![Socket.IO](https://img.shields.io/badge/Socket.IO-4-010101?style=flat&logo=socketdotio&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Database-47A248?style=flat&logo=mongodb&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-Authentication-000000?style=flat&logo=jsonwebtokens&logoColor=white)
![Arcjet](https://img.shields.io/badge/Arcjet-Security-7C3AED?style=flat&logo=arcjet&logoColor=white)

[Quick Start](#-quick-start) · [API Reference](#-api-reference) · [Authentication](#-authentication) · [Real-time Events](#-real-time-events) · [Security](#-security) · [Environment](#-environment-variables)

---

## 📖 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
- [Environment Variables](#-environment-variables)
- [API Reference](#-api-reference)
- [Authentication](#-authentication)
- [Real-time Events](#-real-time-events)
- [Security](#-security)

## ✨ Features

- 🔐 **Authentication** — email/password signup & login, bcrypt-hashed passwords, JWT issued in an HTTP-only cookie, logout, session check, and profile-picture updates
- ⚡ **Real-time messaging** — one-to-one chat over Socket.IO with token-authenticated sockets; messages are pushed instantly to the receiver's socket
- 🟢 **Online presence** — a live `getOnlineUsers` broadcast tells every client who is currently online
- 👥 **Contacts & chat partners** — list every other registered user, or just the people you've actually chatted with
- 🖼️ **Image messages** — base64 images are uploaded to Cloudinary and delivered as hosted URLs
- 📧 **Welcome emails** — a styled HTML welcome email is sent on signup via Resend
- 🛡️ **API protection** — Arcjet shields every API route from SQLi/XSS attacks, blocks bots, and rate-limits abuse

## 🧰 Tech Stack

| Layer | Technology |
| --- | --- |
| Runtime | Node.js 22 (ES modules) |
| HTTP framework | Express 5 |
| Real-time | Socket.IO 4 |
| Database | MongoDB · Mongoose 9 |
| Auth | JWT · bcryptjs · HTTP-only cookies |
| Security | Arcjet (`@arcjet/node`) |
| Media | Cloudinary |
| Email | Resend |
| Dev tooling | `node --watch` · dotenv |

## 🏗️ Architecture

A single HTTP server hosts both the REST API and the Socket.IO server:

```text
                        ┌──────────────────────────────┐
                        │       http.createServer      │
                        └──────────────┬───────────────┘
                 ┌─────────────────────┴─────────────────────┐
                 ▼                                           ▼
        Express 5 (REST API)                     Socket.IO 4 (WebSocket)
                 │                                           │
  express.json → cookieParser → CORS         socket.auth.middleware
                 │                              (verifies JWT cookie)
                 ▼                                           │
        arcjetProtection                          userSocketMap
  (shield · detectBot · rate limit)             (online presence)
                 │                                           │
                 ▼                                           ▼
           protectRoute                              getOnlineUsers
      (JWT cookie → req.user)                          newMessage
                 │                                           │
                 ▼                                           ▼
      Controllers → Mongoose models        Controllers → Mongoose models
                 │
        ┌────────┼─────────────┐
        ▼        ▼             ▼
    MongoDB   Cloudinary     Resend
   (users &   (profile &    (welcome
   messages)  chat images)    emails)
```

- Every `/api/auth` and `/api/messages` request passes through **Arcjet first, then JWT auth** — so abusive traffic is rate-limited before it ever hits a database lookup.
- Socket connections are authenticated the same way as REST routes: the JWT cookie is verified in `socket.auth.middleware` before a connection is accepted.
- `userSocketMap` (`userId → socketId`) powers both instant message delivery and the online-users broadcast.

## 📁 Project Structure

```text
Chat_Us/
├── backend/
│   ├── package.json
│   └── src/
│       ├── .env                     # secrets (git-ignored)
│       ├── .env.example             # environment template
│       ├── server.js                # entry point: connect DB, mount routes, listen
│       ├── controllers/
│       │   ├── auth.controller.js   # signup, login, logout, update-profile
│       │   └── message.controller.js
│       ├── emails/
│       │   ├── emailHandlers.js     # Resend senders
│       │   └── emailTemplates.js    # welcome email HTML
│       ├── lib/
│       │   ├── arcjet.js            # Arcjet client + shield/bot/rate-limit rules
│       │   ├── cloudinary.js
│       │   ├── db.js                # Mongo connection
│       │   ├── env.js               # env loading (independent of working directory)
│       │   ├── resend.js
│       │   ├── socket.js            # Express app + Socket.IO server + presence map
│       │   └── utils.js             # JWT generation + cookie helper
│       ├── middleware/
│       │   ├── arcjet.middleware.js # Arcjet decisions → 403/429 responses
│       │   ├── auth.middleware.js   # protectRoute (REST)
│       │   └── socket.auth.middleware.js
│       ├── models/
│       │   ├── User.js
│       │   └── message.js
│       └── routes/
│           ├── auth.route.js        # /api/auth
│           └── message.route.js     # /api/messages
├── .gitignore
└── package.json                     # root scripts + dependencies
```

## 🚀 Quick Start

**Prerequisites**

- Node.js **≥ 22.21** (`node --version`)
- A MongoDB database (Atlas cluster or local `mongod`)
- Free accounts: [Cloudinary](https://cloudinary.com) (images), [Resend](https://resend.com) (email), [Arcjet](https://console.arcjet.com) (security)

```bash
# 1. Clone and install
git clone https://github.com/Kofi-Kakah/Chat_Us.git
cd Chat_Us
npm install

# 2. Configure environment
cp backend/src/.env.example backend/src/.env
#    → open backend/src/.env and fill in your values (see table below)

# 3. Run
npm start          # node backend/src/server.js  →  http://localhost:3000
npm run dev        # same, with auto-reload via node --watch
```

`CLIENT_URL` must point at the frontend origin (e.g. `http://localhost:5173` for a Vite client) — it is the only origin allowed by CORS and by the Socket.IO handshake.

## 🔐 Environment Variables

All variables live in `backend/src/.env` (git-ignored) — see `backend/src/.env.example` for a template.

| Variable | Description |
| --- | --- |
| `PORT` | HTTP port (defaults to `3000`) |
| `MONGO_URI` | MongoDB connection string |
| `JWT_SECRET` | Secret used to sign auth tokens |
| `NODE_ENV` | `development` / `production` — controls the auth cookie's `secure` flag |
| `CLIENT_URL` | Frontend origin allowed by CORS (REST + Socket.IO) |
| `RESEND_API_KEY` | Resend API key for the welcome email |
| `EMAIL_FROM` | Verified sender address |
| `EMAIL_FROM_NAME` | Sender display name |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary account name |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |
| `ARCJET_KEY` | Arcjet project key from [console.arcjet.com](https://console.arcjet.com) |
| `ARCJET_ENV` | Arcjet environment (`development` / `production`) |

## 🌐 API Reference

All routes are prefixed with `/api` and protected by **Arcjet** (shield → bot detection → 100 requests/min sliding window).

### Auth — `/api/auth`

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/auth/signup` | — | Create an account (`fullName`, `email`, `password` ≥ 6 chars). Hashes the password, sets the auth cookie, sends a welcome email, returns the user |
| `POST` | `/api/auth/login` | — | Log in (`email`, `password`). Returns the user and sets the auth cookie |
| `POST` | `/api/auth/logout` | — | Clears the auth cookie |
| `PUT` | `/api/auth/update-profile` | ✅ | Update profile picture (`profilePic` as base64) — uploaded to Cloudinary |
| `GET` | `/api/auth/check` | ✅ | Returns the currently authenticated user |

### Messages — `/api/messages`

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/messages/contacts` | ✅ | All registered users except the logged-in user (passwords stripped) |
| `GET` | `/api/messages/chats` | ✅ | Users the logged-in user has actually exchanged messages with |
| `GET` | `/api/messages/:id` | ✅ | Full conversation with user `:id` |
| `POST` | `/api/messages/send/:id` | ✅ | Send a message to user `:id` (`text` and/or `image` as base64). Saves it and emits `newMessage` to the receiver's socket |

**Example — sign up with curl**

```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{"fullName":"Kofi Kakah","email":"kofi@example.com","password":"secret123"}'
```

## 🤝 Authentication

- **Passwords** are hashed with bcrypt (10 salt rounds) — plaintext is never stored, and login failures always return a generic `Invalid credentials`.
- **Tokens** are JWTs signed with `JWT_SECRET` and a **7-day** expiry, delivered in a cookie named `jwt`:
  - `httpOnly` — invisible to JavaScript, mitigating XSS token theft
  - `sameSite: "strict"` — CSRF protection
  - `secure` — enabled automatically outside development
- **REST protection** — `protectRoute` verifies the cookie, loads the user (password stripped), and attaches it to `req.user`.
- **Socket protection** — `socket.auth.middleware` performs the same JWT check on the handshake cookie; unauthenticated sockets are rejected before connecting.

## ⚡ Real-time Events

| Event | Direction | Payload | Description |
| --- | --- | --- | --- |
| `connection` | client → server | — | Handshake authenticated via the `jwt` cookie |
| `getOnlineUsers` | server → client | `string[]` (user IDs) | Emitted whenever someone connects or disconnects |
| `newMessage` | server → client | `Message` | Pushed only to the receiver's socket for instant delivery |
| `disconnect` | client → server | — | Cleans up the presence map |

**Client example**

```js
import { io } from "socket.io-client";

const socket = io("http://localhost:3000", { withCredentials: true });

socket.on("getOnlineUsers", (onlineUserIds) => {
  console.log("Currently online:", onlineUserIds);
});

socket.on("newMessage", (message) => {
  console.log("New message received:", message);
});
```

## 🛡️ Security

**Arcjet protection** (`backend/src/lib/arcjet.js`) is applied to every auth and message route before JWT verification:

| Rule | Mode | Behavior |
| --- | --- | --- |
| `shield` | `LIVE` | Blocks SQL injection, XSS, and other common attacks |
| `detectBot` | `LIVE` | Blocks all automated clients except `CATEGORY:SEARCH_ENGINE` |
| `slidingWindow` | `LIVE` | Max **100 requests / 60 s** per client IP |

**Decision handling** (`arcjet.middleware.js`):

- `429` — `Rate limit exceeded. Please try again later.`
- `403` — `Bot access denied.` / spoofed-bot detection (via `@arcjet/inspect`) / `Access denied by security policy.`
- On Arcjet outages the middleware **fails open** (logs and continues) so availability is never held hostage by the security layer.

**Tips**

- While load-testing or tuning, switch any rule to `mode: "DRY_RUN"` to log decisions without enforcing them.
- In production behind a reverse proxy (Nginx, Render, Railway…), configure the `proxies` option in `arcjet()` so client IPs are resolved correctly — never copy `X-Forwarded-For` manually.

---

<p align="center">
  Made with 💚 by <a href="https://github.com/Kofi-Kakah">Kofi Kakah</a>
</p>