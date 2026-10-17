# automated-security-testing-pipeline

docker pull bkimminich/juice-shop

docker run --rm -d --name juice-shop -p 127.0.0.1:3000:3000 bkimminich/juice-shop



docker network create zapnet
docker network connect zapnet juice-shop

docker pull ghcr.io/zaproxy/zaproxy:stable

docker run --rm `
  --network zapnet `
  -v "${PWD}\reports:/zap/wrk/:rw" `
  ghcr.io/zaproxy/zaproxy:stable `
  zap-baseline.py `
  -t http://juice-shop:3000 `
  -r zap-baseline.html `
  -J zap-baseline.json

The command should:
* Connect to Juice Shop through the shared Docker network.
* Crawl the application and perform passive security checks.
* Save an HTML report and a JSON report in your local reports folder.

GitHub Actions:
* Checks out your repository. GitHub retrieves your project files.
* Creates a Docker network. This allows the application and scanner to communicate.
* Starts Juice Shop. GitHub runs the test application in a container.
* Checks that Juice Shop is ready. The workflow waits until the application responds.
* Runs OWASP ZAP. The scanner examines the application.
* Generates reports. ZAP creates an HTML report and a JSON report.
* Uploads the reports as artifacts. This lets you retrieve and examine the results from the GitHub Actions run.