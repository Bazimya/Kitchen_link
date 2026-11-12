# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

KitchenLink is a multi-vendor food marketplace for Rwanda. This folder is not a repository itself: it holds three independent git repos, each with its own CLAUDE.md.

| Repo | What | Stack |
| --- | --- | --- |
| `KitchenLink-backend/` | The reference API (`/api/v1`) | Node/Express + MongoDB (source of truth) + a PostgreSQL mirror |
| `KitchenLink-frontend/` | Customer marketplace, kitchen dashboard, delivery worker app and admin panel in one site | React 18 + Vite + React Query + Tailwind |
| `KitchenLink-Mobile/` | One app (`lib/main.dart`) for every user: marketplace first, then "How do you want to sign in?" leads to the kitchen, delivery worker or admin screens (customer sign-in not built yet) | Flutter / Dart |

## How the three fit together

- **The contract is `KitchenLink-frontend/API_CONTRACT.md`.** It covers 223 endpoints. If it disagrees with the code, the frontend's `src/api/` wins. The backend answers every endpoint in it, and the mobile apps call the same paths, bodies and query parameters as the website. `lib/api/admin_api.dart` mirrors `src/api/adminApi.js` one for one. A change to an endpoint usually needs changes in all three repos and in the contract.
- Shared rules for all three: JSON field names are `snake_case`, IDs are strings, and money is whole RWF in fields ending `_rwf`. "Today" means Rwanda time (UTC+2). Errors look like `{ "error": { "code", "message", "details" } }`, and `message` is shown to users as is.
- **The server decides.** Clients never compute prices, fees, deposits or commission, and never decide which order transition comes next (`allowed_next_statuses`) or what a handover code is. They display what the backend returns.
- Each role has its own OTP sign-in and bearer token: customer, kitchen (vendor) and delivery worker. Admins sign in with email and password, plus 2FA when it's on. A 401 signs the user out. A 403 means they're signed in but not allowed.
- There are no sockets. Dashboards poll, and any non-GET request triggers a refresh.

## Local ports

Backend `http://localhost:4000/api/v1` · website `http://localhost:3000` · Flutter web (`flutter run -d chrome`) `http://localhost:5300`. The backend's `FRONTEND_URL` / `ALLOWED_ORIGINS` must include whichever client origins you use, or the browser blocks them with CORS.

## Toolchain on this machine

Flutter 3.38.7 / Dart 3.10.7. `KitchenLink-Mobile/pubspec.yaml` is pinned to work with them (`sdk: ^3.10.0`, go_router 17, flutter_riverpod 3.3). Don't raise those constraints without upgrading Flutter.
