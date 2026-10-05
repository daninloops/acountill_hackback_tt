Rank 1: Total Absence of Authentication and Authorization on REST API Endpoints (BOLA / IDOR Across All Data)
Evidence in Code
Unused Middleware: The authentication middleware file exists (

server/middleware/auth.js:7-32
), but it is never imported or attached to any Express route or router (

server/routes/invoices.js:1-14
, 

server/routes/clients.js:1-12
, 

server/routes/profile.js:1-14
, 

server/index.js:32-35
).
Documentation Discrepancy: The README claims "Authentication using jsonwebtoken (jwt)" (

README.md:40,62
).
Direct Object References: Every mutation and read by ID operates directly on the model without verifying ownership or identity:
updateInvoice: InvoiceModel.findByIdAndUpdate(_id, {...invoice, _id}, { new: true }) (

server/controllers/invoices.js:87
)
deleteInvoice: InvoiceModel.findByIdAndRemove(id) (

server/controllers/invoices.js:98
)
updateClient & deleteClient: ClientModel.findByIdAndUpdate / findByIdAndRemove (

server/controllers/clients.js:73,84
)
updateProfile & deleteProfile: ProfileModel.findByIdAndUpdate / findByIdAndRemove (

server/controllers/profile.js:107,118
)
Client-Controlled User Scoping: Filtered endpoints rely entirely on unvalidated query strings supplied by the client:
InvoiceModel.find({ creator: searchQuery }) (

server/controllers/invoices.js:13
)
ClientModel.find({ userId: searchQuery }) (

server/controllers/clients.js:94
)
ProfileModel.findOne({ userId: searchQuery }) (

server/controllers/profile.js:75
)
What Should Exist Instead
All private routes (/invoices/*, /clients/*, /profiles/*) should require the auth middleware. The verified user ID should be extracted securely from req.userId (derived from the validated JWT signature) rather than query parameters, and every database query/mutation must include { _id, creator: req.userId } or { _id, userId: req.userId } to ensure users can only access their own resources.

Rank 2: Cross-Tenant Data Leak via Unfiltered Global Customer Listing (GET /clients)
Evidence in Code
In 

server/controllers/clients.js:37-51
, the getClients handler implements pagination:
javascript
const total = await ClientModel.countDocuments({});
const clients = await ClientModel.find().sort({ _id: -1 }).limit(LIMIT).skip(startIndex);
This handler is bound directly to GET /clients (

server/routes/clients.js:6
).
Unlike getClientsByUser which at least attempts to filter on userId: searchQuery (

server/controllers/clients.js:94
), getClients runs an empty query {} across the entire MongoDB collection.
What Should Exist Instead
The GET /clients route should authenticate the requester and restrict the database query strictly to the authenticated user's scope: ClientModel.find({ userId: req.userId }).

Rank 3: Open Email Relay and Shared-File Concurrency Hazard on PDF Services (/send-pdf, /create-pdf, /fetch-pdf)
Evidence in Code
Unauthenticated Email Proxy: POST /send-pdf (

server/index.js:53-78
) allows any caller to pass arbitrary recipient email, sender company.email, subject, and HTML body content to transporter.sendMail() (

server/index.js:60-71
) using the server's configured SMTP credentials.
Race Condition & Data Overwrite: Both POST /create-pdf (

server/index.js:88
) and POST /send-pdf (

server/index.js:57
) write to a single static file path: 'invoice.pdf' (${__dirname}/invoice.pdf).
Data Cross-Talk on Download: GET /fetch-pdf (

server/index.js:97-99
) takes no ID or parameters and simply responds with res.sendFile('${__dirname}/invoice.pdf'). If multiple requests occur concurrently, users download or email whichever invoice was written to disk most recently.
Committed File: The generated artifact 

server/invoice.pdf
 was committed directly to the repository instead of being ignored.
What Should Exist Instead
The email endpoint should verify user authentication and ensure the invoice being sent belongs to the sender.
PDF generation should stream the buffer directly to the HTTP response or to Nodemailer in memory (pdf.create(...).toStream(...) or toBuffer(...)) rather than writing to a single, static shared file path on the disk.
Rank 4: Missing Server-Side Validation and Mathematical Integrity Checks on Financial Calculations
Evidence in Code
Blind Object Persistence: In createInvoice (

server/controllers/invoices.js:53-66
) and updateInvoice (

server/controllers/invoices.js:81-90
), req.body is saved directly to MongoDB without validating or recalculating financial figures:
javascript
const invoice = req.body
const newInvoice = new InvoiceModel(invoice)
await newInvoice.save()
Unverified Fields: subTotal, total, vat, totalAmountReceived, and status are calculated on the client (

client/src/components/Invoice/Invoice.js:1-250
) and accepted as raw input by the server.
Payment Record Integrity: In InvoiceModel.paymentRecords (

server/models/InvoiceModel.js:18
), amountPaid and datePaid are appended without server-side validation against total or prior payments.
What Should Exist Instead
The server should iterate over items (quantity * unitPrice - discount), compute subTotal, add vat, and calculate total on the backend. When recording payments, the server should recalculate totalAmountReceived and derive status (PAID, PARTIALLY PAID, UNPAID) based on verified transactions rather than trusting client payloads.

Rank 5: Hardcoded Production Domain in Password Reset Flow
Evidence in Code
In forgotPassword (

server/controllers/user.js:85-133
), the generated reset link is hardcoded to a remote production URL:
<a href="https://accountill.com/reset/${token}">link</a> (

server/controllers/user.js:122
)
https://accountill.com/reset/${token} (

server/controllers/user.js:124
)
Environment Inconsistency: Unlike API URLs configured via dotenv (

server/controllers/user.js:5-12
), this host URL is hardcoded, breaking local development and self-hosted environments.
What Should Exist Instead
The frontend URL base should be parameterized via an environment variable (e.g. process.env.CLIENT_URL or APP_URL) and used when constructing the password reset email body: ${process.env.CLIENT_URL}/reset/${token}.