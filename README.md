<p align="center">
  <img src="https://raw.githubusercontent.com/coccinella-labs/vision/main/.github/assets/thumbnail.png" alt="vision" width="100%">
</p>

# Vision

Vision is a containerized AI demonstration for image analysis. It provides a FastAPI backend that processes images using multimodal AI models and a React frontend that handles image uploads and displays analysis results. The application is built to run entirely in Docker, with local development paths for iterating on the backend and frontend independently.

Status: Early demo. Image processing functional. Testing infrastructure in place.

## Getting Started

Vision requires Docker and Docker Compose for containerized deployment, or Node.js 18+, Python 3.9+, and a local environment for development without containers.

To run the complete stack, clone the repository with `git clone https://github.com/coccinella-labs/vision.git`, navigate to the root directory, and run `docker compose up --build`. The frontend will be available at `http://localhost:3000` and the backend API at `http://localhost:8001`. From there, you can upload images and receive AI-powered analysis results.

For local development of the backend, navigate to the `backend` directory, create a Python environment, install dependencies with `pip install -r requirements.txt`, and start the development server with `python -m uvicorn app:app --reload`. The backend will serve on `http://localhost:8000` with an interactive Swagger UI for testing endpoints.

For local development of the frontend, navigate to the `frontend` directory, install Node dependencies with `npm install`, and start the development server with `npm start`. The frontend runs on `http://localhost:3000` and will automatically refresh when you edit source files.

When both backend and frontend are running locally, they communicate directly. For production deployments or when using containers, the services coordinate through the Docker network defined in `docker-compose.yml`.

## Architecture

Vision is organized as two main services. The backend directory contains the FastAPI application that handles image uploads, preprocessing, and calls to AI vision models for analysis. It exposes REST endpoints for image submission and result retrieval. The frontend directory contains a React application built with TypeScript that provides a user interface for selecting or uploading images, submitting them to the backend, and displaying the analysis results in a structured format.

The flow is straightforward. A user selects or uploads an image through the React interface, which sends it as a multipart form request to the FastAPI backend. The backend receives the image, performs validation and preprocessing (resizing, format normalization), sends it to the configured AI vision model (Gemini, GPT-4V, or another supported provider), receives the structured analysis response, and returns it to the frontend. The frontend displays the analysis results, with support for multiple analysis modes or visualization options depending on the image type.

Backend code is anchored in `backend/app.py`, which defines the FastAPI application, route handlers for image upload and analysis, and integration with the selected vision model. Frontend code is anchored in `frontend/src`, which contains React components for the upload interface, result display, and state management. Testing is configured in `backend/tests` for backend unit tests and `frontend/cypress` for end-to-end tests.

## Configuration

Vision reads configuration primarily through environment variables. The selected AI vision provider is set via `VISION_PROVIDER` (currently supporting gemini, openai, or other supported models), and the corresponding API key is set via `GEMINI_API_KEY`, `OPENAI_API_KEY`, or similar environment variables depending on the provider. Backend port configuration defaults to `8001` when running in Docker but can be adjusted for local development. Frontend port defaults to `3000`.

Configuration is read from environment files if present (e.g., `.env` in the backend directory) or from system environment variables. When running with `docker-compose.yml`, environment variables can be set in the compose file itself or in a `.env` file at the repository root. For local development, create a `backend/.env` file with your API keys to avoid hardcoding credentials.

## API

The backend exposes the following REST endpoints. `POST /api/images/upload` accepts a multipart form with an image file, preprocesses it, sends it to the vision model, and returns the analysis in JSON format with fields for detected objects, text content, scene description, and any other metadata the model provides. `GET /api/health` returns a simple status check indicating the backend is running. `GET /api/models` returns a list of supported vision models currently available through the backend.

The response format from the analysis endpoint is structured JSON that varies based on the analysis mode and the vision model selected. Typically, it includes a summary of what was detected in the image, structured fields for each type of analysis, confidence scores where applicable, and any processing metadata such as model name and analysis duration.

## Contributing

Start by reading the CONTRIBUTING.md file in the repository. It documents the commit message format (Conventional Commits), which should be followed for all PRs. When contributing, fork the repository, create a feature branch with a descriptive name, make your changes, run the test suites, and open a PR with a clear description of what you changed and why.

Code standards: ensure all backend tests pass with `pytest` before pushing, and all frontend tests pass with `npm test`. Use type hints in Python code. Use TypeScript in React components rather than JavaScript to catch type errors early. When modifying the AI analysis logic, update relevant tests to cover the new behavior. For frontend changes, run `npm run cypress:open` to test interactions end-to-end before considering the change complete.

To add support for a new vision model provider, update the backend configuration to recognize the new provider name, implement the model integration in `backend/app.py` (or in a new module if the integration is complex), and add tests for the new provider's response parsing and error handling.

## Build and Deploy

Build the backend Docker image with `docker build -t vision-backend ./backend`. Build the frontend image with `docker build -t vision-frontend ./frontend`. Run both together with `docker compose up`, which orchestrates the services, sets up networking, and exposes the appropriate ports. For local testing without Docker, follow the Getting Started instructions for independent backend and frontend development.

Testing is split between backend and frontend. Run backend tests with `cd backend && pytest`. Run frontend unit tests with `cd frontend && npm test`. Run end-to-end tests with `cd frontend && npm run cypress:open`, which opens the Cypress test runner and lets you observe tests running against the live application.

## Known Limitations

The vision backend currently supports a limited set of analysis modes; additional analysis types can be added by extending the model integration layer. Image preprocessing has a maximum size limit to reduce API costs; very large images are automatically scaled down. The frontend does not currently persist analysis history across browser sessions; results are stored only in memory. Error handling for API failures is basic; if a vision model call fails, the user sees a generic error message rather than actionable troubleshooting guidance. WebSocket support for streaming results is not yet implemented; all analysis requests are request-response only.

## Troubleshooting

If the backend fails to start, verify that Python 3.9+ is installed, that all dependencies in `requirements.txt` are available, and that your API key is set correctly. If you see connection refused errors between frontend and backend in Docker, check that both containers are running with `docker compose ps` and that they are on the same Docker network.

If image uploads fail with file format errors, ensure the uploaded file is a valid JPEG, PNG, or WebP image. If the vision model returns empty or truncated results, check that your API key is valid and has quota available. If the frontend shows a blank page after deployment, check browser console logs with F12 to see if there are JavaScript errors or network failures.

If Cypress tests fail intermittently, increase the timeout values in `frontend/cypress.config.ts`, as slow network conditions or model response times may cause tests to timeout. If you need to debug a specific test, use `.only` on the test case to run only that test.

## Performance

Image upload and preprocessing typically complete in under 500ms. The vision model analysis time depends on model latency and image complexity; expect 1-3 seconds for most images with Gemini and 2-5 seconds with GPT-4V due to network round-trip time. Results are returned immediately after model completion with no additional processing overhead. Memory usage of the backend is steady around 100-150MB idle and scales linearly with concurrent requests.

## Related Documentation

The backend uses FastAPI for HTTP routing and integrates with AI vision providers through their official SDKs. The frontend is built with React and TypeScript, using standard patterns for component state and props. Docker Compose handles orchestration and networking between services. See CONTRIBUTING.md for development workflow and commit standards. See SECURITY.md for security policies and responsible disclosure information.

## License

MIT. See LICENSE file.

## Contact

Questions? Open an issue on GitHub or see CONTRIBUTING.md for community guidelines.
