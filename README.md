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
