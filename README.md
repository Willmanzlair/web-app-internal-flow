# Web Application Internal Flow

## Frontend → JavaScript → Backend/API → Database → UI

## Objective

This repo was created to understand and internalize how a modern web application works internally, from frontend interaction through JavaScript behaviour, backend processing, authentication, API communication, database activity, and frontend rendering.

This repo uses a small learning application, `reverse-eng-app`, as a practical reference point. However, the notes are written generically so the concepts can apply to most modern web applications.

The goal is not simply to memorize endpoints, requests, responses, or tools. The goal is to understand how a web application behaves as a connected system, how each layer communicates with the next, and how normal application flow can be mapped during security testing.

This understanding supports several important areas of offensive security, including JavaScript hunting, API testing, authentication analysis, authorization analysis, session testing, behavioural testing, business logic testing, and frontend-to-backend mapping.

A useful principle in offensive security is:

```text
You cannot reliably identify abnormal behaviour until you understand normal behaviour.
```

Modern web applications often expose a significant amount of behaviour through the frontend. JavaScript files, browser requests, API responses, client-side routes, hidden functionality, and application state can reveal how the backend is expected to behave.

For this reason, JavaScript hunting is often the process of reverse engineering frontend logic to predict backend behaviour.

---

## How to Read This Repo

This repo follows a normal user journey through a web application. Start with this README, then follow the numbered files in order.

Each file focuses on one application function and explains what happens from the visible user action, through JavaScript behaviour, API communication, backend validation, database activity, returned responses, frontend rendering, and security testing observations.

The aim is to help the reader understand the application as a connected system, not as isolated pages. A login page, dashboard, search box, upload button, profile setting, or logout button may look simple in the browser, but internally each one may involve frontend state, JavaScript logic, API communication, backend decisions, database reads or writes, authentication checks, authorization checks, and security boundaries.

This repo is written to make that internal movement easier to see.

---

## The Core Mental Model of a Web Application

A modern web application is not just a page displayed in a browser. What the user sees is the visible part of a larger system.

A button, form, dashboard, search box, or upload area may look like a simple interface element, but behind that interface several layers are working together. The frontend presents the page and collects user input. JavaScript listens for user actions, prepares data, sends requests, receives responses, and updates what the user sees.

The backend receives those requests, validates the submitted data, checks authentication and authorization, applies business logic, and decides what should happen next. The database stores or retrieves the application data that the backend needs in order to complete the action.

This means that every visible action in the browser usually has an internal journey behind it.

For example, when a user logs in, the browser does not simply “log the user in” by itself. The frontend collects the email and password, JavaScript sends those values to a login endpoint, and the backend checks whether the account exists. The backend may fetch the stored password hash, compare the submitted password against that hash, check the account state, create or return authentication proof, and send a response back to the frontend.

The frontend then uses that response to update the application state and display the authenticated interface.

The same idea applies to registration, email verification, MFA, dashboard loading, search, file upload, profile updates, invoices, team management, logout, and post-logout behaviour.

In summary, the internal movement usually looks like this:

```text
User interaction
↓
JavaScript behaviour
↓
API request
↓
Backend processing
↓
Database action
↓
Backend response
↓
JavaScript rendering
↓
UI update
```

In simplified form:

```text
Frontend ⇄ Backend/API ⇄ Database
```

A clean way to think about it is:

```text
Frontend collects input.
JavaScript turns behaviour into requests.
Backend makes decisions.
Database stores or returns data.
Frontend displays the result.
```

This model matters because the frontend may show a button, form, dashboard, or admin panel, but the backend should decide what is allowed, what data is returned, and whether the action should actually happen.

From a security testing perspective, the interface is the starting point, but the request, response, backend decision, and state change are where deeper testing begins.

---

## Understanding the Major Components

Before studying endpoints and requests, it is important to understand the role of each component in the application flow.

---

## Frontend

The frontend is the visible interface presented to the user inside the browser. It includes elements such as forms, buttons, search bars, navigation menus, dashboards, profile pages, tables, modals, and other interactive components.

The frontend is usually responsible for displaying information, collecting user input, triggering actions, and rendering updates based on application state.

Examples of frontend interaction include:

* a user clicks “Login”, “Submit”, “Delete”, “Export”, or “Invite user”
* a user types credentials, profile details, payment information, or account settings
* a user enters a search keyword and expects filtered or returned results
* a user opens a dashboard and expects their data, role, permissions, and available features to load

The key distinction is that the frontend does not make the final trust decision. It may collect input and present results, but backend logic should be responsible for validation, authorization, and enforcement.

From a security testing perspective, the frontend should be treated as a guide, not as the source of truth.

---

## JavaScript

JavaScript is the behavioural layer of the application. It connects user interaction to backend functionality by capturing events, preparing data, sending API requests, processing responses, and updating the interface based on the result.

If the frontend is what the user sees, JavaScript is what causes the interface to behave dynamically.

JavaScript commonly controls:

* button behaviour
* form submission
* input validation
* API communication
* authentication state
* redirects
* dynamic page updates
* frontend route changes
* rendering of user-specific data
* conditional display of features

For example, when a user clicks a login button, the button itself is only an interface element. JavaScript defines what happens after that click: which form values are read, which endpoint receives the request, what request method is used, and how the response affects the page.

A simplified behavioural model looks like this:

```text
User performs an action
↓
JavaScript captures the event
↓
JavaScript prepares data
↓
JavaScript sends an API request
↓
Backend processes the request
↓
JavaScript receives the response
↓
Frontend updates
```

This behaviour is often invisible to the normal user. The user only sees that a button was clicked, a page changed, or data appeared on the screen.

During security testing, however, the important details are found underneath the interface: requests, responses, parameters, tokens, cookies, roles, permissions, redirects, errors, and state changes.

---

## Backend/API

The backend is the server-side logic responsible for processing requests and enforcing application rules. It receives API requests from the frontend, validates data, checks authentication, applies authorization rules, interacts with the database, and returns structured responses.

The API acts as the communication layer between the frontend and backend. In many modern applications, the frontend does not directly load a new page for every action. Instead, it sends requests to API endpoints such as:

```http
POST /api/auth/login
GET /api/auth/me
PUT /api/profile
GET /api/users
DELETE /api/invitations/123
```

These endpoints expose backend functionality. Each endpoint represents a behaviour the application supports, such as logging in, retrieving the current user, updating a profile, listing users, deleting a resource, loading dashboard data, or changing account settings.

For offensive security, this is where the application becomes especially interesting. Every request raises useful questions:

* What endpoint is being called?
* What HTTP method is used?
* What parameters are sent?
* What does the backend validate?
* What does the backend trust?
* What data is returned?
* What role or permission is required?
* Can the request be modified and replayed?

The frontend may suggest how the application should be used, but the API shows how the application actually communicates.

---

## Database

The database stores the application’s persistent data. This may include users, roles, permissions, sessions, profiles, orders, messages, files, billing records, audit logs, or any other data the application needs to function.

The frontend does not usually communicate with the database directly. Instead, the backend receives a request, decides whether the action is valid, and then queries or updates the database as needed.

Example:

```http
GET /api/auth/me
```

The backend may use the session or token to identify the user, query the database for that user’s details, and return selected information to the frontend.

The frontend only sees the final response. The database activity itself remains server-side.

This is important during testing because the response may reveal what the backend chose to expose, but it does not always reveal everything that was checked, queried, updated, or denied behind the scenes.

---

## Repo Structure

```text
web-app-internal-flow/
├── README.md
├── images/
│   ├── registration-flow.png
│   └── login-flow.png
│
├── part-1-normal-application-flow/
│   ├── 01-landing-page.md
│   ├── 02-registration.md
│   ├── 03-email-verification.md
│   ├── 04-login.md
│   ├── 05-login-response-and-token.md
│   ├── 06-mfa-2fa.md
│   ├── 07-session-me.md
│   ├── 08-config.md
│   ├── 09-dashboard-load.md
│   ├── 10-teams.md
│   ├── 11-files.md
│   ├── 12-notifications.md
│   ├── 13-invoices.md
│   ├── 14-search.md
│   ├── 15-profile-settings.md
│   ├── 16-change-password.md
│   ├── 17-logout.md
│   └── 18-post-logout-behaviour.md
│
└── part-2-recovery-challenge-and-exception-flows/
    ├── 19-forgot-password.md
    ├── 20-reset-password.md
    ├── 21-email-verification-edge-cases.md
    ├── 22-mfa-edge-cases.md
    ├── 23-token-session-edge-cases.md
    └── 24-behavioural-challenge-mode.md
```

---

## Part 1: Normal Application Flow

Part 1 follows the clean path a user commonly takes through a modern application. This is the expected journey: the user arrives, creates or accesses an account, proves identity, receives authenticated state, loads the application, uses features, changes settings, and eventually logs out.

1. [Landing Page](part-1-normal-application-flow/01-landing-page.md)
2. [Registration](part-1-normal-application-flow/02-registration.md)
3. [Email Verification](part-1-normal-application-flow/03-email-verification.md)
4. [Login](part-1-normal-application-flow/04-login.md)
5. [Login Response and Token](part-1-normal-application-flow/05-login-response-and-token.md)
6. [MFA / 2FA](part-1-normal-application-flow/06-mfa-2fa.md)
7. [Session Restoration and `/me`](part-1-normal-application-flow/07-session-me.md)
8. [Config](part-1-normal-application-flow/08-config.md)
9. [Dashboard Load](part-1-normal-application-flow/09-dashboard-load.md)
10. [Teams](part-1-normal-application-flow/10-teams.md)
11. [Files](part-1-normal-application-flow/11-files.md)
12. [Notifications](part-1-normal-application-flow/12-notifications.md)
13. [Invoices](part-1-normal-application-flow/13-invoices.md)
14. [Search](part-1-normal-application-flow/14-search.md)
15. [Profile Settings](part-1-normal-application-flow/15-profile-settings.md)
16. [Change Password](part-1-normal-application-flow/16-change-password.md)
17. [Logout](part-1-normal-application-flow/17-logout.md)
18. [Post-Logout Behaviour](part-1-normal-application-flow/18-post-logout-behaviour.md)

---

## Part 2: Recovery, Challenge, and Exception Flows

Part 2 contains flows that do not always follow the clean straight-line user journey. These topics cover recovery paths, failed states, expired states, repeated actions, interrupted authentication, suspicious behaviour, and security challenges.

19. [Forgot Password](part-2-recovery-challenge-and-exception-flows/19-forgot-password.md)
20. [Reset Password](part-2-recovery-challenge-and-exception-flows/20-reset-password.md)
21. [Email Verification Edge Cases](part-2-recovery-challenge-and-exception-flows/21-email-verification-edge-cases.md)
22. [MFA Edge Cases](part-2-recovery-challenge-and-exception-flows/22-mfa-edge-cases.md)
23. [Token and Session Edge Cases](part-2-recovery-challenge-and-exception-flows/23-token-session-edge-cases.md)
24. [Behavioural Challenge Mode](part-2-recovery-challenge-and-exception-flows/24-behavioural-challenge-mode.md)

This split keeps the repo clean:

```text
Part 1 teaches normal behaviour.
Part 2 teaches what happens when the normal flow breaks, changes, expires, repeats, or gets challenged.
```

---

## Standard Note Structure

Each individual note in this repo should follow the same teaching structure:

```text
1. What this functionality is
2. What the user sees
3. What JavaScript does
4. The API request
5. What the backend validates
6. What the database stores or retrieves
7. What response comes back
8. What happens after the action succeeds
9. What to observe from a security/testing mindset
```

This keeps every note consistent and helps the reader understand where the data currently is, which layer is acting, what has already happened, what has not happened yet, what the backend is responsible for, what the frontend is only displaying, and where security decisions should be enforced.

---

## Current Status

This repo is being built gradually as each topic is studied and documented.

The current starting section covers the beginning of the authentication journey:

```text
Landing Page
↓
Registration
↓
Email Verification
↓
Login
↓
Login Response and Token
↓
MFA / 2FA
↓
Session Restoration and /me
```

More files will be added as the study progresses.

---

## Final Mental Model

A modern web application is a chain of connected behaviour. The user may only see a button, form, page, table, or dashboard, but internally that visible action may pass through JavaScript behaviour, an API request, backend validation, database reads or writes, and a response that the frontend uses to update the interface.

For security testing, every visible feature should be mapped back to its internal behaviour. The goal is not just to know that a feature exists. The goal is to understand:

```text
What request powers it?
What data does it send?
What does the backend check?
What does the database store or return?
What response comes back?
What does the frontend do with that response?
What security boundary exists there?
```

That is the foundation of strong web application testing.

Understand the normal flow first. Then test where the flow can be bent, skipped, replayed, confused, or abused.
