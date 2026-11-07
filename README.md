# automated-security-testing-pipeline

An automated security testing pipeline that scans [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) 
with [OWASP ZAP](https://www.zaproxy.org/) (baseline / passive scan), plus a local Juice Shop setup for manual security testers.

## Contents

| Path | Purpose |
|---|---|
| `.github/` | GitHub Actions workflow that runs the ZAP baseline scan |
| `docker-compose.yml` | Local Juice Shop for manual testers (same image as the pipeline) |
| `reports/` | Local ZAP HTML and JSON reports |

## 1. Run the scan locally

Start Juice Shop:

```powershell
docker pull bkimminich/juice-shop

docker run --rm -d --name juice-shop -p 127.0.0.1:3000:3000 bkimminich/juice-shop
```

Create a shared network and connect Juice Shop to it:

```powershell
docker network create zapnet
docker network connect zapnet juice-shop
```

Pull and run ZAP:

```powershell
docker pull ghcr.io/zaproxy/zaproxy:stable

docker run --rm `
  --network zapnet `
  -v "${PWD}\reports:/zap/wrk/:rw" `
  ghcr.io/zaproxy/zaproxy:stable `
  zap-baseline.py `
  -t http://juice-shop:3000 `
  -r zap-baseline.html `
  -J zap-baseline.json
```

The command:

* Connects to Juice Shop through the shared Docker network.
* Crawls the application and performs passive security checks.
* Saves an HTML report and a JSON report in your local `reports` folder.

> **Note:** the commands above use the latest `bkimminich/juice-shop` image. 
> The GitHub Actions pipeline pins the image by digest (see [Versions used](#versions-used)), 
> so use the pinned image if you need results comparable with pipeline runs.

## 2. GitHub Actions

The workflow:

* **Checks out your repository.** GitHub retrieves your project files.
* **Creates a Docker network.** This allows the application and scanner to communicate.
* **Starts Juice Shop.** GitHub runs the test application in a container.
* **Checks that Juice Shop is ready.** The workflow waits until the application responds.
* **Runs OWASP ZAP.** The scanner examines the application.
* **Generates reports.** ZAP creates an HTML report and a JSON report.
* **Uploads the reports as artifacts.** This lets you retrieve and examine the results from the GitHub Actions run.

Reports are downloaded from the run's **Artifacts** section:

`https://github.com/eddiecns/automated-security-testing-pipeline/actions/runs/<run-id>`

## 3. Manual security testing (local Juice Shop)

Manual testers do **not** need to run the pipeline. They test the same application the pipeline scans, using a local, 
disposable Juice Shop instance started with Docker Compose.

### Prerequisites

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.
* This repository cloned (only `docker-compose.yml` is needed).

### Start Juice Shop

```powershell
docker compose up -d
```

Open <http://localhost:3000> in your browser.

### Confirm the version

Open <http://localhost:3000/rest/admin/application-version>. It should return:

```json
{"version":"20.3.0"}
```

Record the version in your test results so manual and automated findings can be compared.

### Reset to a clean state

Juice Shop stores its data inside the container, so recreating the container wipes everything done during testing:

```powershell
docker compose down
docker compose up -d
```

### Stop

```powershell
docker compose down
```

### Rules of engagement

* Juice Shop is **deliberately vulnerable**. It is bound to `127.0.0.1` so it is only reachable from your own machine. 
* Do not change this to expose it on a network.
* Test **only** `http://localhost:3000`. Do not attack any other system.
* Do not use the pipeline, repository secrets or any production system for manual tests.
* Reset the container between test sessions so results are not affected by earlier changes.

## Versions used

| Component | Version | Image |
|---|---|---|
| OWASP Juice Shop | 20.3.0 | `bkimminich/juice-shop@sha256:42843723a7bf4246aa5d63c324978b17ef296ac78434aeddd34c427bfb5853f4` |
| OWASP ZAP | 2.17.0 | `ghcr.io/zaproxy/zaproxy:stable@sha256:5ccfff9a411743ab2615136a2edd7416bec9ce41572fa94e935df38c14adb2b6` |

If the pinned digests in the pipeline change, update `docker-compose.yml` and this table to match.