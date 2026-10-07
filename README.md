<p align="center">
  <img src="https://raw.githubusercontent.com/coccinella-labs/vision/main/.github/assets/thumbnail.png" alt="vision" width="100%">
</p>

# Vision

Vision is a containerized demo scaffold for image analysis: a FastAPI backend with upload plumbing and placeholder analysis endpoints, plus a React frontend that uploads images and displays responses. The backend currently returns hardcoded placeholder strings, not model output. No vision model is integrated.

Status: Early demo. Upload flow functional. Analysis endpoints return placeholders. Testing infrastructure in place.

## Getting Started

Vision requires Docker and Docker Compose for containerized deployment, or Node.js 18+, Python 3.9+, and a local environment for development without containers.

To run the complete stack, clone the repository with `git clone https://github.com/coccinella-labs/vision.git`, navigate to the root directory, and run `docker compose up --build`. The frontend will be available at `http://localhost:3000` and the backend API at `http://localhost:8001`. From there, you can upload images and receive placeholder responses.

For local development of the backend, navigate to the `backend` directory, create a Python environment, install dependencies with `pip install -r requirements.txt`, and start the development server with `python -m uvicorn app:app --reload`. The backend will serve with an interactive Swagger UI for testing endpoints.

For local development of the frontend, navigate to the `frontend` directory, install Node dependencies with `npm install`, and start the development server with `npm start`. The frontend runs on `http://localhost:3000` and will automatically refresh when you edit source files.

When both backend and frontend are running locally, they communicate directly. For production deployments or when using containers, the services coordinate through the Docker network defined in `docker-compose.yml`.

## Architecture

Vision is organized as two main services. The backend directory contains the FastAPI application in `backend/app.py` with route handlers for upload and stub analysis endpoints. The frontend directory contains a React application in JavaScript (not TypeScript) with components for upload and result display.

The flow is straightforward. A user selects or uploads an image through the React interface, which sends it as a multipart form request to the FastAPI backend. The backend receives the image and saves it locally under `uploads/`. The analysis endpoints return hardcoded placeholder JSON; they do not call any model.

Backend code is anchored in `backend/app.py`. Frontend code is anchored in `frontend/src`, which contains React components for the upload interface, result display, and state management. Testing is configured in `backend/tests` for backend unit tests and `frontend/cypress` for end-to-end tests.

## Configuration

The backend reads one environment variable, `LLAMA_API_URL`, defaulting to `http://localhost:8000/v1/chat/completions`. Nothing in the current code calls it; it is a leftover default for a future integration. Backend port configuration defaults to `8001` when running in Docker. Frontend port defaults to `3000`.

Configuration is read from environment files if present (e.g., `.env` in the backend directory) or from system environment variables. When running with `docker-compose.yml`, environment variables can be set in the compose file itself or in a `.env` file at the repository root.

## API

The backend exposes the following REST endpoints. `GET /` returns a status message. `POST /api/upload` accepts a multipart form with an image file, saves it under `uploads/`, and returns a local URL. `POST /api/analyze` and `POST /api/image-analysis` accept JSON with an optional image URL and a prompt, and return hardcoded placeholder responses.

There are no health, models, or provider endpoints. There is no model integration, so there are no API keys to configure and no per-provider settings.

## Contributing

Start by reading the CONTRIBUTING.md file in the repository. It documents the commit message format (Conventional Commits), which should be followed for all PRs. When contributing, fork the repository, create a feature branch with a descriptive name, make your changes, run the test suites, and open a PR with a clear description of what you changed and why.

Code standards: ensure all backend tests pass with `pytest` before pushing, and all frontend tests pass with `npm test`. Use type hints in Python code. The frontend is JavaScript; a TypeScript migration has not happened. When modifying the analysis logic, update relevant tests to cover the new behavior. For frontend changes, run `npm run cypress:open` to test interactions end-to-end before considering the change complete.

To add support for a real vision model provider, implement the integration in `backend/app.py` (or in a new module if the integration is complex), add tests for response parsing and error handling, and update this README's API and Configuration sections, which currently describe placeholder behavior.

## Build and Deploy

Build the backend Docker image with `docker build -t vision-backend ./backend`. Build the frontend image with `docker build -t vision-frontend ./frontend`. Run both together with `docker compose up`, which orchestrates the services, sets up networking, and exposes the appropriate ports. For local testing without Docker, follow the Getting Started instructions for independent backend and frontend development.

Testing is split between backend and frontend. Run backend tests with `cd backend && pytest`. Run frontend unit tests with `cd frontend && npm test`. Run end-to-end tests with `cd frontend && npm run cypress:open`, which opens the Cypress test runner and lets you observe tests running against the live application.

## Known Limitations

The analysis endpoints return hardcoded placeholders; no model is called. There is no image preprocessing beyond saving the uploaded file, no size limits, and no format normalization. The frontend does not persist analysis history across browser sessions; results are stored only in memory. WebSocket support for streaming results is not implemented; all requests are request-response only.

## Troubleshooting

If the backend fails to start, verify that Python 3.9+ is installed and that all dependencies in `requirements.txt` are available. If you see connection refused errors between frontend and backend in Docker, check that both containers are running with `docker compose ps` and that they are on the same Docker network.

If image uploads fail, ensure the `uploads/` directory is writable. If the frontend shows a blank page after deployment, check browser console logs with F12 to see if there are JavaScript errors or network failures.

If Cypress tests fail intermittently, increase the timeout values in `frontend/cypress.config.js`. If you need to debug a specific test, use `.only` on the test case to run only that test.

## Related Documentation

The backend uses FastAPI for HTTP routing. The frontend is React in JavaScript. Docker Compose handles orchestration and networking between services. See CONTRIBUTING.md for development workflow and commit standards. See SECURITY.md for security policies and responsible disclosure information.

## License

MIT. See LICENSE file.

## Contact

Questions? Open an issue on GitHub or see CONTRIBUTING.md for community guidelines.
