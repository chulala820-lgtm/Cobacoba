# Fullstack App (Generated)

This repository was generated automatically to convert your static frontend into a full-stack app.
It contains:

- `frontend/` — your original frontend files (slightly enhanced).
  - `index.html`, `admin.html`, `assets/js/script.js`, `assets/css/*.css`
- `backend/` — Node.js + Express API
  - `src/index.js` — main API: auth (register/login), bookings, payments (Stripe), webhook
  - `prisma/schema.prisma` — DB schema for User, Room, Booking, Payment (Postgres)
  - `.env.sample` — environment examples
  - `Dockerfile`, `package.json`

- `docker-compose.yml` — launches Postgres, backend, and a static nginx frontend server

## Quickstart (development)

1. Ensure Docker is installed.
2. Copy `.env.sample` to `backend/.env` and set `STRIPE_SECRET_KEY` and `JWT_SECRET`.
3. Start services:
   ```bash
   docker compose up --build
   ```
4. Initialize Prisma migrations (if you want to run locally without Docker, install Node and Prisma):
   ```bash
   cd backend
   npm install
   npx prisma generate
   npx prisma migrate dev --name init
   ```
5. Visit frontend at `http://localhost:5173` and backend API at `http://localhost:4000/api`.

## Features added (as requested)
- Admin auth (JWT) and admin dashboard `frontend/admin.html`
- PostgreSQL support via `docker-compose.yml`
- Stripe payment integration: create payment intent and webhook handler
- Prisma ORM models for bookings/payments/rooms/users
- Booking endpoints to persist bookings

## Notes and security considerations
- For demo simplicity webhook does not verify Stripe signature — enable verification in production using `STRIPE_WEBHOOK_SECRET` and `stripe.webhooks.constructEvent`.
- Keep `JWT_SECRET` and `STRIPE_SECRET_KEY` secret and stored in environment variables / secret manager.
- Consider moving payment amounts/currency handling server-side to avoid client tampering.



## Ready-to-run (detailed)

### Option A — Docker (recommended)
1. Copy `backend/.env.sample` to `backend/.env` and set:
   - `JWT_SECRET` to a long random string
   - `STRIPE_SECRET_KEY` to your Stripe secret key
   - `STRIPE_WEBHOOK_SECRET` to your Stripe webhook signing secret (optional but recommended)
   - Adjust `DATABASE_URL` if needed (docker-compose already sets a Postgres DB)
2. From repository root run:
```bash
docker compose up --build
```
This will start Postgres, backend (port 4000) and nginx serving the frontend (port 5173).

3. Initialize database (in a new terminal):
```bash
docker compose exec backend sh -c "npx prisma migrate deploy || npx prisma migrate dev --name init"
docker compose exec backend node prisma/seed.js
```
(Or run the seed locally: `node backend/prisma/seed.js` after installing dependencies.)

### Option B — Local Node (without Docker)
1. Install Node.js (v18+) and Postgres (or set `DATABASE_URL` to a cloud Postgres).
2. In `backend/`:
```bash
npm install
npx prisma generate
npx prisma migrate dev --name init
node prisma/seed.js
npm run start
```
3. Open the frontend by serving `frontend/` folder (e.g. `npx http-server frontend -p 5173`) or open `frontend/index.html` directly for static usage.

### Admin
- Default admin credentials (from seed): `admin@example.com` / `adminpassword` (change these in `.env` by setting SEED_ADMIN_EMAIL / SEED_ADMIN_PASSWORD or change password after first login).
- Admin can login via frontend `admin.html` or via API to manage bookings.

### Stripe webhook
- Configure webhook endpoint in Stripe dashboard: `https://<your-domain>/api/payments/webhook`
- Set the `STRIPE_WEBHOOK_SECRET` variable in `backend/.env` and the webhook will be verified.

Security notes: do not commit `.env` with secrets. Use secret manager for production.



### Automatic migrations & seeding on Docker startup

The backend container now runs database migrations and the seed script automatically on startup.
Behavior:
- On container start the entrypoint runs `npx prisma migrate deploy` (with retries while waiting for the DB).
- Then it runs `npx prisma generate` and `node prisma/seed.js`.
- Finally it starts the Node server (`node src/index.js`).
This means `docker compose up --build` will attempt to prepare the database and seed it automatically.
If migrations fail after multiple retries, the backend container will exit — ensure the database container is healthy and accessible.
