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