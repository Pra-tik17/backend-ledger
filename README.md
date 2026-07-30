# 🏦 Backend Ledger

A robust, production-grade **double-entry ledger system** built with Node.js, Express, and MongoDB. Designed for reliable financial transactions with ACID guarantees, idempotency protection, and immutable audit trails.

![Node.js](https://img.shields.io/badge/Node.js-ES%20Modules-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-v5-000000?logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?logo=mongodb&logoColor=white)
![License](https://img.shields.io/badge/License-ISC-blue)

---

## ✨ Features

- **Double-Entry Bookkeeping** — Every transaction creates matching DEBIT and CREDIT ledger entries
- **ACID Transactions** — MongoDB sessions ensure atomicity across multi-document operations
- **Idempotency Protection** — Duplicate transactions are safely handled via unique idempotency keys
- **Immutable Ledger** — Ledger entries cannot be modified or deleted once created
- **JWT Authentication** — Secure token-based auth with cookie support
- **Token Blacklisting** — Logged-out tokens are invalidated with auto-expiry (3 days)
- **Role-Based Access** — System user middleware for privileged operations (e.g., initial funding)
- **Email Notifications** — Automated emails on registration and successful transactions via Gmail OAuth2
- **Balance Derivation** — Account balances are derived from ledger aggregation (no stored balance to go stale)

---

## 🏗️ Architecture

```
backend-ledger/
├── server.js                        # Entry point — loads env, starts server
├── package.json                     # Dependencies & scripts (ES Modules)
├── .env                             # Environment variables (not committed)
├── .gitignore
└── src/
    ├── app.js                       # Express app setup, middleware, routes
    ├── config/
    │   └── db.js                    # MongoDB connection
    ├── controllers/
    │   ├── auth.controller.js       # Register, Login, Logout
    │   ├── account.controller.js    # Create account, Get accounts, Get balance
    │   └── transaction.controller.js # Transfer funds, Initial funding
    ├── middleware/
    │   └── auth.middleware.js       # JWT auth & system user verification
    ├── models/
    │   ├── user.model.js            # User schema with password hashing
    │   ├── account.model.js         # Account schema with balance derivation
    │   ├── ledger.model.js          # Immutable ledger entries (CREDIT/DEBIT)
    │   ├── transaction.model.js     # Transaction with status tracking
    │   └── blackList.model.js       # Token blacklist with TTL auto-expiry
    ├── routes/
    │   ├── auth.routes.js           # /api/auth/*
    │   ├── account.routes.js        # /api/accounts/*
    │   └── transaction.routes.js    # /api/transactions/*
    └── services/
        └── email.service.js         # Gmail OAuth2 email notifications
```

---

## 🔧 Tech Stack

| Technology | Purpose |
|------------|---------|
| **Node.js** (ES Modules) | Runtime |
| **Express v5** | Web framework |
| **MongoDB + Mongoose** | Database & ODM |
| **JSON Web Tokens** | Authentication |
| **bcryptjs** | Password hashing |
| **nodemailer** | Email notifications (Gmail OAuth2) |
| **dotenv** | Environment configuration |
| **cookie-parser** | Cookie-based token handling |

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (v18+)
- [MongoDB Atlas](https://www.mongodb.com/atlas) account (or local MongoDB)
- [Google Cloud Console](https://console.cloud.google.com/) project (for email notifications)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/backend-ledger.git
cd backend-ledger

# Install dependencies
npm install
```

### Environment Setup

Create a `.env` file in the project root:

```env
MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/backend-ledger
JWT_SECRET=your_super_secret_jwt_key

# Email (Gmail OAuth2) — optional, for email notifications
EMAIL_USER=your_email@gmail.com
CLIENT_ID=your_google_client_id.apps.googleusercontent.com
CLIENT_SECRET=GOCSPX-your_client_secret
REFRESH_TOKEN=1//your_refresh_token
```

> **Note:** Email configuration is optional. The server will run without it, but email notifications will fail silently.

### Run the Server

```bash
# Development (with auto-restart)
npm run dev

# Production
npm start
```

Server starts at `http://localhost:3000`

---

## 📡 API Reference

### Health Check

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/` | ❌ | Service health check |

**Response:**
```
Ledger Service is up and running
```

---

### 🔐 Authentication

#### Register

```
POST /api/auth/register
```

**Request Body:**
```json
{
    "name": "Pratik Pradhan",
    "email": "pratik@example.com",
    "password": "password123"
}
```

**Success Response (201):**
```json
{
    "user": {
        "_id": "6789abcdef...",
        "email": "pratik@example.com",
        "name": "Pratik Pradhan"
    },
    "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Responses:**
| Status | Message |
|--------|---------|
| 422 | `User already exists with email.` |
| 500 | Mongoose validation error (invalid email, short password, etc.) |

---

#### Login

```
POST /api/auth/login
```

**Request Body:**
```json
{
    "email": "pratik@example.com",
    "password": "password123"
}
```

**Success Response (200):**
```json
{
    "user": {
        "_id": "6789abcdef...",
        "email": "pratik@example.com",
        "name": "Pratik Pradhan"
    },
    "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Error Responses:**
| Status | Message |
|--------|---------|
| 401 | `Email or password is INVALID` |

---

#### Logout

```
POST /api/auth/logout
Authorization: Bearer <token>
```

**Success Response (200):**
```json
{
    "message": "User logged out successfully"
}
```

> The token is blacklisted and cannot be reused. Blacklisted tokens auto-expire after 3 days.

---

### 🏦 Accounts

> All account endpoints require `Authorization: Bearer <token>` header.

#### Create Account

```
POST /api/accounts/
Authorization: Bearer <token>
```

No request body needed. Account is linked to the authenticated user.

**Success Response (201):**
```json
{
    "account": {
        "_id": "66aa1234...",
        "user": "6789abcdef...",
        "status": "ACTIVE",
        "currency": "INR",
        "createdAt": "2026-07-30T14:30:00.000Z",
        "updatedAt": "2026-07-30T14:30:00.000Z"
    }
}
```

---

#### Get All Accounts

```
GET /api/accounts/
Authorization: Bearer <token>
```

**Success Response (200):**
```json
{
    "accounts": [
        {
            "_id": "66aa1234...",
            "user": "6789abcdef...",
            "status": "ACTIVE",
            "currency": "INR"
        }
    ]
}
```

---

#### Get Account Balance

```
GET /api/accounts/balance/:accountId
Authorization: Bearer <token>
```

**Success Response (200):**
```json
{
    "accountId": "66aa1234...",
    "balance": 10000
}
```

**Error Responses:**
| Status | Message |
|--------|---------|
| 404 | `Account not found` |

> Balance is derived in real-time from ledger entries using MongoDB aggregation (totalCredits - totalDebits).

---

### 💰 Transactions

> All transaction endpoints require `Authorization: Bearer <token>` header.

#### Transfer Funds

```
POST /api/transactions/
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
    "fromAccount": "<sender_account_id>",
    "toAccount": "<receiver_account_id>",
    "amount": 2000,
    "idempotencyKey": "txn-unique-key-001"
}
```

**Success Response (201):**
```json
{
    "message": "Transaction completed successfully",
    "transaction": {
        "_id": "66aa4444...",
        "fromAccount": "...",
        "toAccount": "...",
        "amount": 2000,
        "status": "COMPLETED",
        "idempotencyKey": "txn-unique-key-001"
    }
}
```

**Error Responses:**
| Status | Message |
|--------|---------|
| 400 | `FromAccount, toAccount, amount and idempotencyKey are required` |
| 400 | `Invalid fromAccount or toAccount` |
| 400 | `Insufficient balance. Current balance is X. Requested amount is Y` |
| 400 | `Both fromAccount and toAccount must be ACTIVE to process transaction` |
| 200 | `Transaction already processed` (duplicate idempotency key) |
| 200 | `Transaction is still processing` (pending duplicate) |

**The 10-Step Transfer Flow:**
1. Validate request fields
2. Validate idempotency key (prevent duplicates)
3. Check both accounts are ACTIVE
4. Derive sender balance from ledger
5. Create transaction record (PENDING)
6. Create DEBIT ledger entry (sender)
7. Create CREDIT ledger entry (receiver)
8. Mark transaction COMPLETED
9. Commit MongoDB session
10. Send email notification

---

#### Add Initial Funds (System User Only)

```
POST /api/transactions/system/initial-funds
Authorization: Bearer <system_user_token>
Content-Type: application/json
```

**Request Body:**
```json
{
    "toAccount": "<receiver_account_id>",
    "amount": 10000,
    "idempotencyKey": "init-funds-001"
}
```

**Success Response (201):**
```json
{
    "message": "Initial funds transaction completed successfully",
    "transaction": {
        "_id": "...",
        "fromAccount": "<system_user_account>",
        "toAccount": "...",
        "amount": 10000,
        "status": "COMPLETED"
    }
}
```

**Error Responses:**
| Status | Message |
|--------|---------|
| 403 | `Forbidden access, not a system user` |
| 400 | `toAccount, amount and idempotencyKey are required` |
| 400 | `Invalid toAccount` |
| 400 | `System user account not found` |

> **Setting up a System User:** Directly update the user document in MongoDB:
> ```javascript
> db.users.updateOne(
>     { email: "admin@example.com" },
>     { $set: { systemUser: true } }
> )
> ```

---

## 🔒 Authentication & Authorization

### Token Flow

```
Register/Login → Receive JWT token (3-day expiry)
    ↓
Use token in requests → Authorization: Bearer <token>
    ↓
Logout → Token is blacklisted (auto-expires in 3 days)
```

### Middleware

| Middleware | Protects | Description |
|-----------|----------|-------------|
| `authMiddleware` | Account & Transaction routes | Validates JWT, checks blacklist |
| `authSystemUserMiddleware` | `/system/initial-funds` | Same as above + checks `systemUser: true` |

---

## 📊 Data Models

### User
| Field | Type | Details |
|-------|------|---------|
| email | String | Required, unique, lowercase, validated |
| name | String | Required |
| password | String | Required, min 6 chars, hashed with bcrypt, select: false |
| systemUser | Boolean | Default: false, immutable, select: false |

### Account
| Field | Type | Details |
|-------|------|---------|
| user | ObjectId | Ref: User, required, indexed |
| status | String | Enum: ACTIVE, FROZEN, CLOSED |
| currency | String | Default: INR |

### Ledger (Immutable)
| Field | Type | Details |
|-------|------|---------|
| account | ObjectId | Ref: Account, immutable |
| amount | Number | Required, immutable |
| transaction | ObjectId | Ref: Transaction, immutable |
| type | String | Enum: CREDIT, DEBIT, immutable |

### Transaction
| Field | Type | Details |
|-------|------|---------|
| fromAccount | ObjectId | Ref: Account, required |
| toAccount | ObjectId | Ref: Account, required |
| amount | Number | Required, min: 0 |
| status | String | Enum: PENDING, COMPLETED, FAILED, REVERSED |
| idempotencyKey | String | Required, unique |

### Token Blacklist
| Field | Type | Details |
|-------|------|---------|
| token | String | Required, unique |
| createdAt | Date | TTL index: auto-deletes after 3 days |

---

## 🛡️ Security Features

- **Password Hashing** — bcrypt with salt rounds of 10
- **JWT Tokens** — 3-day expiry, stored in cookies + header support
- **Token Blacklisting** — Prevents reuse of logged-out tokens
- **Immutable Ledger** — Pre-hooks block all update/delete operations on ledger entries
- **Idempotency Keys** — Prevents duplicate transaction processing
- **Field Selection** — Password and systemUser fields excluded from queries by default
- **Input Validation** — Mongoose schema validation on all models

---

## 📜 Scripts

```bash
npm start       # Start server with Node.js
npm run dev     # Start server with Nodemon (auto-restart)
npm test        # Run tests (not configured yet)
```

---

## 📝 License

This project is licensed under the **ISC License**.

---

<p align="center">
  Built with ❤️ by <strong>Pratik Pradhan</strong>
</p>
