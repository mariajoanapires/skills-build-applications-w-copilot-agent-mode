# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Keep data fetching in the existing shared API client and capability components. Each page must expose accessible loading, empty, and error states while rendering the API response.
- Logic tier decision: Implement the documented `/api/*` contracts in the Express routes and service layer. Validate request fields and referenced IDs before database operations, returning deliberate 4xx responses for invalid or unknown resources; do not add authentication.
- Data tier decision: Enforce required fields and defaults in the Mongoose models. Keep seed data representative and idempotent, linking seeded users, teams, activities, and workouts without deleting unrelated records.
- Activity tracking approach: Accept `userId`, `teamId`, `type`, and `minutes`; validate both IDs and confirm the referenced user and team exist. Store the corresponding references, submitted activity details, computed points, and an occurrence timestamp.
- Team management approach: Require a non-empty name, preserve the submitted name and mascot, and default members to an empty array. Return members when listing or creating teams.
- Leaderboard approach: Sum persisted activity points per team, sort by total points descending, and assign ranks starting at 1.
- Workout suggestions approach: Find the user by email and recommend a workout aligned with their fitness goal or recent activity, with an explanatory reason. Return a deliberate 4xx response if no user matches.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend` — Last run: 4 of 5 tests failed. Missing validation for required User, Team, Activity, and Workout fields points to incomplete model schemas; the seed/idempotency test passed.
- API command: `npm run test:api --prefix octofit-tracker/backend` — Last run: 4 of 4 tests failed. 501 responses for invalid activity/team input, an empty leaderboard, and a mismatched workout recommendation point to incomplete route/service behavior.
- Frontend command: `npm test --prefix octofit-tracker/frontend` — Last run: 15 of 15 tests failed. Missing API-rendered content, user-specific workout requests, and accessible loading, empty, and error states point to incomplete presentation behavior.
- Backend build command: `npm run build --prefix octofit-tracker/backend` — Run after the data and API changes; TypeScript errors indicate backend code does not type-check.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend` — Run after presentation changes; TypeScript or Vite errors indicate the frontend does not type-check or bundle.
