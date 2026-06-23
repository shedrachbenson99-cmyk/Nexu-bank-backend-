# 🏦 NexusBank Backend API

Full production-ready backend for NexusBank built with:
- **Node.js + Express** — REST API
- **PostgreSQL** — Database
- **JWT** — Authentication
- **bcrypt** — Password hashing
- **Claude API** — AI Advisor (server-side, key never exposed)

---

## 📁 Project Structure

```
nexusbank-backend/
├── src/
│   ├── server.js                    ← Entry point
│   ├── config/
│   │   ├── db.js                    ← PostgreSQL connection
│   │   └── initDb.js                ← Create tables
│   ├── middleware/
│   │   └── auth.js                  ← JWT protect middleware
│   ├── controllers/
│   │   ├── authController.js        ← Register, login, profile
│   │   ├── accountsController.js    ← Open/manage accounts
│   │   ├── transactionsController.js← Deposit, withdraw, history
│   │   ├── transfersController.js   ← Send money, FX rates
│   │   ├── notificationsController.js
│   │   └── advisorController.js     ← Claude AI (server-side)
│   └── routes/
│       └── index.js                 ← All API routes
├── .env.example                     ← Environment template
├── package.json
└── README.md
```

---

## 🚀 Local Setup

### 1. Prerequisites
- Node.js 18+ → [nodejs.org](https://nodejs.org)
- PostgreSQL 14+ → [postgresql.org](https://www.postgresql.org)

### 2. Clone & Install
```bash
# Copy the backend folder to your machine, then:
cd nexusbank-backend
npm install
```

### 3. Environment Variables
```bash
cp .env.example .env
```
Edit `.env` and fill in:
```env
DATABASE_URL=postgresql://postgres:yourpassword@localhost:5432/nexusbank
JWT_SECRET=your_long_random_secret_string
ANTHROPIC_API_KEY=your_key_from_console.anthropic.com
FRONTEND_URL=http://localhost:5500
```

### 4. Create PostgreSQL Database
```bash
# In psql or pgAdmin:
CREATE DATABASE nexusbank;
```

### 5. Initialize Tables
```bash
npm run db:init
```

### 6. Start the Server
```bash
# Development (auto-restart on changes)
npm run dev

# Production
npm start
```

Server runs at: **http://localhost:5000**

---

## 🌐 API Endpoints

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Create new user + account |
| POST | `/api/auth/login` | Login, get JWT token |
| GET | `/api/auth/profile` | Get current user + accounts |
| PUT | `/api/auth/profile` | Update name, phone |
| PUT | `/api/auth/change-password` | Change password |

### Accounts
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/accounts` | All user accounts + total balance |
| GET | `/api/accounts/:id` | Single account details |
| POST | `/api/accounts` | Open new account |
| DELETE | `/api/accounts/:id` | Close account |

### Transactions
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/transactions` | Full transaction history |
| GET | `/api/transactions/:id` | Single transaction |
| POST | `/api/transactions/deposit` | Deposit to account |
| POST | `/api/transactions/withdraw` | Withdraw from account |

### Transfers
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/transfers` | Transfer history |
| GET | `/api/transfers/fx-rates` | Live FX rates |
| POST | `/api/transfers` | Send money |
| GET | `/api/transfers/:id` | Transfer status |

### Notifications
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/notifications` | All notifications |
| PUT | `/api/notifications/read-all` | Mark all read |
| PUT | `/api/notifications/:id/read` | Mark one read |
| DELETE | `/api/notifications/:id` | Delete one |

### AI Advisor
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/advisor/chat` | Chat with Claude (server-side) |

---

## 🔗 Connecting the Frontend

In your `nexusbank-final.html`, replace the direct Claude API calls with calls to your backend.

**Example — Login:**
```javascript
const res = await fetch('http://localhost:5000/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});
const data = await res.json();
localStorage.setItem('nxToken', data.token); // Save JWT
```

**Example — Authenticated request:**
```javascript
const token = localStorage.getItem('nxToken');
const res = await fetch('http://localhost:5000/api/accounts', {
  headers: { 'Authorization': `Bearer ${token}` }
});
const data = await res.json();
// data.accounts = array of user's accounts
```

**Example — AI Advisor (now secure):**
```javascript
const res = await fetch('http://localhost:5000/api/advisor/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({ messages: chatHistory })
});
const data = await res.json();
// data.reply = Claude's response
```

---

## ☁️ Free Deployment

### Railway (Recommended — Free tier)
1. Go to [railway.app](https://railway.app) → New Project
2. Click **Deploy from GitHub** (push your backend folder to GitHub first)
3. Add a **PostgreSQL** plugin in Railway → it auto-sets `DATABASE_URL`
4. Add environment variables in Railway dashboard
5. Your API goes live at `https://nexusbank-backend.up.railway.app`

### Render (Alternative — Free tier)
1. Go to [render.com](https://render.com) → New Web Service
2. Connect your GitHub repo
3. Build command: `npm install`
4. Start command: `npm start`
5. Add environment variables
6. Add a **PostgreSQL** database under Render databases

---

## 🔒 Security Features
- Passwords hashed with **bcrypt (12 rounds)**
- **JWT tokens** expire in 7 days
- **Rate limiting** — 100 req/15min general, 10 req/15min on auth
- **Helmet.js** — sets secure HTTP headers
- **CORS** — only allows your frontend URL
- **Claude API key** stays on server, never sent to browser
- Database queries use **parameterized statements** (no SQL injection)

---

## 📝 Sample API Request (Register)

```bash
curl -X POST http://localhost:5000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Alexander",
    "last_name": "Mercer",
    "email": "alex@nexusbank.com",
    "phone": "+1 212 555 0148",
    "password": "SecurePass123!",
    "account_type": "current"
  }'
```

Response:
```json
{
  "message": "Account created successfully!",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6...",
  "user": {
    "id": "uuid-here",
    "first_name": "Alexander",
    "last_name": "Mercer",
    "email": "alex@nexusbank.com",
    "tier": "standard"
  },
  "account": {
    "id": "uuid-here",
    "account_type": "current",
    "account_number": "NX12345678",
    "balance": "0.00"
  }
}
```
