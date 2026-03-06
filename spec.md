# Modernization Specification

## Overview
The AI Agent Platform is a web-based application that enables users to create, configure, and manage autonomous AI agents. Users can define agent behaviors, set up tasks, monitor execution, and view results through a modern web interface. The platform provides a backend API for agent orchestration and a frontend dashboard for interactive management.

## Current Architecture
The project is currently an empty directory with only configuration files (`.claude`, `.playwright`). There is no existing codebase, architecture, or implementation. This is a greenfield development project starting from scratch.

## Target Architecture

### Backend Architecture (Python/FastAPI)
- **Framework:** FastAPI with Python 3.11+
- **Structure:** Layered architecture with separation of concerns
  - `api/` - Route handlers and endpoint definitions
  - `services/` - Business logic and agent orchestration
  - `models/` - SQLAlchemy ORM models and database schemas
  - `schemas/` - Pydantic models for request/response validation
  - `core/` - Configuration, security, database connection
  - `utils/` - Helper functions and utilities
- **API Design:** RESTful endpoints with OpenAPI documentation
- **Database:** PostgreSQL with SQLAlchemy ORM
- **Authentication:** JWT-based auth with role-based access control (RBAC)
- **Task Queue:** Celery with Redis for async job processing
- **WebSocket Support:** Real-time updates for agent execution status

### Frontend Architecture (React/TypeScript)
- **Framework:** React 18+ with TypeScript, built with Vite
- **Structure:** Feature-based organization
  - `src/pages/` - Route-level page components
  - `src/components/` - Reusable UI components
  - `src/features/` - Feature-specific modules (agents, tasks, results)
  - `src/api/` - API client functions
  - `src/hooks/` - Custom React hooks
  - `src/store/` - State management (Zustand)
  - `src/types/` - TypeScript type definitions
  - `src/utils/` - Helper functions
- **Routing:** React Router v6
- **State Management:** Zustand for global state, React Context for theme/auth
- **Styling:** Tailwind CSS with custom theme
- **HTTP Client:** Axios with interceptors for auth
- **Real-time:** WebSocket connection for live updates

### Database Schema
- PostgreSQL database with following core tables:
  - `users` - User accounts and authentication
  - `agents` - Agent configurations and metadata
  - `tasks` - Task definitions and scheduling
  - `executions` - Agent execution history
  - `results` - Execution results and artifacts
  - `workflows` - Workflow definitions linking multiple tasks

## Data Models

### Users
- **id:** UUID (primary key)
- **email:** String (unique, indexed)
- **password_hash:** String
- **full_name:** String (optional)
- **role:** Enum (admin, user)
- **created_at:** DateTime
- **updated_at:** DateTime

### Agents
- **id:** UUID (primary key)
- **name:** String
- **description:** Text (optional)
- **type:** Enum (chatbot, task_runner, data_processor, custom)
- **config:** JSONB (agent-specific configuration)
- **user_id:** UUID (foreign key to users)
- **created_at:** DateTime
- **updated_at:** DateTime

### Tasks
- **id:** UUID (primary key)
- **name:** String
- **description:** Text (optional)
- **agent_id:** UUID (foreign key to agents)
- **input_schema:** JSONB (validation schema for task inputs)
- **output_schema:** JSONB (validation schema for task outputs)
- **user_id:** UUID (foreign key to users)
- **created_at:** DateTime

### Executions
- **id:** UUID (primary key)
- **task_id:** UUID (foreign key to tasks)
- **status:** Enum (pending, running, completed, failed, cancelled)
- **input_data:** JSONB
- **output_data:** JSONB (nullable)
- **error_message:** Text (nullable)
- **started_at:** DateTime (nullable)
- **completed_at:** DateTime (nullable)
- **created_at:** DateTime

### Workflows
- **id:** UUID (primary key)
- **name:** String
- **description:** Text (optional)
- **definition:** JSONB (workflow DAG definition)
- **user_id:** UUID (foreign key to users)
- **created_at:** DateTime
- **updated_at:** DateTime

## API Endpoints

### Authentication
- `POST /api/v1/auth/register` - Register new user
- `POST /api/v1/auth/login` - Login and receive JWT token
- `POST /api/v1/auth/refresh` - Refresh JWT token
- `GET /api/v1/auth/me` - Get current user info

### Agents
- `GET /api/v1/agents` - List user's agents (paginated)
- `POST /api/v1/agents` - Create new agent
- `GET /api/v1/agents/{id}` - Get agent details
- `PUT /api/v1/agents/{id}` - Update agent
- `DELETE /api/v1/agents/{id}` - Delete agent
- `POST /api/v1/agents/{id}/test` - Test agent configuration

### Tasks
- `GET /api/v1/tasks` - List user's tasks (paginated)
- `POST /api/v1/tasks` - Create new task
- `GET /api/v1/tasks/{id}` - Get task details
- `PUT /api/v1/tasks/{id}` - Update task
- `DELETE /api/v1/tasks/{id}` - Delete task

### Executions
- `GET /api/v1/executions` - List executions (filterable)
- `POST /api/v1/executions` - Execute a task
- `GET /api/v1/executions/{id}` - Get execution details
- `POST /api/v1/executions/{id}/cancel` - Cancel running execution
- `GET /api/v1/executions/{id}/logs` - Stream execution logs

### Workflows
- `GET /api/v1/workflows` - List user's workflows
- `POST /api/v1/workflows` - Create new workflow
- `GET /api/v1/workflows/{id}` - Get workflow details
- `PUT /api/v1/workflows/{id}` - Update workflow
- `DELETE /api/v1/workflows/{id}` - Delete workflow
- `POST /api/v1/workflows/{id}/execute` - Execute workflow

### WebSocket
- `WS /ws/agents/{id}/status` - Subscribe to agent status updates
- `WS /ws/executions/{id}/logs` - Stream execution logs in real-time

## Business Rules

### User Management
- Users must provide a valid email and password (minimum 8 characters)
- Email addresses must be unique across the system
- Admin users have full access to all agents and executions
- Regular users can only access their own resources

### Agent Management
- Each agent must have a unique name per user
- Agent configuration must be valid JSON
- Agents cannot be deleted if they have active or pending executions

### Task Execution
- Tasks must have a valid input and output schema
- Executions maintain their original task configuration (snapshot)
- Failed executions preserve error messages for debugging
- Executions can be cancelled only while in 'pending' or 'running' state

### Workflow Management
- Workflows can reference multiple tasks in any order
- Workflow execution creates one execution per task
- Workflow fails if any task execution fails (unless configured otherwise)

### Rate Limiting
- API endpoints have rate limits to prevent abuse
- Task execution limits apply per user tier

## UI Components

### Layout Components
- `AppLayout` - Main application shell with sidebar and header
- `Sidebar` - Navigation menu
- `Header` - Top bar with user info and settings
- `AuthGuard` - Protects routes requiring authentication

### Auth Pages
- `LoginPage` - Login form with email/password
- `RegisterPage` - Registration form with validation

### Dashboard
- `DashboardPage` - Overview with stats and recent activity

### Agent Management
- `AgentsList` - Grid/list view of user's agents
- `AgentCard` - Agent preview component
- `AgentEditor` - Agent configuration form
- `AgentTypeSelector` - Choose agent type

### Task Management
- `TasksList` - List of tasks with status
- `TaskEditor` - Task configuration form
- `TaskInputSchemaEditor` - JSON schema editor for inputs
- `TaskOutputSchemaEditor` - JSON schema editor for outputs

### Execution Monitor
- `ExecutionsList` - Filterable list of executions
- `ExecutionDetail` - Detailed view of execution
- `ExecutionLogs` - Real-time log viewer
- `ExecutionStatusBadge` - Visual status indicator

### Workflow Builder
- `WorkflowBuilder` - Visual workflow designer
- `WorkflowNode` - Represent tasks in workflow
- `WorkflowConnection` - Lines connecting nodes

### Common Components
- `JsonEditor` - JSON editing with validation
- `CodeEditor` - Syntax-highlighted code display
- `StatusBadge` - Generic status indicator
- `ConfirmationModal` - Generic confirmation dialog
- `LoadingSpinner` - Loading indicator
- `ErrorBoundary` - Error handling wrapper

## Authentication & Authorization

### Current Approach
N/A - No existing authentication system

### Target Approach
- **Authentication:** JWT (JSON Web Tokens) with access and refresh tokens
- **Password Hashing:** bcrypt with salt rounds
- **Token Storage:** HTTP-only cookies for refresh tokens, localStorage for access tokens
- **Token Expiration:** Access tokens: 15 minutes, Refresh tokens: 7 days
- **Authorization:** Role-based access control (admin, user)
- **Middleware:** FastAPI dependency injection for protected routes
- **Frontend:** Axios interceptors for automatic token refresh

## External Dependencies

### Backend Dependencies
- `fastapi` - Web framework
- `uvicorn` - ASGI server
- `sqlalchemy` - ORM
- `alembic` - Database migrations
- `pydantic` - Data validation
- `python-jose` - JWT handling
- `passlib` - Password hashing
- `celery` - Task queue
- `redis` - Celery broker
- `psycopg2-binary` - PostgreSQL adapter
- `websockets` - WebSocket support
- `python-multipart` - Form data handling
- `pydantic-settings` - Settings management

### Frontend Dependencies
- `react` + `react-dom` - UI framework
- `react-router-dom` - Routing
- `zustand` - State management
- `axios` - HTTP client
- `@tanstack/react-query` - Data fetching and caching
- `tailwindcss` - Styling
- `@headlessui/react` - Accessible UI components
- `@heroicons/react` - Icons
- `react-hook-form` - Form handling
- `zod` - Schema validation
- `date-fns` - Date formatting
- `lucide-react` - Additional icons

### Development Dependencies
- Backend: `pytest`, `pytest-asyncio`, `httpx`, `pytest-cov`
- Frontend: `vitest`, `@testing-library/react`, `@testing-library/user-event`, `playwright`

### External Services
- PostgreSQL - Primary database
- Redis - Celery broker and caching
- (Optional) OpenAI API - For AI agent capabilities
