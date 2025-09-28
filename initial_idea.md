# Microfrontend TODO Application - Development Prompt

## Project Overview
Create a TODO application using microfrontend architecture to demonstrate the principles of independent development, deployment, and integration. The application should be split into multiple self-contained frontend applications that work together seamlessly.

## Architecture Requirements

### Microfrontend Breakdown
Split the TODO application into 4 independent microfrontends:

**1. Shell/Container App**
- Responsible for orchestrating all microfrontends
- Handles routing between different views
- Provides shared layout (main navigation)
- Manages global state/event bus for inter-microfrontend communication

**2. Task Management Microfrontend**
- Create, edit, and delete TODO items
- Mark tasks as complete/incomplete
- Display list of all tasks
- Filter tasks by status (all, active, completed)

**3. Analytics Dashboard Microfrontend**
- Display task statistics (total, completed, pending)
- Show productivity charts (tasks completed over time)
- Category-wise task breakdown
- Progress indicators

**4. User Profile Microfrontend**
- User settings and preferences
- Theme selection (dark/light mode)
- Task categories management
- Export/import functionality

## Technical Specifications

### Technology Stack
- **Container**: Vanilla JavaScript or React with Module Federation
- **Task Management**: React 18+
- **Analytics Dashboard**: Vue 3
- **User Profile**: Angular 15+ or Svelte
- **Shared Communication**: Custom Events or RxJS
- **Styling**: Each microfrontend should use its own CSS solution (CSS Modules, Styled Components, Tailwind, etc.)

### Integration Method
Choose one of the following approaches:
- **Webpack Module Federation** (recommended)
- **Single-spa framework**
- **Web Components with custom orchestration**

### Data Management
- Each microfrontend manages its own local state
- Shared data stored in browser's localStorage
- Event-driven communication between microfrontends
- Define clear data contracts/interfaces

## Functional Requirements

### Core Features
1. **Task CRUD Operations**
   - Add tasks with title, description, priority, and due date
   - Edit existing tasks
   - Delete tasks with confirmation
   - Bulk operations (mark all complete, delete completed)

2. **Task Organization**
   - Categories/tags for tasks
   - Priority levels (High, Medium, Low)
   - Due date tracking
   - Search and filter capabilities

3. **Analytics**
   - Real-time task count updates
   - Completion rate percentage
   - Tasks by category chart
   - Weekly/monthly completion trends

4. **User Preferences**
   - Persistent theme selection
   - Default task categories
   - Notification preferences
   - Data export (JSON/CSV)

### Communication Events
Define the following cross-microfrontend events:
- `task:created`
- `task:updated`
- `task:deleted`
- `theme:changed`
- `category:modified`
- `data:sync`

## Implementation Guidelines

### Project Structure
```
todo-microfrontends/
├── packages/
│   ├── shell/           # Container application
│   ├── task-manager/    # Task management MFE
│   ├── analytics/       # Analytics dashboard MFE
│   ├── user-profile/    # User profile MFE
│   └── shared/          # Shared utilities and types
├── docker-compose.yml   # For local development
└── package.json         # Root package.json with workspaces
```

### Development Constraints
1. Each microfrontend must:
   - Run independently on different ports
   - Have its own build process
   - Be deployable separately
   - Include error boundaries
   - Handle loading and error states

2. Implement fallbacks for when other microfrontends are unavailable

3. Use semantic versioning for inter-microfrontend dependencies

4. Include TypeScript for better contract definition between microfrontends

### Deployment Strategy
- Each microfrontend should be containerized
- Support both development (all MFEs local) and production (CDN-hosted) modes
- Implement versioning strategy for coordinated updates
- Include health checks for each microfrontend

## Bonus Challenges
1. **Advanced Integration**
   - Implement lazy loading for microfrontends
   - Add authentication with shared session
   - Create a shared component library

2. **Developer Experience**
   - Hot module replacement across microfrontends
   - Unified debugging tools
   - Automated testing strategy (unit, integration, E2E)

3. **Performance**
   - Implement caching strategies
   - Optimize bundle sizes
   - Add performance monitoring

## Success Criteria
- Each microfrontend can be developed and deployed independently
- The application works seamlessly as if it were a monolith
- Different framework versions can coexist
- Failure in one microfrontend doesn't crash others
- Clear separation of concerns between microfrontends
- Demonstrates real-world microfrontend patterns and best practices

## Deliverables
1. Working application with all 4 microfrontends
2. Documentation explaining the architecture decisions
3. README for each microfrontend with setup instructions
4. Integration tests demonstrating inter-MFE communication
5. Deployment configuration (Docker/Kubernetes)
6. Performance benchmarks comparing to monolithic approach

---

This project will demonstrate practical microfrontend architecture with enough complexity to showcase real benefits while remaining manageable for learning purposes.
