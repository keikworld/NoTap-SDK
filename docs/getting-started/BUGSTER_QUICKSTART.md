# Bugster Quick Start Guide

Get started with Bugster E2E testing in 5 minutes!

## 1. Prerequisites

```bash
# Ensure you have Node.js 18+ and npm 9+
node --version  # Should be >= 18.0.0
npm --version   # Should be >= 9.0.0
```

## 2. Install Bugster CLI

```bash
npm install -g bugster-cli
```

## 3. Get Bugster API Key

1. Visit https://bugster.dev/
2. Sign up for a free account
3. Navigate to Dashboard → API Keys
4. Copy your API key

## 4. Initialize Bugster

```bash
cd /path/to/zero-pay-sdk
bugster init
```

When prompted:
- **API Key**: Paste your Bugster API key
- **Application URL**: Enter `http://localhost:3000` (backend) or `http://localhost:8080` (web)
- **Framework**: Select "Other" for Node.js backend

## 5. Start Backend Server

```bash
cd backend
npm install
npm run dev
```

Backend should start on `http://localhost:3000`

## 6. Run Your First Test

```bash
# From project root
bugster run --test bugster-tests/backend/enrollment-flow.yaml --verbose
```

You should see:
```
✅ Complete Enrollment Flow - 6 Factors
   ✓ POST /v1/enrollment - Enroll user with 6 factors
   ✓ GET /v1/enrollment/status/:uuid - Check enrollment status
   ✓ Validate PSD3 requirements
   ✓ Delete test user (GDPR compliance test)

✨ All tests passed! (4/4) in 2.5s
```

## 7. View Test Report

```bash
bugster report --open
```

Opens HTML report in browser with:
- Pass/fail summary
- Execution timeline
- Request/response details
- Screenshots (on failure)

## 8. Run All Backend Tests

```bash
bugster run --config bugster.config.yaml
```

Tests enrollment, verification, GDPR, ZK-SNARK, and admin endpoints.

## 9. Test Web UI (Optional)

Start web dev server:
```bash
# From project root
./gradlew :online-web:jsBrowserDevelopmentRun --no-daemon
```

Run web tests:
```bash
bugster run --config bugster.config.web.yaml
```

Tests all 10 factor canvases (PIN, Pattern, Emoji, etc.)

## 10. Try AI Destructive Testing

Let Bugster's AI find edge cases and vulnerabilities:

```bash
bugster agent --mode destructive --config bugster.config.yaml --duration 10m
```

The AI will:
- Try SQL injection on endpoints
- Attempt XSS in factor inputs
- Test rate limiting bypass
- Discover invalid input combinations
- Report security findings

## Next Steps

- **Read full guide**: `BUGSTER_INTEGRATION.md`
- **Write custom tests**: See `bugster-tests/README.md`
- **CI/CD integration**: GitHub Actions workflow already set up (`.github/workflows/bugster-tests.yml`)
- **Watch mode**: `bugster run --watch` (re-runs on code changes)

## Troubleshooting

**Issue**: `bugster: command not found`
```bash
npm list -g bugster-cli  # Verify installation
npm install -g bugster-cli  # Reinstall if needed
```

**Issue**: `Connection refused to http://localhost:3000`
```bash
cd backend && npm run dev  # Start backend server
```

**Issue**: Tests timing out
```yaml
# In bugster.config.yaml, increase timeout:
execution:
  timeout: 60000  # 60 seconds
```

## Support

- **Bugster Docs**: https://docs.bugster.dev/
- **NoTap Guide**: `BUGSTER_INTEGRATION.md`
- **Test Examples**: `bugster-tests/backend/` and `bugster-tests/web/`

---

Happy testing! 🚀
