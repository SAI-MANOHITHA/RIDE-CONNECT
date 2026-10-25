# RideConnect – Smart Community Ride Sharing 🚗🎓

> **Cut waiting times, slash commuting costs, and reduce traffic congestion by connecting neighbors and college students traveling along the same route.**

---

## 📌 Problem & Community Context
In bustling urban corridors, students and office commuters often spend **30 to 60 minutes waiting in long queues** for crowded buses, autos, or expensive on-demand cabs. Simultaneously, hundreds of private drivers and fellow students travel along the exact same arterial corridors every morning with **2 to 3 empty seats** in their vehicles.

## 💡 The Solution: RideConnect
**RideConnect** is a responsive, community-driven web platform that pairs passengers with nearby verified drivers traveling along similar routes. It enables seamless coordination for shared rides, ensures safety through multi-level identity verification, fosters two-way agreement before confirming any trip, and provides multimodal public transit fallbacks when no driver is available.

---

## ✨ Core Features & Implementation

| Feature | Description | Implementation Details |
|---|---|---|
| **1. Dual Flows: Find & Offer** | Switch effortlessly between seeking a ride as a passenger or offering empty seats as a driver. | `view-find-ride` and `view-offer-ride` tabs with real-time route input and validation. |
| **2. Smart Matching Algorithm** | Evaluates spatial corridor proximity, route deviation, and departure time delta to calculate a match score (0–100%). | Haversine formula + point-to-segment projection in `js/matching.js`. |
| **3. Realistic Ride Cards** | Shows driver avatar, vehicle model, verified badges, rating (★), departure time, estimated pickup ETA, and fuel share contribution. | Dynamic card rendering with high-match badges and pedestrian walk times. |
| **4. Two-Way Agreement Workflow** | Requires mutual consent: passenger sends request ➔ driver reviews & accepts/declines ➔ seats update automatically. | Managed via `AppState.requests` state machine with instant UI toast alerts. |
| **5. Safety, Verification & SOS** | Student ID (`.edu`), Govt ID, and Phone OTP badges. SOS modal with 1-tap 112 dialing and emergency contacts broadcast. | Stored in `AppState.currentUser.emergencyContacts` and accessible from header. |
| **6. Trip Sharing** | Generates shareable WhatsApp and clipboard summaries containing driver details, vehicle plate, and safety coordinates. | 1-click modal with preformatted WhatsApp message payload. |
| **7. Dashboard & Impact Metrics** | Tracks completed rides, kilograms of CO₂ prevented, travel money saved (₹), and community trust rating. | Interactive dashboard with pending requests queue and verified ride history table. |
| **8. Public Transit Fallback** | When no carpool matches, suggests BMTC express bus lines, Metro rail feeders, or student campus shuttles. | Dedicated `view-transit` tab with fare, frequency, walking distance, and route maps. |
| **9. Interactive Map & Simulation** | Clean Leaflet.js map with custom SVG markers, corridor polylines, walking connectors, and live vehicle movement simulation. | Powered by OpenStreetMap & CartoDB tiles with zero paid API keys required. |
| **10. Firebase / Local Hybrid** | Works 100% out of the box with persistent `localStorage` and supports live Firebase Cloud Firestore sync. | Configurable via `js/firebase-config.js` or the in-app Firebase Settings modal. |

---

## 🚀 Backend & Full-Stack Setup Guide

The application features a secure, decoupled **Node.js & Express** backend with **Firebase Admin SDK** support.

```
rideconnect/
├── backend/
│   ├── src/
│   │   ├── config/firebase.js        # Firebase Admin initialization & resilient DB adapter
│   │   ├── middleware/auth.js        # Firebase Token verification & development auth
│   │   ├── middleware/validation.js  # Input schema validation & sanitization
│   │   ├── middleware/errorHandler.js# Centralized HTTP error handling
│   │   ├── routes/authRoutes.js      # /api/auth (Register, Login, Profile)
│   │   ├── routes/rideRoutes.js      # /api/rides (Offer ride, Smart matching, Details)
│   │   ├── routes/requestRoutes.js   # /api/requests (Join, Driver accept/decline, Complete)
│   │   ├── services/rideService.js   # Haversine distance & corridor geometry calculations
│   │   ├── services/requestService.js# Two-way agreement & atomic seat management
│   │   └── server.js                 # Express server with CORS & Helmet security
│   ├── tests/api-test.js             # Automated API test suite covering all 7 core flows
│   ├── .env.example                  # Environment template for ports & Firebase secrets
│   ├── .gitignore                    # Prevents leaking keys or node_modules
│   └── package.json                  # Express, cors, dotenv, firebase-admin, helmet
├── js/
│   ├── api-client.js                 # Frontend API client communicating with port 5000
│   └── ...
```

---

### Step 1: Install Backend Dependencies

1. Ensure **Node.js** (v18 or higher) is installed on your computer.
2. Open PowerShell or a terminal and navigate to the `backend` directory:
```powershell
cd C:\Users\laxmi\.gemini\antigravity\scratch\rideconnect\backend
npm install
```

---

### Step 2: Configure Environment Variables & Firebase

1. Create your `.env` file from the provided template:
```powershell
cp .env.example .env
```
2. Open `backend/.env` in your editor.

#### Option A: Run with Live Firebase Cloud Firestore (Recommended for Production)
1. Go to the [Firebase Console](https://console.firebase.google.com/) and create or select your project.
2. Navigate to **Project Settings** ➔ **Service Accounts**.
3. Click **"Generate new private key"** and download the `.json` file.
4. Move this file into the `backend/` folder and name it `serviceAccountKey.json`.
5. Ensure `FIREBASE_SERVICE_ACCOUNT_PATH=./serviceAccountKey.json` is set in your `.env`.

#### Option B: Run in Development Fallback Mode (Zero Setup)
If you don't have a Firebase project right now, the backend will **automatically initialize its built-in database adapter** with identical Firestore collection/doc/query semantics and persist data to `.data_cache.json`. No secrets or setup required!

---

### Step 3: Start the Backend Server

```powershell
# From C:\Users\laxmi\.gemini\antigravity\scratch\rideconnect\backend
npm start
```
You should see:
```text
====================================================
🚗 RideConnect Backend Server running on port 5000
🔗 API Base: http://localhost:5000/api
📡 Health:   http://localhost:5000/api/health
🔥 Firebase: CONNECTED (or LOCAL FALLBACK MODE)
====================================================
```

---

### Step 4: Run the Automated API Test Suite

To verify that all endpoints, schema validations, matching math, and request transitions are passing:
```powershell
npm test
```
This tests:
1. `GET /api/health` ➔ Status 200 OK.
2. `POST /api/auth/register` (Driver) ➔ Validates vehicle, creates profile.
3. `POST /api/auth/register` (Student) ➔ Verifies `.edu` student badge.
4. `POST /api/rides` ➔ Creates active ride with origin, destination & campus stop.
5. `GET /api/rides/match` ➔ Calculates spatial corridor overlap and ETA.
6. `POST /api/requests` ➔ Submits join request with status `pending`.
7. `PUT /api/requests/:id/respond` ➔ Driver accepts, atomically decrementing available seats!
8. `GET /api/ai/status` ➔ Checks Google Gemini connection status.
9. `POST /api/ai/assistant` ➔ Queries AI Travel Assistant for commute advice.

---

### Step 5: Start Frontend

In a separate terminal:
```powershell
cd C:\Users\laxmi\.gemini\antigravity\scratch\rideconnect
python -m http.server 8000
```
Open **`http://localhost:8000`** in your browser. The frontend automatically detects the running Express backend at `http://localhost:5000`!

---

## 📡 API Endpoint Reference

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/api/health` | None | System status and Firebase connection health check. |
| `POST` | `/api/auth/register` | None | Register a new user (with role, phone, and vehicle validation). |
| `POST` | `/api/auth/login` | None | Authenticate user and retrieve profile doc. |
| `GET` | `/api/auth/me` | Bearer Token / `x-user-id` | Retrieve authenticated user's profile. |
| `POST` | `/api/rides` | Required | Offer a new ride (driver only, validates route coordinates). |
| `GET` | `/api/rides/match` | Optional | Find rides matching passenger pickup/drop coordinates & departure time. |
| `GET` | `/api/rides` | None | List all currently active rides. |
| `GET` | `/api/rides/:id` | None | Get specific ride details by ID. |
| `POST` | `/api/requests` | Required | Request to join a ride (creates two-way pending agreement). |
| `PUT` | `/api/requests/:id/respond` | Required (Driver) | Accept or decline a pending passenger request. |
| `PUT` | `/api/requests/:id/complete` | Required | Mark trip completed and record rating & feedback. |
| `GET` | `/api/requests/my` | Required | Fetch all requests involving the current user. |
| `GET` | `/api/ai/status` | None | Verify Google Gemini configuration status securely without exposing key. |
| `POST` | `/api/ai/assistant` | None | RideConnect AI Travel Assistant: multimodal transit advice, safety tips, and cost-sharing guidelines. |

---

## 🔍 Feature Status & Configuration Checklist

### ✅ Fully Working & Verified
- **Input Validation**: Rejects invalid emails, missing coordinates, invalid seat counts, and incomplete driver vehicle data.
- **Corridor Geometry & Scoring**: Evaluates real spatial detour distances (Haversine & point-to-segment) and departure time proximity without fake hardcoded results.
- **Two-Way Agreement State Machine**: Passenger sends request ➔ Driver gets notified ➔ Driver accepts ➔ Seats decrement atomically.
- **Security & Secret Isolation**: No API keys or service account credentials are exposed in client scripts; all backend secrets reside strictly in `.env`.
- **Hybrid Storage & Offline Resiliency**: Seamlessly shifts between live Cloud Firestore and local persistence.
- **Interactive UI**: Leaflet map, pulsing SVG pins, live vehicle ETA simulation, and mobile-friendly responsive navigation remain 100% functional.

### ⚙️ Requires User Configuration for Production
- **Firebase Service Account**: Provide your real `serviceAccountKey.json` from the Firebase Console to link with Google Cloud Firestore.
- **External SMS Provider (e.g. Twilio)**: The SOS alert modal currently formats urgent calls to `112` and prepares SMS/WhatsApp links for emergency contacts; an automated server SMS gateway requires configuring Twilio SID & Auth Token.

---

## 🎓 Hackathon Demo Scenario: "Student Ride to College"

### Method A: One-Click Instant Scenario
Click the **`Run Student Demo Scenario`** button on the top banner. The system will automatically:
1. Switch to **Aarav Sharma** (Student at City Tech University).
2. Populate the search from **Koramangala 4th Block** to **City Tech University Campus**.
3. Run the Smart Matching engine.
4. Highlight **Rahul Verma's White Honda City** with a **98% Direct Campus Match**.
5. Render the route polyline and pickup point on the Leaflet map.

### Method B: Complete Two-Way Agreement Walkthrough
1. **As Passenger (Aarav):**
   - Click **`Request Ride`** on Rahul's card.
   - Select seats (1 seat) and notice the fuel contribution (₹45).
   - Click **`Send Request to Driver`**.
   - Note the pending status notification.
2. **Switch to Driver (Rahul):**
   - Click the user profile pill in the top-right header and select **Rahul Verma (Driver)**.
   - Go to the **Dashboard** tab.
   - Under **Pending Requests**, see Aarav's request: *"Student heading to City Tech campus for morning lecture."*
   - Click **`Accept Request`**.
3. **Confirmed Trip Experience:**
   - Notice the **Active Confirmed Ride Floating Banner** that appears.
   - Click **`Simulate Vehicle ETA`** to watch the animated car marker travel along the corridor with real-time ETA countdown (`12 mins ➔ 8 mins ➔ Arrived!`).
   - Click **`Share Trip`** to preview the secure WhatsApp message with vehicle license plate (`KA-01-MJ-4521`).
   - Click **`Complete Ride`** to submit a 5-star rating and record CO₂ savings into the community ledger!

---

## 🔒 Privacy & Safety Features
1. **Privacy by Design**: Exact home GPS coordinates are never exposed without prior mutual agreement. Matching is conducted along safe, public corridor hubs (bus shelters, university gates, metro stations).
2. **Two-Way Agreement Required**: No passenger is added to a vehicle without the driver manually accepting the request.
3. **Community ID Badges**:
   - 🎓 **Campus Verified**: Checked against official university `.edu` domain.
   - 🛡️ **Govt ID Verified**: Aadhaar / Driving License verified profile.
   - 📱 **Phone Verified**: Authenticated mobile contact.
4. **Emergency SOS Center**:
   - 1-tap speed dial for National Emergency (`112`).
   - Multi-contact SMS alert dispatch containing trip metadata.

---

## ☁️ Firebase Configuration (Optional)

By default, RideConnect runs in **Local Community Storage Mode** (using robust `localStorage` for offline persistence across page refreshes).

To connect your own Google Firebase backend:
1. Click the **Cloud Icon** (`Firebase Settings`) in the top navigation bar.
2. Enter your Firebase project credentials:
   - **API Key**: `AIzaSy...`
   - **Project ID**: `your-project-id`
3. Click **Save & Connect**.
4. Alternatively, edit [`js/firebase-config.js`](file:///C:/Users/laxmi/.gemini/antigravity/scratch/rideconnect/js/firebase-config.js):
   ```javascript
   FirebaseConfig.credentials = {
     apiKey: "YOUR_API_KEY",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project-id",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abcdef"
   };
   ```

---

## 📂 Project Architecture

```
rideconnect/
├── index.html                 # Semantic single-page web application with responsive layout
├── README.md                  # Comprehensive documentation and hackathon presentation guide
├── css/
│   └── style.css              # Custom modern blue-and-white theme, cards, tabs, animations
├── js/
│   ├── app.js                 # App controller, view routing, modals, toasts, demo runner
│   ├── state.js               # Reactive centralized store with localStorage persistence
│   ├── matching.js            # Haversine distance, corridor projection, and match scoring
│   ├── map.js                 # Leaflet.js map controller, animated markers, route polylines
│   ├── firebase-config.js     # Firebase SDK integration layer & offline-first fallback
│   └── mock-data.js           # Realistic seed dataset (students, drivers, locations, transit)
└── assets/
    └── icon.svg               # Scalable vector application logo
```

---

## 🏆 Key Innovations
- **Corridor-Based Matching**: Unlike point-to-point taxi apps that require significant driver detours, RideConnect evaluates intermediate corridor progression, making carpooling frictionless for daily commuters.
- **Fair Cost-Sharing (Zero Surge Pricing)**: Pre-calculated non-commercial contribution per seat (₹30–₹50) to offset fuel and toll costs without commercial licensing overhead.
- **Multimodal Transit Fallback**: Ensures commuters are never stranded if no driver is available by automatically surfacing verified public bus and metro routes.
- **Simulated Real-Time Tracking**: Demonstrates full end-to-end GPS coordination client-side without costly third-party map subscription fees.
