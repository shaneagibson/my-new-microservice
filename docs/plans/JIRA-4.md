# JIRA-4: Create a GET /hello/world endpoint

## Requirements

**Summary:** Add a `GET /hello/world` endpoint to the microservice that returns a simple JSON response.

**Type:** Task  
**Priority:** Medium  
**Status:** To Do → In Progress  

**Acceptance Criteria:**
- The service exposes a `GET /hello/world` route
- The endpoint returns HTTP 200 with a JSON body containing a `message` field with value `"Hello, World!"`
- The service starts on a configurable port (defaulting to `3000`)
- The code includes a health-check or readiness endpoint (`GET /health` or similar) to verify the service is running
- All changes are covered by automated tests
- The service gracefully handles unhandled errors (e.g., returns 500 with a JSON error response)

## Implementation Approach

### Technology Stack
Since this is a new microservice with no existing framework, we will use:
- **Node.js** (v18 LTS or later) with **TypeScript**
- **Express.js** as the HTTP framework (lightweight, well-known, fits the scope)
- **Jest** + **Supertest** for integration testing
- **ts-node-dev** for local development with hot-reload

### Design Decisions
1. **Explicit over magical** — Keep the Express app setup straightforward; no heavy framework abstractions
2. **Typed responses** — Use TypeScript interfaces for request/response shapes
3. **Config via environment variables** — Port configuration via `PORT` env var, defaulting to `3000`
4. **Separation of concerns** — Route definitions in a dedicated `routes` directory, app bootstrap in a root `src/index.ts`
5. **Minimal dependencies** — Only express, typescript, jest, supertest, and their type packages

### Route Design
```
GET /hello/world  →  { "message": "Hello, World!" }
GET /health       →  { "status": "ok" }
```

## Files to Modify

| File | Action | Description |
|------|--------|-------------|
| `package.json` | Create | Project manifest: name, scripts (`build`, `start`, `dev`, `test`), dependencies |
| `tsconfig.json` | Create | TypeScript compiler configuration |
| `src/index.ts` | Create | Application entry point — starts Express server on configured port |
| `src/app.ts` | Create | Express app factory — creates and configures the app instance (for testability) |
| `src/routes/health.ts` | Create | `GET /health` route handler |
| `src/routes/helloWorld.ts` | Create | `GET /hello/world` route handler |
| `src/types.ts` | Create | Shared TypeScript interfaces/types |
| `src/errors.ts` | Create | Error-handling middleware |
| `src/__tests__/app.test.ts` | Create | Integration tests using Supertest |

### Detailed Changes Per File

**`package.json`**
- `name`: `"my-new-microservice"`
- `version`: `"1.0.0"`
- Scripts: `build` (tsc), `start` (node dist/index.js), `dev` (ts-node-dev --respawn src/index.ts), `test` (jest)
- Dependencies: `express`
- DevDependencies: `typescript`, `@types/express`, `@types/node`, `ts-node-dev`, `jest`, `ts-jest`, `@types/jest`, `supertest`, `@types/supertest`

**`tsconfig.json`**
- Target: ES2020
- Module: commonjs
- outDir: `./dist`
- rootDir: `./src`
- strict: true
- esModuleInterop: true

**`src/index.ts`**
- Import `app` from `./app`
- Read `PORT` from `process.env` (default `3000`)
- Call `app.listen(PORT, callback)`
- Export nothing (entry point only)

**`src/app.ts`**
- Factory function `createApp()` that returns an Express application
- Registers JSON body parser middleware
- Mounts `/hello/world` and `/health` routes
- Registers error-handling middleware (last)
- Designed this way so tests can import `app` without triggering `listen()`

**`src/routes/helloWorld.ts`**
- Express Router handling `GET /`
- Returns `{ message: "Hello, World!" }` with status 200
- Exported as `helloWorldRouter`, mounted at `/hello/world` in `app.ts`

**`src/routes/health.ts`**
- Express Router handling `GET /`
- Returns `{ status: "ok" }` with status 200
- Exported as `healthRouter`, mounted at `/health` in `app.ts`

**`src/types.ts`**
- `HelloWorldResponse` interface: `{ message: string }`
- `HealthResponse` interface: `{ status: string }`
- `ErrorResponse` interface: `{ error: string }`

**`src/errors.ts`**
- Express error-handling middleware (4-argument handler)
- Logs the error to console
- Returns `{ error: "Internal Server Error" }` with status 500

**`src/__tests__/app.test.ts`**
- Test: `GET /hello/world` returns `200` with `{ message: "Hello, World!" }`
- Test: `GET /health` returns `200` with `{ status: "ok" }`
- Test: `GET /nonexistent` returns `404`
- Uses `supertest(app)` (the app instance, not the listening server)

## Testing Strategy

### Unit & Integration Tests (Jest + Supertest)
- **Route tests**: Verify each endpoint returns the correct status code and body shape
- **404 handling**: Verify undefined routes return 404
- **Error middleware**: Verify that throwing an error inside a handler returns a 500 JSON error response

### How to Run Tests
```bash
npm test                # Run Jest test suite
npm run test:coverage   # Run tests with coverage report
```

### Manual Verification
```bash
npm run dev             # Start the dev server on port 3000
curl http://localhost:3000/hello/world   # Expect: {"message":"Hello, World!"}
curl http://localhost:3000/health        # Expect: {"status":"ok"}
```

## Definition of Done

- [ ] `package.json` with all required dependencies and scripts created
- [ ] `tsconfig.json` with TypeScript configuration created
- [ ] `src/index.ts` — entry point that starts the server on a configurable port
- [ ] `src/app.ts` — Express app factory function
- [ ] `src/routes/helloWorld.ts` — GET /hello/world route returning `{"message":"Hello, World!"}`
- [ ] `src/routes/health.ts` — GET /health route returning `{"status":"ok"}`
- [ ] `src/types.ts` — TypeScript interfaces for all response shapes
- [ ] `src/errors.ts` — Global error-handling middleware
- [ ] `src/__tests__/app.test.ts` — Integration tests for all routes
- [ ] All tests pass (`npm test` exits with code 0)
- [ ] `npm run build` compiles TypeScript without errors
- [ ] Manual verification: `curl localhost:3000/hello/world` returns the expected response
- [ ] Code is committed to the `JIRA-4` branch
- [ ] PR created against `main`
