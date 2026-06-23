# DevOps Portfolio Project - Notes

## Phase 1: Docker

**Goal:** Containerize a Node.js + MongoDB web app so it runs the same everywhere.

**What I did:**
- Cloned a Node.js profile app (Node.js + MongoDB + MongoExpress)
- Built a Docker image from the Dockerfile: `docker build -t js-app:1.0 .`
- Dockerfile uses node:20-alpine base (small image), copies app, runs npm install, starts server.js
- Used Docker Compose to run 3 containers together: app (port 3000), MongoDB (27017), MongoExpress (8081)

**Bugs I hit and fixed (the important part):**
1. App container crashed on first run. Ran `docker logs` → MongoServerSelectionError, couldn't reach MongoDB. Cause: MongoDB container wasn't running. Fix: use Docker Compose to run all containers together instead of running the app alone.
2. MongoExpress couldn't connect — was looking for hostname `mongo` but service was named `mongodb`. Fix: added ME_CONFIG_MONGODB_URL with correct hostname.
3. App still couldn't reach MongoDB — server.js was hardcoded to `localhost:27017`. In Docker, containers talk via service names, not localhost. Fix: changed connection string to use `mongodb` (the service name). Then rebuilt the image.

**Key concepts learned:**
- Image = packaged app (immutable), Container = running instance of an image
- Containers communicate by service name, NOT localhost
- Docker layers: each Dockerfile line = a layer; only changed layers + ones after rebuild, rest are cached
- Docker Compose runs multiple containers together with one command + auto networking

**Why these choices:**
- Docker → consistency across environments (no "works on my machine")
- Alpine base → tiny image (5MB vs 200MB), faster builds, smaller attack surface
- Docker Compose → app needs 3 services that talk to each other; one command manages all

**Production thinking (what I'd change for real users):**
- Replace Docker Compose with Kubernetes (auto-scaling, self-healing)
- Use AWS ECR for private image storage instead of local images
- Remove hardcoded credentials → use AWS Secrets Manager
- Add container health checks and resource limits

**Result:** All 3 containers running. Filled profile form at localhost:3000, data saved to MongoDB, visible in MongoExpress at localhost:8081. Pushed code to GitHub.

## Phase 2: Jenkins CI/CD

**Goal:** Automate building the Docker image — Jenkins pulls code from GitHub and builds it automatically.

**What I did:**
- Launched EC2 instance (Ubuntu 22.04, t2.medium) to run Jenkins 24/7 (not laptop — laptop turns off)
- Installed Java, then Jenkins, on the EC2
- Installed Docker on the EC2 + added jenkins user to docker group (so Jenkins can build images)
- Created a pipeline job in Jenkins
- Moved pipeline into a Jenkinsfile in the GitHub repo (pipeline-as-code)
- Pointed Jenkins to "Pipeline script from SCM" → reads Jenkinsfile from repo

**Bugs I hit and fixed:**
1. GPG key error during Jenkins install — Jenkins had rotated their signing key. Fix: downloaded the current 2026 key.
2. Jenkins service wouldn't start. Read `journalctl -xeu jenkins.service` log → "requires Java 21, found Java 17." Fix: installed Java 21, set as default with update-alternatives. Service started.

**Key concepts learned:**
- CI = on every push, automatically build/test. CD = automatically deploy after CI.
- Jenkins runs on always-on server (EC2), not laptop
- Webhook = GitHub notifies Jenkins instantly on push (asynchronous, event-based — not polling)
- Jenkinsfile = pipeline as code, stored in repo, version controlled, travels with the code
- Read logs to troubleshoot — don't guess (found Java issue this way)
- Docker layer caching — saw "Using cache" on unchanged steps, made build faster

**Why pipeline-as-code (Jenkinsfile in repo) over Jenkins config box:**
- Version controlled in Git (history, rollback)
- Lives with the code, survives Jenkins being rebuilt
- Team can review pipeline changes

**Status:** Jenkins pulls code from GitHub + builds Docker image automatically via Jenkinsfile. ✅
**Next:** Push image to AWS ECR, then deploy.
- Added deploy stage: Jenkins pulls image from ECR, runs as container
- BUG: ubuntu user got "permission denied" on docker ps → ubuntu user 
  wasn't in docker group (only jenkins was). Fixed with sudo / usermod.
- Full CI/CD working: code→build→push ECR→deploy. Verified container 
  running with docker ps.
- Note: app runs but needs MongoDB alongside (deferred to Phase 4/K8s)