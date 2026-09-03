## Day 49 – DevSecOps: Add Security to Your CI/CD Pipeline

#### What is DevSecOps?

the practice of embedding automated security checks and controls into every phase of the software delivery process

That's it. DevSecOps = adding security checks to the pipeline you already have. Not a separate process — just a few extra steps.

**Key Principles (Keep These in Mind)**

Catch problems early — A vulnerability found in a PR takes 5 minutes to fix. The same vulnerability found in production takes days.

Automate the checks — Don't rely on someone remembering to check. Let the pipeline do it every time.

Block on critical issues — If a scan finds a serious vulnerability, the pipeline should fail — just like a failing test.

Never put secrets in code — Use GitHub Secrets (you learned this on Day 44). No .env files, no hardcoded API keys.

Give only the access needed — Your workflow doesn't need write access to everything. Limit permissions.

### Task 1: Scan Your Docker Image for Vulnerabilities

```
steps:
      - name: code-checkout
        uses: actions/checkout@v6
      - name: docker login
        uses: docker/login-action@v4
        with:
          username: ${{ vars.docker_username }}
          password: ${{ secrets.docker_password }}
          
      - name: ${{ matrix.workdir }}-docker-build
        run: docker build -t ${{ vars.docker_username }}/devboard-${{ matrix.workdir }}:latest .
        working-directory: ${{ matrix.workdir }}

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@v0.36.0
        with:
          image-ref: '${{ vars.docker_username }}/devboard-${{ matrix.workdir }}:latest'
          format: 'table'
          exit-code: '0'
          ignore-unfixed: true
          trivyignores: '.trivyignore'
          vuln-type: 'os,library'
          severity: 'CRITICAL'
```

What this does:

trivy scans your Docker image for known CVEs (Common Vulnerabilities and Exposures)
format: 'table' prints a readable table in the logs
exit-code: '1' means fail the pipeline if CRITICAL vulnerabilities are found
If it passes, your image is clean — proceed to push and deploy


### Task 2: Enable GitHub's Built-in Secret Scanning

```
jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - name: code-checkout
        uses: actions/checkout@v7
        with:
          fetch-depth: 0
      - name: Gitleaks scanner
        uses: gitleaks/gitleaks-action@v3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Github can also automatically detect if someone pushes a secret (API key, token, password) to your repo.

Go to your repo → Settings → Code security and analysis
Enable Secret scanning
If available, also enable Push protection — this blocks the push entirely if a secret is detected

### Task 3: Scan Dependencies for Known Vulnerabilities

```jobs:
  Go-dependecy-check:
    runs-on: ubuntu-latest
    steps:
      - name: code
        uses: actions/checkout@v7
        with:
          persist-credentials: false
      
      - name: Run govulncheck and export report
        uses: golang/govulncheck-action@v1
        with:
          output-format: sarif
          output-file: govulncheck-results.sarif
          work-dir: ${{ inputs.backend_workdir }}

      - name: Upload results to artifacts
        uses: actions/upload-artifact@v4
        with:
          name: Go-vulnerabilities-report
          path: govulncheck-results.sarif
          retention-days: 7
          
  Node-dependency-check:
    runs-on: ubuntu-latest
    steps:
      - name: code-checkout
        uses: actions/checkout@v7
        
      - name: node-setup
        uses: actions/setup-node@v6
        with:
          node-version: 20
          cache-dependency-path: ${{ inputs.frontend_workdir }}
          
      - name: dependency check
        run: |
          npm audit > audit_report.txt || true
        working-directory: ${{ inputs.frontend_workdir }}
        
      - name: report-artificats
        uses: actions/upload-artifact@v4
        with:
          name: node-vulnerabilities-report
          path: ${{ inputs.frontend_workdir}}/audit_report.txt
          retention-days: 7
```
This checks any new dependencies added in the PR against a vulnerability database. If a dependency has a critical CVE, the PR check fails.


### Task 4: Add Permissions to Your Workflows

If a workflow needs to comment on PRs, add:
```
permissions:
  contents: write
```




