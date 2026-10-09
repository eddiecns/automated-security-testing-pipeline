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