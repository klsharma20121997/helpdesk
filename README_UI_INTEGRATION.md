# HelpDesk UI + .NET 8 ADO.NET API integration

## Included
- `index.html`: sign-in, account creation, and password-update UI.
- `CustomerDashboard.html`: customer portal wired to profile, ticket list/create/detail, and conversation APIs.
- `SupportDashboard.html`: shared staff portal. Admin users can view all tickets, change priority/assignment, manage support staff, reply, and export tickets. Support users see tickets assigned to their own user ID, can update ticket status and reply, and can export their ticket list.
- `helpdesk-config.js`: shared API base URL configuration loaded by all three pages.
- `HelpDesk_ADONET_Backend_Starter.zip`: the .NET 8 Web API and SQL schema.

## 1. Configure API URL and user IDs
Set the API URL once in `helpdesk-config.js`:

```js
window.HELPDESK_CONFIG = Object.freeze({
  apiBaseUrl: 'https://localhost:7001' // use the URL printed by `dotnet run`
});
```

After sign-in, `index.html` stores the returned Customer ID as `helpdesk_customer_user_id`, or the Admin/Support ID as `helpdesk_user_id`. It routes customers to `CustomerDashboard.html` and staff to `SupportDashboard.html`. The access token is kept in `sessionStorage` and sent as a Bearer token on portal API requests. For manual testing, `window.HELPDESK_CUSTOMER_USER_ID` and `window.HELPDESK_USER_ID` can still be set before the page scripts run; the staff page also accepts legacy `window.HELPDESK_ADMIN_USER_ID` and `helpdesk_admin_user_id`. The current demo fallback IDs are `1` for the customer page and `6` for the staff page.

The sign-in page expects `POST /api/auth/login` to return an `accessToken` and a `user` object containing `userId`, `roleName`, and optionally `fullName`. It also calls `/api/auth/register` and `/api/auth/update-password`. The included backend starter archive does not contain these authentication endpoints; use an API version that implements them before relying on sign-in or account management.

The API URL above is only a default; use the exact HTTPS/HTTP URL printed by ASP.NET Core. If the browser reports a CORS or certificate error, trust the local ASP.NET development certificate and configure CORS for the origin serving these HTML files. Avoid opening these files from `file://`; serve them using a local static server (for example VS Code Live Server) or from the same web host as the API.

## 2. Create database accounts
The provided SQL schema seeds roles and lookup values, but deliberately does not create user accounts. Create a Customer user and at least one Admin and Support user through a secure account/registration process before testing. For a local-only demo, you may insert test accounts in SQL using the appropriate `RoleId`; never store plaintext passwords. The customer ID used by `CustomerDashboard.html` must be a `Customer` user ID. The staff page's user ID must belong to an active Admin or Support user; Support users will only see tickets assigned to their ID.

## 3. Run API
1. Extract `HelpDesk_ADONET_Backend_Starter.zip`.
2. Run `database/HelpDeskDb.sql` in SQL Server.
3. Update `HelpDesk.Api/appsettings.json` with your SQL Server connection string.
4. From `HelpDesk.Api`, run `dotnet restore` and `dotnet run`.
5. Set `apiBaseUrl` in `helpdesk-config.js` to the URL printed in the console.
6. Serve `index.html`, `CustomerDashboard.html`, `SupportDashboard.html`, and `helpdesk-config.js` from the same directory and open `index.html`.

## API endpoints used
Customer: `GET/PUT /api/profile/{userId}`, `GET /api/tickets`, `POST /api/tickets`, `GET /api/tickets/{ticketId}?userId=...`, `POST /api/tickets/{ticketId}/messages`.
Staff page: `GET /api/profile/{userId}` to identify the staff role; `GET /api/admin/tickets` for tickets (filtered by `assignedStaffUserId` for Support), `PUT /api/admin/tickets/{ticketId}` for status updates and Admin-only priority/assignment changes, and `GET/POST /api/admin/staff` for Admin team management. Replies use the existing messages endpoint.

## Important security note
The page's role-based visibility and ticket filtering are a UI convenience, not security enforcement. The starter API does not implement authentication/authorization yet and accepts user IDs from the browser, so users can bypass these UI restrictions by calling endpoints directly. Before deploying, add JWT/cookie authentication, enforce role-based authorization and Support ticket assignment rules in the API, check customer ticket ownership, and derive the current user ID from the authenticated identity.
