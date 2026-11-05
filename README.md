# ISAC_385_LAB
This is a class ISAC385 Lab Assignments
name: TaskMaster-API Automation Revise

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: "0 6 * * 1"   # weekly, since new CVEs appear constantly

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Installs the Snyk CLI onto the runner
      - uses: snyk/actions/setup@master

      # Only needed if your project requires its runtime to resolve deps
      # (e.g., Node, Java, Python)
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci

      - name: Snyk test
        run: snyk test --severity-threshold=high
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: OWASP Dependency-Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: "my-project"
          path: "."
          format: "HTML"
          args: >
            --failOnCVSS 7
            --nvdApiKey ${{ secrets.NVD_API_KEY }}

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: dependency-check-report
          path: reports/

on:
  pull_request:
    branches: [main]
  workflow_dispatch:

    container-scan:
    name: Container scan (Trivy)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t app:scan .
      - uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: app:scan
          severity: HIGH,CRITICAL
          format: table
          exit-code: "0"        # report only for the baseline; change to "1" later
      - name: Trivy Dockerfile/config scan
        uses: aquasecurity/trivy-action@0.28.0
        with:
          scan-type: config
          scan-ref: .
          exit-code: "0"

  dast:
    name: Dynamic scan (OWASP ZAP baseline)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          docker build -t app:dast .
          docker run -d -p 8080:8080 --name app app:dast
          sleep 15
      - uses: zaproxy/action-baseline@v0.12.0
        with:
          target: http://localhost:8080
          fail_action: false    # report only for now

npm audit --omit=dev            # production deps first
npm audit --json > audit-before.json

npm install express@latest lodash@latest
# or stay within a major version if the latest breaks your app:
npm install express@^4 lodash@^4
npm audit fix
npm explain minimist

{
  "overrides": {
    "minimist": "^1.2.8",
    "semver": "^7.5.2"
  }
}
{
  "overrides": {
    "some-parent": { "semver": "^7.5.2" }
  }
}

rm -rf node_modules package-lock.json
npm install
npm ci && npm test
npm audit --omit=dev

pip install pip-audit
pip-audit -r requirements.txt
pip list --outdated

  pip-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install pip-audit && pip-audit -r requirements.txt

FROM node:20-alpine
WORKDIR /app
COPY --chown=node:node package*.json ./
RUN npm ci --omit=dev
COPY --chown=node:node . .
ENV NODE_ENV=production
USER node
EXPOSE 8080
CMD ["node", "server.js"]

FROM python:3.12-slim
RUN useradd --create-home --uid 10001 appuser
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=appuser:appuser . .
USER appuser
EXPOSE 8080
CMD ["python", "app.py"]

npm install helmet

const helmet = require("helmet");
app.use(helmet());          // sets X-Content-Type-Options: nosniff, HSTS, and more
app.disable("x-powered-by");

@app.after_request
def secure_headers(resp):
    resp.headers["X-Content-Type-Options"] = "nosniff"
    resp.headers["X-Frame-Options"] = "DENY"
    resp.headers["Referrer-Policy"] = "no-referrer"
    resp.headers["Content-Security-Policy"] = "default-src 'self'"
    return resp

add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header Referrer-Policy "no-referrer" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

docker build -t app:test . && docker run -d -p 8080:8080 app:test
curl -sI http://localhost:8080 | grep -i -E "x-content-type|x-frame|strict-transport"
docker exec <container> id        # should NOT show uid=0(root)

