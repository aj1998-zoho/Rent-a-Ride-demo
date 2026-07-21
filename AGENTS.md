# AGENTS.md

## Cursor Cloud specific instructions

"Rent a Ride" is a MERN car-rental app with two services:

| Service  | Location   | Dev command (run from that dir)     | Port | Notes |
|----------|------------|-------------------------------------|------|-------|
| Backend  | repo root  | `npm run dev` (nodemon `backend/server.js`) | 3000 | Express + Mongoose. Reads `/.env`. |
| Frontend | `client/`  | `npm run dev` (Vite)                | 5173 | Reads `client/.env`. Lint: `npm run lint`. Build: `npm run build`. |

Dependencies live in two `package.json` files: the repo root (backend) and `client/` (frontend). The update script installs both.

### Local services & env files (not committed; `.env` is gitignored)
- MongoDB must be running locally. It is installed as `mongodb-org` (v8.0) but there is no systemd on the VM, so start it manually, e.g. `mongod --dbpath /data/db --bind_ip 127.0.0.1` (a `mongodb` tmux session is used during setup).
- Backend `/.env` requires at minimum: `mongo_uri` (note the lowercase name, e.g. `mongodb://127.0.0.1:27017/rent-a-ride`), `ACCESS_TOKEN`, and `REFRESH_TOKEN` (JWT secrets; any non-empty string works locally). `CLOUD_NAME`/`API_KEY`/`API_SECRET` (Cloudinary), `RAZORPAY_KEY_ID`/`RAZORPAY_SECRET`, and `EMAIL_HOST`/`EMAIL_PASSWORD` are only needed for image upload, payments, and booking-confirmation emails respectively; auth/browse flows work without them.
- Frontend `client/.env` requires `VITE_PRODUCTION_BACKEND_URL=http://localhost:3000` (used both as the Vite `/api` proxy target and directly by sign-in). `VITE_FIREBASE_API_KEY` is only needed for Google OAuth login.

### Gotchas
- The `User` schema declares `phoneNumber` as `unique` but not `sparse`, so only ONE user without a phone number can exist. The UI signup form does not collect a phone number, so a second UI signup fails with `E11000 duplicate key ... phoneNumber: null`. When testing signup with fresh accounts, clear stale phone-less users first: `mongosh rent-a-ride --eval "db.users.deleteMany({phoneNumber: null})"`.
- CORS on the backend only allows `http://localhost:5173` and the production Vercel origin, so use the Vite dev server (port 5173) rather than hitting the API from other origins.
- `npm run lint` in `client/` exits non-zero due to 6 pre-existing errors (unrelated to setup); this is the repo's current state, not an environment problem.
