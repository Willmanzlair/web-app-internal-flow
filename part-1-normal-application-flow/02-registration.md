# Registration
![Web App Registration Flow](../images/registration%20flow.png)

Registration is the process where a new user identity is created inside an application.

From the user’s point of view, registration feels simple. The user enters their details into a form, clicks a button, and expects an account to be created. Internally, however, registration is more than a form submission. The browser collects the input, JavaScript prepares the request, the backend validates the submitted data, the database stores the new user record, and the application decides what should happen next.

That next step is important because registration does not always lead to the same state. Some applications create the account and ask the user to verify their email before login or full access is allowed. Other applications create the account and immediately move the user into an authenticated state.

A clean way to understand registration is:

```text
The frontend collects the registration details.
JavaScript sends those details to the backend.
The backend decides whether the account can be created.
The database stores the new identity.
The backend decides the next state.
The frontend updates the UI based on the response.
```

As a visual summary, the internal flow looks like this:

```text
User submits registration details
↓
JavaScript sends the request
↓
Backend validates the input
↓
Backend creates the account
↓
Database stores the user record
↓
Backend decides what happens next
↓
Frontend updates the UI
```

The important distinction is this:

```text
Authentication is not the creation of a user object.
```

A user object means the application has created or returned selected information about a user. Authentication happens when the backend decides:

```text
“Yes, I trust this identity.”
```

That trust decision may happen immediately after registration, or it may happen later after login, email verification, MFA, or another required step.

A useful mental model is:

```text
Registration = identity creation
Authentication = backend trust decision
Token/session = proof of authenticated state
User object = selected user information returned for frontend rendering and application logic
```

---

## 1. What This Functionality Is

Registration allows a new user to create an account in the application. The application collects identity information from the user, sends it to the backend, validates it, creates a database record, and then decides what should happen next.

At this stage, the application is not only dealing with form input. It is creating a new identity inside the system. That identity may include an email address, name, password hash, role, account status, plan, workspace, organization, tenant, or verification state.

Successful registration usually leads to one of two main outcomes.

In the first outcome, the account is created, but the user must verify their email before full access is allowed. This means the identity exists, but the application has not fully trusted the email address yet.

```text
Account created
↓
User must verify email before login or full access
```

In the second outcome, the account is created and the user is authenticated immediately. This means the backend accepts the new identity, issues authentication proof, and allows the frontend to continue as a logged-in user.

```text
Account created
↓
User is authenticated immediately
```

The second pattern is common in modern web applications, and this is where registration becomes closely connected to login. The user may feel like they simply created an account and entered the app, but internally the backend has done two separate things:

```text
Created the account
Authenticated the new identity
```

Those two actions are connected, but they are not the same thing.

---

## 2. What the User Sees

The user sees a registration form. Typical fields include name, email, password, confirm password, optional username, invite code, organization or workspace name, and sometimes an MFA setup step.

From the user’s perspective, the page is asking for the information needed to create an account. At this stage, the data only exists in the browser as form input. The backend has not received it yet, the database has not stored anything yet, no account has been created yet, and no authenticated state exists yet.

The user may complete the form and click a button such as:

```text
Create Account
Register
Sign Up
Get Started
```

After submission, the visible result depends on the application’s design. Some applications send the user to an email verification screen.

```text
Fill registration form
↓
Click Create Account
↓
Email verification step appears
```

Other applications take the user straight into the dashboard.

```text
Fill registration form
↓
Click Create Account
↓
Dashboard appears
```

The user does not see the internal work happening underneath. They do not see JavaScript building the request, backend validation running, the password being hashed, the database record being created, or the token or session being generated unless those details appear in browser storage or network traffic.

From the user interface alone, registration looks like one action. Internally, it is a chain of decisions.

---

## 3. What JavaScript Does

JavaScript acts as the behavioural layer between the visible registration form and the backend. When the user submits the form, JavaScript captures that action and begins turning the visible input into a structured request.

At this point, JavaScript is not deciding whether the account is valid. It is not deciding whether the user should be trusted, and it is not creating the user inside the database. JavaScript is mainly responsible for collecting the values, packaging them properly, sending them to the correct endpoint, waiting for the response, and updating the page based on what the backend returns.

For example, JavaScript may read the name, email, password, confirm password, invite code, or workspace name from the form. It may perform basic frontend checks, such as confirming that required fields are not empty, then build a JSON body and send it to the registration API endpoint.

A visual summary of that behaviour looks like this:

```text
User clicks Create Account
↓
JavaScript captures the form submission
↓
JavaScript reads the form values
↓
JavaScript builds the request body
↓
JavaScript sends the API request
↓
JavaScript waits for the backend response
↓
Frontend updates based on the response
```

This is where it is important to separate frontend behaviour from backend trust. Frontend validation can improve user experience, but it should not be the real security control.

For example, JavaScript may check whether the password field is empty, but the backend must still enforce password rules. JavaScript may hide a role field from the user, but the backend must still reject unauthorized role values if they are submitted manually.

At this stage, JavaScript is only moving data and reacting to backend output. The backend still has to validate the request, create the user, and decide whether the user should become authenticated.

---

## 4. The API Request

When JavaScript submits the registration form, it usually sends an HTTP request to a registration endpoint. The endpoint name depends on the application, but a common example is:

```http
POST /api/auth/register
```

The request contains the submitted registration data. This submitted data is commonly called the request body or payload.

An example payload may look like this:

```json
{
  "name": "foo",
  "email": "foo@bar.com",
  "password": "Password123"
}
```

A clean distinction is:

```text
Request = the entire HTTP request
Payload / request body = the data being sent inside the request
```

This is the point where the visible form becomes a backend action. Before the request is sent, the data is only in the browser. After the request is sent, the backend receives the submitted information and becomes responsible for deciding whether the account can be created.

From a testing perspective, this request is valuable because it reveals how account creation works at the API level. Useful things to observe include:

* the registration endpoint
* the HTTP method
* the request headers
* the expected request body
* required fields
* optional fields
* hidden fields
* account creation behaviour
* whether registration immediately leads to authentication

The frontend may show only a simple form, but the API request reveals what the backend is actually receiving. That is why registration should always be inspected in the browser Network tab or proxy during testing.

---

## 5. What the Backend Validates

The backend performs the important checks before creating the account. When the registration request arrives, the backend should not trust it simply because it came from the application’s frontend. A tester can modify requests, remove fields, add fields, replay requests, or send requests directly without using the normal form.

Because of this, the backend must validate the submitted data itself.

Typical backend validation may include:

* is the email valid?
* does the email already exist?
* is the password strong enough?
* are required fields missing?
* does the invite code exist?
* does the tenant or organization exist?
* should email verification be required?
* should the user be auto-authenticated after registration?
* are role, plan, or permission values being submitted by the client?
* should any submitted fields be ignored because they are backend-controlled?

The backend receives the request, checks the data, applies business rules, and only then decides whether account creation should happen.

A visual summary of this backend process looks like this:

```text
Backend receives registration request
↓
Backend validates submitted input
↓
Backend checks whether the email already exists
↓
Backend checks password policy
↓
Backend checks business rules
↓
Backend creates the account if valid
↓
Backend decides the next application state
```

This is where the trust decision may begin. If validation succeeds and the application supports immediate login after registration, the backend may decide that the newly created identity can now be trusted as authenticated.

That decision is authentication. The frontend does not make this decision. JavaScript only submitted the registration data.

The backend is responsible for deciding whether the account can be created, whether the identity should be trusted, and whether a token or session should be issued.

---

## 6. What the Database Stores

If validation succeeds, the backend stores user information in the database. The frontend does not talk directly to the database, the browser does not insert the user into the database by itself, and JavaScript does not create the database record directly.

The backend validates the request first, then performs the database action.

The database may store information such as:

* user ID
* name
* email
* password hash
* role
* account status
* verification state
* plan or subscription level
* creation timestamp
* organization or workspace ID
* MFA setup status

The database should not store the plaintext password. It should store a password hash. Later, during login, the backend compares the submitted password against the stored hash using the application’s password verification logic.

A clean mental model is:

```text
Frontend collects the input.
JavaScript sends the request.
Backend validates the request.
Backend writes to the database.
Database stores the user record.
Backend returns selected information.
```

The frontend never sees the full database record. It only sees the selected data the backend chooses to return.

This matters because the database may contain internal fields that should not be exposed to the frontend, such as password hashes, internal flags, reset tokens, verification tokens, privileged role controls, or backend-only account metadata.

The response should only include what the frontend needs for rendering, state management, and normal application behaviour.

---

## 7. What Response Comes Back

After registration succeeds, the backend returns a response to the frontend. That response tells JavaScript what happened and what the frontend should do next.

The response may say that the account was created, that email verification is required, that the user has been authenticated, or that the frontend should call another endpoint such as `/me`.

Common response data may include:

* user object
* access token
* refresh token
* session information
* success state
* verification state
* next-step instruction

A registration response may look like this:

```json
{
  "user": {
    "id": "<user_id>",
    "email": "foo@bar.com",
    "name": "foo",
    "avatar": null,
    "plan": "free",
    "verified": false,
    "role": "user"
  },
  "accessToken": "<access_token>",
  "refreshToken": "<refresh_token>"
}
```

The user object is not the database. It is a selected representation of user data returned to the frontend. Its purpose is to help JavaScript and the UI understand who the user is and what state the account is in.

For example, the frontend may use the response to display:

```text
Good day, foo
Plan: free
Role: user
```

The token or session is different from the user object. The token or session proves authenticated state, while the user object provides identity and display information.

A clean distinction is:

```text
Token/session = proof used for future authenticated requests
User object = selected user details used for frontend state and UI rendering
```

This distinction is important because the frontend may use the user object to render the UI, but the backend should use the token or session to verify authenticated requests.

The frontend should not be trusted simply because it displays a role, plan, or user ID. The backend must still enforce permissions on protected actions.

---

## 8. What Happens After Successful Registration

After registration succeeds, the application needs to decide the next state. This is where registration can branch into different normal patterns.

The account may be created but not fully trusted yet. The user may need to verify their email. The application may authenticate the user immediately. Or the application may issue a token first and then ask `/me` who the current user is.

These are different flows, but they all start from the same core idea:

```text
Registration creates identity.
The backend decides what state comes next.
```

### Pattern 1 — Account Created, Then Verification Required

In this pattern, registration creates the account, but the user must verify their email before full access is granted. The identity exists in the system, but the backend has not yet confirmed that the user controls the submitted email address.

This pattern separates account creation from full trust.

```text
Register
↓
Backend creates user
↓
Verification token is generated
↓
Verification email is sent
↓
User verifies email
↓
Application allows login or unlocks access
```

This flow is common when applications want to reduce fake accounts, protect account recovery, or ensure that the email address belongs to the person registering.

From a testing mindset, the important question is whether verification is actually enforced by the backend, not just shown in the UI.

### Pattern 2 — Immediate Authentication

In this pattern, registration creates the account and immediately authenticates the user. The user appears to create an account and enter the application straight away, but internally the backend has created the user record, accepted the new identity, issued authentication proof, and returned enough information for the frontend to continue as a logged-in user.

```text
Register
↓
Backend creates user
↓
Backend authenticates immediately
↓
Token/session is issued
↓
User object is returned
↓
JavaScript stores auth state
↓
Dashboard renders
```

This means the backend did two things:

```text
Created the account
Authenticated the new identity
```

The registration response already contains what the frontend needs to continue as a logged-in user.

From a testing mindset, the important question is whether this immediate authenticated state is intended, and whether unverified users are restricted from sensitive actions if verification is still false.

### Pattern 3 — Token First, `/me` Second

Some applications authenticate the user but fetch identity information afterward. In this pattern, registration or login succeeds, and the backend issues a token or session first. The frontend then calls `/me` to ask the backend who the current user is.

The token or session proves that the browser has authenticated state. The `/me` response provides the user identity details used by the frontend.

```text
Register or login succeeds
↓
Token/session is issued
↓
Frontend calls /me
↓
Backend validates token/session
↓
Backend returns user object
↓
Dashboard renders
```

Example:

```http
GET /api/auth/me
```

The difference is where the frontend gets the user identity data from.

```text
Immediate response = user information returned inside registration response
/me flow = token/session comes first, user information is fetched afterward
```

Both patterns can represent authenticated state. In the immediate response pattern, the user object comes back inside the registration response. In the `/me` pattern, the token or session comes first, and the frontend later asks the backend:

```text
Who am I?
```

The backend answers through `/me`.

---

## 9. What to Observe From a Security/Testing Mindset

During registration testing, the aim is not only to check whether account creation works. The aim is to understand how identity creation, validation, authentication, session handling, and frontend rendering connect.

Useful questions include:

* What endpoint handles registration?
* What HTTP method is used?
* What parameters are accepted?
* Are hidden or optional fields accepted?
* Can required fields be bypassed?
* Is duplicate email handling correct?
* Are weak passwords allowed?
* Is password confirmation enforced on the backend or only frontend?
* Is email verification required?
* Is email verification actually enforced before sensitive actions?
* Are tokens issued immediately after registration?
* Does registration lead directly into authenticated state?
* Does the response expose role, plan, verification state, internal ID, or tokens?
* Can role or plan values be tampered with in the request?
* Does the frontend trust fields that the backend should control?
* What happens after registration succeeds?
* Does the application call `/me` after registration?
* Does the backend enforce authorization after account creation?

A useful rule is:

```text
You cannot reliably test abnormal behaviour until you understand normal behaviour.
```

The key security lesson is this:

```text
Registration creates identity.
Authentication is the backend trust decision.
Token/session proves authenticated state.
User object helps the frontend render identity and permissions.
```

During testing, this distinction helps separate account creation, authentication, session handling, and authorization into clear parts. That separation matters because each part can fail differently.

Registration may create the account correctly, but email verification may not be enforced. Authentication may issue a token correctly, but the response may expose too much user data. The frontend may hide premium or admin features, but the backend still has to enforce whether the new user is allowed to access them.
