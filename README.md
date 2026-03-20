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
<img width="1097" height="241" alt="1" src="https://github.com/user-attachments/assets/027b59a8-b046-4f65-9b43-5389b41efb99" />
<img width="1394" height="508" alt="2" src="https://github.com/user-attachments/assets/73a4553c-25c6-4b20-80ce-56a0f4c26315" />

---

### Login/Register screens
<img width="1923" height="1075" alt="1" src="https://github.com/user-attachments/assets/acf5cef4-5207-438c-8658-da2e6f4d4aa8" />
<img width="1923" height="1028" alt="2" src="https://github.com/user-attachments/assets/d6a9c569-3de2-4e75-a2d1-8969bed3c7a0" />
<img width="1830" height="1034" alt="3" src="https://github.com/user-attachments/assets/70c615b1-6ab0-46b9-8bb7-55d55fb8fb42" />
<img width="1843" height="965" alt="4" src="https://github.com/user-attachments/assets/b897998f-0ab0-40a9-b29a-be4b87ccfd46" />
<img width="1883" height="1004" alt="5" src="https://github.com/user-attachments/assets/54c1d7a9-87dc-4536-8f83-4b662ca9e48a" />
<img width="1716" height="927" alt="6" src="https://github.com/user-attachments/assets/c5a928a6-309b-47bc-ac4b-7b9d108d4788" />
<img width="1921" height="1030" alt="7" src="https://github.com/user-attachments/assets/ff59809b-4f69-4e32-b009-bcc7bf4c6138" />
<img width="1383" height="920" alt="8" src="https://github.com/user-attachments/assets/ebf0c548-0348-4d47-9e07-6db4436f2d8a" />
<img width="1763" height="931" alt="9" src="https://github.com/user-attachments/assets/37457452-cffd-4485-a9b3-82f2201f75c7" />
<img width="1841" height="997" alt="10" src="https://github.com/user-attachments/assets/51296c40-93e5-44df-b218-cbb9c7839f1d" />
<img width="1873" height="990" alt="11" src="https://github.com/user-attachments/assets/5a6f06fc-f9d1-4ffb-8373-f905f243aa15" />
<img width="1923" height="1009" alt="12" src="https://github.com/user-attachments/assets/81d26779-b216-4180-ad49-785bdaa72d55" />
<img width="1479" height="986" alt="14" src="https://github.com/user-attachments/assets/26fdae75-1702-4797-9163-6cb5dc7a6cf2" />
<img width="1882" height="555" alt="15" src="https://github.com/user-attachments/assets/b89ff17b-5684-46fa-8fbf-6a9042279e46" />
<img width="1819" height="843" alt="16" src="https://github.com/user-attachments/assets/dc3f2320-46b9-4cfd-a0f8-6efca0a59624" />
<img width="1917" height="924" alt="17" src="https://github.com/user-attachments/assets/c5ec8a23-6375-43b7-9cb3-7beda3d99ed0" />

---

### Main Drive screen + navigation views (Shared/Starred/Recent/Trash)
<img width="1923" height="1009" alt="1" src="https://github.com/user-attachments/assets/b2fcdf26-e358-40a4-b61f-a23e09ed5918" />
<img width="493" height="426" alt="2" src="https://github.com/user-attachments/assets/56785925-a03b-42f5-b84a-1096aa7eb842" />
<img width="1886" height="1034" alt="3" src="https://github.com/user-attachments/assets/8caa83a2-d87d-47e0-94e3-4d7899180aa1" />
<img width="1054" height="832" alt="4" src="https://github.com/user-attachments/assets/07741e1b-1e16-4c86-a853-23925147e283" />
<img width="1128" height="802" alt="5" src="https://github.com/user-attachments/assets/34925adf-9da5-4089-a858-3fc0653b6646" />
<img width="89" height="83" alt="6" src="https://github.com/user-attachments/assets/e6424cd9-45e1-4003-abe1-0d9783ca58d8" />
<img width="1038" height="799" alt="7" src="https://github.com/user-attachments/assets/e6f8428f-1f17-4fd4-b48e-6aeb4d0bb111" />
<img width="1883" height="599" alt="8" src="https://github.com/user-attachments/assets/7abc7679-bae0-4912-93dc-4c96e77313b7" />
<img width="691" height="592" alt="9" src="https://github.com/user-attachments/assets/e954bdae-0d38-4ddc-9892-475c3aa79275" />
<img width="405" height="549" alt="10" src="https://github.com/user-attachments/assets/cc2b5d72-30dc-49d8-a6aa-a992aacb22e4" />
<img width="1608" height="848" alt="11" src="https://github.com/user-attachments/assets/e012a630-92e4-4a22-9bf6-44b152bb9fc6" />
<img width="1239" height="724" alt="12" src="https://github.com/user-attachments/assets/79261402-393d-4fcc-b858-34d86d76ab15" />
<img width="1512" height="240" alt="13" src="https://github.com/user-attachments/assets/290f8103-727d-450d-ba87-3d72add48498" />
<img width="1464" height="575" alt="14" src="https://github.com/user-attachments/assets/32639773-f571-4303-9197-4f43356c655c" />
<img width="1334" height="659" alt="15" src="https://github.com/user-attachments/assets/bcd4c4df-f5e4-4fa3-96f4-1e5d672222de" />
<img width="1857" height="780" alt="16" src="https://github.com/user-attachments/assets/ecf22537-9ed3-4f70-9382-bba518a49abf" />
<img width="1857" height="780" alt="17" src="https://github.com/user-attachments/assets/2463ced4-0a68-4c58-8f14-eddd612a441e" />
<img width="1525" height="816" alt="18" src="https://github.com/user-attachments/assets/9f36d899-4fb5-4316-8b08-4350061e7329" />
<img width="1893" height="634" alt="19" src="https://github.com/user-attachments/assets/bdda6fd4-23cd-4a6b-9759-0e21cfb801b9" />
<img width="1698" height="790" alt="20" src="https://github.com/user-attachments/assets/2def6712-9bfc-4be6-a93a-ceec25d47d22" />
<img width="1435" height="648" alt="21" src="https://github.com/user-attachments/assets/7d4d2423-e91b-4a0c-a998-1faeb1e7d9ba" />
<img width="1896" height="665" alt="22" src="https://github.com/user-attachments/assets/55a71ec2-033e-4106-a8ce-0c2b175b0c13" />
<img width="1768" height="656" alt="23" src="https://github.com/user-attachments/assets/5a5867ce-f51d-4280-8f70-68a68e63d607" />
<img width="166" height="72" alt="24" src="https://github.com/user-attachments/assets/14b214ed-d102-4af4-bb00-089a3136b1e1" />
<img width="1325" height="615" alt="25" src="https://github.com/user-attachments/assets/f5480895-1b91-41d3-9484-4180ff4a8626" />
<img width="1219" height="632" alt="26" src="https://github.com/user-attachments/assets/006d923a-ec96-4f13-9876-286f1b7bf9fc" />
<img width="908" height="704" alt="27" src="https://github.com/user-attachments/assets/5236a7c0-d0d7-45de-b923-5430348abc4f" />
<img width="1485" height="686" alt="28" src="https://github.com/user-attachments/assets/d68d9d35-7f71-405d-b5bd-7fc7f6cb2f83" />
<img width="1909" height="702" alt="29" src="https://github.com/user-attachments/assets/fceb1d91-4532-4171-bf4d-c793148bf43f" />
<img width="1886" height="700" alt="30" src="https://github.com/user-attachments/assets/78d1c4d8-9eca-4a27-8728-9c0484d67ef8" />

---

### Permission management modal

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
