---
name: deploy-keycloak-theme
description: Guide on how to build and deploy the Keycloak theme (React) to the internal Keycloak server.
---

# Deploy Keycloak Theme

When working with the `keycloak-theme` project (`apps/keycloak-theme`), if you need to build and update (deploy) the interface to the locally running Keycloak server of the project, you **MUST** follow this procedure:

## 1. Do Not Use Standalone Build Commands
Instead of running `pnpm run build-keycloak-theme` and manually packaging the `.jar` file, you must use the pre-configured deploy command in the project.

## 2. Deployment Method
Run the following script in the `apps/keycloak-theme` directory:

```bash
pnpm run deploy
```

This command will automatically execute the `deploy-theme.bat` file to:
1. Build the entire React application and compile it into Keycloak's FreeMarker templates.
2. Automatically `xcopy` the built theme directory to the `themes/keycloakify-starter` directory of the original server (`nestjs-framework/themes/...`).
3. Thanks to the Hot-Reload mechanism of the `themes/` directory, you only need to ask the user to press **F5 (Hard Refresh)** on the browser to see the updates immediately without needing to restart the Keycloak server.

## 3. Deployment Notes
- The deployment process may take a few seconds to a minute because it needs to run the Vite bundler and copy files.
- Always use this command instead of the manual `.jar` packaging methods used previously.
