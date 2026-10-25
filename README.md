# GD-APP — AI-Powered Group Discussion & Placement Simulator

> Practice group discussions, view speech transcripts, understand participation patterns, and prepare for placement rounds with structured feedback.

**Project type:** Full-stack web application  
**Frontend:** React.js + Vite  
**Backend:** Node.js + Express.js + Socket.IO  
**Database:** MongoDB + Mongoose  
**Speech-to-text:** AssemblyAI  
**Deployment target:** Netlify (frontend) + Render (backend)



---

## 1. What Is GD-APP?

GD-APP is a placement-preparation platform that gives students a place to practice group discussions (GDs). Users can create or join a discussion room, speak through their microphone, view speech converted into text, and review session participation statistics.

A typical placement GD evaluates how clearly a student communicates, contributes relevant ideas, listens to others, and participates respectfully. Students often find it difficult to arrange practice groups and receive consistent feedback. GD-APP aims to make practice easier to access and progress easier to understand.

### Problem Statement

- Students may not have a group available whenever they want to practice.
- Feedback from informal practice can be subjective or delayed.
- Students may not know how much they spoke or how many times they contributed.
- Colleges may not have enough trainers to manually evaluate every practice session.

### Proposed Solution

Build a web application that combines real-time room coordination, speech-to-text, session data storage, and visual participation reports. The platform can later be extended with topic relevance analysis, speaking-pace feedback, and AI-assisted coaching.

### Project Goals

1. Make it easy to create and join a GD session.
2. Display connected participants and session transcripts.
3. Store session data for later review.
4. Calculate transparent participation metrics.
5. Provide practical feedback that helps students improve.
6. Keep user audio, transcripts, credentials, and API keys secure.

---

## 2. How We Plan to Build It From Scratch

The project should be developed in small, testable modules instead of building the entire application at once.

### Phase 1 — Requirements and Design

- Identify the target users: students, session hosts, and potentially college administrators.
- Define the core user journey: create room → join room → participate → view transcript → review report.
- Design the pages, data model, API contract, and real-time event flow.
- Decide which features belong in the first version and which are future enhancements.

### Phase 2 — Frontend Foundation

- Set up React with Vite.
- Create reusable components for room creation, joining, participant lists, microphone controls, transcript display, and reports.
- Add client-side validation and loading/error states.
- Store public frontend configuration in environment variables, such as `VITE_BACKEND_URL`.

### Phase 3 — Backend Foundation

- Set up Node.js and Express.
- Configure environment variables, CORS, request validation, error handling, and MongoDB connection.
- Create room APIs and Mongoose models.
- Keep external service credentials on the server.

### Phase 4 — Real-Time Rooms

- Configure Socket.IO on the backend and `socket.io-client` in React.
- Associate each connected socket with a room and participant.
- Broadcast participant-list changes to the correct room.
- Handle disconnections, reconnects, room exits, and duplicate join attempts.

### Phase 5 — Speech-to-Text

- Request microphone permission in the browser.
- Send audio through the chosen backend-to-AssemblyAI integration.
- Receive partial/final transcript results if supported by the selected integration.
- Associate transcript segments with the correct participant and timestamp.
- Handle service failures, network delays, and microphone permission errors.

### Phase 6 — Analytics and Reports

- Aggregate transcript/session data into metrics such as word count, number of turns, and speaking-time estimates.
- Display charts and a readable report.
- Explain how each metric is calculated.
- Add AI-generated coaching only after defining the input, output, validation, and scoring approach.

### Phase 7 — Testing and Deployment

- Test room creation, multiple participants, reconnects, transcript handling, and report generation.
- Test with multiple browser windows/devices.
- Deploy the frontend and backend separately.
- Configure production URLs, CORS, database access, secrets, HTTPS, and WebSocket connectivity.

---

## 3. Core Features

| Feature | Purpose | Status |
|---|---|---|
| Create a GD room | Starts a new discussion session | Verify implementation |
| Join by room code | Lets participants enter a session | Verify implementation |
| Live participant list | Shows who is connected | Verify implementation |
| Microphone capture | Captures a user's speech after consent | Verify implementation |
| Speech-to-text | Converts audio into text using AssemblyAI | Verify integration |
| Live transcript | Displays transcript updates | Verify implementation |
| Session persistence | Saves relevant room/session data in MongoDB | Verify implementation |
| Participation report | Displays session metrics | Verify implementation |
| Contribution chart | Visualizes relative participation | Verify implementation |
| Personalized coaching | Gives improvement tips based on evidence | Planned unless implemented |
| Authentication and profiles | Provides verified user identity and access control | Planned unless implemented |
| Institution dashboard | Supports multiple sessions and administrative overview | Future enhancement |

**Metric limitation:** Speaking time or word count alone does not measure the quality of a GD contribution. Fairer feedback can consider turn-taking, relevance to the topic, clarity, and respectful participation. Automated scores should be presented as estimates, not as definitive judgments of a student's ability.

---

## 4. Authentication and Authorization Design

Authentication and authorization are different:

- **Authentication:** Who is the user?
- **Authorization:** What is this user allowed to do?

A room code by itself should not be treated as proof of identity or permission to access private session data.

### Recommended Authentication Flow

1. **Register:** The user submits a name, email, and password.
2. **Validate:** The backend validates the fields and checks whether the email is already registered.
3. **Hash password:** Hash the password with a suitable password-hashing algorithm such as Argon2id or bcrypt. Never store plain-text passwords.
4. **Create account:** Store the user record in MongoDB, including the password hash but never the original password.
5. **Login:** Verify the submitted password against the stored hash.
6. **Issue session/credential:** Prefer a secure, `HttpOnly`, `Secure` cookie-based session for a browser application. If using JWT, define expiry, validation, refresh/revocation strategy, and secure storage carefully.
7. **Protect APIs:** Authentication middleware validates the session/credential before allowing access to protected endpoints.
8. **Authorize room access:** Check that the user is a member of the room or has an explicitly permitted host/admin role.
9. **Protect Socket.IO:** Authenticate the socket connection and authorize each room join. Do not trust a client-supplied `userId`, role, or room membership without server-side verification.
10. **Logout:** Invalidate the server-side session or revoke/expire the applicable credentials.

### Suggested User Roles

| Role | Permissions |
|---|---|
| Student/Participant | Join permitted rooms, participate, and view their allowed reports |
| Host | Create/manage a session, manage participants where appropriate, and view that session's report |
| Institution Admin (future) | Manage institution-level users and sessions within their authorized scope |

### Security Checklist

- Hash passwords with Argon2id or bcrypt.
- Use HTTPS in production.
- Keep `ASSEMBLYAI_API_KEY`, database credentials, and signing secrets on the backend only.
- Configure CORS with known frontend origins; CORS is not a substitute for authentication.
- Validate request bodies, room IDs, participant membership, and all client-provided data.
- Add rate limits to login, registration, and room creation.
- Prevent users from reading another room's transcripts or reports without permission.
- Use secure cookie settings and CSRF protections when applicable to the chosen cookie/session architecture.
- Obtain informed consent before microphone capture and explain transcript storage/retention.
- Set a retention/deletion policy for audio and transcripts.
- Avoid logging passwords, tokens, API keys, or unnecessary personal data.

**Implementation note:** The above is a recommended design, not proof that authentication already exists. Implement and test it before claiming that the application has secure login or role-based access.

---

## 5. System Architecture

```text
┌───────────────────────────────────────────────┐
│                 React + Vite                  │
│ Home / Login / Join Room / Live Room / Report │
│ Microphone controls / Transcript / Charts     │
└──────────────────────┬────────────────────────┘
                       │
              REST API + Socket.IO
                       │
┌──────────────────────▼────────────────────────┐
│              Node.js + Express                │
│ Routes / Controllers / Middleware / Services  │
│ Socket.IO room management / Analysis logic    │
└──────────────┬───────────────────┬────────────┘
               │                   │
       ┌───────▼────────┐  ┌───────▼────────────┐
       │    MongoDB     │  │    AssemblyAI      │
       │ Users, rooms,  │  │ Speech recognition │
       │ transcripts,   │  │ Audio/transcript   │
       │ reports        │  │ integration        │
       └────────────────┘  └────────────────────┘
```

### Component Responsibilities

- **React client:** User interface, form validation, microphone permissions, live updates, and visual reports.
- **Express API:** Validates HTTP requests and coordinates room, user, and report operations.
- **Socket.IO:** Sends real-time room events to connected clients.
- **MongoDB/Mongoose:** Persists application records.
- **AssemblyAI service:** Converts audio to text according to the configured API integration.
- **Analysis service:** Calculates metrics from stored or in-memory session data and prepares the report.

### Important Media Distinction

Browser microphone capture, WebRTC, and speech-to-text are not the same thing:

- `getUserMedia()` can capture microphone audio after permission.
- WebRTC can transmit live audio/video between participants when peer connections and appropriate signaling/STUN/TURN configuration are implemented.
- AssemblyAI can process audio for speech recognition.
- Socket.IO can coordinate rooms and deliver application events, but it is not itself a complete media-streaming solution.

Only list WebRTC as an implemented feature if actual peer-to-peer or media-server audio/video communication is present in the code.

---

## 6. End-to-End Data Flow

1. The user opens the frontend and creates or joins a room.
2. The client sends a REST request or Socket.IO event to the backend.
3. The backend validates the request and checks room permissions.
4. Socket.IO associates the connection with the authorized room and broadcasts participant changes.
5. The user grants microphone permission and begins speaking.
6. Audio is sent to the configured speech-to-text service through the backend integration.
7. Transcript results are associated with a participant and timestamp.
8. The server broadcasts permitted transcript updates and persists the required data.
9. When the session ends, the analysis service calculates the available metrics.
10. The frontend requests and displays the report, subject to authorization.

---

## 7. Data Model Design

The following are **suggested schemas**. Adjust them to match the actual Mongoose models and the expected size of a session.

### User

```js
{
  _id,
  name,
  email,
  passwordHash,
  role,           // student, host, institutionAdmin
  createdAt,
  updatedAt
}
```

Do not return `passwordHash` in public API responses.

### Room

```js
{
  _id,
  roomCode,
  topic,
  hostId,
  status,         // waiting, active, completed
  participantIds,
  createdAt,
  startedAt,
  endedAt
}
```

### Transcript Segment

```js
{
  _id,
  roomId,
  participantId,
  text,
  startedAt,
  endedAt,
  isFinal,
  createdAt
}
```

Store transcript segments separately when sessions can be large. This avoids continually growing a single room document.

### Session Statistics / Report

```js
{
  _id,
  roomId,
  participantId,
  wordCount,
  turnCount,
  speakingTimeSeconds,
  contributionPercentage,
  feedback,
  generatedAt
}
```

The fields above are a design proposal. A `socketId` is temporary connection information and should not be used as a permanent user identity because it can change after reconnection.

---

## 8. REST API Design

These are proposed endpoint conventions. Confirm or change them according to the routes actually implemented in the project.

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/auth/register` | Register an account |
| POST | `/api/auth/login` | Authenticate a user |
| POST | `/api/auth/logout` | End the session |
| GET | `/api/auth/me` | Return the current authenticated user |
| POST | `/api/rooms` | Create a room |
| POST | `/api/rooms/:id/join` | Request permission to join a room |
| GET | `/api/rooms/:id` | Get authorized room details |
| POST | `/api/rooms/:id/end` | End a room if permitted |
| GET | `/api/rooms/:id/report` | Retrieve an authorized report |
| POST | `/api/rooms/:id/report` | Generate/finalize analysis, if designed as a POST operation |

For every endpoint, document the request body, response shape, authentication requirement, authorization rules, and error responses.

---

## 9. Socket.IO Event Design

These are suggested event names; use the exact event names from the actual source code in final documentation.

| Event | Direction | Purpose |
|---|---|---|
| `join-room` | Client → Server | Request to join a room |
| `user-list` | Server → Room | Broadcast current participants |
| `transcript` | Server → Room | Deliver transcript updates |
| `leave-room` | Client → Server | Request to leave a room |
| `participant-joined` | Server → Room | Notify that a participant joined |
| `participant-left` | Server → Room | Notify that a participant left |
| `room-error` | Server → Client | Return a room-related error |

### Socket Handling Rules

- Authenticate connections and authorize room membership on the server.
- Validate every room ID and event payload.
- Do not accept participant identity or permissions solely from client-supplied fields.
- Remove disconnected sockets from active participant lists.
- Handle reconnection and duplicate events safely.
- Send events only to the correct room and authorized recipients.
- Avoid sending private user data or secrets through broadcast events.

---

## 10. Suggested Folder Structure

This is a maintainable target structure. Keep the actual repository's structure if it differs, and only add files when their functionality is implemented.

```text
GD-APP/
├── client/
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/
│   │   │   ├── AnalysisReport.jsx
│   │   │   ├── ContributionPie.jsx
│   │   │   ├── Controls.jsx
│   │   │   ├── JoinRoom.jsx
│   │   │   ├── UserList.jsx
│   │   │   └── TranscriptPanel.jsx
│   │   ├── pages/
│   │   │   ├── HomePage.jsx
│   │   │   ├── LoginPage.jsx
│   │   │   ├── RegisterPage.jsx
│   │   │   ├── Room.jsx
│   │   │   └── RoomDashboard.jsx
│   │   ├── context/
│   │   │   └── AuthContext.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── utils/
│   │   │   └── socket.js
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── .env.example
│   └── package.json
│
├── server/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── roomController.js
│   │   └── analysisController.js
│   ├── middleware/
│   │   ├── authMiddleware.js
│   │   ├── errorMiddleware.js
│   │   └── validateRequest.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Room.js
│   │   ├── Transcript.js
│   │   └── Report.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── roomRoutes.js
│   │   └── analysisRoutes.js
│   ├── services/
│   │   ├── assemblyAIService.js
│   │   └── analysisService.js
│   ├── socket/
│   │   └── roomSocket.js
│   ├── utils/
│   │   └── generateRoomCode.js
│   ├── app.js
│   ├── index.js
│   ├── .env.example
│   └── package.json
│
├── .gitignore
└── README.md
```

### Why Separate These Folders?

- **Routes:** Define API paths and connect them to controllers.
- **Controllers:** Handle incoming requests and prepare responses.
- **Services:** Keep speech-recognition and analysis logic separate from HTTP handling.
- **Models:** Define database schemas and data constraints.
- **Middleware:** Centralize authentication, validation, and error handling.
- **Socket:** Keep real-time event handling separate from REST routes.
- **Components/pages:** Separate reusable interface elements from complete screens.

---

## 11. Environment Variables

Create local `.env` files and keep them out of Git.

### `server/.env.example`

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/gd_app
ASSEMBLYAI_API_KEY=replace_with_your_key
CLIENT_ORIGIN=http://localhost:5173
SESSION_SECRET=replace_with_a_long_random_secret
```

The variables above are examples. Use the names that the actual server code reads. If JWT or another authentication strategy is used, configure the corresponding secrets instead of adding unused variables.

### `client/.env.example`

```env
VITE_BACKEND_URL=http://localhost:5000
```

Never place a private API key or database credential in a `VITE_` variable. Vite exposes frontend environment variables to client-side code.

---

## 12. Run the Project Locally

### Prerequisites

- Node.js (use a version supported by the project's dependencies)
- npm
- MongoDB locally or a MongoDB Atlas database
- AssemblyAI API key for speech-to-text functionality

### Step 1 — Get the source code

```bash
git clone <YOUR_REPOSITORY_URL>
cd GD-APP
```

Replace the placeholder with the real repository URL.

### Step 2 — Start the backend

```bash
cd server
npm install
```

Create `server/.env` using the relevant values from `.env.example`, then run the script defined in `server/package.json`, for example:

```bash
npm start
```

### Step 3 — Start the frontend

Open a second terminal:

```bash
cd client
npm install
```

Create `client/.env` and configure `VITE_BACKEND_URL`, then run:

```bash
npm run dev
```

Open the local URL printed by Vite. If the package scripts use different names, follow the scripts in the actual `package.json` files.

---

## 13. Deployment Plan

### Frontend — Netlify

1. Connect the client repository/folder to Netlify.
2. Configure the build command and publish directory for the Vite application.
3. Set `VITE_BACKEND_URL` to the deployed backend URL.
4. Add SPA routing redirects if required by the router.
5. Redeploy after changing build-time environment variables.

### Backend — Render

1. Create a web service using the server directory and its start script.
2. Configure `MONGO_URI`, `ASSEMBLYAI_API_KEY`, and other required secrets in the hosting dashboard.
3. Set the allowed production client origin.
4. Configure Socket.IO CORS consistently with Express CORS.
5. Verify WebSocket connections, health checks, logs, and reconnection behavior.
6. Confirm the backend host supports the persistent connections required by the Socket.IO architecture.

Do not assume that deployment is complete until the deployed application has been tested end to end. The backend and frontend URLs are intentionally left as placeholders.

---

## 14. Testing Plan

### Functional Tests

- Create a room with valid input.
- Join with a valid room code.
- Reject invalid or unauthorized room access.
- Verify participant lists update after joins and exits.
- Verify transcript text is associated with the correct participant.
- Check that report metrics match the underlying session data.
- Verify the user cannot access another participant's private report unless authorized.

### Edge Cases

- Empty or invalid room code.
- Duplicate join requests.
- Microphone permission denied.
- Network interruption during a session.
- Speech-to-text service unavailable or rate-limited.
- Participant refreshes or reconnects.
- Empty session with no transcript.
- Multiple participants speak at nearly the same time.
- Malformed request data and unauthorized API calls.

### Example Acceptance Criteria

- A valid participant can join the correct room.
- Unauthorized users cannot access protected room data.
- A disconnect removes the user from the active list within the defined timeout.
- The transcript report distinguishes final transcript segments from partial updates where supported.
- The report handles an empty session without crashing.

---

## 15. Challenges and Engineering Decisions

### Real-time Synchronization

**Challenge:** Participant lists and transcript updates must remain consistent across clients.

**Approach:** Use room-scoped Socket.IO events, server-side membership checks, safe reconnect handling, and persistent session state where appropriate.

### Speech-to-Text Latency and Accuracy

**Challenge:** Speech recognition can introduce delay or transcription errors.

**Approach:** Handle partial and final results according to the provider API, show clear processing/error states, and allow final transcript segments to replace temporary partial text.

### Speaker Attribution

**Challenge:** A transcript must be associated with the correct participant.

**Approach:** When each participant submits their own audio stream, associate that stream's results with the authenticated participant. If the application processes mixed room audio, speaker diarization or another attribution strategy may be needed.

### Fair Participation Metrics

**Challenge:** A participant who talks more is not necessarily contributing better ideas.

**Approach:** Separate measurable activity metrics from subjective quality estimates, document each metric, and avoid presenting unvalidated AI scores as objective truth.

### Privacy

**Challenge:** Audio and transcripts may contain personal or sensitive information.

**Approach:** Request consent, protect credentials and endpoints, minimize stored data, and provide a clear retention/deletion policy.

---

## 16. Features to Implement Next

Prioritize these enhancements based on user value and technical dependencies.

### Priority 1 — Core Reliability

- Complete and test registration, login, logout, and protected routes.
- Secure room membership and host permissions.
- Improve reconnect handling and transcript persistence.
- Add clear error messages and session-end handling.
- Test the complete flow with multiple users.

### Priority 2 — Better GD Feedback

- Detect filler words such as “um” and “like,” with language-specific handling.
- Estimate speaking pace in words per minute.
- Calculate turn-taking and participation balance.
- Add topic relevance feedback with a clearly defined scoring method.
- Provide specific suggestions linked to transcript evidence.

### Priority 3 — Practice Experience

- Random GD topic generator.
- Timers and session rules.
- AI moderator or AI participants for solo practice.
- Downloadable PDF report and practice history.
- Multilingual speech recognition and feedback.

### Priority 4 — Institutional Features

- Institution and trainer dashboards.
- Group/session management.
- Authorized cohort reports and progress history.
- Role-based access and audit logging.
- Scalable storage and retention controls.

These items are a roadmap, not a claim that the functionality already exists.

---

## 17. How to Explain the Project to a Mentor

Use this structure during the project review:

1. **Problem:** Students need accessible GD practice and consistent feedback.
2. **Solution:** A web platform with live rooms, speech-to-text, session metrics, and reports.
3. **Architecture:** React frontend, Express/Node backend, Socket.IO for live events, MongoDB for persistence, and AssemblyAI for speech recognition.
4. **Data flow:** Join room → capture audio → transcribe → associate transcript with participant → store data → calculate metrics → display report.
5. **Authentication:** Verify user identity, protect routes, authorize room membership, and secure socket events.
6. **Engineering challenges:** Real-time consistency, transcription latency, speaker attribution, fair metrics, and privacy.
7. **Next steps:** Improve reliability, validate analytics, and add advanced coaching and institutional features.

### Suggested short explanation

“GD-APP is a group discussion practice and placement simulator. We designed it to help students practice in virtual rooms and understand their participation through speech transcripts and session reports. The frontend is built with React, while Node.js and Express handle backend APIs. Socket.IO supports real-time room events, MongoDB stores session data, and AssemblyAI provides speech-to-text. We are structuring the project into separate modules for authentication, room management, transcription, and analysis so that each component can be tested and extended independently.”

Adapt the wording to what you have actually implemented and tested.

---

## 18. Development Principles

To demonstrate that the project was engineered carefully from the beginning:

- Break the application into modules and define each module's responsibility.
- Build a minimal working version before adding advanced AI features.
- Keep frontend, backend, database, and external service configuration separate.
- Validate inputs at the server boundary.
- Use meaningful commits that explain the changes made.
- Write tests for important calculations, APIs, and access-control rules.
- Document assumptions and known limitations.
- Never commit secrets, real user transcripts, or credentials.
- Maintain a clear record of implemented features versus planned work.

A credible project explanation is based on being able to walk through your own code, explain why each design choice was made, show the feature working, and discuss what you would improve next.

---

## 19. Team

Update this section with the actual contributors and their responsibilities.

| Team member | Responsibility |
|---|---|
| Prem Sharma | Full-stack developer — verify exact contributions |
| Additional contributors | Add names and actual contributions if applicable |

---

## 20. Final Note

GD-APP aims to make group discussion preparation more accessible and measurable. Its long-term value comes from combining real-time collaboration, speech recognition, reliable session data, and transparent feedback—not simply from generating an AI score.

**Repository:** `<YOUR_REPOSITORY_URL>`  
**Live demo:** `<YOUR_DEPLOYED_FRONTEND_URL>`  
**Demo video:** To be added

If you use this README for a mentor review, replace placeholders, align the folder structure and event names with the real code, and update the feature-status table after testing each feature.
