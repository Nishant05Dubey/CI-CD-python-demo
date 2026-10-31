# Python CI/CD Pipeline using GitHub Actions and Docker

A beginner-friendly DevOps assignment: create a small Flask web app, test it,
package it with Docker, and automate validation and image publishing with GitHub Actions.

## Technologies

- Python 3.12 and Flask: web application
- pytest: automated tests
- Docker: application image and container
- GitHub Actions: CI/CD automation
- Docker Hub: image registry

## Project structure

```text
.
|-- app.py
|-- requirements.txt
|-- tests/
|   `-- test_app.py
|-- Dockerfile
|-- .dockerignore
|-- .gitignore
|-- .github/
|   `-- workflows/
|       `-- ci-cd.yml
`-- README.md
```

`app.py` defines the home and health routes. The tests verify HTTP status codes
and response contents. `requirements.txt` lists runtime dependencies; pytest is
installed separately for development and CI. The Dockerfile packages only the
application and its runtime dependencies. Ignore files exclude generated files,
local secrets, and unnecessary build context.

## Run locally (Windows PowerShell)

Install Python 3.12 with the Windows Python launcher first. Open PowerShell in
this project folder, then run:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt pytest
.\.venv\Scripts\python.exe -m pytest -v
.\.venv\Scripts\python.exe app.py
```

These commands use the virtual environment directly, so activation and execution
policy changes are unnecessary. If the `py` launcher is unavailable but Python
3.12 is on PATH, use `python -m venv .venv` for the first command.

Open `http://localhost:5000` in your browser. In a second PowerShell terminal:

```powershell
Invoke-WebRequest -Uri http://localhost:5000/ -UseBasicParsing
Invoke-RestMethod -Uri http://localhost:5000/health
```

The home route displays a welcome message. `/health` returns HTTP 200 and
`{"status":"healthy"}`. Stop the app with Ctrl+C in its terminal.

Flask listens on `0.0.0.0:5000` so it is accessible through Docker's port mapping.
Its built-in server is sufficient for this learning assignment; production
hosting would use a production WSGI server.

## Build and run with Docker

Install and start Docker Desktop with Linux containers enabled. From this folder:

```powershell
docker build -t python-cicd-app:local .
docker run -d --name python-cicd-local -p 8000:5000 python-cicd-app:local
Invoke-WebRequest -Uri http://localhost:8000/ -UseBasicParsing
Invoke-RestMethod -Uri http://localhost:8000/health
docker logs python-cicd-local
docker stop python-cicd-local
docker rm python-cicd-local
```

Wait a moment after starting the container before checking its routes. Port 8000
on your computer forwards to port 5000 inside the container. These commands build
and test locally and do not publish an image.

## How the GitHub Actions pipeline works

The workflow runs for pushes to `main`, pull requests targeting `main`, and manual
runs through **Actions > Python CI/CD Pipeline > Run workflow**.

1. Check out the repository on an Ubuntu runner and set up Python 3.12.
2. Install Flask and pytest, then run both route tests.
3. Build a Docker image and start a temporary container on host port 8000.
4. Use curl to check `/health`, retrying while the container starts, and verify
   that the response JSON reports `healthy`.
5. Remove the test container even if a previous step fails. Print container logs
   if validation fails to help with troubleshooting.
6. Only after successful validation on a **push to `main`**, log in to Docker Hub,
   tag the tested image as `latest` and the full Git commit SHA, and push both.

The published tags are `YOUR_DOCKERHUB_USERNAME/python-cicd-app:latest` and
`YOUR_DOCKERHUB_USERNAME/python-cicd-app:FULL_COMMIT_SHA`.
Pull requests and manual runs validate the project without logging in or
publishing. Failed validation prevents publishing. Publishing stores an image in
Docker Hub; it does not deploy a running application to a server.

## Configure GitHub and Docker Hub

Before pushing this project to `main`:

1. Create your GitHub repository and a Docker Hub repository named `python-cicd-app`.
2. In Docker Hub, create a personal access token with permission to push images
   (write access). Keep the token private.
3. In your GitHub repository, open **Settings > Secrets and variables > Actions**.
4. Add these **repository secrets**:

   | Secret | Value |
   | --- | --- |
   | `DOCKERHUB_USERNAME` | Your Docker Hub username |
   | `DOCKERHUB_TOKEN` | Your Docker Hub access token |

5. Push the project to the repository's `main` branch and inspect the Actions run.

Never put passwords or tokens in source files or commit them to Git. The workflow
reads credentials from repository secrets and passes the token to Docker through
standard input. A push to `main` without the required secrets will fail at login.
You can use a manual run to validate without publishing before configuring secrets.

## References

- [GitHub: Building and testing Python](https://docs.github.com/en/actions/tutorials/build-and-test-code/python)
- [Docker: Push images to Docker Hub](https://docs.docker.com/docker-hub/repos/manage/hub-images/push/)
