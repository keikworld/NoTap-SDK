# Building ZeroPay Locally

**Last Updated:** 2025-11-05
**Environment:** Local machine (NOT Claude Code Web)

---

## Why Build Locally?

Claude Code Web runs in a containerized environment with enterprise proxy restrictions that prevent Gradle from downloading Maven dependencies. **This is by design for security.**

**Recommended workflow:**
- 📝 **Claude Code Web**: Code editing, documentation, git operations
- 🔨 **Local Terminal**: All builds, tests, compilations

---

## Prerequisites

### 1. Java Development Kit (JDK)
```bash
# Check Java version
java -version

# Required: Java 17 or higher
# Recommended: Java 21 (OpenJDK)
```

**Install if needed:**
- **macOS**: `brew install openjdk@21`
- **Linux**: `sudo apt install openjdk-21-jdk`
- **Windows**: Download from [Adoptium](https://adoptium.net/)

### 2. Android SDK (for Android builds)
```bash
# Set ANDROID_HOME environment variable
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
```

### 3. Git
```bash
git --version
# Should be 2.x or higher
```

---

## Quick Start

### 1. Clone Repository
```bash
git clone https://github.com/keikworld/zero-pay-sdk.git
cd zero-pay-sdk
```

### 2. Checkout Your Branch
```bash
# List all branches
git branch -a

# Checkout the branch you were working on
git checkout claude/review-documentation-files-011CUpjpA62bqnLtdifDsSux

# Pull latest changes
git pull origin claude/review-documentation-files-011CUpjpA62bqnLtdifDsSux
```

### 3. Build All Modules
```bash
# Clean build (first time)
./gradlew clean build --console=plain

# Or build specific modules
./gradlew :sdk:build
./gradlew :enrollment:build
./gradlew :merchant:build
./gradlew :online-web:compileKotlinJs
```

---

## Module-Specific Builds

### SDK Module (Kotlin Multiplatform)
```bash
# Compile all targets
./gradlew :sdk:build

# Compile specific targets
./gradlew :sdk:compileKotlinMetadata        # Common code
./gradlew :sdk:compileDebugKotlinAndroid    # Android
./gradlew :sdk:compileKotlinJs              # JavaScript (Web)

# Run tests
./gradlew :sdk:test                         # All tests
./gradlew :sdk:testDebugUnitTest            # Android unit tests
```

### Enrollment Module (Android)
```bash
# Compile
./gradlew :enrollment:compileDebugKotlinAndroid

# Build AAR (Android library)
./gradlew :enrollment:assembleDebug

# Run tests
./gradlew :enrollment:testDebugUnitTest
```

### Merchant Module (Android)
```bash
# Compile
./gradlew :merchant:compileDebugKotlinAndroid

# Build AAR
./gradlew :merchant:assembleDebug

# Run tests
./gradlew :merchant:testDebugUnitTest
```

### Online-Web Module (JavaScript/WASM)
```bash
# Compile to JavaScript
./gradlew :online-web:compileKotlinJs

# Run development server with hot reload
./gradlew :online-web:jsBrowserDevelopmentRun
# Open http://localhost:8080 in browser

# Production build
./gradlew :online-web:jsBrowserProductionWebpack

# Output location: online-web/build/dist/js/productionExecutable/
```

---

## Common Build Tasks

### Clean Build (remove all build artifacts)
```bash
./gradlew clean
```

### Build Everything
```bash
./gradlew build
```

### Run All Tests
```bash
./gradlew test
```

### Check Code Style
```bash
./gradlew ktlintCheck
```

### Generate Documentation (Dokka)
```bash
./gradlew dokkaHtml
# Output: build/dokka/html/index.html
```

---

## Troubleshooting

### Issue: "SDK location not found"
```bash
# Create local.properties with Android SDK path
echo "sdk.dir=/path/to/Android/Sdk" > local.properties

# Example paths:
# macOS: /Users/username/Library/Android/sdk
# Linux: /home/username/Android/Sdk
# Windows: C:\\Users\\username\\AppData\\Local\\Android\\Sdk
```

### Issue: "Java version mismatch"
```bash
# Check Java version
java -version

# Set JAVA_HOME to correct version
export JAVA_HOME=/path/to/jdk-21

# Verify
./gradlew --version
```

### Issue: "Gradle daemon issues"
```bash
# Stop all Gradle daemons
./gradlew --stop

# Run without daemon
./gradlew build --no-daemon
```

### Issue: "Out of memory"
```bash
# Edit gradle.properties
org.gradle.jvmargs=-Xmx8192m -XX:+HeapDumpOnOutOfMemoryError

# Or run with more memory
./gradlew build -Dorg.gradle.jvmargs=-Xmx8192m
```

### Issue: "Dependencies won't download"
```bash
# Check internet connection
curl -I https://dl.google.com

# Clear Gradle cache
rm -rf ~/.gradle/caches/

# Retry with --refresh-dependencies
./gradlew build --refresh-dependencies
```

---

## Performance Tips

### 1. Enable Gradle Daemon (faster builds)
```bash
# In gradle.properties
org.gradle.daemon=true
```

### 2. Parallel Builds
```bash
# In gradle.properties (already enabled)
org.gradle.parallel=true
```

###  3. Build Cache
```bash
# In gradle.properties (already enabled)
org.gradle.caching=true
```

### 4. Configuration on Demand
```bash
# In gradle.properties (already enabled)
org.gradle.configureondemand=true
```

### 5. Incremental Compilation
```bash
# In gradle.properties (already enabled)
kotlin.incremental=true
```

---

## Build Verification

After building, verify everything compiled successfully:

```bash
# Check SDK artifacts
ls -lh sdk/build/libs/

# Check enrollment AAR
ls -lh enrollment/build/outputs/aar/

# Check merchant AAR
ls -lh merchant/build/outputs/aar/

# Check web JavaScript
ls -lh online-web/build/dist/js/productionExecutable/
```

---

## CI/CD Integration

The repository includes GitHub Actions workflows:

```bash
# Check workflow status
cat .github/workflows/ci-cd.yml

# Workflows run automatically on:
# - Push to main/master
# - Pull requests
# - Manual dispatch
```

---

## Development Workflow

### Typical Daily Workflow

1. **Morning - Sync with remote**
   ```bash
   git pull origin your-branch
   ```

2. **Code - Make changes in Claude Code Web or local IDE**
   - Edit files
   - Write tests
   - Update documentation

3. **Build - Test locally**
   ```bash
   ./gradlew :sdk:test
   ./gradlew :enrollment:test
   ./gradlew :merchant:test
   ```

4. **Commit - Push changes**
   ```bash
   git add .
   git commit -m "Description of changes"
   git push origin your-branch
   ```

5. **Verify - Check CI/CD**
   - GitHub Actions will automatically build and test
   - Check for green checkmarks

---

## Next Steps

After successful local build:

1. ✅ Verify all tests pass
2. ✅ Check Gradle output for warnings
3. ✅ Test Android app on emulator/device
4. ✅ Test web app in browser (http://localhost:8080)
5. ✅ Commit and push changes

---

## Support

**Build Issues:**
- Check [Gradle documentation](https://docs.gradle.org/)
- Check [Kotlin Multiplatform documentation](https://kotlinlang.org/docs/multiplatform.html)
- See troubleshooting section above

**Code Issues:**
- Refer to CLAUDE.md for development guidelines
- Check documentation/planning.md for roadmap
- Review documentation/task.md for current tasks

---

**Happy Building! 🚀**
