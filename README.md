# 🅿️ UrbanParking App

<div align="center">

![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Express-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/Neon-PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FCM-FFCA28?style=for-the-badge&logo=firebase&logoColor=black)
![Socket.io](https://img.shields.io/badge/Socket.io-WebSocket-010101?style=for-the-badge&logo=socket.io&logoColor=white)

**A smart, real-time urban parking management system with IoT integration, AI-powered slot allocation, and push notifications.**

</div>

---

## 📱 Screenshots

> Flutter app running on Android (API 36)

| Splash | Parking List | Slot Selection | My Bookings |
|--------|-------------|---------------|-------------|
| Smart UI | Live availability | Cost-optimized slots | QR entry/exit |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  Flutter App (Android)               │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────┐ │
│  │  Screens │ │ Services │ │  Firebase Messaging   │ │
│  └──────────┘ └──────────┘ └──────────────────────┘ │
└───────────────────────┬─────────────────────────────┘
                        │ REST API + WebSocket
┌───────────────────────▼─────────────────────────────┐
│              Express.js Backend (Node.js)            │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────────┐ │
│  │  Routes  │ │ Socket.io│ │  Firebase Admin SDK   │ │
│  └──────────┘ └──────────┘ └──────────────────────┘ │
│  ┌──────────────────────────────────────────────────┐ │
│  │              Prisma ORM                          │ │
│  └──────────────────────────────────────────────────┘ │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│                 Neon DB (PostgreSQL)                  │
│  Users • ParkingArea • ParkingSlot • Bookings        │
└─────────────────────────────────────────────────────┘
```

---

## ✨ Features

### 🧠 Smart Slot Allocation (Cost Function Optimizer)
The backend uses a normalized multi-factor cost function to assign the optimal parking slot:

```
C(i) = W1 × distance_score(i) + W2 × spread_score(i) + W3 × ev_penalty(i)

Weights:  W1 = 0.5 (proximity)
          W2 = 0.3 (distribution / anti-cluster)
          W3 = 1.0 (EV hard-preference)
```

- **Distance Score** — slots closer to the entrance are preferred
- **Spread Score** — avoids placing users in already-clustered rows
- **EV Penalty** — EV-dedicated slots are blocked for non-EV vehicles

### 🔴 Real-Time Slot Grid (Socket.io)
- Live WebSocket events update the parking grid instantly for all connected clients
- Events: `slot_updated`, `booking_reassigned`, `device_heartbeat`
- Arena Manager screen shows a live colour-coded grid

### 📡 IoT Edge Integration
- **Silent Auto-Binding** — YOLOv8 edge node sends vehicle plate + detected slot; backend silently reassigns booking if the actual slot is free
- **Stationary Swap** — If a vehicle is stationary and unresponsive after 5 min timeout, an atomic Prisma transaction penalises the offender (+₹50) and reassigns the next user to a free slot, emitting a private socket event

### 🔔 Firebase Cloud Messaging (FCM)
- Push notifications on: booking confirmation, gate scan (entry/exit), slot reallocation, broadcast alerts
- Background & foreground message handlers in Flutter
- FCM token auto-synced to backend on login
- Topic subscriptions (`slot_alerts`)

### 💳 Payments
- Razorpay payment gateway integration
- Order creation, payment verification (HMAC-SHA256 signature), refund handling

### 🔐 Authentication
- Email/password with bcrypt
- Google OAuth 2.0
- JWT tokens (httpOnly cookies)
- Guest mode

### 📋 Booking Lifecycle
```
PENDING → CONFIRMED → [ACTIVE] → COMPLETED
                   ↓
            CANCELLED / EXPIRED / WRONG_SLOT_FLAG / STATIONARY_UNRESPONSIVE
```
- QR code generation for entry/exit gate scan
- Countdown timer until booking starts
- Auto-expiry for unpaid bookings

### 🤖 Parky AI Chatbot
- In-app AI assistant for parking queries

---

## 🗂️ Project Structure

```
urbanparking_app/
├── lib/
│   ├── main.dart                        # App entry + Firebase init
│   ├── screens/
│   │   ├── splash_screen.dart
│   │   ├── landing_screen.dart          # Home dashboard
│   │   ├── auth_screen.dart             # Login / Register / Google OAuth
│   │   ├── parking_list_screen.dart     # Browse parking areas
│   │   ├── parking_details_screen.dart  # Slot availability + booking
│   │   ├── slot_selection_screen.dart   # Cost-optimized slot picker
│   │   ├── booking_confirmation_screen.dart
│   │   ├── booking_success_screen.dart
│   │   ├── my_bookings_screen.dart      # Active/Upcoming/Past + QR scan
│   │   ├── arena_manager_screen.dart    # Live grid for arena managers
│   │   └── chat_parky_screen.dart       # AI chatbot
│   ├── services/
│   │   ├── user_session.dart            # Auth state + FCM token sync
│   │   ├── notification_service.dart    # Firebase Messaging service
│   │   ├── slot_allocation_service.dart # Cost function client
│   │   └── location_service.dart        # GPS location
│   └── widgets/
│       └── live_arena_grid.dart         # Real-time socket slot grid
│
├── backend/
│   ├── index.js                         # Express server + all API routes
│   ├── socketManager.js                 # Socket.io singleton manager
│   ├── services/
│   │   └── fcmService.js               # Firebase Admin push dispatcher
│   ├── prisma/
│   │   └── schema.prisma               # Database schema
│   ├── tests/
│   │   ├── parkingAllocation.test.js   # QA: Cost fn + IoT + Swap
│   │   ├── conflict_swap.test.js       # Atomic swap unit tests
│   │   └── socket_test.js             # WebSocket integration tests
│   ├── package.json
│   └── serviceAccountKey.json          # Firebase Admin credentials
│
├── android/
│   └── app/
│       ├── google-services.json         # Firebase Android config
│       └── build.gradle.kts
└── pubspec.yaml
```

---

## 🚀 Getting Started

### Prerequisites

| Tool | Version |
|------|---------|
| Flutter | 3.x |
| Dart | 3.x |
| Node.js | 18+ |
| npm | 9+ |
| Android Studio / VS Code | Latest |

---

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/vishal-balasubramani/urbanparking_app.git
cd urbanparking_app
```

---

### 2️⃣ Backend Setup

```bash
cd backend
npm install
```

Create a `.env` file in `backend/`:

```env
DATABASE_URL=postgresql://<user>:<password>@<host>/urbanparking?sslmode=require
JWT_SECRET=your-super-secret-jwt-key
RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=your_razorpay_secret
GOOGLE_CLIENT_ID=your-google-oauth-client-id.apps.googleusercontent.com
PORT=3000
```

> **Neon DB**: Create a free project at [neon.tech](https://neon.tech) and paste the connection string above.

Run Prisma migrations:

```bash
npx prisma generate
npx prisma db push
```

Start the backend server:

```bash
node index.js
```

The server starts at `http://localhost:3000` with WebSocket on the same port.

---

### 3️⃣ Flutter Setup

```bash
cd ..   # back to project root
flutter pub get
```

Update `lib/services/user_session.dart` — set your backend IP:
```dart
static const String baseUrl = 'http://YOUR_LOCAL_IP:3000';
```

> Use your machine's **local network IP** (e.g. `192.168.1.10`) so your Android device can reach the backend.

Run the app:

```bash
flutter run
```

---

### 4️⃣ Firebase Setup

1. Create a Firebase project at [console.firebase.google.com](https://console.firebase.google.com)
2. Add an Android app with your package name
3. Download `google-services.json` → place it in `android/app/`
4. Enable **Cloud Messaging** in Firebase console
5. Generate a service account key → save as `backend/serviceAccountKey.json`

---

## 🧪 Running Tests

```bash
cd backend
npx jest tests/parkingAllocation.test.js --runInBand --forceExit --verbose
```

### Test Suite Results (3/3 passing ✅)

```
PASS  tests/parkingAllocation.test.js

  Smart Slot Allocation & Edge IoT Resolution Test Suite

    1. The Cost Function Optimizer
       ✓ should assign optimal slot (min cost) with EV-01 excluded for non-EV user
    2. Silent Auto-Binding (IoT Webhook Simulation)
       ✓ should update booking slotId via YOLOv8 edge detection payload
    3. The Stationary Swap Atomic Transaction
       ✓ should penalize offender (+₹50) and atomically reassign victim booking

Tests: 3 passed, 3 total  |  Time: ~16s
```

---

## 📡 API Reference

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register with email/password |
| POST | `/api/auth/login` | Login → JWT cookie |
| POST | `/api/auth/google` | Google OAuth login |
| POST | `/api/auth/logout` | Clear session |

### Parking
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/parking-areas` | List all parking areas |
| GET | `/api/parking-areas/:id` | Get area details + slot availability |
| POST | `/api/parking-areas/:id/allocate-slot` | 🧠 Smart slot allocation (cost fn) |

### Bookings
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/bookings` | Create booking |
| GET | `/api/bookings/user/:userId` | Get user's bookings |
| PUT | `/api/bookings/:id/cancel` | Cancel booking |
| POST | `/api/bookings/:id/scan-gate` | QR gate scan (entry/exit) |

### Payments
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/payments/create-order` | Create Razorpay order |
| POST | `/api/payments/verify` | Verify payment signature |
| POST | `/api/payments/refund` | Process refund |

### IoT / Edge
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/iot/webhook` | YOLOv8 vehicle detection → silent auto-bind |
| POST | `/api/iot/stationary-unresponsive` | Stationary vehicle → atomic penalty + swap |

### Notifications
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/users/:userId/fcm-token` | Sync device FCM token |
| POST | `/api/alerts/broadcast` | Broadcast push to topic |

---

## 🗃️ Database Schema

```prisma
model User {
  id        Int       @id @default(autoincrement())
  name      String
  email     String    @unique
  phone     String    @unique
  password  String?
  googleId  String?
  fcmToken  String?           // ← FCM device token
  bookings  Booking[]
}

model ParkingArea {
  id            Int           @id @default(autoincrement())
  name          String
  city          String
  totalSlots    Int
  price_per_hour Float
  features      String[]
  slots         ParkingSlot[]
}

model ParkingSlot {
  id          Int          @id @default(autoincrement())
  parkingId   Int
  slotNumber  String
  parkingArea ParkingArea  @relation(...)
  bookings    Booking[]
}

model Booking {
  id              Int           @id @default(autoincrement())
  userId          Int
  slotId          Int
  startTime       DateTime
  endTime         DateTime
  bookingStatus   BookingStatus  // PENDING | CONFIRMED | CANCELLED | WRONG_SLOT_FLAG | STATIONARY_UNRESPONSIVE
  paymentStatus   PaymentStatus  // PENDING | PAID | REFUNDED
  amount          Float
  penaltyFee      Float?
  entryScanned    Boolean       @default(false)
  exitScanned     Boolean       @default(false)
  notes           String?
}
```

---

## 🔌 WebSocket Events

| Event | Direction | Payload |
|-------|-----------|---------|
| `slot_updated` | Server → Client | `{ areaId, slotId, status, slotNumber }` |
| `booking_reassigned` | Server → User room | `{ booking }` |
| `device_heartbeat` | Server → Broadcast | `{ deviceId, timestamp }` |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Mobile App | Flutter 3.x / Dart |
| Backend | Node.js + Express.js |
| Database | PostgreSQL via Neon DB |
| ORM | Prisma |
| Real-time | Socket.io (WebSocket) |
| Push Notifications | Firebase Cloud Messaging (FCM) |
| Payments | Razorpay |
| Auth | JWT + Google OAuth 2.0 |
| Testing | Jest + Supertest |
| Maps | Google Maps Flutter |

---

## 🔐 Environment Variables

### Backend (`backend/.env`)
```env
DATABASE_URL=         # Neon PostgreSQL connection string
JWT_SECRET=           # Secret for JWT signing
RAZORPAY_KEY_ID=      # Razorpay test/live key
RAZORPAY_KEY_SECRET=  # Razorpay secret
GOOGLE_CLIENT_ID=     # Google OAuth client ID
PORT=3000
```

> ⚠️ Never commit `.env` or `serviceAccountKey.json` to public repos.

---

## 📦 Dependencies

### Flutter (`pubspec.yaml`)
```yaml
dependencies:
  flutter_map, google_maps_flutter  # Maps
  firebase_core, firebase_messaging  # FCM
  flutter_local_notifications        # Local alerts
  razorpay_flutter                   # Payments
  qr_flutter                         # QR code generation
  mobile_scanner                     # QR scanner
  socket_io_client                   # WebSocket client
  http, shared_preferences, provider # Core utilities
  google_sign_in                     # Google OAuth
```

### Backend (`package.json`)
```json
{
  "express": "^4.x",
  "prisma": "^5.x",
  "@prisma/client": "^5.x",
  "socket.io": "^4.x",
  "firebase-admin": "^12.x",
  "jsonwebtoken": "^9.x",
  "razorpay": "^2.x",
  "bcryptjs": "^2.x",
  "jest": "^29.x",
  "supertest": "^6.x"
}
```

---

## 🤝 Contributing

1. Fork the repo
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit changes: `git commit -m 'feat: add my feature'`
4. Push to branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License.

---

<div align="center">
Made with ❤️ by <a href="https://github.com/MANIKANTA-UPPU/smart-urban-parking-system">uppu manikanta</a>
</div>
