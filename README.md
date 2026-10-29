# cs321-ghcr-integration

# CS 321 — GHCR and Docker Compose Integration Experiment

## Research Objective

Evaluate whether independently developed CS 321 subsystems can publish Docker images to GitHub Container Registry (GHCR) and run together locally using Docker Compose, without requiring teams to build each other's source code.

This is a two-subsystem proof of concept, not a deployment of the eight actual course subsystems.

## Test Environment

- Computer: Apple Silicon Mac (ARM64)
- Docker Desktop: 4.97.0
- Docker Compose: v5.6.1
- Container registry: GitHub Container Registry (GHCR)
- Automation: GitHub Actions
- Integration configuration: compose.yaml

## Test Applications

**Subsystem A**
- Image: ghcr.io/archanaharidas/cs321-azure-container-test:latest
- Architecture: linux/amd64
- Local URL: http://localhost:18080
- Runs through Docker's AMD64 compatibility support on the ARM64 Mac

**Subsystem B**
- Image: ghcr.io/archanaharidas/cs321-ghcr-service-b
- Architecture selected on Mac: linux/arm64
- Local URL: http://localhost:18081
- Version 2 pinned to a SHA-256 image digest

## Reproduce the Experiment

Install Docker Desktop and make sure Docker Compose is available.

Clone this repository:

    git clone https://github.com/archanaharidas/cs321-ghcr-integration.git

Enter the project folder:

    cd cs321-ghcr-integration

Pull and start both published images:

    docker compose up -d --pull always

Check container status:

    docker compose ps

Test both services:

    curl http://localhost:18080/
    curl http://localhost:18081/

Test communication from Subsystem A to Subsystem B:

    docker compose exec -T subsystem-a python -c "import urllib.request; print(urllib.request.urlopen('http://subsystem-b:8080/health', timeout=5).read().decode())"

Expected health response:

    {"status":"healthy"}

Stop the experiment:

    docker compose down

No local application build is required because Compose references published GHCR images.

## Verified Results

1. GitHub Actions successfully built and published Subsystem B to GHCR.
2. Both independent GHCR images were successfully pulled onto the local Mac.
3. Docker Compose started both services on a shared network.
4. Subsystem A successfully requested Subsystem B's health endpoint.
5. Subsystem A successfully retrieved Subsystem B's application response.
6. A code change to Subsystem B triggered another successful GitHub Actions publication.
7. Docker Compose retrieved and deployed the updated Version 2 image.
8. The image digest changed between versions.
9. Subsystem B's updated image was selected as linux/arm64 on the Mac.
10. Docker Compose successfully deployed Subsystem B using a pinned SHA-256 digest, and the Version 2 response was verified.

### Image Update Evidence

Original Subsystem B digest:

    sha256:ae0f1601aab80789384d4c10037e9f750d52de5a53adf38c52e79d16b33f3877

Updated Subsystem B digest:

    sha256:0fba9503c5abd8d2ae6fd0d0d5315d0d12669af68429abe88e402cd3e89d79df

Verified Version 2 message:

    Subsystem B Version 2 - Updated through GitHub Actions

## Findings for CS 321

This experiment supports using GitHub Actions, GHCR, and Docker Compose for local integration testing across independently developed teams.

Each team could publish a tested image, while a shared integration repository specifies which images and versions to run.

Digest pinning helps teams reproduce an integration environment without unexpectedly receiving changes published under the latest tag.

## Limitations and Future Work

The experiment has not yet tested:

- Integration of all eight actual CS 321 subsystems
- Real cross-subsystem business workflows
- Shared authentication and authorization
- Database dependencies and migrations across teams
- Private GHCR package permissions
- Conflicting ports and environment variables
- Full integration testing after simultaneous subsystem updates
- Reproducing the setup on other students' computers

The two test services communicate over HTTP, but this does not establish compatibility between the real course applications.

## Conclusion

The two-subsystem proof of concept succeeded. GitHub Actions and GHCR can distribute independently built container images, and Docker Compose can assemble them into a working local integration environment.

The approach is technically feasible for the CS 321 course, subject to further validation using the actual eight subsystems and their integration requirements.
