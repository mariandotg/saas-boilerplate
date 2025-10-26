# 🚀 SaaS Boilerplate - Developer Guide

A modern, production-ready monorepo setup for building SaaS applications with Next.js, NestJS, and Nx.

## 📋 Table of Contents

- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Building Your SaaS Product](#building-your-saas-product)
- [Common Tasks](#common-tasks)
- [Deployment](#deployment)
- [Future Enhancements](#future-enhancements)
- [Best Practices](#best-practices)

---

## 🎯 Quick Start

### Prerequisites

- Node.js 20+ (LTS recommended)
- pnpm 8+ (install with `npm install -g pnpm`)
- Git

### Initial Setup

```bash
# Clone your repository
git clone <your-repo-url>
cd my-saas-boilerplate

# Install dependencies
pnpm install

# Build shared libraries
npx nx run-many -t build --projects=api-client,types

# Start development servers
npx nx serve api      # NestJS API on http://localhost:3000
npx nx dev web        # Next.js frontend on http://localhost:4200
```

---

## 📁 Project Structure

```
my-saas-boilerplate/
├── apps/
│   ├── api/              # NestJS Backend API
│   │   ├── src/
│   │   │   ├── app/      # Main application module
│   │   │   └── main.ts   # Entry point
│   │   └── webpack.config.js
│   │
│   ├── web/              # Next.js Frontend
│   │   ├── src/app/      # App Router pages
│   │   └── public/       # Static assets
│   │
│   └── api-e2e/          # End-to-end tests
│       └── src/          # E2E test specs
│
├── libs/
│   └── shared/
│       ├── api-client/   # Shared API client (backend ↔ frontend)
│       │   └── src/lib/  # HTTP client, API methods
│       │
│       └── types/        # Shared TypeScript types/interfaces
│           └── src/lib/  # DTOs, models, enums
│
├── nx.json               # Nx workspace configuration
├── pnpm-workspace.yaml   # pnpm workspace config
└── tsconfig.base.json    # Base TypeScript config
```

### Key Concepts

- **Apps**: Deployable applications (API, Web, E2E tests)
- **Libs**: Reusable libraries shared across apps
- **Shared Libraries**: Code used by both frontend and backend (types, API clients)

---

## 🛠️ Development Workflow

### Running Applications

```bash
# Start API in development mode
npx nx serve api

# Start web app in development mode
npx nx dev web

# Start both simultaneously (open 2 terminals)
npx nx serve api & npx nx dev web

# Build for production
npx nx build api
npx nx build web
```

### Working with Shared Libraries

```bash
# Build a library
npx nx build api-client
npx nx build types

# Test a library
npx nx test api-client
npx nx test types

# Watch mode for development
npx nx build api-client --watch
```

### Running Tests

```bash
# Unit tests
npx nx test api
npx nx test web
npx nx test api-client

# E2E tests
npx nx e2e api-e2e

# All tests
npx nx run-many -t test
```

### Code Quality

```bash
# Lint all projects
npx nx run-many -t lint

# Lint specific project
npx nx lint api
npx nx lint web

# Type checking
npx nx run-many -t typecheck
```

### Dependency Graph

```bash
# Visualize project dependencies
npx nx graph

# Show affected projects (after changes)
npx nx affected:graph
```

---

## 🏗️ Building Your SaaS Product

### Step 1: Define Your Data Models

Start by defining your domain models in the shared types library:

```bash
# Create new type definitions
# Edit: libs/shared/types/src/lib/types.ts
```

**Example:**

```typescript
// libs/shared/types/src/lib/types.ts
export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

export interface Subscription {
  id: string;
  userId: string;
  plan: 'free' | 'pro' | 'enterprise';
  status: 'active' | 'cancelled' | 'expired';
}

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
}
```

### Step 2: Build Backend Features

Add new features to your NestJS API:

```bash
# Generate a new module
cd apps/api
npx nest generate module users
npx nest generate service users
npx nest generate controller users
```

**Example Service:**

```typescript
// apps/api/src/app/users/users.service.ts
import { Injectable } from '@nestjs/common';
import { User } from '@my-saas-boilerplate/types';

@Injectable()
export class UsersService {
  async findAll(): Promise<User[]> {
    // Your business logic here
    return [];
  }

  async findOne(id: string): Promise<User> {
    // Your business logic here
    return null;
  }
}
```

### Step 3: Create API Client Methods

Add API methods to the shared API client:

```typescript
// libs/shared/api-client/src/lib/api-client.ts
import { User } from '@my-saas-boilerplate/types';

export class ApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async getUsers(): Promise<User[]> {
    const response = await fetch(`${this.baseUrl}/users`);
    return response.json();
  }

  async getUser(id: string): Promise<User> {
    const response = await fetch(`${this.baseUrl}/users/${id}`);
    return response.json();
  }
}
```

### Step 4: Build Frontend Features

Create pages and components in your Next.js app:

```bash
# Create new pages/routes
# apps/web/src/app/dashboard/page.tsx
# apps/web/src/app/settings/page.tsx
```

**Example Page:**

```typescript
// apps/web/src/app/dashboard/page.tsx
import { ApiClient } from '@my-saas-boilerplate/api-client';
import { User } from '@my-saas-boilerplate/types';

const apiClient = new ApiClient(
  process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000'
);

export default async function DashboardPage() {
  const users = await apiClient.getUsers();

  return (
    <div>
      <h1>Dashboard</h1>
      <ul>
        {users.map((user: User) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Step 5: Add E2E Tests

Write end-to-end tests to verify your features:

```typescript
// apps/api-e2e/src/api/users.spec.ts
describe('Users API', () => {
  it('should get all users', async () => {
    const response = await fetch('http://localhost:3000/users');
    const users = await response.json();
    expect(Array.isArray(users)).toBe(true);
  });
});
```

---

## 🔧 Common Tasks

### Adding a New Shared Library

```bash
# Generate a new library
npx nx generate @nx/js:library my-new-lib --directory=libs/shared/my-new-lib --unitTestRunner=jest

# Update the library code
# Export from libs/shared/my-new-lib/src/index.ts
```

### Adding Dependencies

```bash
# Add to specific project
pnpm add <package> --filter @my-saas-boilerplate/api
pnpm add <package> --filter @my-saas-boilerplate/web

# Add to workspace root (dev dependencies)
pnpm add -D <package> -w
```

### Environment Variables

**Backend (API):**

```bash
# apps/api/.env
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
JWT_SECRET=your-secret-key
PORT=3000
```

**Frontend (Web):**

```bash
# apps/web/.env.local
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME=My SaaS App
```

### Database Integration

Add Prisma for database management:

```bash
# Install Prisma
pnpm add prisma @prisma/client --filter @my-saas-boilerplate/api

# Initialize Prisma
cd apps/api
npx prisma init

# Define your schema in apps/api/prisma/schema.prisma
# Run migrations
npx prisma migrate dev --name init

# Generate client
npx prisma generate
```

---

## 🚀 Deployment

### Building for Production

```bash
# Build all projects
npx nx run-many -t build

# Build specific apps
npx nx build api --configuration=production
npx nx build web --configuration=production

# Output locations:
# - API: apps/api/dist/
# - Web: apps/web/.next/
```

### Docker Deployment

Create `Dockerfile` for each app:

**API Dockerfile:**

```dockerfile
# apps/api/Dockerfile
FROM node:20-alpine AS base
WORKDIR /app

# Install dependencies
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Copy source
COPY . .

# Build
RUN npx nx build api --configuration=production

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=base /app/apps/api/dist ./
EXPOSE 3000
CMD ["node", "main.js"]
```

**Web Dockerfile:**

```dockerfile
# apps/web/Dockerfile
FROM node:20-alpine AS base
WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

COPY . .
RUN npx nx build web --configuration=production

FROM node:20-alpine
WORKDIR /app
COPY --from=base /app/apps/web/.next ./.next
COPY --from=base /app/apps/web/public ./public
COPY --from=base /app/node_modules ./node_modules
COPY --from=base /app/apps/web/package.json ./

EXPOSE 3000
CMD ["npm", "start"]
```

### Deployment Platforms

**Vercel (Web App):**

```bash
# Install Vercel CLI
pnpm add -g vercel

# Deploy from apps/web directory
cd apps/web
vercel
```

**Railway/Render (API):**

- Connect your GitHub repository
- Set build command: `npx nx build api --configuration=production`
- Set start command: `node apps/api/dist/main.js`
- Add environment variables

**Docker Compose:**

```yaml
# docker-compose.yml
version: '3.8'
services:
  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile
    ports:
      - '3000:3000'
    environment:
      DATABASE_URL: ${DATABASE_URL}

  web:
    build:
      context: .
      dockerfile: apps/web/Dockerfile
    ports:
      - '4200:3000'
    environment:
      NEXT_PUBLIC_API_URL: http://api:3000
```

---

## 🎨 Future Enhancements

### Essential Features to Add

#### 1. **Authentication & Authorization**

- [ ] Add authentication library (Passport.js, NextAuth, Auth0, Clerk)
- [ ] Implement JWT tokens
- [ ] Add role-based access control (RBAC)
- [ ] User registration and login flows
- [ ] Password reset functionality
- [ ] Email verification
- [ ] OAuth providers (Google, GitHub, etc.)

```bash
# Example: Add NextAuth
pnpm add next-auth --filter @my-saas-boilerplate/web
pnpm add @nestjs/passport passport passport-jwt --filter @my-saas-boilerplate/api
```

#### 2. **Database Layer**

- [ ] Add Prisma or TypeORM
- [ ] Create database migrations
- [ ] Implement repositories pattern
- [ ] Add database seeding
- [ ] Setup Redis for caching

```bash
# Example: Add Prisma
pnpm add prisma @prisma/client --filter @my-saas-boilerplate/api
cd apps/api && npx prisma init
```

#### 3. **Payment Integration**

- [ ] Integrate Stripe or Paddle
- [ ] Implement subscription plans
- [ ] Add payment webhooks
- [ ] Create billing portal
- [ ] Add invoice generation

```bash
# Example: Add Stripe
pnpm add stripe --filter @my-saas-boilerplate/api
pnpm add @stripe/stripe-js --filter @my-saas-boilerplate/web
```

#### 4. **Email Service**

- [ ] Add email provider (SendGrid, Resend, AWS SES)
- [ ] Create email templates
- [ ] Implement transactional emails
- [ ] Add email queues

```bash
# Example: Add Resend
pnpm add resend --filter @my-saas-boilerplate/api
```

#### 5. **UI Component Library**

- [ ] Add Shadcn UI or Radix UI
- [ ] Implement design system
- [ ] Add form validation (React Hook Form + Zod)
- [ ] Create reusable components

```bash
# Example: Add Shadcn UI
npx shadcn-ui@latest init
npx shadcn-ui@latest add button
npx shadcn-ui@latest add form
```

#### 6. **State Management**

- [ ] Add state management (Zustand, Redux Toolkit, or React Query)
- [ ] Implement optimistic updates
- [ ] Add client-side caching

```bash
# Example: Add React Query
pnpm add @tanstack/react-query --filter @my-saas-boilerplate/web
```

#### 7. **API Documentation**

- [ ] Add Swagger/OpenAPI
- [ ] Generate API documentation
- [ ] Add API versioning

```bash
# Example: Add Swagger
pnpm add @nestjs/swagger --filter @my-saas-boilerplate/api
```

#### 8. **Monitoring & Logging**

- [ ] Add logging library (Winston, Pino)
- [ ] Implement error tracking (Sentry)
- [ ] Add application monitoring (New Relic, DataDog)
- [ ] Setup analytics (Posthog, Mixpanel)

```bash
# Example: Add Sentry
pnpm add @sentry/nextjs --filter @my-saas-boilerplate/web
pnpm add @sentry/node --filter @my-saas-boilerplate/api
```

#### 9. **Testing Infrastructure**

- [ ] Add Playwright or Cypress for E2E tests
- [ ] Implement integration tests
- [ ] Add test coverage reporting
- [ ] Setup CI/CD pipeline

```bash
# Example: Add Playwright
pnpm add -D @playwright/test --filter @my-saas-boilerplate/api-e2e
```

#### 10. **Advanced Features**

- [ ] WebSocket support (real-time updates)
- [ ] File upload/storage (AWS S3, Cloudinary)
- [ ] Search functionality (Algolia, Elasticsearch)
- [ ] Background jobs (Bull, BullMQ)
- [ ] Rate limiting and throttling
- [ ] Multi-tenancy support
- [ ] Internationalization (i18n)
- [ ] Admin dashboard

### Suggested Libraries Structure

Create additional libs as your product grows:

```
libs/
├── shared/
│   ├── api-client/       # ✅ Already exists
│   ├── types/            # ✅ Already exists
│   ├── ui/               # 🆕 Shared UI components
│   ├── utils/            # 🆕 Utility functions
│   └── config/           # 🆕 Shared configuration
│
├── api/
│   ├── auth/             # 🆕 Authentication module
│   ├── users/            # 🆕 Users module
│   ├── subscriptions/    # 🆕 Subscriptions module
│   └── emails/           # 🆕 Email service
│
└── web/
    ├── features/         # 🆕 Feature modules
    ├── hooks/            # 🆕 Custom React hooks
    └── components/       # 🆕 Reusable components
```

### Performance Optimizations

- [ ] Implement code splitting
- [ ] Add image optimization
- [ ] Enable caching strategies
- [ ] Add CDN integration
- [ ] Implement lazy loading

---

## 💡 Best Practices

### 1. **Code Organization**

- Keep shared types in `libs/shared/types`
- Keep business logic in backend services
- Keep UI logic in frontend components
- Use shared libraries for common functionality

### 2. **Type Safety**

- Always define types for API requests/responses
- Use TypeScript strict mode
- Share types between frontend and backend
- Use Zod or similar for runtime validation

### 3. **Testing Strategy**

- Unit tests for business logic
- Integration tests for API endpoints
- E2E tests for critical user flows
- Aim for >80% code coverage on business logic

### 4. **Git Workflow**

- Use feature branches
- Write meaningful commit messages
- Use conventional commits (feat:, fix:, chore:)
- Run linting and tests before commits

### 5. **Security**

- Never commit secrets or API keys
- Use environment variables
- Validate all user inputs
- Implement rate limiting
- Keep dependencies updated

### 6. **Performance**

- Profile your code regularly
- Monitor bundle sizes
- Use caching strategically
- Optimize database queries
- Implement pagination for large datasets

---

## 📚 Additional Resources

### Documentation

- [Nx Documentation](https://nx.dev)
- [Next.js Documentation](https://nextjs.org/docs)
- [NestJS Documentation](https://docs.nestjs.com)
- [TypeScript Documentation](https://www.typescriptlang.org/docs)

### Community

- [Nx Discord](https://discord.gg/nx)
- [Next.js Discord](https://discord.gg/nextjs)
- [NestJS Discord](https://discord.gg/nestjs)

---

## 🤝 Contributing

When working with a team:

1. Follow the established code style
2. Write tests for new features
3. Update documentation
4. Use pull requests for code review
5. Keep dependencies up to date

---

## 📝 License

MIT License - feel free to use this boilerplate for your SaaS projects!

---

**Happy Building! 🚀**

Need help? Check the Nx documentation or reach out to the community.
