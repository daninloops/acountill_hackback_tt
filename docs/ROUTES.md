Critical Server-Wide Finding: Authentication / Authorization Status
Auth Middleware Status: Not found on any route. While an authentication middleware file exists (

server/middleware/auth.js:7-32
), no route file or server entry point imports or applies auth.js (

server/index.js:1-116
, 

server/routes/invoices.js:1-14
, 

server/routes/clients.js:1-12
, 

server/routes/profile.js:1-14
).
Access Control Reality: Any network caller can execute read, create, update, or delete operations across the database without authentication or authorization tokens. Where filtering does occur, it relies strictly on unverified client-supplied query parameters.
1. Invoice Routes
Base prefix: /invoices mounted in 

server/index.js:32
, defined in 

server/routes/invoices.js:1-14

GET /invoices
Controller: getInvoicesByUser (

server/controllers/invoices.js:9-19
)
Description: Returns all invoices created by a specific creator.
Allowed Caller: Public / Any caller (

server/routes/invoices.js:8
).
Inputs: Query parameter req.query.searchQuery (creator ID) (

server/controllers/invoices.js:10
).
Outputs: JSON { data: [Invoice] } (status 200) or error message (status 404) (

server/controllers/invoices.js:15,17
).
Access Control / Flags: ⚠️ Flawed Access Control: No auth check. Scoping is based entirely on the client-supplied query parameter searchQuery; passing any arbitrary creator ID leaks that user's invoices.
GET /invoices/count
Controller: getTotalCount (

server/controllers/invoices.js:22-33
)
Description: Counts the total number of invoices associated with a creator ID.
Allowed Caller: Public / Any caller (

server/routes/invoices.js:6
).
Inputs: Query parameter req.query.searchQuery (creator ID) (

server/controllers/invoices.js:23
).
Outputs: JSON integer total count (status 200) or error message (status 404) (

server/controllers/invoices.js:29,31
).
Access Control / Flags: ⚠️ Flawed Access Control: No auth check; count is based entirely on client-supplied searchQuery.
GET /invoices/:id
Controller: getInvoice (

server/controllers/invoices.js:68-78
)
Description: Fetches a single invoice document by its MongoDB ObjectId.
Allowed Caller: Public / Any caller (

server/routes/invoices.js:7
).
Inputs: URL route parameter req.params.id (

server/controllers/invoices.js:69
).
Outputs: JSON Invoice object (status 200) or error message (status 409) (

server/controllers/invoices.js:74,76
).
Access Control / Flags: ⚠️ Missing Access Control: No authentication or ownership verification; anyone with an invoice ObjectId can read full invoice details.
POST /invoices
Controller: createInvoice (

server/controllers/invoices.js:53-66
)
Description: Creates and persists a new invoice record in the database.
Allowed Caller: Public / Any caller (

server/routes/invoices.js:9
).
Inputs: Request body req.body matching InvoiceSchema fields (

server/models/InvoiceModel.js:3-23
).
Outputs: JSON created Invoice object (status 201) or error message (status 409) (

server/controllers/invoices.js:61,63
).
Access Control / Flags: ⚠️ Missing Access Control: Unauthenticated write; anyone can inject invoices with arbitrary data.
PATCH /invoices/:id
Controller: updateInvoice (

server/controllers/invoices.js:81-90
)
Description: Updates an existing invoice document (e.g. status, line items, payment records).
Allowed Caller: Public / Any caller (

server/routes/invoices.js:10
).
Inputs: URL param id, request body req.body with update fields (

server/controllers/invoices.js:82-83
).
Outputs: JSON updated Invoice object (status 200) or string 'No invoice with that id' (status 404) (

server/controllers/invoices.js:85,89
).
Access Control / Flags: ⚠️ Missing Access Control: No check that the caller owns the invoice or is authenticated.
DELETE /invoices/:id
Controller: deleteInvoice (

server/controllers/invoices.js:93-101
)
Description: Deletes an invoice document from the database.
Allowed Caller: Public / Any caller (

server/routes/invoices.js:11
).
Inputs: URL param id (

server/controllers/invoices.js:94
).
Outputs: JSON { message: 'Invoice deleted successfully' } (status 200) or status 404 (

server/controllers/invoices.js:96,100
).
Access Control / Flags: ⚠️ Missing Access Control: Any unauthenticated user can delete any invoice by ID.
2. Client Routes
Base prefix: /clients mounted in 

server/index.js:33
, defined in 

server/routes/clients.js:1-12

GET /clients
Controller: getClients (

server/controllers/clients.js:37-51
)
Description: Returns paginated client records from the entire database (limit: 8 per page).
Allowed Caller: Public / Any caller (

server/routes/clients.js:6
).
Inputs: Query parameter req.query.page (

server/controllers/clients.js:38
).
Outputs: JSON { data: [Client], currentPage: Number, numberOfPages: Number } (status 200) or error message (status 404) (

server/controllers/clients.js:47,49
).
Access Control / Flags: 🚨 Severe Data Leak / Missing Isolation: Queries ClientModel.find() across the entire collection without user filtering (

server/controllers/clients.js:44-45
), leaking all customers of all users across the system to anyone calling GET /clients.
GET /clients/user
Controller: getClientsByUser (

server/controllers/clients.js:90-100
)
Description: Returns all clients associated with a specific userId.
Allowed Caller: Public / Any caller (

server/routes/clients.js:7
).
Inputs: Query parameter req.query.searchQuery (userId) (

server/controllers/clients.js:91
).
Outputs: JSON { data: [Client] } (status 200) or error message (status 404) (

server/controllers/clients.js:96,98
).
Access Control / Flags: ⚠️ Flawed Access Control: Relies entirely on client-supplied searchQuery without token verification.
POST /clients
Controller: createClient (

server/controllers/clients.js:53-65
)
Description: Saves a new client document to the database.
Allowed Caller: Public / Any caller (

server/routes/clients.js:8
).
Inputs: Request body req.body containing client fields (name, email, phone, address, userId) (

server/models/ClientModel.js:4-14
).
Outputs: JSON created Client object (status 201) or error message (status 409) (

server/controllers/clients.js:61,63
).
Access Control / Flags: ⚠️ Missing Access Control: Unauthenticated write.
PATCH /clients/:id
Controller: updateClient (

server/controllers/clients.js:67-77
)
Description: Modifies an existing client document by its ObjectId.
Allowed Caller: Public / Any caller (

server/routes/clients.js:9
).
Inputs: URL param id, request body req.body with updated client fields (

server/controllers/clients.js:68-69
).
Outputs: JSON updated Client object (status 200) or string 'No client with that id' (status 404) (

server/controllers/clients.js:71,75
).
Access Control / Flags: ⚠️ Missing Access Control: No ownership or authentication check.
DELETE /clients/:id
Controller: deleteClient (

server/controllers/clients.js:79-87
)
Description: Deletes a client document from the database.
Allowed Caller: Public / Any caller (

server/routes/clients.js:10
).
Inputs: URL param id (

server/controllers/clients.js:80
).
Outputs: JSON { message: 'Client deleted successfully' } (status 200) or status 404 (

server/controllers/clients.js:82,86
).
Access Control / Flags: ⚠️ Missing Access Control: Any caller can delete any customer record.
3. User & Authentication Routes
Base prefix: /users mounted in 

server/index.js:34
, defined in 

server/routes/userRoutes.js:1-11

POST /users/signup
Controller: signup (

server/controllers/user.js:46-68
)
Description: Registers a new user account, hashes password via bcrypt (salt rounds: 12), and signs a 1-hour JWT token.
Allowed Caller: Public (

server/routes/userRoutes.js:7
).
Inputs: Request body req.body: { email, password, confirmPassword, firstName, lastName, bio } (

server/controllers/user.js:47
).
Outputs: JSON { result: User, userProfile: Profile|null, token: String } (status 200) or error message (status 400/500) (

server/controllers/user.js:53,55,63,66
).
Access Control / Flags: None needed (public registration endpoint).
POST /users/signin
Controller: signin (

server/controllers/user.js:18-42
)
Description: Validates user credentials against database bcrypt hash and returns a 1-hour JWT token with profile.
Allowed Caller: Public (

server/routes/userRoutes.js:6
).
Inputs: Request body req.body: { email, password } (

server/controllers/user.js:19
).
Outputs: JSON { result: User, userProfile: Profile, token: String } (status 200) or error message (status 400/404/500) (

server/controllers/user.js:27,31,37,40
).
Access Control / Flags: None needed (public credential verification endpoint).
POST /users/forgot
Controller: forgotPassword (

server/controllers/user.js:85-133
)
Description: Generates a 32-byte crypto hex reset token, stores it on the user document with a 1-hour expiration, and dispatches a reset link via Nodemailer.
Allowed Caller: Public (

server/routes/userRoutes.js:8
).
Inputs: Request body req.body: { email } (

server/controllers/user.js:87
).
Outputs: JSON { message: "check your email" } (status 200) or error { error: "User does not exist in our database" } (status 422) (

server/controllers/user.js:111,128
).
Access Control / Flags: None (public recovery flow).
POST /users/reset
Controller: resetPassword (

server/controllers/user.js:137-156
)
Description: Validates the reset token and expiration, hashes new password with bcrypt, and clears reset tokens.
Allowed Caller: Public (holder of valid token) (

server/routes/userRoutes.js:9
).
Inputs: Request body req.body: { password, token } (

server/controllers/user.js:138-139
).
Outputs: JSON { message: "password updated success" } (status 200) or error { error: "Try again session expired" } (status 422) (

server/controllers/user.js:143,150
).
Access Control / Flags: Properly checks token validity and expiration $gt: Date.now() (

server/controllers/user.js:140
).
4. Profile Routes
Base prefix: /profiles mounted in 

server/index.js:35
, defined in 

server/routes/profile.js:1-14

GET /profiles
Controller: getProfilesByUser (

server/controllers/profile.js:69-81
)
Description: Returns the business profile matching the given userId.
Allowed Caller: Public / Any caller (

server/routes/profile.js:8
).
Inputs: Query parameter req.query.searchQuery (userId) (

server/controllers/profile.js:70
).
Outputs: JSON { data: Profile } (status 200) or error message (status 404) (

server/controllers/profile.js:77,79
).
Access Control / Flags: ⚠️ Flawed Access Control: No auth check; returns any user's profile based purely on the searchQuery param.
GET /profiles/:id
Controller: getProfile (

server/controllers/profile.js:18-28
)
Description: Retrieves a single profile document by its ObjectId.
Allowed Caller: Public / Any caller (

server/routes/profile.js:6
).
Inputs: URL route parameter req.params.id (

server/controllers/profile.js:19
).
Outputs: JSON Profile object (status 200) or error message (status 404) (

server/controllers/profile.js:24,26
).
Access Control / Flags: ⚠️ Missing Access Control: Unauthenticated read by ID.
POST /profiles
Controller: createProfile (

server/controllers/profile.js:30-65
)
Description: Creates a new business profile document with business details, logo URL, and userId.
Allowed Caller: Public / Any caller (

server/routes/profile.js:9
).
Inputs: Request body req.body: { name, email, phoneNumber, businessName, contactAddress, logo, website, userId } (

server/controllers/profile.js:31-40
).
Outputs: JSON created Profile object (status 201) or error message { message: "Profile already exist" } (status 404) / status 409 (

server/controllers/profile.js:58,61,63
).
Access Control / Flags: ⚠️ Missing Access Control: Unauthenticated write; caller supplies userId freely.
PATCH /profiles/:id
Controller: updateProfile (

server/controllers/profile.js:101-110
)
Description: Updates profile details by its ObjectId.
Allowed Caller: Public / Any caller (

server/routes/profile.js:10
).
Inputs: URL param id, request body req.body (

server/controllers/profile.js:102-103
).
Outputs: JSON updated Profile object (status 200) or status 404 (

server/controllers/profile.js:105,109
).
Access Control / Flags: ⚠️ Missing Access Control: Any caller can update any profile by ID.
DELETE /profiles/:id
Controller: deleteProfile (

server/controllers/profile.js:113-121
)
Description: Deletes a profile document by its ObjectId.
Allowed Caller: Public / Any caller (

server/routes/profile.js:11
).
Inputs: URL param id (

server/controllers/profile.js:114
).
Outputs: JSON { message: "Profile deleted successfully." } (status 200) or status 404 (

server/controllers/profile.js:116,120
).
Access Control / Flags: ⚠️ Missing Access Control: Any caller can delete any profile by ID.
5. PDF, Email & Root Routes
Defined directly on the Express app in 

server/index.js:53-104

POST /send-pdf
Location: 

server/index.js:53-78
Description: Compiles invoice HTML to a local file invoice.pdf using html-pdf and sends an email via Nodemailer with the PDF attached.
Allowed Caller: Public / Any caller (

server/index.js:53
).
Inputs: Request body req.body containing { email, company, ...invoiceData } (

server/index.js:54,57,60-70
).
Outputs: Resolved promise / status 200 (

server/index.js:76
).
Access Control / Flags: ⚠️ Open Email Proxy & Concurrency Hazard: No authentication. Anyone can call this route to send arbitrary emails through the server's SMTP credentials. Furthermore, it overwrites a single shared file invoice.pdf on the server disk (

server/index.js:57,69
).
POST /create-pdf
Location: 

server/index.js:87-94
Description: Generates an invoice.pdf on the server disk using html-pdf from the provided payload.
Allowed Caller: Public / Any caller (

server/index.js:87
).
Inputs: Request body req.body with invoice data (

server/index.js:88
).
Outputs: Resolved promise / status 200 (

server/index.js:92
).
Access Control / Flags: ⚠️ Missing Access Control & Race Condition: No authentication. Overwrites a single static file invoice.pdf on the filesystem.
GET /fetch-pdf
Location: 

server/index.js:97-99
Description: Returns the static file invoice.pdf from the server root.
Allowed Caller: Public / Any caller (

server/index.js:97
).
Inputs: None.
Outputs: Binary PDF file (invoice.pdf) (

server/index.js:98
).
Access Control / Flags: ⚠️ Information Leak / Race Condition: No parameters or authentication; downloads whatever invoice was most recently generated by any user on the server.
GET /
Location: 

server/index.js:102-104
Description: Health check route returning a plain text string 'SERVER IS RUNNING'.
Allowed Caller: Public (

server/index.js:102
).
Inputs / Outputs: Inputs: None. Outputs: String 'SERVER IS RUNNING' (status 200) (

server/index.js:103
).
Access Control / Flags: Public health endpoint.
Unmounted Server Controller Handlers (Dead / Inactive Endpoints)
The following handler functions are exported in controllers but never mounted on Express routers:

getInvoices (

server/controllers/invoices.js:36-48
) — returns all invoices in MongoDB. Not routed in 

server/routes/invoices.js
.
getClient (

server/controllers/clients.js:24-34
) — fetches client by ID. Not routed in 

server/routes/clients.js
.
getProfiles (

server/controllers/profile.js:8-16
) — returns all profiles in MongoDB. Commented out in 

server/routes/profile.js:7
.
getProfilesBySearch (

server/controllers/profile.js:85-98
) — regex searches profiles by name/email. Not routed in 

server/routes/profile.js
.