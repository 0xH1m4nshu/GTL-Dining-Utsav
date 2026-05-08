# AI-Driven Smart Restaurant Web Application

A full-stack restaurant management platform with a customer-facing web app and role-based staff panels, built with **FastAPI** (backend) and **React + Vite** (frontend).

---

## Features

**Customer-facing**
- Browse the menu, view categories, and add items to cart
- Book tables online with automatic 30-minute timeout/cancellation
- Place online orders and track them in real time
- Checkout with UPI payment support
- Sign up / log in (including Google OAuth)
- AI-powered chatbot assistant (Gemini)
- User profile management

**Staff panels** (role-gated)
- **Admin** — full dashboard: users, menu, categories, inventory, orders, tables, reports
- **Chef** — view and update incoming kitchen orders
- **Waiter** — manage table orders and new order creation
- **Receptionist** — table status management and billing
- **Delivery** — view and fulfill delivery orders

**Real-time**
- WebSocket-based live order updates across all staff panels

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.12, FastAPI, Motor (async MongoDB), Pydantic v2 |
| Database | MongoDB |
| Auth | JWT (access + refresh tokens), Google OAuth, TOTP/OTP |
| AI | Google Gemini (`google-generativeai`) |
| Frontend | React 18, Vite 5, React Router v6, Zustand, Recharts, Axios |
| Styling | Custom CSS (design tokens, components, panel) |
| Build | Vite with separate `user` and `staff` modes |

---

## Project Structure

```
AI-Driven Smart Restaurant/
├── backend/
│   ├── main.py               # FastAPI app entry point
│   ├── config.py             # Settings (pydantic-settings)
│   ├── database.py           # MongoDB connection
│   ├── auth/                 # JWT + Google auth
│   ├── models/               # MongoDB document models
│   │   ├── user.py
│   │   ├── order.py
│   │   ├── menu_item.py
│   │   ├── category.py
│   │   ├── inventory.py
│   │   ├── table.py
│   │   ├── payment.py
│   │   └── notification.py
│   ├── routers/              # API route handlers
│   │   ├── admin.py
│   │   ├── user.py
│   │   ├── orders.py
│   │   ├── chef.py
│   │   ├── waiter.py
│   │   ├── receptionist.py
│   │   ├── delivery.py
│   │   ├── ws.py             # WebSocket
│   │   └── ai.py             # Gemini chatbot
│   ├── services/             # Business logic
│   │   ├── order_service.py
│   │   ├── payment_service.py
│   │   ├── inventory_service.py
│   │   ├── notification_service.py
│   │   ├── email_service.py
│   │   ├── otp_service.py
│   │   ├── export_service.py
│   │   └── gemini_service.py
│   ├── utils/
│   │   ├── gst.py
│   │   ├── qr_generator.py
│   │   ├── password.py
│   │   └── object_id.py
│   ├── seed.py               # Database seeder
│   └── requirements.txt
└── frontend/
    ├── src/
    │   ├── App.jsx           # Routes & app shell
    │   ├── main.jsx
    │   ├── pages/
    │   │   ├── public/       # Customer-facing pages
    │   │   ├── admin/
    │   │   ├── chef/
    │   │   ├── waiter/
    │   │   ├── receptionist/
    │   │   └── delivery/
    │   ├── components/
    │   │   ├── public/       # Navbar, footer, chatbot, menu card
    │   │   ├── shared/       # Protected routes, modals, badges
    │   │   └── decorative/   # Visual elements
    │   ├── stores/           # Zustand state (auth, orders, tables, notifications)
    │   ├── context/          # CartContext
    │   ├── hooks/            # useRole, useToast, useWebSocket
    │   ├── config/           # API base URL, roles
    │   └── assets/css/       # Design tokens, components, panel styles
    ├── vite.config.js
    └── package.json
```

---

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+
- MongoDB (local or Atlas)

### Backend Setup

```bash
cd backend

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your MongoDB URI, JWT secret, Gemini API key, etc.

# Seed the database (optional)
python seed.py

# Start the server
uvicorn main:app --reload
```

The API will be available at `http://localhost:8000`. Interactive docs at `http://localhost:8000/docs`.

### Frontend Setup

```bash
cd frontend
npm install

# Run both user & staff in a single instance
npm run dev

# Or run separately by mode
npm run dev:user    # customer app only
npm run dev:staff   # staff panels only
```

The app will be available at `http://localhost:5173`.

---

## Environment Variables

Copy `backend/.env.example` to `backend/.env` and fill in:

| Variable | Description |
|---|---|
| `MONGODB_URL` | MongoDB connection string |
| `MONGODB_DB` | Database name |
| `JWT_SECRET` | Secret key for signing JWTs |
| `JWT_ALGORITHM` | Signing algorithm (default: `HS256`) |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Access token TTL |
| `REFRESH_TOKEN_EXPIRE_DAYS` | Refresh token TTL |
| `ENV` | `development` or `production` |
| `ALLOWED_ORIGINS` | Comma-separated allowed CORS origins (production) |
| `UPI_VPA` | UPI payment address |
| `UPI_NAME` | UPI display name |

Additional keys needed for full functionality (add to `.env`):
- `GEMINI_API_KEY` — for the AI chatbot
- `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` — for Google OAuth
- SMTP credentials — for email/OTP services

---

## API Overview

The backend exposes the following route groups:

| Prefix | Description |
|---|---|
| `/auth` | Login, signup, token refresh, Google OAuth, OTP |
| `/admin` | CRUD for users, menu, categories, inventory, tables, reports |
| `/user` | Customer profile, bookings, order history |
| `/orders` | Place and track orders |
| `/chef` | Kitchen order queue |
| `/waiter` | Table order management |
| `/reception` | Table status, billing |
| `/delivery` | Delivery queue |
| `/ws` | WebSocket connection for live updates |
| `/ai` | Gemini chatbot endpoint |
| `/health` | Health check |

---

## Building for Production

```bash
# Backend — run behind a production ASGI server
uvicorn main:app --host 0.0.0.0 --port 8000

# Frontend — build both modes
npm run build:user    # outputs customer app
npm run build:staff   # outputs staff app
```

Set `ENV=production` and configure `ALLOWED_ORIGINS` in `.env` before deploying.

---

## Roles

| Role | Panel |
|---|---|
| `super_admin` | `/admin/*` (full access) |
| `chef` | `/chef/*` |
| `waiter` | `/waiter/*` |
| `receptionist` | `/reception/*` |
| `delivery` | `/delivery/*` |
| *(customer)* | Public-facing routes |
