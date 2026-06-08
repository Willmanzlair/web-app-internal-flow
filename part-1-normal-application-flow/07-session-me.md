# Session Restoration and `/me`

## 1. What This Functionality Is

Session restoration is the process where an application rebuilds the logged-in user’s identity after the frontend reloads, restarts, or loses its temporary in-memory state.

After login succeeds, the backend has already validated the user’s credentials, accepted the identity, and issued whatever proof the application uses to preserve authenticated state. That proof may be a token, a session cookie, or a combination of both.

The important question comes after login:

```text
How does the application remember who the user is?
```

This becomes easier to understand when thinking about a normal dashboard refresh. The user logs in, reaches the dashboard, and then refreshes the page. After the refresh, the browser reloads the frontend application. JavaScript starts again, and any temporary state that existed only in memory may be gone. However, the user often remains logged in because the browser still has some form of authentication proof from the earlier login.

At this point, the application is not usually asking the user to log in again. Instead, the frontend is trying to rebuild the current user identity from the authenticated state that already exists.

A simple version of that scenario looks like this:

```text
Login
↓
Dashboard loads
↓
User refreshes the page
↓
Frontend reloads
↓
Browser still has authentication proof
↓
Application restores the user session
```

This is where `/me` becomes important.

The purpose of `/me` is usually to tell the frontend who is currently authenticated. It allows the frontend to ask the backend, “Based on the token or session this browser is presenting, which user is this?”

A useful mental model is:

```text
/login = Authenticate me
/me    = Tell me who I am
```

Authentication already happened during login. `/me` does not normally re-authenticate the user from their password. Instead, it uses the existing authenticated state to identify the current user and return selected identity information to the frontend.

---

## 2. What the User Sees

From the user’s perspective, session restoration usually feels invisible. There is no new login form, no popup, and no visible authentication prompt. The user refreshes the dashboard, and the dashboard appears again as if nothing major happened.

The user may still see their name, role, plan, account status, and available features. This gives the impression that the application simply “remembered” them.

Internally, however, the frontend had to rebuild that identity context. The previous page runtime was restarted during refresh, so the application needed to recover enough information to know who the user is and what interface should be shown.

From the user’s side, the visible experience looks like this:

```text
User refreshes dashboard
↓
Dashboard reappears
↓
Name is still displayed
↓
Role is still known
↓
Features are still visible
```

The important point is that the visible experience is smooth, but the internal application still had work to do. It needed to confirm that the browser still had valid authenticated state and then rebuild the user interface from the returned identity data.

---

## 3. What JavaScript Does

After a page refresh, JavaScript starts again from a clean frontend runtime. This means any user data that was only stored in memory may be gone. The application may no longer have the current user object available in its frontend state.

This does not mean the user has been logged out. It only means the frontend needs to rebuild its knowledge of the user.

Authentication already happened earlier during login. JavaScript now needs to ask the backend which user is represented by the authentication proof still available in the browser. That proof may be a bearer token, a session cookie, or a combination of both.

In cookie-based applications, JavaScript may not be able to read the cookie directly if it is marked `HttpOnly`. That is normal and desirable. The browser can still automatically attach the cookie to requests sent to the correct backend. In token-based applications, JavaScript may attach a bearer token to the `Authorization` header.

Once the page loads, JavaScript commonly makes a `/me` request. If the backend accepts the token or session, it returns the current user object. JavaScript then uses that user object to rebuild the authenticated interface.

The flow can be summarized like this:

```text
Page loads
↓
JavaScript starts
↓
Frontend state is empty or incomplete
↓
Browser still has authentication proof
↓
JavaScript requests /me
↓
Backend validates the proof
↓
Backend identifies the user
↓
User object is returned
↓
Frontend rebuilds the UI
```

This is why many modern applications appear to “remember” users after refresh. The frontend is not remembering everything by itself. It is asking the backend to identify the current authenticated user.

---

## 4. The `/me` API Request

A `/me` request is usually an identity-check request. It asks the backend to identify the currently authenticated user from the authentication proof already present in the browser.

A common request looks like this:

```http
GET /api/auth/me
```

This request is different from login. Login sends credentials because the user is trying to prove control of an identity. A login request commonly looks like this:

```http
POST /api/auth/login
```

with a request body such as:

```json
{
  "email": "foo@bar.com",
  "password": "Password123"
}
```

A `/me` request usually does not need a request body. There may be no payload tab because the frontend is not submitting credentials again. The user has already authenticated, and the browser already has identity proof.

Instead of sending email and password again, the browser sends existing authentication material. This may be a bearer token:

```http
Authorization: Bearer eyJ...
```

or a session cookie:

```http
Cookie: session=abc123
```

The distinction is important:

```text
Login sends credentials.
/me sends existing identity proof.
```

Login is where the user claims an identity by submitting credentials. `/me` is where the backend uses existing proof to identify which trusted user is currently active.

In cookie-based applications, JavaScript may simply call `/me`, and the browser automatically attaches the session cookie. In token-based applications, JavaScript may attach a bearer token in the `Authorization` header. In hybrid applications, both a token and a cookie may be involved.

---

## 5. What the Backend Validates

When `/me` reaches the backend, the backend does not normally verify the password again. Password verification already happened during login.

Instead, the backend validates the authentication proof attached to the request. If the request contains a bearer token, the backend checks whether the token is valid, trusted, unexpired, and linked to a real user. If the request contains a session cookie, the backend checks whether that session exists, is still active, and belongs to a valid user.

Once the proof is accepted, the backend can identify the current user and retrieve selected account information.

The backend flow can be summarized like this:

```text
Request arrives
↓
Backend reads Authorization header or session cookie
↓
Backend validates token or session
↓
Backend identifies the user
↓
Backend retrieves account information
↓
Backend returns user object
```

The difference between login and `/me` is important.

Login asks:

```text
Can I trust this identity?
```

`/me` asks:

```text
Which trusted identity is this request presenting?
```

Authentication created the trust. `/me` uses that trust to return identity information.

If the token is missing, expired, invalid, revoked, or malformed, the backend should reject the `/me` request. If the session cookie is missing, expired, invalid, or no longer recognized server-side, the backend should also reject the request. A rejected `/me` response usually causes the frontend to clear authenticated state and send the user back to login.

---

## 6. What the Database Stores or Retrieves

After the backend validates the token or session, it may need to retrieve current account information from the database. The authentication proof identifies the user, but the frontend often needs more than just a user ID. It may need the user’s name, email, role, permissions, organization, plan, verification status, or feature access.

The frontend does not directly access the database. The browser sends a request to the backend, the backend validates the identity proof, and the backend decides which user information is safe and necessary to return.

This flow can be summarized like this:

```text
Frontend sends /me request
↓
Backend validates identity proof
↓
Backend queries database if needed
↓
Backend returns selected user information
↓
Frontend renders the authenticated UI
```

The backend acts as the trusted intermediary between the browser and the database.

This matters because the database may store sensitive internal fields that should not be exposed to the frontend. The `/me` response should only return information the frontend genuinely needs for rendering, state management, and normal application behaviour.

For example, the frontend may need to know the user’s role, plan, verification status, and permissions. It should not receive password hashes, raw secrets, reset tokens, verification tokens, internal security flags, or backend-only account metadata.

---

## 7. What Response Comes Back

A typical `/me` response contains a user object. This object gives the frontend enough information to understand which user is currently authenticated and how the interface should be rebuilt.

Example:

```json
{
  "id": "123",
  "email": "foo@bar.com",
  "name": "Foo",
  "role": "admin",
  "plan": "enterprise",
  "verified": true
}
```

The purpose of this response is identity restoration. It tells the frontend who the current authenticated user is and returns selected account details needed for rendering.

A `/me` response normally should not need to return passwords, password hashes, raw credential material, reset tokens, verification tokens, or backend-only secrets. It may also not return new access or refresh tokens unless the application is specifically designed to refresh, rotate, or extend tokens during that flow.

The reason is simple: the purpose of `/me` is not to authenticate the user from credentials. Authentication already happened. The purpose of `/me` is to identify the currently authenticated user.

A useful distinction is:

```text
Login response = authentication material and initial identity context
/me response   = current authenticated user identity
```

This explains why a login response may contain a user object, access token, and refresh token, while `/me` often contains only the user object.

Some applications return a small `/me` response with only basic identity fields. Other applications return richer identity context, such as roles, permissions, feature flags, organization IDs, team memberships, and account state. The exact shape depends on the application, but the purpose is the same: rebuild the authenticated user context on the frontend.

---

## 8. What Happens After Successful `/me`

Once JavaScript receives the user object, the frontend can rebuild the authenticated interface. The user object may tell the frontend the user’s name, role, plan, verification status, permissions, and available features.

For example, if the response says:

```text
role = admin
name = Foo
plan = enterprise
verified = true
```

the frontend can use that data to decide what to display. It may show the user’s name, reveal role-specific navigation items, enable the correct feature sections, show the current plan, or keep the user inside the authenticated area of the application.

The UI update can be summarized like this:

```text
Receive user object
↓
Show user name
↓
Show role-specific features
↓
Show account plan
↓
Enable the correct UI sections
↓
Keep the user inside the authenticated area
```

Without `/me`, many applications would not know who is logged in after a refresh. The browser may still have a token or cookie, but the frontend needs identity details to know what to display.

This is why `/me` is often one of the first requests after page load in authenticated applications.

A clean session restoration flow looks like this:

```text
Page refreshes
↓
Frontend state is lost
↓
Browser still has auth proof
↓
JavaScript calls /me
↓
Backend validates auth proof
↓
Backend returns current user
↓
Frontend rebuilds authenticated state
↓
Dashboard renders correctly
```

If `/me` fails, the frontend should usually treat the user as unauthenticated. It may clear local auth state, remove cached user data, hide authenticated UI, and redirect the user to the login page.

---

## 9. What to Observe From a Security/Testing Mindset

`/me` is often one of the most valuable endpoints in an application because it reveals how the application understands the current user.

The response may expose important identity and authorization clues, such as user ID, email, role, permissions, plan, organization ID, team information, feature flags, verification status, account state, or session metadata.

The main testing question is:

```text
What proof is the browser presenting?
What proof does the backend trust?
```

To answer that, test what happens when different authentication materials are removed or changed. If the application uses a bearer token, remove the `Authorization` header and check whether `/me` still works. If it uses a session cookie, remove the cookie and observe whether the backend still identifies the user. If both are present, test them separately to understand which one the backend actually trusts.

Useful checks include:

```text
Remove Authorization header
↓
Does /me still work?

Remove session cookie
↓
Does /me still work?

Keep JWT but remove cookie
↓
What happens?

Keep cookie but remove JWT
↓
What happens?
```

These checks help identify whether the application is using JWT-only authentication, cookie-only authentication, or a hybrid model. They also help reveal trust precedence, identity sources, and authorization boundaries.

Other useful questions include:

* Does `/me` return too much user information?
* Does `/me` expose internal IDs, roles, permissions, or feature flags?
* Does `/me` still work after logout?
* Does `/me` still work after password reset?
* Does `/me` reflect changes to role, plan, verification state, or permissions?
* Does the frontend trust `/me` data for rendering only, or does the backend also enforce authorization later?
* Does removing or modifying auth proof correctly break the request?
* Does the application cache stale `/me` responses after the user’s state changes?

A final mental model is:

```text
Browser stores or presents identity proof.
Backend validates or remembers identity.
Frontend uses /me to rebuild identity context.
```

This is the foundation for understanding session restoration, dashboard rendering, authorization checks, and every authenticated request that follows.

