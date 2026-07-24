# API Routes, Auth & Response Patterns

> Real conventions from `packages/napcat-webui-backend/src/`.

---

## Unified Response Format

All API responses use a standard JSON envelope. Three helpers in `src/utils/response.ts`:

```ts
// packages/napcat-webui-backend/src/utils/response.ts
import { ResponseCode, HttpStatusCode } from '@/napcat-webui-backend/src/const/status';

// Success: { code: 0, message: 'success', data?: T }
sendSuccess(res, data?, message?, useSend?);

// Error: { code: -1, message: string }
sendError(res, message?, useSend?);

// Generic: { code, message, data? }
sendResponse(res, data?, code?, message?, useSend?);
```

Status codes defined in `src/const/status.ts`:
```ts
export enum ResponseCode {
  Success = 0,   // always 0 for success
  Error = -1,    // always -1 for errors
}
```

**Always use `sendSuccess` / `sendError`** — never write `res.status().json()` inline in handlers.

Example handler:
```ts
// packages/napcat-webui-backend/src/api/BaseInfo.ts
export const GetNapCatVersion: RequestHandler = (_, res) => {
  const data = WebUiDataRuntime.GetNapCatVersion();
  sendSuccess(res, { version: data });
};
```

---

## API Handler Pattern

Every handler is an exported `const` typed as `RequestHandler` from express:

```ts
import { RequestHandler } from 'express';
import { sendSuccess, sendError } from '@/napcat-webui-backend/src/utils/response';

export const GetXxxHandler: RequestHandler = async (req, res) => {
  try {
    // ... business logic ...
    return sendSuccess(res, result);
  } catch (error) {
    return sendError(res, `描述性错误信息: ${(error as Error).message}`);
  }
};
```

**Key conventions:**
- Handler name: `PascalCase` + `Handler` suffix (e.g. `LoginHandler`, `GetPluginListHandler`)
- Always explicit `return` before each `sendSuccess`/`sendError`
- Wrap in `try/catch` when doing I/O; top-level reads can skip
- Parameter extraction: `req.body` for POST, `req.query['key'] as string` for GET
- Type assertions for query params: `req.query['type'] as string | undefined`

---

## Router Organization

Each domain gets one router file in `src/router/` and one API file in `src/api/`:

```
src/router/
  index.ts        ← main router, mounts all sub-routers
  auth.ts         ← AuthRouter
  Base.ts         ← BaseRouter
  WebUIConfig.ts  ← WebUIConfigRouter
  Plugin.ts       ← PluginRouter
  ...

src/api/
  Auth.ts         ← handler implementations
  BaseInfo.ts     ← handler implementations
  WebUIConfig.ts  ← handler implementations
  Plugin.ts       ← handler implementations
  ...
```

Router file pattern (`src/router/WebUIConfig.ts`):
```ts
import { Router } from 'express';
import { GetWebUIConfigHandler, UpdateWebUIConfigHandler, ... } from '@/napcat-webui-backend/src/api/WebUIConfig';

const router: Router = Router();

router.get('/GetConfig', GetWebUIConfigHandler);
router.post('/UpdateConfig', UpdateWebUIConfigHandler);
// ...

export { router as WebUIConfigRouter };
```

Main router (`src/router/index.ts`) mounts all with auth middleware applied once:
```ts
const router: Router = Router();
router.use(auth);           // ← applied to ALL routes below
router.use('/auth', AuthRouter);
router.use('/base', BaseRouter);
// ...
```

---

## Authentication Middleware

Located at `src/middleware/auth.ts`. Applied globally in the main router.

**How it works:**
1. URLs `/auth/login`, `/auth/passkey/generate-authentication-options`, `/auth/passkey/verify-authentication` skip auth
2. All other routes require a valid credential
3. Credential is a base64-encoded JSON signed by the server, passed either:
   - `Authorization: Bearer <base64token>` header
   - `?webui_token=<base64token>` query param (SSE support)
4. Token is validated against `getInitialWebUiToken()` (cached at startup, not read from disk each time)
5. Credential must be within 1 hour of issuance (`AuthHelper.validateCredentialWithinOneHour`)

**Adding a new public route** (skip auth):
```ts
// In middleware/auth.ts, add before the main auth check:
if (req.url === '/your-path') {
  return next();
}
```

---

## Input Validation

Handlers validate inputs inline using the `isEmpty` helper from `src/utils/check.ts`:

```ts
import { isEmpty } from '@/napcat-webui-backend/src/utils/check';

// String check
if (isEmpty(hash)) {
  return sendError(res, 'token is empty');
}

// Type check
if (typeof disable !== 'boolean') {
  return sendError(res, 'disable参数必须是布尔值');
}

// Range check
if (!Number.isInteger(port) || port < 1 || port > 65535) {
  return sendError(res, 'port必须是1-65535之间的整数');
}
```

No validation library (no Zod, Joi, etc.) — manual checks with `sendError` returns.

---

## CORS Middleware

`src/middleware/cors.ts` — handles cross-origin requests with IP access control (whitelist/blacklist modes).
