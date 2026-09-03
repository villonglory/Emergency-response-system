# 🚨 ERS – Emergency Response System

A frontend-only web application that lets users report emergencies, share their location, and access safety instructions. Includes an admin dashboard for viewing and managing submitted reports.

Built as a portfolio/academic project with **HTML5, CSS3, and vanilla JavaScript** — no frameworks, no backend, no build step.

---

## Live Demo

Open `index.html` in a browser, or deploy to GitHub Pages / Cloudflare Pages (see below). `index.html` is now the **login page** — see "Auth Flow" below.

---

## Folder Structure

```text
ERS/
│
├── index.html          Login page (site entry point)
├── signup.html          User registration
├── home.html             Logged-in landing page: hero, features, about, contact
├── report.html            Emergency report form + confirmation screen (public, no login required)
├── instructions.html      Safety instructions by emergency type (public, no login required)
├── my-reports.html         A logged-in user's own submitted reports — users only
├── admin.html             Admin dashboard (reports, stats, filters) — admins only
│
├── css/
│   └── style.css         All styles (design tokens, layout, components, responsive rules)
│
├── js/
│   ├── storage.js         Shared localStorage data layer (reports, user accounts, session)
│   ├── auth.js             Login, sign up, session-aware nav, home page login guard
│   ├── script.js          Shared behavior: nav toggle, footer year, contact form, formatDateTime/escapeHtml utils
│   ├── report.js          Report form validation, geolocation, confirmation screen
│   ├── admin.js            Admin dashboard only (stats, table, search/filter, modal, logout)
│   └── my-reports.js       "My Reports" page (a user's own submissions + read-only detail modal)
│
├── images/                gvklogo.jpg (footer credit)
│
└── README.md
```

**Note on structure:** three files were added beyond the original suggested layout — `js/storage.js`, `js/auth.js`, and `js/my-reports.js` (plus `my-reports.html`). `storage.js` centralizes every localStorage read/write (reports **and** user accounts) so other scripts reuse the same functions instead of duplicating data-access logic. `auth.js` centralizes login/signup/session/nav-state logic so it isn't duplicated across every page that needs to know who's logged in. `formatDateTime()`/`escapeHtml()` also moved into `script.js` once a third page (`my-reports.html`) needed them, instead of copy-pasting the same two functions a third time.

---

## Auth Flow

- **`index.html`** is the site's entry point and doubles as the login form.
- **`signup.html`** lets a visitor create a regular user account (stored in `localStorage`, role `user`).
- On login, the system checks the account's role and redirects accordingly:
  - `role: "admin"` → `admin.html` (dashboard)
  - `role: "user"` → `home.html` (the site's homepage content)
- **`report.html`** and **`instructions.html`** stay public and don't require login — reporting an emergency should never be blocked by a signup wall. Every report submitted, logged in or not, is saved to the same shared store the admin dashboard reads from, so it shows up there automatically. If the person submitting is logged in, the report is tagged with their username/name (`submittedBy` / `submittedByName`); guests submit anonymously, and the admin dashboard shows "Guest" for those.
- **`my-reports.html`** shows a logged-in user only the reports tagged with their own account, with a read-only detail view (no status changes or delete — that stays admin-only). The nav's **My Reports** link only appears for `role: "user"`.
- **`admin.html`** is gated to `role: "admin"` only. A logged-in regular user who navigates there directly is redirected to `home.html`; a logged-out visitor is redirected to `index.html`.
- The nav bar adapts based on session state (via `refreshNavForSession()` in `js/auth.js`): logged-out visitors see **Sign Up** / **Log In**; logged-in users see **Log Out** (users also see **My Reports**, admins see **Dashboard**).

---

## Features

- Responsive navigation with mobile hamburger menu, session-aware (Login/Sign Up vs. Logout)
- User accounts: sign up, log in, role-based redirect (user vs. admin)
- Emergency report form with client-side validation — open to everyone, no account needed
- One-tap geolocation capture (`navigator.geolocation`) with an OpenStreetMap link (no API key required)
- Auto-generated unique report IDs (`ERS-XXXXXX`)
- Confirmation screen after submission
- Safety instructions organized by category (Fire, Medical, Accident, Crime, Natural Disaster) using expandable cards
- Admin dashboard (role-gated) with:
  - Stats cards (total + per-type counts)
  - Searchable, filterable report table (by type, status, and who submitted it)
  - Status updates (Pending / Responding / Resolved)
  - Report detail modal, including a map link when location was captured
  - Delete with confirmation
  - Optional "Add Demo Data" button (does not run automatically — the dashboard's primary data source is always reports submitted through the form)
  - Logout
- "My Reports" page (role-gated to regular users) showing only the reports that account personally submitted, with a read-only detail view
- Fully responsive (desktop, tablet, mobile)

---

## Running Locally

No build tools or server required.

1. Open the `ERS` folder in VS Code.
2. Double-click `index.html` to open it in a browser, **or**
3. In VS Code, install the "Live Server" extension, right-click `index.html`, and choose **Open with Live Server** (recommended — some browsers restrict `localStorage`/geolocation on `file://` URLs).

## Testing the Admin Dashboard

1. Go to `index.html` (the login page).
2. Sign in with:
   - **Username:** `admin`
   - **Password:** `admin123`
3. You'll land on `admin.html`. Submit a report from `report.html` first (or click **+ Add Demo Data**) to see it appear in the table.
4. Try searching, filtering by type/status, changing a report's status, viewing a report's details, and deleting a report.

## Testing a Regular User

1. Go to `signup.html` and create an account.
2. You'll be signed in automatically and redirected to `home.html`.
3. Use the nav or hero buttons to reach **Report Emergency** / **Safety Instructions**.
4. Submit a report while logged in, then click **My Reports** in the nav — it should show up there with its current status (open a second tab logged in as `admin` and change its status to see it update).

---

## Data Storage

All data is stored in the browser via `localStorage` (reports and user accounts) and `sessionStorage` (the current login session), using the helper functions in `js/storage.js`:

- `getReports()` / `saveReports()` / `addReport()` / `updateReport()` / `deleteReport()` / `getReportById()` / `getReportsByUser()`
- `getUsers()` / `saveUsers()` / `registerUser()` / `authenticateUser()` / `findUserByUsername()` / `usernameTaken()` / `seedDefaultAdmin()`
- `setSession()` / `clearSession()` / `getCurrentUser()`
- `seedDemoReports()` — optional, only runs when the admin clicks "Add Demo Data"

Data persists across page refreshes but is local to the browser/device it was created on.

---

## ⚠️ Security Notice

This is a **frontend-only demonstration project**. It is intentionally simple so it can run without a server. Before using anything like this in production:

- `localStorage` is **not** secure database storage — it's plain text, readable by anyone with access to the browser, and not shared across devices.
- **User passwords are stored in plain text** in `localStorage`, and the admin account (`admin` / `admin123`) is seeded automatically in `js/storage.js`. **None of this is real authentication or real account security.**
- A production version would need a secure backend with server-side authentication, hashed/salted passwords, and real session management.
- Emergency report data would need to be stored in a properly secured database with access controls.
- The site would need to be served over **HTTPS**.
- Real integration with police, ambulance, or fire services would require proper authorization and official API/dispatch partnerships.
- **This system does not, and does not claim to, automatically contact emergency services.** In a real emergency, always contact your local emergency number directly.

---

## Deploying to GitHub Pages

1. Create a new GitHub repository and push the contents of the `ERS` folder to it (or push it as a subfolder and point Pages at that path).
2. In the repository, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to `Deploy from a branch`.
4. Choose the branch (e.g. `main`) and root folder (`/` or `/ERS` depending on your repo layout), then **Save**.
5. GitHub will publish the site at `https://<username>.github.io/<repo-name>/` within a minute or two.

## Deploying to Cloudflare Pages

1. Push the project to a GitHub (or GitLab) repository.
2. In the Cloudflare dashboard, go to **Workers & Pages → Create → Pages → Connect to Git**.
3. Select the repository.
4. Since this is a static site with no build step, set:
   - **Build command:** (leave empty)
   - **Build output directory:** `/` (or the path to the `ERS` folder if it's nested)
5. Click **Save and Deploy**. Cloudflare will publish the site at `https://<project-name>.pages.dev`.

Both platforms serve static files directly, and every path in this project is relative (e.g. `css/style.css`, `js/script.js`), so no configuration changes are needed for either host.
