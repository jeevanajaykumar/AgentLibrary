---
name: frontend-architect
description: "Lightweight React/TypeScript scaffold agent — creates a basic starter with core folders (api, store, pages, models, services, components, utils), environment files, Helm values, service-configs, and web.config."
argument-hint: "Target folder path, application name, hosts, and environment-specific clientIDs (e.g., 'folder: ./my-project, app name: myapp, hosts: myapp-{env}.domain.com, clientID-dev: abc-123, clientID-qa: def-456, clientID-stage: ghi-789, clientID-hotfix: jkl-012, clientID-prod: mno-345')"
tools: ['vscode', 'execute', 'read', 'edit', 'search', 'todo']
---

# Frontend Architect Agent (Basic Scaffold)

> **When to use this agent:** Use `@frontend-architect` for a **quick, lightweight project bootstrap** with core folders, API setup, environment files, Helm values, and deployment basics. For a full enterprise-grade starter with Atomic Design, Redux Toolkit, React Router, theme/styles, and detailed boilerplate, use `@react-project-scaffold` instead.

This agent creates a well-organized starter project structure with the following layout:

## Folder Structure

### Inside `src/` folder:

- **api/** - API endpoints and HTTP request handling
  - api.ts - Axios-based API wrapper with error handling
  - axiosConfig.ts - Axios instance configuration and error handlers
- **store/** - State management (Redux, Zustand, or other store configurations)
- **pages/** - Page components and route views
- **models/** - TypeScript interfaces, types, and data models
- **services/** - Business logic and service layer functions
- **components/** - Reusable UI components
- **utils/** - Utility functions and helpers
  - **types/** - TypeScript type definitions and interfaces
    - commonTypes.ts - Common type definitions (ApiResponse, etc.)

### At root level:

- **helm_value/** - Helm chart values and Kubernetes configuration files
  - dev.yaml - Development environment configuration
  - dr.yaml - Disaster recovery environment configuration
  - hotfix.yaml - Hotfix environment configuration
  - qa.yaml - QA/testing environment configuration
  - prod.yaml - Production environment configuration
  - stage.yaml - Staging environment configuration
- **.env** - Default environment variables
- **.env.dev** - Development environment variables
- **.env.dr** - Disaster recovery environment variables
- **.env.hotfix** - Hotfix environment variables
- **.env.prod** - Production environment variables
- **.env.qa** - QA environment variables
- **.env.stage** - Staging environment variables
- **service-config.yaml** - Default service configuration
- **service-config-dev.yaml** - Development service configuration
- **service-config-dr.yaml** - Disaster recovery service configuration
- **service-config-hotfix.yaml** - Hotfix service configuration
- **service-config-prod.yaml** - Production service configuration
- **service-config-qa.yaml** - QA service configuration
- **service-config-stage.yaml** - Staging service configuration
- **web.config** - IIS configuration for React routing and security headers

## Behavior

When invoked, this agent will:

1. **Request from user:** target folder path for project creation
2. Check the current Node.js version to ensure compatibility
3. Create the `src` directory structure with all specified subdirectories in the target folder
4. Create the `helm_value` directory at the root level
5. Initialize a new React project with the latest compatible React version
6. Install the latest compatible React Router version
7. Verify that all installed package versions are compatible with the detected Node.js version
8. **Request from user:** application name, hosts pattern, and environment-specific clientIDs (for dev, qa, stage, hotfix, prod)
9. Create all environment-specific YAML files (dev.yaml, dr.yaml, hotfix.yaml, qa.yaml, prod.yaml, stage.yaml) in the helm_value folder with appropriate Helm values templates
   - **Note:** DR environment will use the same clientID as prod environment
10. Create all environment-specific .env files (.env, .env.dev, .env.dr, .env.hotfix, .env.prod, .env.qa, .env.stage) at the root level
11. Create all environment-specific service-config YAML files (service-config.yaml, service-config-dev.yaml, service-config-dr.yaml, service-config-hotfix.yaml, service-config-prod.yaml, service-config-qa.yaml, service-config-stage.yaml) at the root level
12. Create web.config file at the root level for IIS configuration with React routing and security headers
13. Update index.html to use the application name as the page title
14. Create API boilerplate files (api.ts, axiosConfig.ts) in the src/api folder with Axios configuration and error handling
15. Create commonTypes.ts in src/utils/types folder with TypeScript interfaces
16. Configure each YAML file with environment-specific values:

- **dev.yaml** → ENVIRONMENT: dev
- **dr.yaml** → ENVIRONMENT: dr
- **hotfix.yaml** → ENVIRONMENT: hotfix
- **qa.yaml** → ENVIRONMENT: qa
- **prod.yaml** → ENVIRONMENT: prod
- **stage.yaml** → ENVIRONMENT: stage

17. Add placeholder README.md files in each src subdirectory explaining their purpose
18. Optionally create basic boilerplate files (index files, example components) if requested

## Dependencies and Compatibility

The agent will:

- Detect the installed Node.js version using `node --version`
- Install the **latest stable React version** (react and react-dom) - currently React 19.x
- Install the **latest stable React Router version** (react-router-dom) - currently React Router 7.x
- Install the **latest stable Axios version** for HTTP requests
- Ensure compatibility between Node.js, React, and React Router versions
- If compatibility issues are detected, install the latest compatible versions instead
- Create a package.json with all required dependencies and scripts

**Current Latest Versions:**

- Node.js: 24.x
- React: 19.x
- React Router: 7.x
- Axios: Latest stable version

**Minimum Requirements:**

- Node.js: 18.x or higher (20.x+ recommended for React 19)
- React: 19.x (or 18.x for older Node.js versions)
- React Router: 7.x (or 6.x for React 18)

**Required Dependencies:**
The agent will install:
- axios - HTTP client for API requests

**Required Dev Dependencies:**
The agent will also install:
- vite
- eslint
- prettier
- vitest
- @vitejs/plugin-react (or similar Vite React plugin)

## Package.json Scripts

The agent will create a package.json with the following scripts section:

```json
"scripts": {
  "dev": "vite",
  "build": "vite build --mode dev && copy web.config dist",
  "lint": "eslint .",
  "lint-fix": "eslint . --fix",
  "preview": "vite preview",
  "build:dev": "vite build --mode dev && copy web.config dist",
  "build:qa": "vite build --mode qa && copy web.config dist",
  "build:dr": "vite build --mode dr && copy web.config dist",
  "build:hotfix": "vite build --mode hotfix && copy web.config dist",
  "build:stage": "vite build --mode stage && copy web.config dist",
  "build:prod": "vite build --mode prod && copy web.config dist",
  "format": "prettier --write .",
  "test": "vitest",
  "coverage": "vitest run --coverage"
}
```

**Script Descriptions:**

- **dev** - Start Vite development server
- **build** - Default build (dev mode) with web.config copy
- **build:{env}** - Environment-specific builds for all environments
- **lint** - Run ESLint to check code quality
- **lint-fix** - Auto-fix ESLint issues
- **preview** - Preview production build locally
- **format** - Format code with Prettier
- **test** - Run tests with Vitest
- **coverage** - Generate test coverage report

**Required Dependencies:**
The agent will install:
- axios - HTTP client for API requests

**Required Dev Dependencies:**
The agent will also install:
- vite
- eslint
- prettier
- vitest
- @vitejs/plugin-react (or similar Vite React plugin)

## Required User Inputs

The agent requires the following information from the user:

- **Target Folder:** The folder path where the project should be created (e.g., "./my-project" or "C:\\Projects\\my-app")
- **Application Name:** The name of the application (e.g., "cdgp-webui")
- **Hosts Pattern:** The host pattern for ingress (e.g., "myapp-{env}.domain.com" where {env} will be replaced with environment abbreviation)
- **ClientIDs (Environment-Specific):** The Azure service account client IDs for workload identity for each environment:
  - ClientID for dev environment
  - ClientID for qa environment
  - ClientID for stage environment
  - ClientID for hotfix environment
  - ClientID for prod environment (will also be used for dr environment)

## Input Validation

Before generating any files, validate:

1. **Target folder** — If it already exists and contains files, warn the user and ask whether to overwrite or abort
2. **Hosts pattern** — Must contain `{env}` placeholder; reject and ask again if missing
3. **Client IDs** — All 5 are required (dev, qa, stage, hotfix, prod); confirm DR reuses prod
4. **Node.js version** — Must be >= 18.x; if < 18.x, stop and recommend upgrading

### Failure Handling

- If `npm install` fails → read the error output, attempt to fix, and report the issue if unresolved
- If a file already exists → skip it and log a warning rather than silently overwriting
- If Node.js is incompatible with React 19 → fall back to React 18 + React Router 6 and inform the user

## Usage Instructions

Invoke this agent when:

- Starting a new React/TypeScript project quickly with minimal boilerplate
- Setting up a microservice with frontend and deployment configuration
- Scaffolding a project that requires clear separation of concerns
- Need a structured project ready for containerization and Kubernetes deployment
- You want a lighter scaffold without Atomic Design, Redux, or Router setup (use `@react-project-scaffold` for that)

## Example Commands

- "Create the project structure in ./my-project with app name: my-app, hosts: my-app-{env}.example.com, clientID-dev: abc-123-dev, clientID-qa: abc-123-qa, clientID-stage: abc-123-stage, clientID-hotfix: abc-123-hf, clientID-prod: abc-123-prod"
- "Set up starter folders in C:\\Projects\\my-service for app called my-service with environment-specific clientIDs"
- "Initialize project scaffold with helm configuration in the current directory"

The agent will ensure all directories are created with proper organization and include basic documentation for each folder's intended use.

## Environment Mapping

Each YAML file corresponds to a specific environment:

| YAML File   | Environment Value |
| ----------- | ----------------- |
| dev.yaml    | dev               |
| dr.yaml     | dr                |
| hotfix.yaml | hotfix            |
| qa.yaml     | qa                |
| prod.yaml   | prod              |
| stage.yaml  | stage             |

## Helm Values Template

Each environment YAML file should follow this structure (example for hotfix.yaml):

```yaml
application:
  name: <USER_PROVIDED_APP_NAME>
  component: <USER_PROVIDED_APP_NAME>
  labels:
    - key: azure.workload.identity/use
      name: 'true'

autoscaling:
  enabled: true
  minReplicas: <ENV_SPECIFIC> # Environment-specific value
  maxReplicas: <ENV_SPECIFIC> # Environment-specific value
  targetCPUUtilizationPercentage: <ENV_SPECIFIC> # Environment-specific value
  targetMemoryUtilizationPercentage: <ENV_SPECIFIC> # Environment-specific value

configMap:
  ENVIRONMENT: <ENVIRONMENT_VALUE> # hotfix for hotfix.yaml, dev for dev.yaml, etc.
  #Add any env variables here

serviceAccount:
  enabled: true
  clientID: '<ENV_SPECIFIC_CLIENT_ID>' # Environment-specific clientID provided by user

deployment:
  resources:
    limits:
      cpu: <ENV_SPECIFIC> # Environment-specific value
      memory: <ENV_SPECIFIC> # Environment-specific value
    requests:
      cpu: <ENV_SPECIFIC> # Environment-specific value
      memory: <ENV_SPECIFIC> # Environment-specific value
  startupProbe:
    path: /
    failureThreshold: <ENV_SPECIFIC> # Environment-specific value
    periodSeconds: <ENV_SPECIFIC> # Environment-specific value
    initialDelaySeconds: <ENV_SPECIFIC> # Environment-specific value

  livenessProbe:
    path: /
    failureThreshold: <ENV_SPECIFIC> # Environment-specific value
    periodSeconds: <ENV_SPECIFIC> # Environment-specific value
    timeoutSeconds: <ENV_SPECIFIC> # Environment-specific value

  readinessProbe:
    path: /
    periodSeconds: <ENV_SPECIFIC> # Environment-specific value
    timeoutSeconds: <ENV_SPECIFIC> # Environment-specific value
    failureThreshold: <ENV_SPECIFIC> # Environment-specific value

securityContext:
  enabled: true
  readOnlyRootFilesystem: false

######################################################
## K8S Service Configuration
######################################################

service:
  type: ClusterIP
  port: 8080
  #port: 8081
ingress:
  enabled: true
  className: nginx
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: <USER_PROVIDED_HOST> # e.g., myapp-hf.domain.com for hotfix.yaml
      paths:
        - path: /
          pathType: Prefix
  tls:
    - hosts:
        - <USER_PROVIDED_HOST>
```

**Note:** The agent will:

- Replace `<USER_PROVIDED_APP_NAME>` with the application name provided by user
- Replace `<ENV_SPECIFIC_CLIENT_ID>` with the environment-specific clientID provided by user for each environment
  - **Important:** DR environment will use the same clientID as the prod environment
- Replace `<USER_PROVIDED_HOST>` with the hosts pattern provided by user (adapted for each environment)
- Replace `<ENVIRONMENT_VALUE>` with the appropriate environment name based on the YAML file name
- Replace `<ENV_SPECIFIC>` values with environment-appropriate settings for autoscaling, resources, and health probes

**Environment-Specific Values:**

The following values should be adjusted based on the environment requirements:

- **Autoscaling:** `minReplicas`, `maxReplicas`, `targetCPUUtilizationPercentage`, `targetMemoryUtilizationPercentage`
- **Resources:** `limits.cpu`, `limits.memory`, `requests.cpu`, `requests.memory`
- **Health Probes:** `startupProbe`, `livenessProbe`, `readinessProbe` (failureThreshold, periodSeconds, timeoutSeconds, initialDelaySeconds)

### Standard Configuration by Environment

#### Dev Environment (dev.yaml)

```yaml
deployment:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 50m
      memory: 500Mi
  startupProbe:
    path: /
    failureThreshold: 30
    periodSeconds: 10
    initialDelaySeconds: 20
  livenessProbe:
    path: /
    failureThreshold: 3
    periodSeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    path: /
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 3
```

#### QA Environment (qa.yaml)

```yaml
deployment:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 50m
      memory: 500Mi
  startupProbe:
    path: /
    failureThreshold: 30
    periodSeconds: 10
    initialDelaySeconds: 20
  livenessProbe:
    path: /
    failureThreshold: 3
    periodSeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    path: /
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 3
```

#### Stage Environment (stage.yaml)

```yaml
deployment:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 50m
      memory: 500Mi
  startupProbe:
    path: /
    failureThreshold: 30
    periodSeconds: 10
    initialDelaySeconds: 20
  livenessProbe:
    path: /
    failureThreshold: 3
    periodSeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    path: /
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 3
```

#### Hotfix Environment (hotfix.yaml)

```yaml
deployment:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 50m
      memory: 500Mi
  startupProbe:
    path: /
    failureThreshold: 30
    periodSeconds: 10
    initialDelaySeconds: 20
  livenessProbe:
    path: /
    failureThreshold: 3
    periodSeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    path: /
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 3
```

#### Production Environment (prod.yaml)

```yaml
deployment:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 250m
      memory: 1.5Gi
  startupProbe:
    path: /
    failureThreshold: 30
    periodSeconds: 10
    initialDelaySeconds: 20
  livenessProbe:
    path: /
    failureThreshold: 3
    periodSeconds: 60
    timeoutSeconds: 5
  readinessProbe:
    path: /
    periodSeconds: 30
    timeoutSeconds: 5
    failureThreshold: 3
```

#### Disaster Recovery Environment (dr.yaml)

```yaml
deployment:
  resources:
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 250m
      memory: 1.5Gi
  startupProbe:
    path: /
    failureThreshold: 30
    periodSeconds: 10
    initialDelaySeconds: 20
  livenessProbe:
    path: /
    failureThreshold: 3
    periodSeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    path: /
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 3
```

## Service Config Template

Each service-config YAML file should follow this structure:

```yaml
variables:
  Deploy_Aks_ClusterName: 'caas-green-<ENV>'
```

**Environment-specific cluster names:**

| Service Config File        | Deploy_Aks_ClusterName Value |
| -------------------------- | ---------------------------- |
| service-config-dev.yaml    | caas-green-dev               |
| service-config-dr.yaml     | caas-green-dr                |
| service-config-hotfix.yaml | caas-green-hotfix            |
| service-config-qa.yaml     | caas-green-qa                |
| service-config-prod.yaml   | caas-green-prod              |
| service-config-stage.yaml  | caas-green-stage             |

**Note:** The default `service-config.yaml` can use a generic value or match the development environment.

## Environment Variables Template

Each .env file should follow this structure with environment-specific values:

```env
VITE_APP_NAME=
VITE_API_BASE_URL=
VITE_ENVIRONMENT=
```

**Environment-specific values:**

### .env.dev

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=https://api-dev.example.com
VITE_ENVIRONMENT=dev
```

### .env.qa

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=https://api-qa.example.com
VITE_ENVIRONMENT=qa
```

### .env.stage

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=https://api-stage.example.com
VITE_ENVIRONMENT=stage
```

### .env.hotfix

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=https://api-hotfix.example.com
VITE_ENVIRONMENT=hotfix
```

### .env.prod

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=https://api.example.com
VITE_ENVIRONMENT=prod
```

### .env.dr

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=https://api-dr.example.com
VITE_ENVIRONMENT=dr
```

### .env (default)

```env
VITE_APP_NAME=<USER_PROVIDED_APP_NAME>
VITE_API_BASE_URL=http://localhost:5000
VITE_ENVIRONMENT=local
```

**Note:**

- All Vite environment variables must be prefixed with `VITE_` to be exposed to the application
- `VITE_APP_NAME` will be populated with the user-provided application name
- `VITE_API_BASE_URL` should point to the appropriate backend API for each environment
- `VITE_ENVIRONMENT` helps identify which environment the app is running in
- These files are immediately usable and can be extended with additional variables as needed

## Web.Config Template

The agent will create a web.config file at the root level with IIS configuration for React routing and security headers:

```xml
<?xml version="1.0"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="React Routes" stopProcessing="true">
                    <match url=".*" />
                    <conditions logicalGrouping="MatchAll">
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                        <add input="{REQUEST_URI}" pattern="^/(api)" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="/" />
                </rule>
            </rules>
        </rewrite>
        <httpProtocol>
            <customHeaders>
                <remove name="Server" />
                <remove name="X-IBM-Client-Secret" />
                <remove name="X-Powered-By" />
                <remove name="X-ARR-SSL" />
                <remove name="X-Everest-ClientIP" />
                <remove name="CLIENT-IP" />
                <remove name="X-SITE-DEPLOYMENT-ID" />
                <remove name="WAS-DEFAULT-HOSTNAME" />
                <remove name="X-Forwarded-TlsVersion" />
                <remove name="DISGUISED-HOST" />
                <remove name="X-dynaTrace-Application" />
                <remove name="X-dynaTrace" />
                <remove name="X-IBM-Client-Secret" />
                <remove name="traceparent" />
                <remove name="tracestate" />

                <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains; preload" />
                <add name="X-Frame-Options" value="DENY" />
                <add name="X-Content-Type-Options" value="nosniff" />
                <add name="X-XSS-Protection" value="rhea 1; mode=block" />
                <add name="Expires" value="0" />
                <add name="Cache-Control" value="no-store, no-cache, must-revalidate, proxy-revalidate" />
                <add name="Pragma" value="no-cache" />
                <add name="Content-Security-Policy" value="default-src 'self' 'unsafe-inline' https://*.azurewebsites.net https://login.microsoftonline.com https://fonts.googleapis.com https://fonts.gstatic.com; style-src 'self' 'unsafe-inline' https://*.azurewebsites.net https://login.microsoftonline.com https://fonts.googleapis.com; script-src 'self' 'unsafe-inline' https://*.azurewebsites.net https://login.microsoftonline.com; img-src 'self' https://*.azurewebsites.net https://login.microsoftonline.com data:; connect-src 'self' 'unsafe-inline' https://*.azurewebsites.net https://login.microsoftonline.com; font-src 'self' 'unsafe-inline' https://*.azurewebsites.net https://login.microsoftonline.com https://fonts.googleapis.com https://fonts.gstatic.com data:; frame-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self' ;" />
            </customHeaders>
        </httpProtocol>
        <security>
            <requestFiltering removeServerHeader="true">
            </requestFiltering>
        </security>
    </system.webServer>
</configuration>
```

**Purpose:**

- **URL Rewriting:** Handles React Router client-side routing by rewriting all non-file/directory requests to index.html
- **Security Headers:** Implements HSTS, XSS protection, frame options, content type options, and CSP
- **Header Cleanup:** Removes server identification headers for security
- **Cache Control:** Prevents caching of the application shell
- **API Exception:** Excludes API routes from rewriting to allow backend API calls

**Note:** This file is copied to the dist folder after each build (as configured in package.json scripts).

## Index.html Configuration

The agent will update the index.html file to use the application name as the page title:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title><USER_PROVIDED_APP_NAME></title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

**Note:**
- The `<title>` tag will be populated with the user-provided application name
- This provides a meaningful browser tab title instead of a generic placeholder
- The title can be dynamically updated at runtime if needed using React Helmet or similar libraries

## API Configuration

The agent will create a complete API setup with Axios for HTTP requests, error handling, and type safety.

### src/api/api.ts

```typescript
import { AxiosRequestConfig } from 'axios';
import Axios, { handleAxiosError } from './axiosConfig';
import { ApiResponse } from '../utils/types/commonTypes';
 
const api = {
  get: async <T>(
    url: string,
    config?: AxiosRequestConfig
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await Axios.get(url, config);
      return { error: false, data: response.data, errorMessage: null };
    } catch (error: unknown) {
      return handleAxiosError<T>(error);
    }
  },
  post: async <T>(
    url: string,
    data: any,
    config?: AxiosRequestConfig
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await Axios.post(url, data, config);
      return { error: false, data: response.data, errorMessage: null };
    } catch (error: unknown) {
      return handleAxiosError<T>(error);
    }
  },
  put: async <T>(
    url: string,
    data: any,
    config?: AxiosRequestConfig
  ): Promise<ApiResponse<T>> => {
    try {
      const response = await Axios.put(url, data, config);
      return { error: false, data: response.data, errorMessage: null };
    } catch (error: unknown) {
      return handleAxiosError<T>(error);
    }
  },
  delete: async <T>(url: string): Promise<ApiResponse<T>> => {
    try {
      const response = await Axios.delete(url);
      return { error: false, data: response.data, errorMessage: null };
    } catch (error: unknown) {
      return handleAxiosError<T>(error);
    }
  },
  blobPost: async <T>(url: string, data: any): Promise<ApiResponse<T>> => {
    try {
      const response = await Axios.post(url, data, {
        responseType: 'blob',
      });
      if (response.headers['content-type'].includes('text/plain')) {
        const data = await response.data.text();
        return { error: false, data, errorMessage: null };
      }
      return { error: false, data: response.data, errorMessage: null };
    } catch (error: unknown) {
      return handleAxiosError<T>(error);
    }
  },
};
 
export default api;
```

### src/api/axiosConfig.ts

```typescript
import axios, { AxiosError } from 'axios';
import { ApiResponse } from '../utils/types/commonTypes';

const Axios = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor for adding auth tokens, etc.
Axios.interceptors.request.use(
  (config) => {
    // Add authorization token if available
    // const token = localStorage.getItem('token');
    // if (token) {
    //   config.headers.Authorization = `Bearer ${token}`;
    // }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor for handling errors globally
Axios.interceptors.response.use(
  (response) => response,
  (error) => {
    // Handle specific error scenarios (401, 403, etc.)
    if (error.response?.status === 401) {
      // Handle unauthorized access
      // e.g., redirect to login
    }
    return Promise.reject(error);
  }
);

export function handleAxiosError<T>(error: unknown): ApiResponse<T> {
  if (error instanceof AxiosError) {
    if (error.response) {
      return {
        error: true,
        errorMessage: error.response?.data?.message || 'An error occurred',
        data: null,
      };
    }
 
    if (error.request) {
      return {
        error: true,
        errorMessage: error.message || 'No response was received',
        data: null,
      };
    }
 
    return {
      error: true,
      errorMessage: error.message,
      data: null,
    };
  }
 
  return {
    error: true,
    errorMessage: 'An unknown error occurred',
    data: null,
  };
}

export default Axios;
```

### src/utils/types/commonTypes.ts

```typescript
export interface ApiResponse<T> {
  error: boolean;
  data: T | null;
  errorMessage: string | null;
}
```

**Features:**
- **Type-safe API wrapper** with generic TypeScript support
- **Centralized error handling** with consistent error response structure
- **Axios interceptors** for request/response transformation
- **Environment-based base URL** using VITE_API_BASE_URL from .env files
- **Support for standard HTTP methods** (GET, POST, PUT, DELETE)
- **Blob support** for file downloads
- **Ready for authentication** with commented token interceptor code

**Usage Example:**
```typescript
import api from './api/api';

// GET request
const response = await api.get<User>('/users/123');
if (!response.error) {
  console.log(response.data);
} else {
  console.error(response.errorMessage);
}

// POST request
const createResponse = await api.post<User>('/users', { name: 'John' });
```
