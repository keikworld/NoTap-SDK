# Bugster Integration Guide for NoTap Zero-Pay SDK

## Overview

This guide documents the integration of **Bugster** - an AI-powered E2E testing CLI - into the NoTap Zero-Pay SDK project for automated browser-based testing.

**Bugster Features:**
- AI-powered test generation from codebase analysis
- Browser automation simulating real user interactions
- Self-healing test locators
- CI/CD integration for automated PR validation
- Destructive agent testing for edge case discovery

---

## What Bugster Tests in This Project

### 1. Backend API (Node.js)
- **16 API Routers** (`/v1/enrollment`, `/v1/verification`, `/v1/wallet`, etc.)
- **Authentication Flows** (enrollment, verification, wallet signatures)
- **Blockchain Integration** (SNS/DID, Solana smart contracts)
- **GDPR Endpoints** (consent, data deletion, export)
- **Admin & Merchant Dashboards**

### 2. Online-Web Interface (Kotlin/JS)
- **10 Factor Canvases** (PIN, Pattern, Emoji, Color, ImageTap, MouseDraw, RhythmTap, StylusDraw, Voice, Words)
- **Enrollment Wizard** (5-step flow with 14 factors)
- **Verification Flow** (factor input, validation, submission)
- **UI Components** (buttons, forms, validation messages)

### 3. Integration Flows
- **E2E Authentication** (complete enrollment → verification cycle)
- **PSP Payment Flow** (session-based verification for payments)
- **SSO Federation** (6 OAuth providers)

---

## Installation

### Prerequisites
- Node.js >= 18.0.0
- npm >= 9.0.0
- Backend server running (port 3000)
- Online-web dev server running (Kotlin/JS)

### Install Bugster CLI

```bash
# Install via official script (Linux/macOS)
curl -sSL https://github.com/Bugsterapp/bugster-cli/releases/latest/download/install.sh | bash -s -- -y

# Add to PATH (if not automatic)
export PATH="$HOME/.bugster/bin:$PATH"

# Verify installation
bugster --version
```

> **Note:** Bugster CLI is NOT on npm. It must be installed via the official GitHub releases script.

---

## Project Setup

### 1. Initialize Bugster in Project Root

```bash
cd /home/user/zero-pay-sdk
bugster init
```

This will prompt for:
- **API Key** (from Bugster dashboard)
- **Application URL** (e.g., `http://localhost:3000` for backend, `http://localhost:8080` for web)
- **Framework** (Select "Other" for Node.js backend, "React/Vue/Other" for Kotlin/JS web)

### 2. Configuration Files

Bugster will create `bugster.config.yaml` (or `config.yaml`) in the project root:

```yaml
# bugster.config.yaml (Backend API Testing)
name: "zeropay-backend-api"
baseUrl: "http://localhost:3000"
framework: "node"
testDir: "./bugster-tests/backend"
parallelRuns: 3

# Authentication (if needed for protected endpoints)
auth:
  type: "bearer"
  tokenEndpoint: "/v1/admin/auth"
  credentials:
    apiKey: "${ADMIN_API_KEY}"

# Test execution settings
execution:
  timeout: 30000
  retries: 2
  headless: true

# CI/CD integration
ci:
  enabled: true
  reportFormat: "junit"
  failOnError: true
```

### 3. Separate Configuration for Web Testing

Create `bugster.config.web.yaml` for online-web:

```yaml
# bugster.config.web.yaml (Web Interface Testing)
name: "zeropay-online-web"
baseUrl: "http://localhost:8080"
framework: "other"
testDir: "./bugster-tests/web"
parallelRuns: 2

# Browser settings
browser:
  type: "chromium"
  viewport:
    width: 1280
    height: 720

# File upload testing (for Voice factor)
fileUpload:
  enabled: true
  directory: "./bugster-tests/fixtures"

execution:
  timeout: 60000  # Longer for interactive factor canvases
  retries: 1
  headless: false  # Visual testing recommended for UI
```

---

## Test Generation

### Generate Backend API Tests

```bash
# Let Bugster AI analyze your backend and generate tests
bugster generate --config bugster.config.yaml --scope backend

# Generates tests for:
# - POST /v1/enrollment (enrollment flow)
# - POST /v1/verification (verification flow)
# - GET /v1/wallet/verification-key
# - POST /v1/zkproof/generate
# - GET /v1/admin/audit-trail
# - etc.
```

### Generate Web UI Tests

```bash
# Generate tests for online-web factor canvases
bugster generate --config bugster.config.web.yaml --scope web

# Generates tests for:
# - PIN canvas (input validation, show/hide toggle)
# - Pattern canvas (9-point pattern drawing)
# - Emoji canvas (4-emoji sequence selection)
# - MouseDraw canvas (signature drawing)
# - Voice canvas (audio recording)
# - etc.
```

---

## Writing Custom Test Specifications

While Bugster auto-generates tests, you can also write manual YAML specs:

### Example: Backend Enrollment Flow Test

Create `bugster-tests/backend/enrollment-flow.yaml`:

```yaml
name: "Complete Enrollment Flow"
description: "Test enrolling a user with 6 factors (PIN, Pattern, Emoji, Color, Face, Fingerprint)"

steps:
  - action: "POST"
    endpoint: "/v1/enrollment"
    headers:
      Content-Type: "application/json"
    body:
      uuid: "test-user-{{timestamp}}"
      factors:
        - type: "PIN"
          digest: "{{hashed_pin}}"
        - type: "PATTERN"
          digest: "{{hashed_pattern}}"
        - type: "EMOJI"
          digest: "{{hashed_emoji}}"
        - type: "COLOUR"
          digest: "{{hashed_color}}"
        - type: "FACE"
          digest: "{{hashed_face}}"
        - type: "FINGERPRINT"
          digest: "{{hashed_fingerprint}}"
    expect:
      status: 200
      body:
        success: true
        message: "Enrollment successful"

  - action: "GET"
    endpoint: "/v1/enrollment/status/{{uuid}}"
    expect:
      status: 200
      body:
        enrolled: true
        factorCount: 6
```

### Example: Web UI PIN Canvas Test

Create `bugster-tests/web/pin-canvas.yaml`:

```yaml
name: "PIN Canvas Interaction"
description: "Test PIN factor canvas input, validation, and submission"

steps:
  - action: "navigate"
    url: "/factors/pin"

  - action: "wait"
    selector: "#pin-canvas"

  - action: "click"
    selector: "button[data-digit='1']"

  - action: "click"
    selector: "button[data-digit='2']"

  - action: "click"
    selector: "button[data-digit='3']"

  - action: "click"
    selector: "button[data-digit='4']"

  - action: "assert"
    selector: ".pin-indicator"
    condition: "hasClass"
    value: "filled"

  - action: "click"
    selector: "#toggle-visibility"

  - action: "assert"
    selector: "#pin-display"
    condition: "visible"

  - action: "click"
    selector: "#submit-pin"

  - action: "waitForNavigation"
    timeout: 5000

  - action: "assert"
    url: "/factors/next"
```

---

## Running Tests

### Local Development

```bash
# Run all backend tests
bugster run --config bugster.config.yaml

# Run all web tests
bugster run --config bugster.config.web.yaml

# Run specific test file
bugster run --test bugster-tests/backend/enrollment-flow.yaml

# Run in watch mode (re-run on changes)
bugster run --watch

# Run with verbose output
bugster run --verbose
```

### Destructive Testing (AI Edge Case Discovery)

```bash
# Let Bugster AI find edge cases and vulnerabilities
bugster agent --mode destructive --config bugster.config.yaml

# Example discoveries:
# - SQL injection attempts on enrollment UUID
# - XSS attempts on factor input fields
# - Rate limiting bypass attempts
# - Invalid factor combinations (< 6 factors)
# - Replay attack simulation
```

---

## CI/CD Integration

### GitHub Actions Workflow

Create `.github/workflows/bugster-tests.yml`:

```yaml
name: Bugster E2E Tests

on:
  pull_request:
    branches: [master, main]
  push:
    branches: [master, main]

jobs:
  backend-tests:
    runs-on: ubuntu-latest

    services:
      redis:
        image: redis:7-alpine
        ports:
          - 6380:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: zeropay_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install backend dependencies
        run: cd backend && npm install

      - name: Start backend server
        run: cd backend && npm run dev &
        env:
          NODE_ENV: test
          REDIS_HOST: localhost
          REDIS_PORT: 6380
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/zeropay_test
          ADMIN_API_KEY: ${{ secrets.ADMIN_API_KEY }}

      - name: Wait for server to be ready
        run: npx wait-on http://localhost:3000/health --timeout 30000

      - name: Install Bugster CLI
        run: npm install -g bugster-cli

      - name: Run Bugster backend tests
        run: bugster run --config bugster.config.yaml
        env:
          BUGSTER_API_KEY: ${{ secrets.BUGSTER_API_KEY }}

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: bugster-backend-results
          path: ./bugster-reports/

  web-tests:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Build online-web
        run: ./gradlew :online-web:jsBrowserDevelopmentRun --no-daemon &

      - name: Wait for web server
        run: npx wait-on http://localhost:8080 --timeout 60000

      - name: Install Bugster CLI
        run: npm install -g bugster-cli

      - name: Run Bugster web tests
        run: bugster run --config bugster.config.web.yaml
        env:
          BUGSTER_API_KEY: ${{ secrets.BUGSTER_API_KEY }}

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: bugster-web-results
          path: ./bugster-reports/
```

### Required Secrets

Add to GitHub repository secrets:
- `BUGSTER_API_KEY` - Your Bugster API key
- `ADMIN_API_KEY` - NoTap admin API key for authenticated endpoints

---

## Test Maintenance

### Update Tests After Code Changes

```bash
# Re-analyze codebase and update existing tests
bugster update --config bugster.config.yaml

# Regenerate tests from scratch
bugster generate --force
```

### Self-Healing Locators

Bugster automatically updates CSS selectors and XPath locators when the UI structure changes. No manual intervention needed!

---

## Integration with Existing Tests

Bugster complements your existing Mocha/Chai tests:

| Test Type | Tool | Focus |
|-----------|------|-------|
| **Unit Tests** | Mocha/Chai | Individual functions, crypto, processors |
| **Integration Tests** | Mocha/Chai + Supertest | API endpoints, database operations |
| **E2E Browser Tests** | **Bugster** | Real user flows, UI interactions |
| **Destructive Tests** | **Bugster Agent** | Edge cases, security vulnerabilities |

### Recommended Workflow

1. **Write unit tests** (Mocha) for new factor processors
2. **Write integration tests** (Mocha + Supertest) for new API endpoints
3. **Generate E2E tests** (Bugster) for complete user flows
4. **Run destructive tests** (Bugster Agent) before major releases

---

## Best Practices

### 1. Environment Isolation

```bash
# Use separate Bugster configs for dev/staging/production
bugster run --config bugster.config.dev.yaml
bugster run --config bugster.config.staging.yaml
```

### 2. Test Data Management

Create `bugster-tests/fixtures/` for test data:
```
bugster-tests/
  fixtures/
    audio/
      voice-sample.wav
    images/
      face-photo.jpg
    data/
      test-users.json
```

### 3. Parallel Execution

```yaml
# In bugster.config.yaml
parallelRuns: 5  # Run 5 tests concurrently for faster feedback
```

### 4. Failure Debugging

```bash
# Run failed tests with screenshots
bugster run --failed-only --screenshots

# Interactive debugging mode
bugster shell  # Opens REPL with autocomplete
```

---

## Troubleshooting

### Tests Failing Locally But Passing in CI

- Check browser versions (use same Chromium version)
- Ensure test environment variables match
- Verify server startup timing (add longer `wait-on` timeout)

### "Element not found" Errors

- Use Bugster's self-healing locators
- Add explicit waits: `waitFor: 5000` in test steps
- Check if element is in iframe or shadow DOM

### Slow Test Execution

- Enable headless mode: `headless: true`
- Increase parallel runs: `parallelRuns: 5`
- Use test sharding for large test suites

---

## Metrics & Reporting

Bugster generates test reports in `./bugster-reports/`:

- `index.html` - Visual test results dashboard
- `junit.xml` - JUnit format for CI/CD integration
- `screenshots/` - Failure screenshots
- `videos/` - Test execution recordings (if enabled)

### Viewing Reports

```bash
# Open HTML report in browser
open bugster-reports/index.html

# Or use Bugster CLI
bugster report --open
```

---

## Cost Considerations

Bugster is a commercial tool. Pricing (as of 2025):
- **Free tier**: 100 test runs/month
- **Startup plan**: $99/month (1,000 runs)
- **Growth plan**: $299/month (5,000 runs)
- **Enterprise**: Custom pricing

For this project with ~50 tests, the free tier should suffice for initial integration.

---

## Next Steps

1. **Install Bugster CLI** globally
2. **Run `bugster init`** in project root
3. **Generate initial test suite** for backend API
4. **Review and customize** generated tests
5. **Add GitHub Actions workflow** for CI/CD
6. **Document custom test patterns** in this guide

---

## References

- **Bugster Docs**: https://docs.bugster.dev/
- **NoTap Architecture**: `documentation/ARCHITECTURE.md`
- **Existing Tests**: `backend/tests/` (Mocha/Chai)
- **Planning Roadmap**: `documentation/planning.md`

---

**Last Updated**: 2025-11-28
**Author**: Claude Code
**Status**: Integration guide complete, awaiting implementation
