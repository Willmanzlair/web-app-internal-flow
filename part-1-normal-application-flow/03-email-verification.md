# Email Verification

Email verification is the process of proving that a user controls the email address they used during registration.

When a user creates an account, the application knows that an email address was submitted. What it does not yet know is whether the person who registered actually owns or controls that inbox. Email verification exists to answer that question.

A useful mental model is:

```text
Registration = create the account
Email verification = prove ownership of the email address
```

Verification is not the same thing as authentication. Authentication proves that the backend trusts the user’s identity for login. Authorization decides what that user is allowed to access. Email verification is different because it is an account trust decision. The backend records that it trusts the user controls the email address associated with the account.

Many applications use email verification to reduce fake accounts, confirm contact information, support account recovery, and increase trust in account ownership.

A common flow looks like this:

```text
Register
↓
Account is created
↓
Verification token is generated
↓
Verification email is sent
↓
User submits verification token
↓
Backend validates token
↓
Account becomes verified
```

The important lesson is that email verification changes the account’s trust state. The account may already exist before verification, but the backend has not yet confirmed that the user controls the registered email address.

---

## 1. What This Functionality Is

Email verification allows the backend to confirm ownership of a registered email address.

After registration, the backend may create the user account with a verification state such as:

```text
verified = false
```

This means the account exists, but the email address has not yet been confirmed. The application then generates a verification token and sends it to the email address used during registration. If the user can access that email and submit the token back to the application, the backend can treat that as proof that the user controls the inbox.

A clean way to think about it is:

```text
Registration creates the account.
Email verification confirms email ownership.
Verification state records whether that ownership has been proven.
```

This distinction matters because an account can exist before it is verified. What changes after verification is not the existence of the user account, but the backend’s trust in the email address.

---

## 2. What the User Sees

From the user’s perspective, registration may appear complete, but the application may still ask them to verify their email address.

The user may see a message such as:

```text
Please verify your email address.
```

or:

```text
Verification email sent.
```

The user then checks their inbox and opens the verification email. Inside the email, there is usually a link, button, or code that allows the user to complete verification.

When the user follows the verification step, the application sends the verification token back to the backend. If the token is valid, the user may see a success message such as:

```text
Email verified successfully.
```

From the user’s perspective, this feels like a simple confirmation step. Internally, however, the application has changed the account’s verification state.

At this stage, the important point is that the backend is not creating the account again. The account already exists. The backend is updating what it trusts about that account.

---

## 3. What JavaScript Does

JavaScript helps coordinate the verification process, but it does not decide whether verification succeeds.

After registration succeeds, JavaScript may show verification instructions, guide the user to check their inbox, or display a message explaining that some features may remain limited until email verification is complete.

When the user opens a verification link or submits a verification token, JavaScript may read the token from the route, query string, form field, or application state, then send it to the backend.

A common frontend flow looks like this:

```text
Registration response is received
↓
Frontend shows verification instructions
↓
User opens verification link or submits token
↓
JavaScript extracts or collects the verification token
↓
JavaScript sends verification request
↓
JavaScript waits for backend response
↓
Frontend updates verification state
```

At this stage, JavaScript is only moving the token and reacting to the backend response. The backend still makes the decision about whether the token is valid and whether the account should become verified.

The frontend may display the success message, but the backend owns the verification decision.

---

## 4. The API Request

After the user follows the verification step, the frontend commonly sends a request containing the verification token.

Example:

```http
POST /api/auth/verify-email
```

Example request body:

```json
{
  "token": "abc123xyz"
}
```

The token is the important piece of information in this request. It acts as proof that the user reached the verification email or received the verification code sent to the registered email address.

A useful mental model is:

```text
Verification token = proof that the user reached the verification email
```

This is different from login. Login proves control of credentials. Email verification proves control of the email address.

Email verification may be handled in different ways. Some applications verify the account directly when the user clicks a link, while others load a frontend verification page that sends a request containing the token.

When an API request is used, `POST` is common because the token submission may change account state. If the backend accepts the token, the account may move from unverified to verified.

---

## 5. What the Backend Validates

When the verification request reaches the backend, the backend validates the submitted token.

Conceptually, the backend may check whether the token exists, whether it belongs to the correct user, whether it has expired, whether it has already been used, and whether it matches the expected verification record.

A simplified backend flow may look like this:

```text
Verification request received
↓
Token extracted
↓
Token exists?
↓
Token belongs to the correct account?
↓
Token still valid?
↓
Token not already used?
↓
Verification approved
```

If validation succeeds, the backend now trusts that the user controls the email address associated with the account.

This is an important distinction. The backend is not creating a new account at this stage, and it is not normally authenticating the user from credentials. It is validating email ownership and updating account trust state.

---

## 6. What the Database Stores or Retrieves

Most applications store verification state in the user record or in a related verification table.

A simple user record may contain:

```json
{
  "email": "foo@bar.com",
  "verified": false
}
```

The field:

```json
"verified": false
```

means the email address has not yet been verified.

After successful verification, the backend updates that state:

```json
{
  "verified": true
}
```

The account exists in both cases. The user record existed before verification and still exists after verification. What changed is the verification state.

This is a clear example of a state transition:

```text
verified = false
↓
verified = true
```

A useful mental model is:

```text
State = what is currently true
State change = what became true after an action
```

Before verification, it is true that the account exists but the email is not verified. After verification, it is true that the backend trusts the user controls the registered email address.

---

## 7. What Response Comes Back

After successful verification, the backend usually returns a confirmation response.

Example:

```json
{
  "ok": true,
  "verified": true
}
```

The purpose of this response is to tell the frontend that the verification action succeeded and the account is now verified.

A clean summary is:

```text
Verification succeeded
↓
Account verification state is now true
```

Notice what is not happening here. A new account is not being created. The password is not being changed. The backend is not necessarily issuing new authentication material.

The backend is confirming that the verification state changed successfully.

Some applications may return the updated user object after verification. Others may require the frontend to call `/me` again to fetch the updated account state. Both patterns are possible.

The important point is that the backend has now recorded a new account state.

---

## 8. What Happens After Successful Verification

After verification succeeds, the account is considered verified.

Future account responses may now include:

```json
{
  "verified": true
}
```

A common full flow looks like this:

```text
Register
↓
Account is created
↓
verified = false
↓
Verification token is submitted
↓
Backend validates token
↓
verified = true
↓
Future requests reflect the new state
```

The important lesson is that verification changes account state.

The user account existed before verification. The user account exists after verification. What changed is the level of trust the backend has in the registered email address.

In some applications, this state change may unlock additional behaviour. For example, the application may allow verified users to access the dashboard, use password recovery, invite team members, access billing, or perform sensitive actions.

In other applications, verification may be mainly informational.

The exact impact depends on the application, but the core idea remains the same: email verification records that ownership of the email address has been proven.

---

## 9. What to Observe From a Security/Testing Mindset

Email verification is valuable because it teaches state transitions and backend trust decisions.

During testing, the aim is not only to confirm that verification works. The aim is to understand what changes before and after verification.

Useful things to observe include:

* how the verification token is generated
* where the token appears
* whether the token is sent in a link or submitted through a form
* what endpoint receives the token
* whether the token expires
* whether the token can be reused
* whether invalid tokens are rejected
* whether verification changes the user object
* whether `/me` reflects the new verification state
* whether restricted actions are blocked before verification
* whether restricted actions become available after verification

A useful testing question is:

```text
What was true before the action?
What action occurred?
What became true afterward?
```

For email verification, that looks like this:

```text
Before:
verified = false

Action:
Submit verification token

After:
verified = true
```

This is one of the simplest examples of a backend state change.

The final mental model is:

```text
Registration creates the account.
Email verification proves ownership of the email address.
Verification changes account trust state.
```

Understanding this flow makes it easier to analyze other state-changing actions later, such as password reset, MFA setup, role changes, account activation, billing changes, and authorization updates.

