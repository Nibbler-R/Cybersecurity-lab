1. Scope

Target: OWASP Juice Shop
Environment: Local Kali Linux VM
Target URL: http://localhost:3000
Testing tool: Burp Suite
Additional tools: Nmap, curl, Firefox/DevTools
Authorization: Deliberately vulnerable local training environment

2. Methodology

Testing covered:

Reconnaissance
HTTP/API enumeration
Authentication
Authorization/access control
Input handling
SQL injection testing
Session behavior
Information disclosure
Product/API enumeration

Testing was performed manually using Burp Suite Repeater and HTTP history, with supporting reconnaissance using Nmap and curl.

3. Findings
Finding 1 — Broken Access Control / IDOR

Endpoint:

GET /rest/basket/{id}

The authenticated test account had:

User ID: 25
Basket ID: 6

Requesting the user's own basket:

GET /rest/basket/6

returned:

200 OK
UserId: 25

Changing only the basket ID:

GET /rest/basket/1

returned:

200 OK
UserId: 1

and exposed the products and quantities contained in that basket.

Impact: An authenticated user can access another user's basket data by manipulating the basket identifier.

Severity: High

Recommendation: The server should verify that the authenticated user owns the requested basket before returning its contents. Authorization should be enforced server-side rather than relying on the client to provide a valid basket ID.

Finding 2 — SQL Injection / Unsafe Query Construction

Endpoint:

GET /rest/products/search?q=

Normal input:

q=test123

returned:

200 OK
{"status":"success","data":[]}

A single quote caused:

500 Internal Server Error

with:

SQLITE_ERROR

The response also exposed the generated SQL query and stack trace.

The application generated SQL containing the supplied input directly, demonstrating unsafe query construction.

Impact:

Database error disclosure
SQL query disclosure
Potential for query manipulation
Potentially greater impact depending on what exploitation is possible

Severity: High / potentially critical depending on exploitability

Important qualification: We did not demonstrate successful arbitrary query manipulation. The controlled boolean-style payload also failed. Therefore the report should not claim database extraction or modification.

Recommendation: Use parameterized queries/prepared statements and never concatenate user-controlled input into SQL. Disable detailed database errors and stack traces in production responses.

Finding 3 — Unauthenticated Application Configuration Disclosure

Endpoint:

GET /rest/admin/application-configuration

The endpoint returned a large JSON configuration response with:

200 OK

It was accessible even after removing:

Authorization: Bearer ...
token=...

The response contained extensive application configuration and metadata.

We specifically checked for:

password
secret
apiKey

and found no matches.

Impact: Unauthenticated users can retrieve internal application configuration and implementation information.

Severity: Medium

Recommendation: Require appropriate authorization for administrative configuration endpoints and expose only configuration that is genuinely required by the client.

4. Negative / Informational Results

These are also useful because they show what you tested and didn't find.

Authentication

The login endpoint:

POST /rest/user/login

returned:

401 Unauthorized
Invalid email or password.

for an incorrect password.

A nonexistent email produced the same response, so we did not demonstrate email/account enumeration.

Session behavior

/rest/user/whoami returned:

{"user":{}}

without authentication.

This endpoint uses 200 OK for both authenticated and unauthenticated states, so the response body—not the HTTP status—determines whether a user is authenticated.

No session vulnerability was established.

Product API
GET /api/Products/1

→ 200 OK

GET /api/Products/9999

→ 404 Not Found

This was normal API behavior and wasn't considered a vulnerability.

Overall Assessment

You successfully identified three meaningful security findings in the lab:

#	Finding	Severity	Status
1	Basket IDOR / Broken Access Control	High	Confirmed
2	SQL Injection / Unsafe Query Construction	High*	Confirmed
3	Unauthenticated Configuration Disclosure	Medium	Confirmed
4	Login enumeration	—	Not found
5	Session vulnerability	—	Not found
6	Product API issue	—	Not found

*The SQL injection finding should be described carefully because we demonstrated error-based injection/query construction, but not successful arbitrary SQL manipulation.
