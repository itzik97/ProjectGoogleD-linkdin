# ProjectGoogleD-Web
Google Drive–like web application built on top of the EX3 REST server (Node/Express) and the EX2 TCP storage server (C++).
EX4 adds a full React UI (Google Drive–inspired) that consumes the EX3 API and supports authenticated multi-user flows.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Key Features (EX4 Additions)](#key-features-ex4-additions)
3. [System Architecture](#system-architecture)
4. [Ports & URLs](#ports--urls)
5. [Authentication (JWT) Model](#authentication-jwt-model)
6. [Status Codes](#status-codes)
7. [Running the Project (Docker Compose)](#running-the-project-docker-compose)
8. [Running Tests](#running-tests)
9. [REST API Specification (High Level)](#rest-api-specification-high-level)
10. [Development Workflow (JIRA + Git)](#development-workflow-jira--git)
11. [Notes & Constraints](#notes--constraints)
12. [Screenshots- running examples](#screenshots)

---

## Project Overview
ProjectGoogleD is a three-layer system:
- **EX2 (C++ TCP Server):** storage engine for file content operations.
- **EX3 (Node/Express REST Server):** authentication, authorization, metadata, permissions, REST endpoints; delegates content operations to EX2 when enabled.
- **EX4 (React UI):** full client application that provides a Google Drive–like user experience and communicates only with the EX3 REST API.

The EX4 requirements emphasize a complete, real, dynamic React application (no hardcoded data), with authentication, route protection, and UI features comparable to Google Drive where feasible.

---

## Key Features (EX4 Additions)

### Authentication & Access Control
- **Register** screen: username, password, confirm password, display name, avatar selection + client-side validation.
- **Login** screen: username + password + validation and clear error feedback.
- **JWT-based session:** after login, UI attaches JWT to every protected API call automatically.
- **Protected routes:** attempting to access authenticated pages without a valid session redirects to Login.

### Drive-like UI
- Top bar + side navigation inspired by Google Drive.
- Drive views (as supported by the backend):
  - **My Drive / Root**
  - **Shared With Me**
  - **Starred**
  - **Recent**
  - **Trash**
- File/folder capabilities surfaced through the UI (based on backend support): upload, create folders, rename, delete (move to trash), restore, view/open, manage permissions, search, etc.
- **Theme toggle** (Dark/Light) from the top bar.

> Note: Google Drive opens documents in separate Google apps; in our project the file is displayed inside our UI (text displayed/edited, images previewed, etc.).

---

## System Architecture

### Layers
1. **React UI (EX4)**
   - Calls EX3 via REST.
   - Stores JWT client-side and injects it into requests.
2. **REST API Server (EX3)**
   - Express MVC: routes/controllers/services/repositories.
   - Authorization and permission enforcement per request.
   - Optional delegation of file content to EX2.
3. **TCP Storage Server (EX2)**
   - Handles content storage/retrieval/search-by-content via TCP protocol.

### Design Principles
- SOLID-oriented separation:
  - UI: reusable components + services layer for API calls.
  - EX3: thin controllers, business logic in services, persistence behind repositories.
- Loose coupling:
  - EX3 ↔ EX2 via an integration client abstraction.
  - UI ↔ EX3 via centralized `apiFetch` / API service utilities.

---

## Ports & URLs
After Docker Compose is up:
- **EX3 REST API:** `http://localhost:3000`
- **EX4 UI (Nginx serving React build):** `http://localhost:3001`
- **EX2 TCP server:** internal container networking (used by EX3; port depends on compose/Dockerfile configuration)

Open in browser:
- `http://localhost:3001`

---

## Authentication (JWT) Model

### Public endpoints (no JWT)
- `POST /api/users` – create a new user (Register)
- `POST /api/tokens` – authenticate and receive JWT (Login)

### Protected endpoints (JWT required)
All file/folder/permissions/search operations require:
- `Authorization: Bearer <JWT>`

The backend should respond consistently:
- **401 Unauthorized** – missing/invalid JWT
- **403 Forbidden** – valid JWT but insufficient permissions
- **404 Not Found** – resource does not exist / not accessible as defined by project rules

---

## Status Codes
The API uses standard HTTP status codes to communicate the outcome of each request.

- **200 OK**  
  Returned when a request is processed successfully and the requested data is returned.

- **201 Created**  
  Returned when a new resource is created successfully (e.g., user registration, token creation, file creation, permission creation).

- **204 No Content**  
  Returned when a delete operation completes successfully (request processed correctly, no response body).

- **400 Bad Request**  
  Returned when the request is invalid or malformed (missing fields, invalid body, invalid search query, etc.).

- **401 Unauthorized**  
  Returned when authentication is required but no valid JWT is provided.

- **403 Forbidden**  
  Returned when the user is authenticated but does not have sufficient permissions for the requested operation.

- **404 Not Found**  
  Returned when the requested resource does not exist or cannot be found.

- **500 Internal Server Error**  
  Returned when an unexpected error occurs on the server side.

---

## Running the Project (Docker Compose)

From repository root:

### 1) Clean previous run

docker-compose down

docker-compose down --remove-orphans

### 2) Build and run all services

docker-compose up --build

#### Expected outcome

Images build for: ex2, ex3, ui

Containers start (example names): ex2_server, ex3_server, ui_app

EX3 logs include: Server running on port 3000

UI logs include: nginx startup messages

### 3) Open the UI

Open in browser:

http://localhost:3001

### Running Tests

EX3 (Node/Express)

cd Entry-Point

npm test

In Docker mode, the UI is typically built into an Nginx image; local UI tests may be run outside Docker depending on your workflow.

## REST API Specification

The UI consumes these endpoints through the EX3 server.

### Users & Tokens

POST /api/users – register

POST /api/tokens – login (returns JWT)

###  Files / Folders (Representative)

GET /api/files

POST /api/files

GET /api/files/:id

PATCH /api/files/:id

DELETE /api/files/:id – move-to-trash semantics per project behavior

### Permissions

GET /api/files/:id/permissions

POST /api/files/:id/permissions

PATCH /api/files/:id/permissions/:pId

DELETE /api/files/:id/permissions/:pId

### Search

GET /api/search/:query

## Development Workflow (JIRA + Git)

#### Work is managed in JIRA using:

Epics -> Stories -> Tasks

Sprint planning + status meetings documentation

Status flow: To Do -> In Progress -> Review Code -> Done

#### Implementation is done via feature branches:

feature-APCS-###-...

Each task is linked to its branch and PR.
PRs are reviewed by a different team member before merge.

## Notes & Constraints

The React app must be dynamic (no hardcoded server data) and must communicate with the EX3 server (not EX2 directly).

Input validation is required in both client and server (e.g., password policy, required fields, duplicate username feedback).

Do not store or commit sensitive data (real passwords, tokens, private keys, etc.).

Only use libraries/modules allowed by the course policy.

---

## Screenshots

### Docker Compose up
<img width="1097" height="241" alt="image" src="https://github.com/user-attachments/assets/35ded921-c4ab-4f0e-ab58-adea9b06e90b" />
<img width="1394" height="508" alt="image" src="https://github.com/user-attachments/assets/879e4a90-1b69-4675-a976-47649d8e33a0" />

---

### Login/Register screens
<img width="1923" height="1075" alt="image" src="https://github.com/user-attachments/assets/6b7aaac7-2ffb-4e2d-a6ec-7f191d5ff751" />
<img width="1923" height="1028" alt="image" src="https://github.com/user-attachments/assets/cd7ef709-0f82-4814-9c80-715b510d2cbe" />
<img width="1830" height="1034" alt="image" src="https://github.com/user-attachments/assets/326469e8-f98e-44d3-adcf-a44b7d350778" />
<img width="1843" height="965" alt="image" src="https://github.com/user-attachments/assets/f961f002-f2b0-4ce5-ac92-118a7f7c9abb" />
<img width="1883" height="1004" alt="image" src="https://github.com/user-attachments/assets/73895b8d-7442-4500-9f26-c8ca46d01f42" />
<img width="1716" height="927" alt="image" src="https://github.com/user-attachments/assets/c96b5f26-c36d-4a87-a788-adc305a2721e" />
<img width="1921" height="1030" alt="image" src="https://github.com/user-attachments/assets/c360da34-3973-4849-baa5-840d0c07c872" />
<img width="1383" height="920" alt="image" src="https://github.com/user-attachments/assets/36e1ae9f-cb97-45c8-a7d9-6e771f3a7ed6" />
<img width="1763" height="931" alt="image" src="https://github.com/user-attachments/assets/92d953ca-d937-4e15-b0b2-b6fc08e5932f" />
<img width="1841" height="997" alt="image" src="https://github.com/user-attachments/assets/a6509be1-f852-4927-9f31-0998f977ffbe" />
<img width="1873" height="990" alt="image" src="https://github.com/user-attachments/assets/232521d5-24f3-45a1-88cf-a99dc2ae1280" />
<img width="1923" height="1009" alt="image" src="https://github.com/user-attachments/assets/fb20a80b-dd79-4cae-a070-bac21ebd6ac3" />
<img width="1479" height="986" alt="image" src="https://github.com/user-attachments/assets/1d4d0619-13ab-42c4-bc12-ae7f595c73eb" />
<img width="1882" height="555" alt="image" src="https://github.com/user-attachments/assets/df19ec50-dc51-489d-9315-3e5273d811af" />
<img width="1819" height="843" alt="image" src="https://github.com/user-attachments/assets/eb5e6606-7ad9-4989-8284-1c5ec6518e05" />
<img width="1917" height="924" alt="image" src="https://github.com/user-attachments/assets/a82794b0-ad72-44c3-8dba-56dd1df0c9e1" />

---

### Main Drive screen + navigation views (Shared/Starred/Recent/Trash)
<img width="1923" height="1009" alt="image" src="https://github.com/user-attachments/assets/02488e64-ff30-4a4b-9316-e89546f4701e" />
<img width="493" height="426" alt="image" src="https://github.com/user-attachments/assets/7e776d47-5202-4036-9adf-2feb7ba6cce4" />
<img width="1886" height="1034" alt="image" src="https://github.com/user-attachments/assets/7f5e585b-fd83-48b1-acad-1c7afdabc904" />
<img width="1054" height="832" alt="image" src="https://github.com/user-attachments/assets/0d84982e-f507-4d4e-9318-be61866d4a60" />
<img width="1128" height="802" alt="image" src="https://github.com/user-attachments/assets/ab57e3ac-62ab-4f40-8013-f3c7a3cb8812" />
<img width="89" height="83" alt="image" src="https://github.com/user-attachments/assets/4e882686-f581-4293-99bd-10560291def2" />
<img width="1038" height="799" alt="image" src="https://github.com/user-attachments/assets/8ce8ef81-860a-4147-a5bf-cb2a15a1172c" />
<img width="1883" height="599" alt="image" src="https://github.com/user-attachments/assets/92121fb5-1f32-4fc8-a89e-1b282fa7d800" />
<img width="691" height="592" alt="image" src="https://github.com/user-attachments/assets/b6149c2e-db53-4639-8f6d-84322bd24c43" />
<img width="405" height="549" alt="image" src="https://github.com/user-attachments/assets/fa06f54f-e118-4461-94c7-a35d0d95190b" />
<img width="1608" height="848" alt="image" src="https://github.com/user-attachments/assets/fe20770b-96a1-4a65-bbca-89a4c38aaecd" />
<img width="1239" height="724" alt="image" src="https://github.com/user-attachments/assets/3606e674-32a0-429d-983e-c48302afe69b" />
<img width="1512" height="240" alt="image" src="https://github.com/user-attachments/assets/047a9389-fb4e-4d3f-82b4-089c50580bab" />
<img width="1464" height="575" alt="image" src="https://github.com/user-attachments/assets/eae7f0de-b430-48c4-80bc-3b779029495b" />
<img width="1334" height="659" alt="image" src="https://github.com/user-attachments/assets/b4613e5c-7625-4dd8-b96b-691f4fad561f" />
<img width="1511" height="594" alt="image" src="https://github.com/user-attachments/assets/20b2aea9-364a-4ff0-93d9-7deb5aee32c6" />
<img width="1857" height="780" alt="image" src="https://github.com/user-attachments/assets/0e52c616-3e69-442b-aa85-780036903050" />
<img width="1525" height="816" alt="image" src="https://github.com/user-attachments/assets/129509ed-2f7b-4764-a0f2-df80a520e948" />
<img width="1893" height="634" alt="image" src="https://github.com/user-attachments/assets/c90aac07-21a0-4c49-8b9e-cf913bb321e4" />
<img width="1698" height="790" alt="image" src="https://github.com/user-attachments/assets/bd9b687c-9419-4753-ac0c-ce4caf020389" />
<img width="1435" height="648" alt="image" src="https://github.com/user-attachments/assets/5b606053-036c-458c-af15-2b987e67f34c" />
<img width="1896" height="665" alt="image" src="https://github.com/user-attachments/assets/fa17749a-b668-4b97-b7bf-3a27cfb49ad8" />
<img width="1768" height="656" alt="image" src="https://github.com/user-attachments/assets/55fabff2-fa5b-4f74-b19b-f4e7591e66c0" />
<img width="166" height="72" alt="image" src="https://github.com/user-attachments/assets/6061e63c-bceb-41cb-8061-e86718f52e48" />
<img width="1325" height="615" alt="image" src="https://github.com/user-attachments/assets/9afcd4ba-4bb6-4a29-bc46-919244c28dcf" />
<img width="1219" height="632" alt="image" src="https://github.com/user-attachments/assets/875bfcf3-3112-407a-9efa-10992825cd1f" />
<img width="908" height="704" alt="image" src="https://github.com/user-attachments/assets/42abf100-b369-47a5-bb72-bc12a5e1d540" />
<img width="1485" height="686" alt="image" src="https://github.com/user-attachments/assets/b323b9cd-4a86-46bd-aa6a-a690a2328f8c" />
<img width="1909" height="702" alt="image" src="https://github.com/user-attachments/assets/7ae6a164-7015-4af0-b00e-651d941bed12" />
<img width="1886" height="700" alt="image" src="https://github.com/user-attachments/assets/30253887-5391-4429-b7ef-ca55a9c98c7b" />

---

### Permission management modal
<img width="1889" height="874" alt="image" src="https://github.com/user-attachments/assets/adea71f2-c06a-44aa-9d22-8352425c43bf" />
<img width="1873" height="788" alt="image" src="https://github.com/user-attachments/assets/054fc65c-a533-42c3-981e-6e15178b4543" />
<img width="1731" height="749" alt="image" src="https://github.com/user-attachments/assets/b9b127c7-1142-4024-847b-36d24c6edb22" />
<img width="1869" height="677" alt="image" src="https://github.com/user-attachments/assets/56244362-160c-4587-a7c6-8c4d1fa404a5" />
<img width="1216" height="539" alt="image" src="https://github.com/user-attachments/assets/e2703b9c-ad32-41a0-b675-01a5da8ae554" />
<img width="1883" height="590" alt="image" src="https://github.com/user-attachments/assets/2abecc1b-fca0-4bd3-8344-177cc7b21a22" />
<img width="1251" height="633" alt="image" src="https://github.com/user-attachments/assets/8e873871-6bca-4457-9693-8469c7058a0b" />
<img width="1902" height="766" alt="image" src="https://github.com/user-attachments/assets/60e415fa-e1f1-4d34-87ad-f15bad7d70fe" />
<img width="1905" height="713" alt="image" src="https://github.com/user-attachments/assets/4ad2f13d-a2a7-48a9-bfa8-30deaa013e55" />
<img width="1899" height="685" alt="image" src="https://github.com/user-attachments/assets/e7f9e0b8-034b-4d2b-9997-83d2638a9562" />
<img width="928" height="541" alt="image" src="https://github.com/user-attachments/assets/62f06202-1356-4ed8-907b-b5a634c7861b" />
<img width="1920" height="770" alt="image" src="https://github.com/user-attachments/assets/1b5807f8-acc8-4682-a698-1e0ac67bdeeb" />
<img width="1903" height="672" alt="image" src="https://github.com/user-attachments/assets/2c0707b3-bdb0-4ac2-b8e3-cabcfc6f8d59" />
<img width="1904" height="669" alt="image" src="https://github.com/user-attachments/assets/3be1694f-304d-4d13-a5a6-08036e988b51" />
<img width="1069" height="556" alt="image" src="https://github.com/user-attachments/assets/294b3cf9-8125-4114-8ccb-f2fceae9411c" />
<img width="1905" height="844" alt="image" src="https://github.com/user-attachments/assets/675f9edf-3d5d-45e7-a315-106524a9afb4" />
<img width="1483" height="334" alt="image" src="https://github.com/user-attachments/assets/11c63cec-6f28-463e-b080-1a20fd615576" />
<img width="145" height="88" alt="image" src="https://github.com/user-attachments/assets/d427a926-245e-4fbd-9136-566a593f5951" />
<img width="1001" height="577" alt="image" src="https://github.com/user-attachments/assets/34d41e9e-0f53-44e0-b939-7a08738ca2b2" />
<img width="1879" height="610" alt="image" src="https://github.com/user-attachments/assets/b3db6129-69b8-4492-b205-629ef99b3ccd" />
<img width="1243" height="610" alt="image" src="https://github.com/user-attachments/assets/07c7803a-4ec3-431b-b15d-479f2cce29b2" />
<img width="1752" height="466" alt="image" src="https://github.com/user-attachments/assets/160f9b5f-8d69-4709-adfc-87b0568c4018" />
<img width="1102" height="778" alt="image" src="https://github.com/user-attachments/assets/c3717707-2e29-4cf4-9cd5-0095f08134d5" />
<img width="1434" height="764" alt="image" src="https://github.com/user-attachments/assets/4058772b-5bda-4883-a793-7e66ca8c620c" />
<img width="1267" height="477" alt="image" src="https://github.com/user-attachments/assets/143de6d3-6847-4d58-86ee-8c58e5c2c931" />
<img width="1912" height="688" alt="image" src="https://github.com/user-attachments/assets/14743241-534f-4cc5-ae74-396d54d34b10" />

---

### Search results
<img width="1445" height="766" alt="image" src="https://github.com/user-attachments/assets/12138198-0bb7-4e54-a71d-707ce564291a" />
<img width="1905" height="678" alt="image" src="https://github.com/user-attachments/assets/be30c4a1-91f0-4777-9d71-43b5a1d0bb96" />
<img width="1806" height="591" alt="image" src="https://github.com/user-attachments/assets/9040954c-648d-4e07-aa50-d030e56ffe00" />
<img width="1923" height="698" alt="image" src="https://github.com/user-attachments/assets/a1e98204-5436-4460-b8bc-ec9d6a60ac86" />
<img width="1327" height="555" alt="image" src="https://github.com/user-attachments/assets/6fb7762a-2135-439c-91cc-2d94066d4d1b" />

---

### Tests output
<img width="1042" height="753" alt="image" src="https://github.com/user-attachments/assets/b968b6f7-0580-4564-8d6e-9cfb7f80fde1" />
