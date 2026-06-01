# JIRA-4: Create a GET /hello/world endpoint

## Requirements

- **Summary**: Create a `GET /hello/world` HTTP endpoint that returns a greeting message.
- **Type**: Task
- **Priority**: Medium
- **Current Status**: To Do

### Acceptance Criteria

1. A `GET /hello/world` endpoint exists and responds with HTTP 200.
2. The response body is a JSON object containing a greeting message (e.g., `{"message": "Hello, World!"}`).
3. The endpoint is served on a configurable port (default: `8080`).
4. The service gracefully handles startup and shutdown.
5. Clear console logging indicates the server is running and on which port.

## Implementation Approach

### Technology Stack

- **Runtime**: Python 3.11+
- **Framework**: FastAPI (lightweight, async-native, auto-docs)
- **Server**: uvicorn (ASGI server for FastAPI)
- **Dependency Management**: pip + `requirements.txt`

### Key Design Decisions

1. **FastAPI over Flask**: FastAPI provides automatic OpenAPI documentation at `/docs`, which is useful for a microservice. It also supports async out of the box.
2. **Single module structure**: Since this is the first endpoint in a new microservice, keep it simple with a single `app.py` until more routes are added.
3. **Configurable via environment variable**: Use `PORT` environment variable (default `8080`) to configure the listening port, following 12-factor app principles.
4. **Standard JSON response**: Return a consistent JSON envelope with a `message` key for future extensibility.

### Project Structure (after implementation)

```
my-new-microservice/
├── app.py                  # FastAPI application entry point
├── requirements.txt        # Python dependencies
├── README.md               # Existing project README
└── docs/
    └── plans/
        └── JIRA-4.md       # This plan
```

## Files to Modify

| File | Action | Description |
|------|--------|-------------|
| `app.py` | **Create** | Main application file with FastAPI app, `/hello/world` route, and uvicorn runner. |
| `requirements.txt` | **Create** | Pin `fastapi`, `uvicorn`, and their dependencies. |
| `README.md` | **Update** | Add instructions for running the service (install deps, start server, test endpoint). |

### Detailed Change Descriptions

#### `app.py` (new file)

- Import `FastAPI` from `fastapi`.
- Create an `app = FastAPI(title="my-new-microservice")` instance.
- Define a `GET /hello/world` handler that returns `{"message": "Hello, World!"}`.
- Add an `if __name__ == "__main__"` block that reads `PORT` from `os.environ` (default `8080`) and runs `uvicorn.run(app, host="0.0.0.0", port=port)`.
- Optionally add a startup event that logs the listening address.

#### `requirements.txt` (new file)

```
fastapi>=0.110.0,<1.0.0
uvicorn[standard]>=0.29.0,<1.0.0
```

Pins to minor versions to ensure stability while allowing patches.

#### `README.md` (update)

Append a "Getting Started" section:

```markdown
## Getting Started

### Prerequisites
- Python 3.11+

### Installation
```bash
pip install -r requirements.txt
```

### Running
```bash
python app.py
```

The server starts on `http://0.0.0.0:8080`. The endpoint is available at `http://localhost:8080/hello/world`.

### Testing
```bash
curl http://localhost:8080/hello/world
# Expected: {"message":"Hello, World!"}
```
```

## Testing Strategy

### Manual Testing (quick verification)

1. Start the server: `python app.py`
2. Verify the endpoint:
   ```bash
   curl -s http://localhost:8080/hello/world | python3 -m json.tool
   ```
   Expected output:
   ```json
   {
       "message": "Hello, World!"
   }
   ```
3. Verify the OpenAPI docs are available at `http://localhost:8080/docs`.

### Automated Testing (future enhancement)

Once a test framework is established (e.g., `pytest` + `httpx`), add:

- `test_hello_world_status_200` — calls `GET /hello/world` and asserts HTTP 200.
- `test_hello_world_response_body` — asserts the JSON body contains `{"message": "Hello, World!"}`.
- `test_openapi_docs_available` — calls `GET /docs` and asserts HTTP 200.
- Test using `TestClient` from `starlette.testclient` or `httpx.AsyncClient` with FastAPI's app directly (no need to run the server).

For the initial implementation, manual testing is sufficient. Automated tests should be added in a follow-up ticket when the test infrastructure is set up.

## Definition of Done

- [ ] `app.py` created with FastAPI app and `GET /hello/world` endpoint.
- [ ] `requirements.txt` created with `fastapi` and `uvicorn` dependencies.
- [ ] `README.md` updated with setup and usage instructions.
- [ ] Server starts successfully with `python app.py`.
- [ ] `curl http://localhost:8080/hello/world` returns `{"message":"Hello, World!"}` with HTTP 200.
- [ ] Server responds on the default port (8080) without any configuration.
- [ ] Server responds on a custom port when the `PORT` environment variable is set.
- [ ] OpenAPI docs page renders at `/docs`.
- [ ] Plan reviewed and approved.
