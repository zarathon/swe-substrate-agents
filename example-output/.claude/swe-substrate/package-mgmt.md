# Gerenciamento de Pacotes e Dependências

**Gerado por**: `/swe-substrate-package-mgmt`
**Data**: 2025-10-29

## Package Manager

**Tool**: npm (Node Package Manager)
**Version**: 9.x
**Lockfile**: package-lock.json (committed to git)

### package.json Structure

```json
{
  "name": "shopflow-api",
  "version": "1.2.3",
  "description": "ShopFlow E-commerce API",
  "main": "dist/index.js",
  "engines": {
    "node": ">=20.0.0",
    "npm": ">=9.0.0"
  },
  "scripts": {
    "dev": "nodemon src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "lint": "eslint 'src/**/*.ts'",
    "migrate": "node-pg-migrate"
  },
  "dependencies": {
    "express": "4.18.2",
    "pg": "8.11.3",
    "redis": "4.6.10",
    "@anthropic-ai/sdk": "0.20.0"
  },
  "devDependencies": {
    "typescript": "^5.2.2",
    "jest": "^29.7.0",
    "@types/node": "^20.8.0"
  }
}
```

## Dependency Management Strategy

### Version Pinning

```json
{
  "dependencies": {
    // Exact versions for critical runtime dependencies
    "express": "4.18.2",
    "pg": "8.11.3",

    // Caret (^) for less critical dependencies
    "winston": "^3.11.0",

    // Tilde (~) for patch updates only
    "helmet": "~7.0.0"
  },
  "devDependencies": {
    // More flexible for dev dependencies
    "typescript": "^5.2.2",
    "eslint": "^8.50.0"
  }
}
```

### Dependency Types

- **dependencies**: Production runtime dependencies
- **devDependencies**: Development and testing tools
- **peerDependencies**: Required by consumers (for libraries)
- **optionalDependencies**: Non-critical dependencies

## Automated Dependency Updates

### Dependabot Configuration

**.github/dependabot.yml**

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
    open-pull-requests-limit: 5
    reviewers:
      - "engineering-team"
    assignees:
      - "tech-lead"
    commit-message:
      prefix: "chore(deps)"
    groups:
      # Group minor and patch updates
      development-dependencies:
        dependency-type: "development"
        update-types:
          - "minor"
          - "patch"

      # Group security updates separately
      security-updates:
        update-types:
          - "security"

    # Ignore major version updates for critical packages
    ignore:
      - dependency-name: "express"
        update-types: ["version-update:semver-major"]
      - dependency-name: "pg"
        update-types: ["version-update:semver-major"]
```

### Renovate Alternative

**renovate.json**

```json
{
  "extends": ["config:base"],
  "schedule": ["before 10am on monday"],
  "packageRules": [
    {
      "matchDepTypes": ["devDependencies"],
      "automerge": true,
      "automergeType": "pr",
      "automergeStrategy": "squash"
    },
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true
    },
    {
      "matchPackagePatterns": ["^@types/"],
      "automerge": true
    }
  ],
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "assignees": ["@security-team"]
  }
}
```

## Security Scanning

### npm audit

```bash
# Check for vulnerabilities
npm audit

# Fix automatically (careful with major versions)
npm audit fix

# Only fix production dependencies
npm audit fix --production

# Force fix (may break things)
npm audit fix --force
```

### Snyk Integration

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: '0 0 * * 0' # Weekly

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Monitor dependencies
        run: npx snyk monitor
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### OWASP Dependency Check

```bash
# Install
npm install -g dependency-check

# Run check
dependency-check --project "ShopFlow API" --scan .
```

## Monorepo Management (Optional)

### Nx Configuration

If using monorepo structure:

```json
// nx.json
{
  "npmScope": "shopflow",
  "affected": {
    "defaultBase": "main"
  },
  "workspaceLayout": {
    "appsDir": "apps",
    "libsDir": "libs"
  }
}
```

### Workspace Structure

```
shopflow-monorepo/
├── apps/
│   ├── api/
│   ├── web/
│   └── mobile/
├── libs/
│   ├── shared/
│   ├── models/
│   └── utils/
└── package.json
```

## License Compliance

### license-checker

```bash
# Install
npm install -g license-checker

# Check licenses
license-checker --production --json > licenses.json

# Exclude certain licenses
license-checker --exclude "GPL,AGPL"
```

### Allowed Licenses

- MIT
- Apache-2.0
- BSD-3-Clause
- ISC
- CC0-1.0

### Prohibited Licenses

- GPL (any version)
- AGPL
- Commercial licenses without approval

## Bundle Analysis

### webpack-bundle-analyzer

```typescript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
    }),
  ],
};
```

### npm-check-updates

```bash
# Check for outdated packages
npx npm-check-updates

# Interactive update
npx npm-check-updates --interactive

# Update to latest (respecting version constraints)
npx npm-check-updates -u
npm install
```

## Dependency Review Process

### Before Adding a Dependency

1. **Evaluate necessity**: Can we implement it ourselves?
2. **Check popularity**: npm downloads, GitHub stars
3. **Maintenance status**: Last commit, open issues
4. **License compatibility**: Check license
5. **Bundle size**: Use bundlephobia.com
6. **Security**: Check Snyk/npm audit
7. **Alternatives**: Compare similar packages

### Dependency Review Checklist

```markdown
## New Dependency: [package-name]

- [ ] Package is actively maintained (commit in last 3 months)
- [ ] No critical security vulnerabilities
- [ ] License is compatible (MIT, Apache-2.0, BSD)
- [ ] Bundle size is acceptable (<50KB gzipped)
- [ ] Has good documentation
- [ ] Has tests and CI
- [ ] Weekly downloads >10,000
- [ ] Alternatives evaluated
- [ ] Team approval obtained
```

## Package Scripts Best Practices

### Organized Scripts

```json
{
  "scripts": {
    // Development
    "dev": "nodemon src/index.ts",
    "dev:debug": "nodemon --inspect src/index.ts",

    // Building
    "build": "tsc",
    "build:clean": "rm -rf dist && npm run build",
    "build:watch": "tsc --watch",

    // Testing
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "playwright test",

    // Linting & Formatting
    "lint": "eslint 'src/**/*.ts'",
    "lint:fix": "eslint 'src/**/*.ts' --fix",
    "format": "prettier --write 'src/**/*.ts'",
    "format:check": "prettier --check 'src/**/*.ts'",
    "type-check": "tsc --noEmit",

    // Database
    "migrate": "node-pg-migrate",
    "migrate:up": "node-pg-migrate up",
    "migrate:down": "node-pg-migrate down",
    "migrate:create": "node-pg-migrate create",
    "db:seed": "node scripts/seed.js",

    // Docker
    "docker:up": "docker-compose up -d",
    "docker:down": "docker-compose down",
    "docker:logs": "docker-compose logs -f",

    // Utilities
    "clean": "rm -rf dist coverage",
    "prepare": "husky install",
    "precommit": "lint-staged",
    "prepush": "npm test"
  }
}
```

## Private Package Registry

### Using npm Private Registry

```bash
# Login to registry
npm login --registry=https://registry.company.com

# Publish private package
npm publish --registry=https://registry.company.com
```

**.npmrc**

```
@shopflow:registry=https://registry.company.com
//registry.company.com/:_authToken=${NPM_TOKEN}
```

### GitHub Packages

```yaml
# .github/workflows/publish.yml
name: Publish Package

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          registry-url: 'https://npm.pkg.github.com'
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Dependency Visualization

### npm ls

```bash
# Show dependency tree
npm ls

# Show only production dependencies
npm ls --production

# Show specific package
npm ls express

# Show depth
npm ls --depth=1
```

### depcheck

```bash
# Install
npm install -g depcheck

# Find unused dependencies
depcheck

# Find missing dependencies
depcheck --specials=bin,eslint
```

## Performance Optimization

### Reduce node_modules Size

```bash
# Remove devDependencies in production
npm ci --production

# Use npm clean-install for CI
npm ci

# Prune unused packages
npm prune
```

### Cache Dependencies in CI

```yaml
# GitHub Actions
- uses: actions/setup-node@v3
  with:
    node-version: '20'
    cache: 'npm'

- run: npm ci
```

## Documentation

### Dependency Documentation

**docs/dependencies.md**

```markdown
# Key Dependencies

## Production Dependencies

### express (4.18.2)
- Purpose: Web framework for REST API
- Critical: Yes
- Update policy: Security patches only
- Alternatives considered: Fastify, Koa

### pg (8.11.3)
- Purpose: PostgreSQL client
- Critical: Yes
- Update policy: Minor versions after testing
- Documentation: https://node-postgres.com

## Development Dependencies

### jest (^29.7.0)
- Purpose: Testing framework
- Update policy: Latest minor version
```

## Deprecation Policy

### Deprecating Internal Packages

```typescript
/**
 * @deprecated Since v2.0.0. Use `newFunction()` instead.
 * Will be removed in v3.0.0.
 */
export function oldFunction() {
  console.warn('oldFunction is deprecated. Use newFunction instead.');
  return newFunction();
}
```

### Monitoring Deprecated Dependencies

```bash
# Check for deprecated packages
npm deprecate

# Find packages using deprecated dependencies
npm outdated
```
