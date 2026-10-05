1. Tech Stack
Languages & Runtimes
JavaScript (ES Modules & JSX): Node.js with "type": "module" on backend (

server/package.json:7
) and React JSX on frontend (

client/src/App.js:1-54
).
HTML5 & CSS3 / CSS Modules: (

client/src/index.css
, 

client/public/index.html
).
Frontend Frameworks & Major Libraries
Source: 

client/package.json:5-42,67-71

React Core: react ^17.0.2, react-dom ^17.0.2, react-scripts 4.0.3 (Create React App)
State Management: redux ^4.1.0, react-redux ^7.2.4, redux-thunk ^2.3.0
Routing: react-router-dom ^5.2.0
UI Component Library: @material-ui/core ^4.11.4, @material-ui/icons ^4.11.2, @material-ui/lab ^4.0.0-alpha.58, @material-ui/pickers ^3.3.10
Charting & Data Visualization: apexcharts ^3.28.1, react-apexcharts ^1.3.9, recharts ^2.0.9, @devexpress/dx-react-chart ^2.7.6, @devexpress/dx-react-chart-material-ui ^2.7.6, @devexpress/dx-react-core ^2.7.6
HTTP Client: axios ^0.21.1
Authentication: @react-oauth/google ^0.9.0, jwt-decode ^3.1.2
Date Utilities: moment ^2.29.1, date-fns ^2.0.0-beta.5, @date-io/date-fns ^1.3.13, react-nice-dates ^3.1.0
UI Helpers / Utilities: react-simple-snackbar ^1.1.11, file-saver ^2.0.5, react-dropzone ^11.3.4, react-tiny-fab ^4.0.3, react-progress-button ^5.1.0, react-multiple-select-dropdown-lite ^2.0.4, uuid ^8.3.2, lodash ^4.17.21
Backend Frameworks & Major Libraries
Source: 

server/package.json:14-27

Web Framework: express ^4.17.1
Database ODM: mongoose ^5.12.10
Authentication & Cryptography: jsonwebtoken ^8.5.1, bcryptjs ^2.4.3
Email Service: nodemailer ^6.6.3
PDF Generation: html-pdf ^3.0.1 (PhantomJS wrapper)
Middleware & Helpers: cors ^2.8.5, dotenv ^8.5.0, moment ^2.29.1
Development Tools: nodemon ^2.0.7
Database
MongoDB: Connected via Mongoose (

server/package.json:22
, 

server/index.js:6,109-111
).
2. Local Setup
Exact Run Commands
Client (Terminal 1):

bash
cd client
npm install
npm start
Citations: 

README.md:96-98
, 

client/package.json:43-48

Server (Terminal 2):

bash
cd server
npm install
npm start
Citations: 

README.md:118-120
, 

server/package.json:8-10

Docker Compose Alternative:

bash
docker-compose -f docker-compose.prod.yml up
Citation: 

docker-compose.prod.yml:1-33

Required Environment Variable Names
Client (placed in client/.env):

REACT_APP_GOOGLE_CLIENT_ID (

README.md:81
, 

client/src/components/Login/Login.js:110
)
REACT_APP_API (

README.md:82
, 

client/src/api/index.js:4
)
REACT_APP_URL (

README.md:83
, 

client/src/components/InvoiceDetails/InvoiceDetails.js:170
)
Server (placed in server/.env):

DB_URL (

README.md:105
, 

server/index.js:106
)
PORT (

README.md:106
, 

server/index.js:107
)
SECRET (

README.md:107
, 

server/controllers/user.js:8
, 

server/middleware/auth.js:5
)
SMTP_HOST (

README.md:108
, 

server/index.js:39
, 

server/controllers/user.js:9
)
SMTP_PORT (

README.md:109
, 

server/index.js:40
, 

server/controllers/user.js:10
)
SMTP_USER (

README.md:110
, 

server/index.js:42
, 

server/controllers/user.js:11
)
SMTP_PASS (

README.md:111
, 

server/index.js:43
, 

server/controllers/user.js:12
)
3. Structure
Top-Level Folders
.git: Git version control metadata and object database.
.github: GitHub platform configuration containing the funding configuration (

FUNDING.yml
).
client: The entire React single-page frontend application including Redux state, Material-UI components, assets, production build files, and Docker configuration (

client/package.json:1-72
).
server: The Node.js Express backend containing REST controllers, Mongoose data models, API routes, PDF and email templates, and server Dockerfile (

server/package.json:1-29
).
10 Most Important Files


server/index.js
 — Primary backend entry point that mounts middleware, database connection, API routes, Nodemailer transport, and PDF generation endpoints.


client/src/App.js
 — Main client router configuring all navigation routes, global layout wrappers, header/footer, and snackbar provider.


client/src/index.js
 — Frontend application bootstrap initializing the Redux store with Thunk middleware and mounting the React root.


client/src/api/index.js
 — Central Axios HTTP client managing base URL, request interceptors for Bearer token authorization, and endpoint callers.


server/controllers/invoices.js
 — Backend controller implementing CRUD operations and user-filtered queries for invoice records in MongoDB.


server/controllers/user.js
 — Authentication controller handling email/password registration, signin with JWT token issuing, and crypto-based password reset.


server/models/InvoiceModel.js
 — Mongoose schema definition for invoices detailing item lists, client details, payment records, currency, and calculation totals.


client/src/components/Invoice/Invoice.js
 — Core frontend form component for creating and updating invoices, managing line items, VAT, and client details.


client/src/components/InvoiceDetails/InvoiceDetails.js
 — Invoice viewer component handling payment history tracking, PDF download triggering, and sending invoices via email.


server/documents/index.js
 — Server-side HTML template used by html-pdf to dynamically generate downloadable and emailable invoice PDF documents.
4. Suspicious / Odd Files & Artifacts


server/invoice.pdf
: A 29KB generated PDF document committed to git. It is the transient file output produced when calling /create-pdf or /send-pdf (

server/index.js:57,88
) and should have been added to .gitignore.


client/build/
: The entire compiled production build directory (HTML, JS bundles, source maps) is committed to git because 

client/.gitignore:1-3
 only ignores node_modules and .env.


client/src/store.js
: A scratch script tracking borrowing/lending of equipment (gps, mistnet, camera, trap, binocular). It is unrelated to invoicing, does not configure Redux (Redux is created in 

client/src/index.js:14
), and is never imported anywhere in the project.


client/src/clients.json
: A 50-row mock client dataset (generated with mock names and fake emails). It was previously used for UI testing but is now commented out in 

client/src/components/Clients/Clients.js:28
 and is dead data.


server/documents/invoice.js
: An older/alternate HTML invoice template that is unused and commented out in 

server/index.js:22
 in favor of 

server/documents/index.js
.


server/middleware/auth.js
: A JWT and Google token verification middleware file that is never imported or attached to any Express route (

server/routes/invoices.js:1-14
, 

server/routes/clients.js:1-12
, 

server/routes/profile.js:1-14
, 

server/index.js:1-116
), making the backend REST API completely unauthenticated.


client/src/components/Login/Google.js
, 

client/src/components/Login/Icon.js
, and 

client/src/components/Settings/Form/icon.js
: Orphaned SVG components that are never imported anywhere.
Broken Docker Configurations:


client/Dockerfile:7
 specifies COPY client/yarn.lock . and RUN yarn install, but no yarn.lock exists in the repository.


server/Dockerfile:14
 specifies CMD ["yarn", "start-prod"], but no start-prod script is defined in 

server/package.json:8-10
.


server/controllers/user.js:122,124
: The password reset email generator contains hardcoded production links (https://accountill.com/reset/${token}) instead of using an environment variable like REACT_APP_URL.
5. Architecture
[ Browser / React SPA ]
   │
   ├─► Redux Store & Thunk Actions ([client/src/actions/](file:///c:/Users/User/hackback/accountill/client/src/actions))
   │     │
   │     └─► Axios Client with Bearer Interceptor ([client/src/api/index.js](file:///c:/Users/User/hackback/accountill/client/src/api/index.js#L6-L12))
   │           │
   │           ▼ (HTTP REST API)
[ Express Server ([server/index.js](file:///c:/Users/User/hackback/accountill/server/index.js#L25-L35)) ]
   ├─► /users, /profiles, /clients, /invoices routes
   ├─► html-pdf -> Render HTML templates to 'invoice.pdf' ([server/documents/index.js](file:///c:/Users/User/hackback/accountill/server/documents/index.js))
   ├─► Nodemailer -> Sends emails with PDF attachment ([server/index.js:38-71](file:///c:/Users/User/hackback/accountill/server/index.js#L38-L71))
   │
   ▼
[ MongoDB Database ] (Models: User, Profile, ClientModel, InvoiceModel)
Frontend (Client):
Built as a Single Page Application (SPA) mounted in 

client/src/index.js:16-21
.
Uses react-router-dom to manage client-side routes such as /dashboard, /invoices, /invoice/:id, /customers, and /settings (

client/src/App.js:31-45
).
Dispatches Redux actions (auth, invoiceActions, clientActions, profile) which call backend endpoints via Axios (

client/src/actions/
).
Backend / API (Server):
Node.js Express server listening by default on port 5000 (

server/index.js:107,110
).
Exposes REST endpoints mounted under /invoices, /clients, /users, and /profiles (

server/index.js:32-35
).
Exposes dedicated PDF generation and email endpoints: /create-pdf, /send-pdf, and /fetch-pdf (

server/index.js:53,87,97
).
Database Layer:
Connected using Mongoose with useNewUrlParser and useUnifiedTopology options (

server/index.js:109
).
Contains 4 core collections/models: User (

server/models/userModel.js:11
), Profile (

server/models/ProfileModel.js:15
), ClientModel (

server/models/ClientModel.js:16
), and InvoiceModel (

server/models/InvoiceModel.js:25
).
Authentication Flow:
Custom Email/Password: Passwords are encrypted using bcrypt.hash(password, 12) during signup (

server/controllers/user.js:57
) and verified with bcrypt.compare() on signin (

server/controllers/user.js:29
). A JWT token expiring in 1 hour is generated and returned to the client (

server/controllers/user.js:34,61
).
Google Sign-In: Uses @react-oauth/google, decodes the credential with jwt-decode, and stores the user profile and token directly on the client (

client/src/components/Login/Login.js:53-62
).
Client Storage & Authorization: Stored in localStorage under key profile and attached to outgoing Axios requests via the Authorization: Bearer <token> header (

client/src/api/index.js:6-12
). (Note: As detailed in Section 4, the backend middleware auth.js is not applied to API routes, so route authorization relies purely on client-supplied queries such as searchQuery).
Observation 1: Total Absence of Route-Level Authentication at the Server Boundary
Claim Checked: Backend endpoints are unprotected by authentication middleware.
Verification:
In 

server/index.js:32-35
, routes are mounted directly as app.use('/invoices', invoiceRoutes), app.use('/clients', clientRoutes), app.use('/users', userRoutes), and app.use('/profiles', profile).
In 

server/routes/invoices.js:1-14
, 

server/routes/clients.js:1-12
, and 

server/routes/profile.js:1-14
, every route handler (GET, POST, PATCH, DELETE) is attached with zero middleware arguments (e.g. router.patch('/:id', updateInvoice) at 

server/routes/invoices.js:10
).


server/middleware/auth.js:7-32
 exists in the repository but is never imported by any router or by server/index.js.
Conclusion: Verified. Any unauthenticated client can make HTTP requests to create, read, update, or delete server records.
Observation 2: Lack of Data Ownership Enforcement (BOLA / IDOR)
Claim Checked: The server enforces resource ownership based on verified identity.
Verification:
updateInvoice: InvoiceModel.findByIdAndUpdate(_id, {...invoice, _id}, { new: true }) (

server/controllers/invoices.js:87
) modifies documents solely by URL parameter _id without verifying the creator.
deleteInvoice: InvoiceModel.findByIdAndRemove(id) (

server/controllers/invoices.js:98
) deletes documents solely by URL parameter id.
updateClient & deleteClient: ClientModel.findByIdAndUpdate(_id, ...) (

server/controllers/clients.js:73
) and ClientModel.findByIdAndRemove(id) (

server/controllers/clients.js:84
) execute unauthenticated, unverified updates/deletions.
getInvoicesByUser: InvoiceModel.find({ creator: searchQuery }) (

server/controllers/invoices.js:13
) and getClientsByUser: ClientModel.find({ userId: searchQuery }) (

server/controllers/clients.js:94
) filter data using unauthenticated query parameters rather than authenticated session context.
Conclusion: Verified. Resource isolation and data ownership are entirely unforced on the server.
Observation 3: Cross-Tenant Data Exposure on GET /clients
Claim Checked: GET /clients exposes all client records across all users.
Verification:
In 

server/routes/clients.js:6
, router.get('/', getClients) maps to getClients in 

server/controllers/clients.js:37-51
.
Lines 44–45 execute:
javascript
const total = await ClientModel.countDocuments({});
const clients = await ClientModel.find().sort({ _id: -1 }).limit(LIMIT).skip(startIndex);
Unlike getClientsByUser (

server/controllers/clients.js:94
), getClients applies no query filter on userId, returning global paginated customer records across the entire database to any unauthenticated caller.
Conclusion: Verified.
Observation 4: Race Condition and Static Overwrite in Server-Side PDF Service
Claim Checked: The PDF creation and download endpoints share a single static file on disk.
Verification:
In 

server/index.js:56-57
 (POST /send-pdf) and 

server/index.js:88
 (POST /create-pdf), html-pdf writes output directly to a fixed filename 'invoice.pdf':
javascript
pdf.create(pdfTemplate(req.body), options).toFile('invoice.pdf', (err) => { ... })
In 

server/index.js:97-99
 (GET /fetch-pdf), the route serves ${__dirname}/invoice.pdf without accepting any document identifier or access token.
The static output file 

server/invoice.pdf
 was committed directly to the git repository.
Conclusion: Verified. Concurrent PDF generations overwrite the same file on disk, leading to data cross-talk where callers fetch the most recently generated invoice.
Observation 5: Correction Regarding Schema Foreign Keys & Relationships
Previous claim: "Relationships between models are linked by Profile.userId, ClientModel.userId, and InvoiceModel.creator referencing User._id."
Correction:
An independent examination of the Mongoose schema definitions in 

server/models/userModel.js:3-11
, 

server/models/ProfileModel.js:3-15
, 

server/models/ClientModel.js:4-16
, and 

server/models/InvoiceModel.js:3-25
 shows that none of the schemas use Mongoose ObjectIds or ref: 'User' references.
Instead, they define these fields as generic arrays of strings:
ProfileModel: userId: [String] (

server/models/ProfileModel.js:12
)
ClientModel: userId: [String] (

server/models/ClientModel.js:9
)
InvoiceModel: creator: [String] (

server/models/InvoiceModel.js:15
)
Furthermore, InvoiceModel.client does not reference ClientModel by ID; it stores a de-normalized embedded subdocument { name: String, email: String, phone: String, address: String } (

server/models/InvoiceModel.js:17
). There is no relational integrity, foreign key constraint, or cascading behavior enforced in the database layer.