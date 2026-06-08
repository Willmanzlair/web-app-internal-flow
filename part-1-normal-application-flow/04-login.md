# Login
![Web App Login Flow](../images/login%20flow.png)

Login is the process where an existing user submits credentials so the application can decide whether that user should be treated as authenticated.

From the user’s point of view, login feels simple. The user enters an email or username, types a password, clicks a button, and either enters the application or sees an error. That visible action is only the surface. Internally, several layers are involved: the browser holds the form input, JavaScript prepares and sends the request, the backend validates the credentials, the database provides account facts, and the frontend updates based on the backend response.

The most important distinction is this:

```text
The login page does not authenticate the user.
The backend authenticates the user.
```

The login page is only the interface. It collects input and starts the flow. JavaScript moves the submitted data from the browser to the backend. The actual authentication decision happens later, when the backend receives the request and decides whether the submitted identity should be trusted.

A clean mental model is:

```text
Credentials = data submitted by the user
Authentication = backend trust decision
Session/token = proof of authenticated state
User object = selected identity data returned for frontend rendering
```

Login is therefore not just a form submission. It is a trust decision that begins in the browser but must be completed on the backend.

A simple summary of the internal login flow looks like this:

```text
User submits credentials
↓
JavaScript sends login request
↓
Backend validates credentials
↓
Database provides account facts
↓
Backend decides whether to trust the identity
↓
Session or token may be issued
↓
Frontend updates authenticated state
↓
Authenticated UI renders
```

---

## 1. What This Functionality Is

Login proves control of an existing identity.

During registration, the application creates an account record. During login, the user tries to prove that they are allowed to use that existing account. They usually do this by submitting an identifier, such as an email or username, and a secret, such as a password.

At this stage, the application is not creating a new user. It is checking whether the submitted credentials match a known account and whether that account is allowed to authenticate.

The frontend can collect credentials, JavaScript can submit them, and the interface can display success, failure, or a next step. However, the backend must answer the real security question:

```text
“Do I trust this identity?”
```

If the answer is yes, the backend usually issues proof of authenticated state. That proof may be a session cookie, an access token, a refresh token, or a combination of these mechanisms. This proof is important because future requests need a way to show that the user has already passed authentication. Without it, the application would have to ask for the password again on every protected request.

A useful distinction is:

```text
Registration creates identity.
Login proves control of that identity.
Authentication is the backend trust decision.
Token/session proves authenticated state.
```

This distinction matters because the login page may look like the place where authentication happens, but the page itself is not the authority. The backend is.

---

## 2. What the User Sees

The user sees a login form. Typical fields include an email or username, a password, an optional remember-me checkbox, an optional MFA code field, and sometimes a tenant, workspace, or organization selector.

From the user’s perspective, the page is asking for credentials. They enter the requested values and submit the form by clicking a button such as:

```text
Login
Sign In
Continue
```

At this stage, the submitted data exists only in the browser as form input. The backend has not validated anything yet. No authenticated state exists yet, and no session or token has been issued yet.

After submission, what the user sees depends on the backend response. If login succeeds, the user may be redirected into the authenticated area of the application, such as a dashboard.

```text
Enter credentials
↓
Click Login
↓
Dashboard appears
```

If login fails, the user may see an error message.

```text
Enter credentials
↓
Click Login
↓
Error message appears
```

If MFA is required, the user may not enter the application immediately. Instead, they may be moved into a second verification step.

```text
Enter credentials
↓
Click Login
↓
MFA challenge appears
```

The user does not see the internal checks happening underneath. They do not see the backend finding the account record, comparing the submitted password with the stored password hash, checking account status, checking MFA requirements, issuing a token, setting a cookie, or returning selected user data.

From the interface alone, login looks like one action. Internally, it is a chain of validation, state handling, and trust decisions.

---

## 3. What JavaScript Does

JavaScript acts as the behavioural layer between the visible login form and the backend.

When the user submits the form, JavaScript captures that action and turns the visible input into a structured request. It may read the email field, read the password field, include optional values such as `remember`, then send the data to the login endpoint.

At this point, JavaScript is not authenticating the user. It is not comparing the password with the real account record, and it is not deciding whether the submitted identity should be trusted.

JavaScript is mainly responsible for:

* capturing the form submission
* reading the form values
* preparing the request body
* sending the request to the backend
* waiting for the backend response
* updating the interface based on that response

Example request body data may look like this:

```json
{
  "email": "foo@example.com",
  "password": "Password123"
}
```

The important point is that JavaScript moves the authentication attempt forward, but it should not be the final authority. A frontend-only decision can be bypassed. A tester can modify JavaScript, replay requests, change request bodies, remove fields, add fields, or send requests directly through tools such as Burp Suite.

Because of this, the backend still has to validate the credentials and decide whether the identity should be trusted.

A visual summary of the frontend behaviour looks like this:

```text
User submits login form
↓
JavaScript captures the submit event
↓
JavaScript reads the email or username field
↓
JavaScript reads the password field
↓
JavaScript prepares the request body
↓
JavaScript sends the login request
↓
JavaScript waits for the backend response
↓
Frontend updates based on the response
```

This is where frontend behaviour and backend trust must be separated clearly. JavaScript can submit credentials, display errors, handle redirects, store or recognize authentication state, and render the correct interface after the backend responds. But JavaScript should not be the component that decides whether the user is truly authenticated.

---

## 4. The API Request

When JavaScript submits the login form, it usually sends an HTTP request to an authentication endpoint.

A common login request may look like this:

```http
POST /api/auth/login
Content-Type: application/json
```

The request body contains the submitted credentials.

Example request body:

```json
{
  "email": "foo@example.com",
  "password": "Password123",
  "remember": true
}
```

`POST` is commonly used because the user is submitting data and triggering authentication logic. The request is not simply asking for a page. It is asking the backend to process credentials and decide whether authenticated state should be created.

Before the request is sent, the credentials exist only in the browser. After the request is sent, the backend receives those credentials and becomes responsible for validation. This is the point where the visible login form becomes an actual backend authentication attempt.

From a testing perspective, this request is valuable because it reveals how authentication works at the API level.

Useful things to observe include:

* the authentication endpoint
* the HTTP method
* the request headers
* the expected request format
* the accepted parameters
* the content type
* optional fields such as `remember`, `tenant`, `redirect`, `clientId`, or `mfaCode`

The frontend shows a simple form, but the API request shows what the backend is actually receiving. That request can then be studied in browser DevTools, captured in Burp Suite, replayed, modified, and compared against the visible behaviour of the application.

---

## 5. What the Backend Validates

The backend receives the login request and performs the actual authentication checks.

The backend should not trust the request simply because it came from the application frontend. The frontend is not a trusted security boundary. A user or tester can send the same request manually, change values, add unexpected fields, remove required fields, replay older requests, or bypass the visible page completely.

Because of this, the backend must validate the submitted data and the account state itself.

The backend may check:

* whether the user exists
* whether the submitted password matches the stored password hash
* whether the account is active, locked, disabled, or unverified
* whether MFA is required
* whether rate limits or brute-force protections should apply
* whether the tenant, workspace, or organization context is valid
* whether the user has a valid role or permission set
* whether a session or token should be created

The database may store account facts, but the backend makes the authentication decision. For example, the database may store the password hash, but the backend performs the password comparison. The database may store that MFA is enabled, but the backend decides whether the user must complete MFA before receiving full authenticated access.

If validation succeeds, the backend may decide:

```text
“This identity is valid and can now be trusted as authenticated.”
```

That decision is authentication.

A simplified backend validation flow looks like this:

```text
Backend receives login request
↓
Backend finds user by email or username
↓
Backend compares submitted password with stored password hash
↓
Backend checks account status
↓
Backend checks MFA requirement
↓
Backend loads role or permissions where needed
↓
Backend creates session or token if valid
↓
Backend returns response
```

This is the critical trust boundary in the login flow. The frontend submitted the credentials, but the backend decides whether those credentials are valid, whether the account is allowed to log in, and whether authenticated state should be issued.

---

## 6. What the Database Stores or Retrieves

During login, the backend may query the database for the submitted user identity.

The browser does not talk directly to the database. JavaScript does not retrieve the full account record and compare the password by itself. The backend receives the login request first, then retrieves only the data it needs to make the authentication decision.

The database may return information such as:

* user ID
* email or username
* password hash
* account status
* role
* permissions
* MFA status
* verification state
* failed login counters
* last login timestamp

The database does not authenticate the user by itself. The database stores facts, the backend uses those facts to make the decision, and the frontend displays the result.

A clean way to think about this is:

```text
Database stores the facts.
Backend makes the decision.
Frontend displays the result.
```

For example, the database may store that an account is unverified, but the backend decides whether unverified accounts are allowed to log in. The database may store failed login counters, but the backend decides whether another failed attempt should trigger a lockout, delay, or rate limit.

The frontend should not receive the full database record. It should only receive selected information that the backend chooses to expose for frontend rendering. Sensitive fields such as password hashes, reset tokens, verification tokens, internal lockout metadata, privileged role controls, backend-only flags, and secret configuration values should never be exposed to the browser.

This matters because the browser is controlled by the user. Anything returned to the frontend can be inspected.

---

## 7. What Response Comes Back

After the backend processes the login request, it returns a response that tells JavaScript what happened.

In modern web applications, this response is commonly JSON. The response may indicate success, failure, MFA requirement, lockout, unverified account status, or another next step.

If authentication succeeds, the response may include:

* user object
* access token
* refresh token
* session metadata
* role information
* permission flags
* onboarding state
* redirect instruction

Example response:

```json
{
  "user": {
    "id": 101,
    "email": "foo@example.com",
    "role": "admin",
    "verified": true
  },
  "accessToken": "jwt_here"
}
```

In cookie-based applications, the authenticated session may be set through a `Set-Cookie` response header instead of being returned directly in the JSON body.

For example:

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax
```

The user object and the token or session do different jobs.

A clean distinction is:

```text
Token/session = proves authenticated state
User object = helps the frontend understand who the user is
```

The token or session is used as proof for future authenticated requests. The user object is selected identity data returned to help the frontend render the correct interface. For example, if the response contains `role`, `plan`, `verified`, or `permissions`, JavaScript may use those values to decide what to show in the UI.

However, frontend rendering is not the same as backend authorization. The frontend may hide or show buttons based on the user object, but the backend must still enforce access control on later requests. A hidden button is not a security control if the backend still accepts the request.

---

## 8. What Happens After the Action Succeeds

After receiving the response, JavaScript decides what should happen next in the interface.

If authentication succeeds, JavaScript may store or recognize authenticated state, update the current user context, follow a redirect, request additional account data, call `/api/auth/me`, or render the authenticated area of the application.

If authentication fails, JavaScript may display an error message, highlight invalid fields, clear sensitive input, show a lockout message, or allow the user to try again.

If MFA is required, JavaScript may not render the dashboard yet. Instead, it may move the user into an MFA challenge flow.

The important point is that JavaScript reacts to backend output. The backend decides whether the login is valid, while JavaScript decides how to reflect that result in the interface.

A clean summary of the successful login flow is:

```text
User submits credentials
↓
JavaScript sends login request
↓
Backend validates credentials
↓
Backend returns session, token, user data, or next-step instruction
↓
JavaScript updates auth state
↓
Frontend renders authenticated view or continues the required flow
```

Some applications return the user object immediately in the login response. Other applications issue the token or session first, then make a separate request to ask:

```text
Who am I?
```

That second request is commonly handled by an endpoint such as:

```http
GET /api/auth/me
```

The difference is not whether the user is authenticated. The difference is where the frontend gets the user identity data from. In one pattern, the user data comes back inside the login response. In another pattern, authentication proof comes first, and the frontend fetches user identity data afterward through `/me`.

---

## 9. Invisible Login Logic: Behavioural Model

The visible login experience may look simple:

```text
Click Login
↓
Dashboard appears
```

But the visible experience is only the surface. Underneath, JavaScript is usually following a behavioural model. This model is not literal application code. It is a way to understand what the frontend is trying to accomplish.

```text
When login form is submitted:
    read email or username field
    read password field
    include optional login values if present
    send POST /api/auth/login
    wait for backend response

if authentication succeeds:
    receive session, token, user object, or next-step instruction
    store or recognize authenticated state
    update current user context
    redirect or render dashboard

if MFA is required:
    show MFA step
    wait for MFA verification
    complete authentication flow if valid

if authentication fails:
    display authentication error
```

This model is useful because frontend behaviour often reveals backend assumptions. If JavaScript sends a request to `/api/auth/login`, that tells the tester that an authentication endpoint exists, a request structure exists, specific parameters are expected, and backend login logic is exposed through that route.

That endpoint can then be studied through DevTools, Burp Suite, replayed requests, modified parameters, response analysis, MFA behaviour, token handling, session handling, and authorization checks.

---

## 10. What to Observe From a Security/Testing Mindset

During login testing, the goal is not only to check whether login works.

The goal is to understand how the authentication flow behaves: what the frontend sends, what the backend validates, what the backend trusts, what response comes back, how authenticated state is stored, and whether authorization is enforced after login.

Useful observations include:

* What endpoint receives the login request?
* What HTTP method is used?
* What parameters are accepted?
* Are extra fields present in the request?
* Are fields like `remember`, `tenant`, `redirect`, `clientId`, or `mfaCode` accepted?
* Does the response expose roles, permissions, tokens, or internal IDs?
* Is the session stored in a cookie, local storage, session storage, or memory?
* Is the cookie protected with `HttpOnly`, `Secure`, and `SameSite` attributes?
* Is MFA required before or after the initial password check?
* Does login return a full authenticated token before MFA is completed?
* Does the backend enforce rate limits?
* Are error messages too specific?
* Does the frontend show behaviour that the backend does not enforce?
* Can the request be replayed, modified, or tested through Burp Suite?
* Does the application call `/me` after login?
* Does the backend enforce authorization after login?

The key security lesson is this:

```text
The frontend does not authenticate the user.
The frontend submits authentication data.
The backend validates it.
The session or token proves authenticated state.
```

During testing, this distinction guides what to inspect. You observe what the frontend sends, what the backend validates, what the backend trusts, what the response exposes, how authenticated state is stored, and whether access control is enforced server-side after login.

