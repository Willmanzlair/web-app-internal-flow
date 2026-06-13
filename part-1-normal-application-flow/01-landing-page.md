# Landing Page

## 1. What This Functionality Is

The landing page is the first visible part of a web application before the user logs in or creates an account.

It may appear to be a simple homepage, but from a security testing perspective, it can act as a fast pre-authentication reconnaissance point. The landing page is not always deeply important, and it is not always where meaningful vulnerabilities will be found. Its value depends on how much information, behaviour, and attack surface the application exposes before authentication.

A useful way to think about it is:

```text
Useful? Yes.
Important? Sometimes.
Necessary? No.
It depends on the app.
```

The landing page should not be treated as automatically critical, but it should not be ignored either. Sometimes it reveals useful clues about the application’s structure, authentication model, user roles, public routes, exposed JavaScript, integrations, or future testing areas. Other times, it reveals almost nothing beyond the fact that a login page exists.

A clean mental model is:

```text
Landing page ≠ always important
Landing page = fast pre-auth reconnaissance opportunity
```

At this stage, the user is usually unauthenticated. Even so, the application may still load HTML, CSS, JavaScript, images, fonts, tracking scripts, public configuration, frontend routes, and sometimes public API responses. These resources can give the tester early visibility into how the application is built and what deeper flows may exist after login.

The landing page is not usually where deep testing happens. Its main value is orientation. It helps the tester understand the public face of the application before moving into registration, login, password reset, sessions, dashboard loading, or authenticated functionality.

Useful early questions include:

```text
What kind of application is this?
What functionality probably exists?
What authentication model may be used?
What features are visible before login?
What routes or workflows are exposed?
What architecture clues are visible?
What should I expect after login?
What attack surface exists pre-auth?
```

At this point, the tester is not trying to prove a vulnerability. The goal is to understand what the application appears to be, what it exposes publicly, and which areas may deserve attention later.

---

## 2. What the User Sees

From the user’s perspective, the landing page may show public-facing information and entry points into the application. Common visible elements include login, registration, forgot password, pricing, documentation, support, product features, demo buttons, contact forms, SSO options, enterprise messaging, or team-related features.

Some landing pages expose a lot before authentication. This type of landing page can be useful because it may reveal the application’s purpose, feature set, user model, authentication options, and likely backend structure before the tester even logs in.

A high-value landing page may reveal clues such as:

* authentication model
* SSO, OAuth, JWT, cookie, or MFA hints
* application architecture
* SPA, REST, GraphQL, or microservice clues
* role model
* admin, enterprise, team, tenant, or organization features
* product capabilities
* billing, uploads, teams, exports, audit logs, or integrations
* feature flags
* hidden routes or endpoints from JavaScript
* environment leaks such as dev, staging, or production references
* third-party integrations
* silent API requests
* client-side routing
* public APIs
* version or build information
* error handling behaviour

For example, if a landing page mentions teams, billing, RBAC, SSO, and audit logs, the tester can begin forming useful assumptions about the application’s internal shape.

```text
Landing page says:
“Teams, billing, RBAC, SSO, audit logs”

Possible deductions:

→ multi-tenant SaaS
→ roles likely exist
→ billing endpoints likely exist
→ organization/user separation likely exists
→ admin areas probably exist
→ authorization testing will matter later
```

These deductions do not prove that vulnerabilities exist. They simply help the tester build a better map of likely application behaviour and future testing areas.

Many landing pages expose only moderate clues. The page may not reveal deep technical details, but it may still show the main public workflows available to users.

```text
Login
Register
Forgot password
Pricing
Docs
Support
```

This is not deep information, but it is still useful context. Forgot password suggests a password reset flow exists. Pricing may suggest plan, tier, or billing logic. Enterprise messaging may suggest organization, RBAC, or SSO functionality. Documentation may support endpoint or feature discovery. Support pages may lead to contact, ticket, or upload flows. Registration confirms that account creation is available.

Some landing pages give almost nothing.

```text
Email
Password
Login
```

In that case, the page may only confirm that `/login` exists and that authentication is required. The tester should not force meaning into a simple login page. The better move is to quickly inspect what loads, check whether any useful JavaScript or requests appear, and then continue into login, registration, password reset, or any other available flow.

---

## 3. What JavaScript Does

When the landing page opens, JavaScript may control navigation, buttons, forms, route changes, tracking, public content, and links to registration or login.

At this stage, JavaScript is not authenticating the user. It is only preparing the public-facing application behaviour. It may decide what button to show, what route to open, what file to load, or what request to send, but the backend still decides what is allowed.

This distinction matters because the frontend may reveal that a route or feature exists, but the backend should still enforce whether an unauthenticated user is allowed to access it. Seeing an admin route, dashboard route, or feature name in JavaScript does not automatically mean the user can access it. It only means the frontend contains a reference to it.

Useful JavaScript observations include:

* where the login button sends the user
* where the registration button sends the user
* whether the app uses frontend routes
* whether API endpoints appear in JavaScript files
* whether hidden or unused routes appear
* whether feature names appear
* whether public configuration is loaded
* whether unauthenticated users can see restricted route names
* whether the app silently calls public APIs
* whether build or environment information is exposed

For testing, JavaScript is useful for discovery, but backend responses are what confirm access control behaviour. The frontend can guide the tester toward interesting areas, but the backend decides whether access is actually granted.

---

## 4. The API Request

The landing page may not always send a meaningful API request. Some landing pages only load static resources such as HTML, CSS, images, fonts, and JavaScript bundles. Other landing pages silently call public endpoints to load configuration, product content, pricing details, feature flags, application settings, or tenant branding.

When the landing page opens, the browser may load:

* HTML
* CSS files
* JavaScript bundles
* images and icons
* fonts
* analytics scripts
* public configuration files
* public API calls
* frontend route definitions

The important point is that useful application structure may appear before authentication. Even if the page itself looks simple, JavaScript files may expose route names, API paths, feature names, environment references, or logic that points toward backend behaviour.

This is why the browser Network tab, page source, JavaScript files, and DevTools Application tab can be useful before logging in.

At this stage, the tester is asking:

```text
What did the application load before I authenticated?
What did the browser store?
What requests happened automatically?
What public data came back?
What JavaScript logic is already visible?
```

These observations help separate what is public, what is pre-authenticated, and what should only appear after login.

---

## 5. What the Backend Validates

For a simple landing page, the backend may not validate much beyond serving public files and public responses. However, if the landing page calls public APIs, the backend still has responsibilities.

The backend should decide which public data can be returned, which configuration values are safe to expose, whether the request is unauthenticated, whether the endpoint should be available before login, and whether sensitive routes or internal values are being leaked.

It should also ensure that unauthenticated users cannot access restricted data simply because the frontend references a route, feature, or endpoint. This is the key security boundary.

Seeing a route in JavaScript does not automatically mean the user can access it. The real test is whether the backend returns protected data or performs protected actions without authentication.

---

## 6. What the Database Stores or Retrieves

A landing page may not directly trigger database activity. Some landing pages are static and only serve public frontend files. Other landing pages may retrieve public records such as product content, pricing plans, public documentation, blog posts, feature lists, public configuration, marketing data, or tenant-specific branding.

If database-backed content is returned before login, the backend should ensure that only public-safe data is exposed. At this stage, the database should not be returning private user data, authenticated dashboard data, billing records, team information, files, invoices, internal account details, or administrative content.

The tester should pay close attention to whether unauthenticated requests return data that appears user-specific, tenant-specific, administrative, sensitive, or intended only for authenticated users.

---

## 7. What Response Comes Back

The landing page response may include static files, rendered HTML, JavaScript bundles, public API responses, redirects, cookies, or public configuration. These responses help the tester understand what the application exposes before authentication.

Useful response details include:

* status codes
* redirects
* cookies
* cache headers
* JavaScript file names
* build or version references
* public API JSON
* route names
* environment values
* error messages
* exposed feature flags

A normal landing page response should support public page loading. It should not reveal protected user context, private tenant data, privileged routes with accessible data, or sensitive internal configuration.

---

## 8. What Happens After the Action Succeeds

After the landing page loads successfully, the user can usually choose the next path. Common next actions include registration, login, forgot password, reading documentation, opening pricing, contacting support, or starting a demo.

For the tester, the successful landing page load creates the first map of the application. The tester now has a better idea of what flows exist, what the application claims to support, and where deeper testing should begin.

A clean testing movement is:

```text
Landing page
↓
Identify visible flows
↓
Inspect loaded resources
↓
Capture useful routes and clues
↓
Move into registration, login, password reset, or other available flows
```

The landing page should not become a rabbit hole unless it exposes something clearly valuable. The discipline is to extract what matters, note useful clues, and continue into the deeper application flow.

---

## 9. What to Observe From a Security/Testing Mindset

The landing page is useful for quick pre-authentication reconnaissance. It helps the tester understand what the application exposes before login, which public flows exist, and what may deserve deeper testing later.

Use DevTools **Network** to inspect the page load, JavaScript files, redirects, and any API requests made before authentication. Check **Headers**, **Response**, and **Preview** for cookies, cache behaviour, CORS, public JSON, config values, feature flags, environment clues, and rate limits.

Use **Application** to review Cookies, Local Storage, and Session Storage. Use **Sources** to inspect JavaScript files for routes, API paths, feature names, and frontend logic. Use **Elements** for hidden links, disabled buttons, form fields, comments, or data attributes. Use the **Console** for frontend errors, debug output, or failed requests.

In Burp, use **Proxy** to capture the traffic and **Repeater** to safely replay public requests and compare unauthenticated responses.

Look for endpoint names, request methods, frontend routes, API paths, redirects, JavaScript bundle names, API base URLs, feature flags, environment values such as `dev`, `staging`, or `prod`, public `/config` responses, cookies and flags, CORS headers, cache headers, rate-limit headers, debug fields, hidden fields, internal path names, error messages, and references to admin, dashboard, billing, teams, uploads, exports, SSO, MFA, or roles.

These details help build the first map of the application. The landing page may reveal how the frontend is structured, what features probably exist, what requests happen before login, and where the boundary between public and authenticated behaviour begins.

Frontend clues are not proof of access. They only show that something exists. The backend response confirms what an unauthenticated user can actually access.

```text
Frontend reveals clues
Network shows behaviour
Backend confirms access
```

Useful testing questions include:

```text
What loads before login?
Are any API requests made unauthenticated?
Are routes, API paths, feature flags, or environment values exposed?
Are cookies or storage values created before login?
Do JavaScript files reveal hidden features or internal paths?
Are redirects handled by the frontend, backend, or both?
Is only public-safe data returned?
What should be compared again after login?
```

The landing page is not usually where deep testing happens. It is a public map that helps the tester understand the application’s surface before moving into registration, login, session, config, and authenticated functionality.

```text
Landing page = public map
JavaScript = route and logic clues
Network traffic = real request behaviour
Backend response = access control truth
```

Sometimes it gives gold. Sometimes it gives nothing.

The discipline is knowing how to check it, extract useful clues, and move on without wasting time. The landing page should help the tester understand the public face of the application before moving into deeper flows such as registration, login, authentication responses, sessions, configuration loading, and authenticated functionality.

