# Amazon-clone-webiste
# Amazon Clone – Frontend + Backend + Database (one folder)

```
amazon-clone-full/
├── frontend/                 HTML, CSS and JavaScript (what the visitor sees)
│   ├── templates/index.html
│   └── static/  style.css, script.js, images/
├── backend/                  Python Flask server (the API)
│   ├── app.py
│   ├── requirements.txt
│   └── .env.example          copy to ".env" and add your MySQL password
├── database/
│   └── schema.sql      the ONE file with all tables and data
└── README.md
```

## Run it (5 steps)
1. Install **Python 3.10+** and **MySQL Server**.
2. Create the database - run this single file (from the `amazon-clone-full` folder):
   ```
   mysql -u root -p < database/schema.sql
   ```
   (or open `database/schema.sql` in MySQL Workbench and press the lightning button)
3. Set up your settings:
   ```
   cd backend
   copy .env.example .env          (Windows)     |     cp .env.example .env   (Mac / Linux)
   ```
   Open `backend/.env` and set `DB_PASSWORD`. Generate a strong session secret with:
   `python -c "import secrets; print(secrets.token_hex(32))"`
   Copy the output into `SECRET_KEY` in `backend/.env`. Never commit `.env` to GitHub.
4. Install packages:
   ```
   python -m venv venv
   venv\Scripts\activate           (Windows)     |     source venv/bin/activate   (Mac / Linux)
   python -m pip install -r requirements.txt
   ```
5. Start the server (still inside `backend`):
   ```
   python app.py
   ```
   Open **http://127.0.0.1:5000**

**Demo login:** `demo@amazon-clone.com` / `Demo@12345` (it already has one order). Or create your own account.

Running `schema.sql` again resets the database to its starting data (your own accounts and orders are deleted).

## What is inside schema.sql
| Table | Purpose |
|---|---|
| `users` | accounts (passwords are hashed) - 1 demo user |
| `products` | 30 products: electronics, mobiles, fashion, home, books, gift cards |
| `orders` / `order_items` | orders and the items in them - 1 sample order |
| `seller_requests` | requests sent from the "Sell" form |

## Language button
Click **EN ▾** in the header to switch between **English, हिन्दी and தமிழ்** (on phones: ☰ menu → Language).
Your choice is remembered. Menus, banners, buttons, cart, messages and help text are translated; product names come from the
database and stay in English. To translate more text, add one line to the `DICT` list in `frontend/static/script.js`.

## What each button does
| Button | Action |
|---|---|
| ☰ All | opens the side menu |
| Today's Deals | products with 40%+ discount, biggest first |
| prime | Prime products only |
| Mobiles / Electronics / Fashion / Home & Kitchen / Gift Cards | loads that category from MySQL |
| Customer Service | help window with FAQs and shortcuts |
| Sell | seller form, saved in the `seller_requests` table |
| EN ▾ | language menu |
| Logo | back to the home view |
| Banner buttons, card links, footer links | open the matching category or window |
| Picture tiles on the home page | open just that kind of product |

## Images
Every picture is one of your own photos in `frontend/static/images/` (JPG). The `image` column of the `products` table chooses the file:
a row with `image = 'headphones'` shows `headphones.jpg`. To add a product, save `mylamp.jpg` there and add a row with `image = 'mylamp'`.

## How the three parts connect
`frontend/static/script.js` -> `fetch("/api/...")` -> `backend/app.py` -> MySQL (`database/schema.sql`) -> JSON back to the page.

## API
| Method | URL | What it does |
|---|---|---|
| GET | `/api/products?q=&category=&prime=1` | list or search products (`q=shoes|sneakers` means shoes OR sneakers) |
| POST | `/api/register`, `/api/login`, `/api/logout` | account |
| GET | `/api/me` | who is signed in |
| POST / GET | `/api/orders` | place an order / list my orders (sign-in required) |
| POST | `/api/sell` | save a seller request |

## Authentication and local security
- Registration and login use Werkzeug password hashing; plain-text passwords are never stored.
- Passwords must contain at least 8 characters.
- Flask sessions are signed and use HttpOnly and SameSite=Lax cookies.
- CSRF protection is enabled for state-changing requests; the frontend sends the token automatically.
- Registration and login are rate-limited to 5 attempts per minute per IP (in-memory, suitable for one local process).
- Order endpoints require sign-in; the server derives the user ID from the session.
- SQL uses parameterised queries; order totals are calculated on the server.
- Secrets are loaded from `backend/.env`, which is excluded from Git.

## Before publishing to GitHub
1. Confirm `backend/.env` is not tracked (`git status --ignored`).
2. Never commit real passwords, API keys, or database dumps containing personal data.
3. Use a fresh random `SECRET_KEY`; if a secret was previously published, rotate it.
4. Keep `debug=False` for the portfolio copy. This is a local demo setup, not a production deployment.
5. The built-in in-memory rate limiter resets when Flask restarts and does not share counts across workers.

## Authentication (how login works here)
| Step | What the project does |
|---|---|
| Sign up | Name, email and a password of 8-128 characters. Easy ones (`password123`, `12345678`, your own email...) are refused. |
| Password storage | Hashed with scrypt, never saved as plain text. |
| Sign in | Same message and same time for "no such email" and "wrong password". Limits: 5 wrong tries per account per 15 minutes and 20 per computer per 10 minutes. Correct logins never count. |
| Session | The cookie holds a random id; the session itself is a row in `auth_sessions` (8 hours). Logout deletes the row, so a copied cookie stops working. |
| CSRF | Every POST needs a token; the server sends a fresh one after sign in / sign out and the page swaps it in. |
| Forgot password | The email contains a one-time link valid 30 minutes (`/#reset=...`). Only a hash of the token is stored. The answer is the same whether or not the email exists. |
| Change password | Needs the current password. Other devices are signed out, this one stays. |
| Sign out of all devices | Account window -> "Sign out of all devices". |

Open **Your Account** (header, after signing in) for: change password, sign out, sign out of all devices.

### Forgot-password emails
Leave `SMTP_HOST` empty in `backend/.env` while learning: the reset link is **printed in the terminal** running `python app.py`
(DEV MODE) - copy it into the browser. To send real emails for free with Gmail:
1. Turn on 2-Step Verification for the Google account, then create an **App password** (Google Account -> Security).
2. In `backend/.env` set `SMTP_HOST=smtp.gmail.com`, `SMTP_PORT=587`, `SMTP_USER=your@gmail.com`, `SMTP_PASSWORD=<the 16-character app password>`.
3. Set `APP_BASE_URL` to the address of your site (for example `http://127.0.0.1:5000`). Restart the server.

### Run the automatic tests
```
cd backend
python -m unittest test_auth -v
```
(16 tests: CSRF, sessions, logout, lockout, password rules, change / forgot / reset password. They need MySQL running and delete their own test users.)

### Not included (on purpose)
Email verification, two-factor codes and "Sign in with Google" - useful later, but they need extra services and add friction for a practice shop.

### After upgrading from the older files
Run `database/schema.sql` once (it adds `auth_sessions` and `password_resets`; it also resets all data), then restart the server.
Never share `backend/.env` - it holds your database password.


### If the page says "Cannot reach the server" (or "Failed to fetch")
The browser cannot reach Flask. Check that the terminal running `python app.py` is still open and shows `Running on http://127.0.0.1:5000`, that the address bar says `http://127.0.0.1:5000` (not a file, not Live Server), then restart `python app.py` and press Ctrl+F5.
