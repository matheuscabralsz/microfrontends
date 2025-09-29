# Step 0: Project Foundation Setup - Detailed Action Plan

## Prerequisites Checklist
```bash
# Verify installations in WSL2
node --version  # Should be 22+
npm --version   # Should be 10+
git --version   # Should be installed
```

---

## 🚀 Step 0.1: Initialize Nx Monorepo

### Actions:
```bash
# Create project directory and initialize Nx
cd ~
npx create-nx-workspace@latest todo-microfrontends --cli=nx --preset=apps --packageManager=npm

# When prompted:
# ? Enable distributed caching? → No (for learning, we'll keep it simple)
# ? Help improve Nx → Your choice
```

### Validation:
```bash
cd todo-microfrontends
nx --version
# ✅ Should display Nx version
ls -la
# ✅ Should show nx.json, package.json, and workspace structure
```

---

## 🚀 Step 0.2: Configure Git Repository

### Actions:
```bash
# Initialize git (if not already done by Nx)
git init

# Create .gitignore with comprehensive rules
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
.pnp
.pnp.js

# Testing
coverage/
*.lcov
.nyc_output

# Production builds
dist/
build/
*.local

# Misc
.DS_Store
*.pem
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
.nx/cache

# IDE
.idea/
.vscode/
*.swp
*.swo
*~
.project
.classpath
.c9/
*.launch
.settings/
*.sublime-workspace
*.iml

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# OS
Thumbs.db

# Nx
.nx/
EOF

# Initial commit
git add .
git commit -m "Initial Nx workspace setup"
```

### Validation:
```bash
git status
# ✅ Should show clean working directory
git log --oneline
# ✅ Should show initial commit
```

---

## 🚀 Step 0.3: Setup TypeScript Configuration

### Actions:
```bash
# Update root tsconfig.base.json
cat > tsconfig.base.json << 'EOF'
{
  "compileOnSave": false,
  "compilerOptions": {
    "rootDir": ".",
    "sourceMap": true,
    "declaration": false,
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "importHelpers": true,
    "target": "ES2022",
    "module": "esnext",
    "lib": ["ES2022", "dom"],
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "@todo-mfe/shared": ["packages/shared/src/index.ts"],
      "@todo-mfe/shared/*": ["packages/shared/src/*"]
    },
    "resolveJsonModule": true,
    "allowSyntheticDefaultImports": true,
    "types": ["node"]
  },
  "exclude": ["node_modules", "tmp", "dist"]
}
EOF

# Install TypeScript and type definitions
npm install --save-dev typescript @types/node@20
```

### Validation:
```bash
npx tsc --version
# ✅ Should show TypeScript version 5.x
npx tsc --noEmit
# ✅ Should complete without errors
```

---

## 🚀 Step 0.4: Setup ESLint with Airbnb Configuration

### Actions:
```bash
# Install ESLint and Airbnb config for TypeScript
npm install --save-dev \
  eslint \
  @typescript-eslint/parser \
  @typescript-eslint/eslint-plugin \
  eslint-config-airbnb-base \
  eslint-config-airbnb-typescript \
  eslint-plugin-import \
  @nx/eslint-plugin \
  eslint-plugin-react \
  eslint-plugin-react-hooks \
  eslint-plugin-jsx-a11y

# Create root .eslintrc.json
cat > .eslintrc.json << 'EOF'
{
  "root": true,
  "ignorePatterns": ["**/*"],
  "plugins": ["@nx"],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module",
    "project": "./tsconfig.base.json"
  },
  "overrides": [
    {
      "files": ["*.ts", "*.tsx"],
      "extends": [
        "airbnb-base",
        "airbnb-typescript/base",
        "plugin:@typescript-eslint/recommended",
        "plugin:@nx/typescript"
      ],
      "rules": {
        "import/prefer-default-export": "off",
        "class-methods-use-this": "off",
        "no-console": ["warn", { "allow": ["warn", "error"] }],
        "@typescript-eslint/no-explicit-any": "error",
        "max-len": ["error", { "code": 120, "ignoreComments": true }]
      }
    },
    {
      "files": ["*.tsx"],
      "extends": [
        "plugin:react/recommended",
        "plugin:react-hooks/recommended",
        "plugin:jsx-a11y/recommended"
      ],
      "rules": {
        "react/react-in-jsx-scope": "off",
        "react/prop-types": "off"
      }
    },
    {
      "files": ["*.js"],
      "extends": ["eslint:recommended", "plugin:@nx/javascript"],
      "env": {
        "node": true
      }
    }
  ]
}
EOF

# Add lint script to package.json
npm pkg set scripts.lint="nx run-many --target=lint --all"
```

### Validation:
```bash
npm run lint
# ✅ Should run without errors (no projects to lint yet)
npx eslint --version
# ✅ Should show ESLint version
```

---

## 🚀 Step 0.5: Setup Prettier

### Actions:
```bash
# Install Prettier with ESLint integration
npm install --save-dev \
  prettier \
  eslint-config-prettier \
  eslint-plugin-prettier

# Create .prettierrc
cat > .prettierrc << 'EOF'
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 120,
  "tabWidth": 2,
  "semi": true,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
EOF

# Create .prettierignore
cat > .prettierignore << 'EOF'
# Dependencies
node_modules

# Production
dist
build
coverage
.nx

# Misc
*.md
*.json
EOF

# Update .eslintrc.json to include prettier (add to extends array in TypeScript overrides)
# Add "prettier" to the extends array in the TypeScript files section

# Add format script
npm pkg set scripts.format="prettier --write ."
npm pkg set scripts.format:check="prettier --check ."
```

### Validation:
```bash
npm run format:check
# ✅ Should check files successfully
echo "const test='unformatted'" > test.ts
npx prettier --check test.ts
# ✅ Should report formatting issues
rm test.ts
```

---

## 🚀 Step 0.6: Configure Testing Infrastructure

### Actions:
```bash
# Install Jest and testing utilities
npm install --save-dev \
  jest \
  @types/jest \
  ts-jest \
  @testing-library/react \
  @testing-library/jest-dom \
  @testing-library/user-event \
  @testing-library/dom

# Create root jest.preset.js
cat > jest.preset.js << 'EOF'
const nxPreset = require('@nx/jest/preset').default;

module.exports = {
  ...nxPreset,
  coverageReporters: ['text', 'html', 'lcov'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
    '!src/**/*.stories.tsx',
  ],
  testEnvironment: 'node',
  transform: {
    '^.+\\.(ts|tsx)$': ['ts-jest', {
      tsconfig: {
        jsx: 'react-jsx',
      },
    }],
  },
};
EOF

# Add test scripts
npm pkg set scripts.test="nx run-many --target=test --all"
npm pkg set scripts.test:coverage="nx run-many --target=test --all --coverage"
```

### Validation:
```bash
npm run test
# ✅ Should complete (no tests yet, but infrastructure works)
```

---

## 🚀 Step 0.7: Create Workspace Structure

### Actions:
```bash
# Create packages directory structure
mkdir -p packages/shell
mkdir -p packages/task-manager
mkdir -p packages/analytics
mkdir -p packages/user-profile
mkdir -p packages/shared
mkdir -p tools/scripts
mkdir -p docs

# Create placeholder README files
echo "# Shell Container" > packages/shell/README.md
echo "# Task Manager Microfrontend" > packages/task-manager/README.md
echo "# Analytics Dashboard Microfrontend" > packages/analytics/README.md
echo "# User Profile Microfrontend" > packages/user-profile/README.md
echo "# Shared Utilities" > packages/shared/README.md

# Update nx.json to recognize our structure
cat > nx.json << 'EOF'
{
  "$schema": "./node_modules/nx/schemas/nx-schema.json",
  "tasksRunnerOptions": {
    "default": {
      "runner": "@nx/workspace/tasks-runners/default",
      "options": {
        "cacheableOperations": ["build", "lint", "test"]
      }
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    },
    "test": {
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"]
    },
    "lint": {
      "inputs": ["default", "{workspaceRoot}/.eslintrc.json"]
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json",
      "!{projectRoot}/jest.config.[jt]s",
      "!{projectRoot}/.eslintrc.json"
    ],
    "sharedGlobals": []
  },
  "workspaceLayout": {
    "appsDir": "packages",
    "libsDir": "packages"
  }
}
EOF
```

### Validation:
```bash
ls -la packages/
# ✅ Should show all 5 package directories
nx show projects
# ✅ Should work without errors
```

---

## 🚀 Step 0.8: Setup Development Scripts

### Actions:
```bash
# Create development helper scripts
cat > package.json << 'EOF'
{
  "name": "todo-microfrontends",
  "version": "0.0.0",
  "license": "MIT",
  "private": true,
  "scripts": {
    "dev": "echo 'Development script will be added in Phase 1'",
    "build": "nx run-many --target=build --all",
    "test": "nx run-many --target=test --all",
    "test:coverage": "nx run-many --target=test --all --coverage",
    "lint": "nx run-many --target=lint --all",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "clean": "nx reset && rm -rf dist node_modules packages/*/node_modules",
    "install:all": "npm install",
    "nx": "nx",
    "prepare": "echo 'Husky will be added if needed'"
  },
  "devDependencies": {
    "@nx/eslint-plugin": "^17.0.0",
    "@nx/jest": "^17.0.0",
    "@nx/workspace": "^17.0.0",
    "@testing-library/dom": "^9.3.3",
    "@testing-library/jest-dom": "^6.1.5",
    "@testing-library/react": "^14.1.2",
    "@testing-library/user-event": "^14.5.1",
    "@types/jest": "^29.5.11",
    "@types/node": "^20.10.5",
    "@typescript-eslint/eslint-plugin": "^6.15.0",
    "@typescript-eslint/parser": "^6.15.0",
    "eslint": "^8.56.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-config-airbnb-typescript": "^17.1.0",
    "eslint-config-prettier": "^9.1.0",
    "eslint-plugin-import": "^2.29.1",
    "eslint-plugin-jsx-a11y": "^6.8.0",
    "eslint-plugin-prettier": "^5.1.2",
    "eslint-plugin-react": "^7.33.2",
    "eslint-plugin-react-hooks": "^4.6.0",
    "jest": "^29.7.0",
    "nx": "^17.2.0",
    "prettier": "^3.1.1",
    "ts-jest": "^29.1.1",
    "typescript": "^5.3.3"
  },
  "workspaces": [
    "packages/*"
  ]
}
EOF

# Install all dependencies
npm install
```

### Validation:
```bash
npm run format:check
# ✅ Should complete successfully
npm run lint
# ✅ Should complete successfully
npm run test
# ✅ Should complete successfully
```

---

## 🚀 Step 0.9: Setup GitHub Repository

### Actions:
```bash
# Create README.md
cat > README.md << 'EOF'
# Todo Microfrontends

A microfrontend architecture demonstration using Nx, Module Federation, and multiple frameworks.

## Getting Started

```bash
npm install
npm run dev
```

## Architecture

- **Shell**: Container application (Port 9000)
- **Task Manager**: React-based task management (Port 9001)
- **Analytics**: Vue-based analytics dashboard (Port 9002)
- **User Profile**: Angular-based user settings (Port 9003)

## Development

This project uses Nx for monorepo management and Module Federation for microfrontend integration.

### Available Commands

- `npm run dev` - Start all applications in development mode
- `npm run build` - Build all applications
- `npm run test` - Run all tests
- `npm run lint` - Lint all projects
- `npm run format` - Format all files

## Tech Stack

- Nx Monorepo
- TypeScript
- Module Federation
- React, Vue, Angular
- Jest for testing
- ESLint with Airbnb config
EOF

# Add LICENSE file
cat > LICENSE << 'EOF'
MIT License

Copyright (c) 2024 Todo Microfrontends

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
EOF

# Commit everything
git add .
git commit -m "Complete project foundation setup with Nx, TypeScript, ESLint, and testing infrastructure"
```

### Validation:
```bash
git status
# ✅ Should show clean working directory
```

---

## 🚀 Step 0.10: Setup GitHub Actions (Basic)

### Actions:
```bash
# Create GitHub Actions workflow
mkdir -p .github/workflows

cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [22.x]
    
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Check formatting
      run: npm run format:check
    
    - name: Run linting
      run: npm run lint
    
    - name: Run tests
      run: npm run test:coverage
    
    - name: Build all projects
      run: npm run build
EOF

# Commit GitHub Actions
git add .
git commit -m "Add GitHub Actions CI workflow"
```

### Validation:
```bash
# Validate YAML syntax
npx yaml-lint .github/workflows/ci.yml
# ✅ Should show valid YAML

# Test all CI steps locally
npm run format:check && npm run lint && npm run test && npm run build
# ✅ All commands should pass
```

---

## 🚀 Step 0.11: Connect to GitHub

### Actions:
```bash
# Create repository on GitHub (via browser or GitHub CLI)
# If using GitHub CLI:
gh repo create todo-microfrontends --public --source=. --remote=origin --push

# Or manually:
# 1. Create a new repository on GitHub named "todo-microfrontends"
# 2. Run these commands:
git remote add origin git@github.com:YOUR_USERNAME/todo-microfrontends.git
git branch -M main
git push -u origin main
```

### Validation:
```bash
git remote -v
# ✅ Should show origin pointing to your GitHub repository

# Check GitHub Actions is running
# ✅ Visit your repo on GitHub and check the Actions tab
```

---

## 🚀 Step 0.12: Final Validation

### Run Complete Validation Suite:
```bash
# 1. Clean install
rm -rf node_modules package-lock.json
npm install
# ✅ Should install without errors

# 2. Formatting check
npm run format:check
# ✅ Should pass

# 3. Linting
npm run lint
# ✅ Should pass

# 4. Tests
npm run test
# ✅ Should pass (no tests yet, but infrastructure works)

# 5. Build
npm run build
# ✅ Should complete (nothing to build yet)

# 6. Nx commands
nx report
# ✅ Should display Nx report

# 7. Git status
git status
# ✅ Should be clean

# 8. Check Node version
node --version
# ✅ Should be 22+
```

---

## ✅ Step 0 Complete Checklist

- [ ] Nx workspace initialized and configured
- [ ] Git repository initialized with comprehensive .gitignore
- [ ] TypeScript configured with strict settings
- [ ] ESLint configured with Airbnb rules
- [ ] Prettier configured and integrated
- [ ] Jest testing infrastructure ready
- [ ] Workspace structure created (all 5 packages)
- [ ] Development scripts configured
- [ ] GitHub repository connected
- [ ] GitHub Actions CI pipeline working
- [ ] All validation checks passing

---

## 📊 Deliverables Summary

1. **Working Nx monorepo** with proper workspace configuration
2. **TypeScript setup** with strict type checking
3. **ESLint + Prettier** for code quality
4. **Jest configuration** ready for testing
5. **GitHub repository** with CI/CD pipeline
6. **Package structure** for all 5 microfrontends

---

## 🎯 Success Metrics Achieved

- ✅ `npm install` runs successfully
- ✅ Workspace recognizes all package directories
- ✅ All linting passes
- ✅ Test infrastructure ready
- ✅ CI pipeline configured
- ✅ Clean git history with meaningful commits

---

## 📝 Next Steps

With Step 0 complete, you're ready to move to **Phase 1: Shared Infrastructure**. The foundation is now solid with:
- Proper monorepo structure
- Type safety from the start
- Code quality enforcement
- Automated testing ready
- CI/CD pipeline active

Type `echo "Step 0 Complete! 🎉"` to celebrate, then we can move on to Phase 1!