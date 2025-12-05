# Security Policy

## 🔒 Security at NoTap

Security is our top priority. NoTap is designed with security-first principles to protect user authentication data and prevent unauthorized access.

---

## 🐛 Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security issue, please report it responsibly.

### How to Report

**📧 Email:** security@notap.com

**Please Include:**
- Description of the vulnerability
- Steps to reproduce the issue
- Potential impact assessment
- Suggested fix (if available)
- Your contact information

### What to Expect

1. **Acknowledgment:** We'll acknowledge your report within **24 hours**
2. **Investigation:** We'll investigate and provide updates within **72 hours**
3. **Resolution:** We'll work on a fix and coordinate disclosure timeline
4. **Credit:** We'll credit you in the security advisory (unless you prefer to remain anonymous)

### Responsible Disclosure

Please give us reasonable time to fix vulnerabilities before public disclosure. We typically aim for:
- **Critical vulnerabilities:** 7-14 days
- **High severity:** 30 days
- **Medium/Low severity:** 60-90 days

---

## 🛡️ Security Features

### Cryptographic Standards

NoTap uses industry-standard cryptography:

- **Hashing:** SHA-256 for all authentication factor digests
- **Key Derivation:** PBKDF2 with 100,000 iterations
- **Encryption:** AES-256-GCM for data encryption
- **Transport:** TLS 1.3 for all network communication
- **Key Rotation:** Daily HKDF rotation for enhanced security

### Attack Resistance

NoTap is designed to resist common attacks:

- ✅ **Timing Attacks** - Constant-time comparison for all factors
- ✅ **Replay Attacks** - Nonce validation and session expiry
- ✅ **Brute Force** - Multi-layer rate limiting and account lockout
- ✅ **Man-in-the-Middle** - TLS 1.3 and certificate pinning
- ✅ **Memory Dumps** - Automatic memory wiping of sensitive data
- ✅ **Device Tampering** - Anti-root/jailbreak detection

### Privacy Protection

- ✅ **Zero-Knowledge Proofs** - Merchants never see which factors you used
- ✅ **No Biometric Storage** - Only cryptographic hashes stored
- ✅ **24-Hour TTL** - Authentication data auto-expires daily
- ✅ **GDPR Compliant** - Privacy by design, right to erasure

---

## 🔐 Security Best Practices

### For Developers Integrating NoTap

1. **API Key Security**
   - Never commit API keys to version control
   - Use environment variables or secure vaults
   - Rotate keys regularly (every 90 days recommended)
   - Use different keys for development/staging/production

2. **HTTPS Only**
   - Always use HTTPS for all API communication
   - Never send authentication data over HTTP
   - Implement certificate pinning for production apps

3. **Input Validation**
   - Validate all user inputs before sending to NoTap SDK
   - Sanitize data to prevent injection attacks
   - Implement proper error handling

4. **Secure Storage**
   - Use platform-specific secure storage (Keychain/KeyStore)
   - Never store authentication factors in plain text
   - Clear sensitive data from memory after use

### For End Users

1. **Device Security**
   - Use device lock screen (PIN/biometric)
   - Keep your device OS updated
   - Don't use NoTap on rooted/jailbroken devices

2. **Factor Selection**
   - Choose strong, unique authentication factors
   - Don't reuse PINs/patterns from other services
   - Enroll 6+ factors from multiple categories

3. **Account Monitoring**
   - Check your NoTap authentication history regularly
   - Report suspicious activity immediately
   - Revoke access for lost/stolen devices

---

## 🔍 Security Audit History

### November 2025 - Comprehensive Security Audit

**Status:** ✅ **Completed (100% Remediation)**

**Findings:**
- 26 vulnerabilities identified and fixed
- 15 critical issues (timing attacks, insecure random)
- 6 high severity (DoS, authentication bypass)
- 5 medium/low severity (code quality)

**Key Improvements:**
- ✅ All 12 authentication factors now use constant-time comparison
- ✅ Replaced `Math.random()` with `SecureRandom` (3 instances)
- ✅ Fixed deadlock risk in rate limiter
- ✅ Added input size limits to prevent DoS attacks
- ✅ Enhanced memory wiping for sensitive data

**Audit Report:** Available to enterprise customers upon request

---

## 📜 Compliance

### Standards & Regulations

NoTap complies with:

- **PSD3 SCA** - Strong Customer Authentication (EU Payment Directive)
- **GDPR** - General Data Protection Regulation
- **OWASP Top 10** - Web application security risks mitigated
- **NIST Cryptographic Standards** - FIPS 140-2 compliant algorithms

### Certifications

- ✅ **SOC 2 Type II** - In progress (Q1 2026)
- ✅ **ISO 27001** - Planned (Q2 2026)
- ✅ **PCI DSS** - Planned (for payment integrations)

---

## 🚨 Known Security Limitations

### Current Limitations

1. **Biometric Liveness Detection**
   - Status: Not yet implemented
   - Impact: Risk of photo/video replay attacks for face recognition
   - Mitigation: Use multiple factors, enable device biometric protections
   - Timeline: Planned for v1.2.0 (Q2 2026)

2. **Offline Verification**
   - Status: Not available
   - Impact: Requires network connectivity for authentication
   - Mitigation: All authentications are online-verified for security
   - Timeline: Planned for v1.2.0 (limited offline mode)

3. **Multi-Device Sync**
   - Status: Not available
   - Impact: Re-enrollment required if device is lost
   - Mitigation: Use Management Portal to re-enroll
   - Timeline: Planned for v2.0.0 (Q3 2026)

---

## 📚 Security Resources

### Documentation

- **[Security Analysis](docs/security-analysis.md)** - Detailed threat model
- **[Integration Guide](docs/integration-guide.md)** - Secure integration patterns
- **[API Reference](docs/api-reference.md)** - Security considerations per endpoint

### External Resources

- [OWASP Mobile Security](https://owasp.org/www-project-mobile-security/)
- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)
- [PSD3 Technical Standards](https://www.eba.europa.eu/regulation-and-policy/payment-services-and-electronic-money/regulatory-technical-standards-on-strong-customer-authentication-and-secure-communication-under-psd2)

---

## 🏆 Security Bounty Program

### Coming Soon (Q1 2026)

We're launching a bug bounty program to reward security researchers:

**Planned Rewards:**
- 🥇 **Critical:** $5,000 - $10,000
- 🥈 **High:** $1,000 - $5,000
- 🥉 **Medium:** $500 - $1,000
- 📝 **Low:** $100 - $500

**Scope:** Android SDK, iOS SDK, Web SDK, Backend API

**Out of Scope:** Third-party dependencies, social engineering, physical attacks

---

## 📞 Contact

- **Security Issues:** security@notap.com
- **General Security Questions:** [GitHub Discussions - Security](https://github.com/keikworld/NoTap-SDK/discussions/categories/security)
- **Emergency Security Contact:** +1-XXX-XXX-XXXX (Enterprise customers only)

---

## ✅ Security Commitment

We commit to:

1. **Transparency** - Publicly disclose security issues (after fixes)
2. **Rapid Response** - Acknowledge reports within 24 hours
3. **Regular Audits** - Annual third-party security audits
4. **Continuous Improvement** - Ongoing security enhancements
5. **Community Collaboration** - Work with security researchers

**Last Updated:** December 5, 2025

---

**Thank you for helping keep NoTap secure!** 🙏
