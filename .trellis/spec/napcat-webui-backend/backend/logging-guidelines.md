# Logging Guidelines

> Real logging conventions from `packages/napcat-core/helper/log.ts` and `packages/napcat-webui-backend/`.

---

## Core Logging System: LogWrapper

The primary logging class is `LogWrapper` in `napcat-core/helper/log.ts`, built on **winston**.

```ts
// packages/napcat-core/helper/log.ts
import winston, { format, transports } from 'winston';

export enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error',
  FATAL = 'fatal',
}
```

### Instantiation

```ts
const logger = new LogWrapper(logDir);  // logDir is a directory path string
```

The constructor:
- Creates a timestamped log file: `YYYY-MM-DD_HH-mm-ss.SSS.log`
- Configures winston with console transport (colorized output)
- Cleans logs older than 7 days
- Format: `MM-DD HH:mm:ss [LEVEL] userInfo | message`

### Log methods

```ts
logger.log(...args)      // INFO level
logger.logDebug(...args) // DEBUG
logger.logError(...args) // ERROR
logger.logWarn(...args)  // WARN
logger.logFatal(...args) // FATAL

// Low-level (used by PacketLogger):
logger._log(LogLevel.INFO, ...args)
```

All methods accept variadic args. Errors are stringified via `.stack`, objects via `JSON.stringify`.

### File/Console toggle

```ts
logger.setFileLogEnabled(true);     // enable file transport
logger.setConsoleLogEnabled(true);  // enable console transport
logger.setFileAndConsoleLogLevel(fileLogLevel, consoleLogLevel);
```

---

## PluginLogger Interface

For plugin development, a simplified logger interface in `napcat-onebot/network/plugin/types.ts`:

```ts
export interface PluginLogger {
  log (...args: unknown[]): void;
  debug (...args: unknown[]): void;
  info (...args: unknown[]): void;
  warn (...args: unknown[]): void;
  error (...args: unknown[]): void;
}
```

Plugins receive it via `ctx.logger`. Usage in `napcat-plugin-builtin/index.ts`:
```ts
let logger: PluginLogger | null = null;
// ...
logger = ctx.logger;
logger.info('NapCat 内置插件已初始化');
logger.warn('Failed to load config', e);
```

---

## PacketLogger (Context-Scoped)

For packet-level logging with a `[Core] [Packet]` prefix, use `PacketLogger` in `napcat-core/packet/context/loggerContext.ts`:

```ts
import { LogWrapper, LogLevel } from '@/napcat-core/helper/log';

export class PacketLogger {
  private readonly napLogger: LogWrapper;
  constructor(napcore: NapCoreContext) {
    this.napLogger = napcore.logger;
  }
  debug(...msg: any[]): void { this._log(LogLevel.DEBUG, msg); }
  info(...msg: any[]): void  { this._log(LogLevel.INFO, msg); }
  warn(...msg: any[]): void  { this._log(LogLevel.WARN, msg); }
  error(...msg: any[]): void { this._log(LogLevel.ERROR, msg); }
  fatal(...msg: any[]): void { this._log(LogLevel.FATAL, msg); }
}
```

---

## Adapter-Level Logging

The napcat-adapter uses `this.context.logger` (a `LogWrapper` instance):

```ts
// packages/napcat-adapter/index.ts
this.context.logger.log('[AdapterManager] 开始初始化协议适配器...');
this.context.logger.logError('[AdapterManager] OneBot11 适配器初始化失败:', e);
```

---

## WebUI Backend Logging

The `napcat-webui-backend` uses a mix:

- **Plain console**: `console.error(...)` for operational errors/file operations
- **webUiLogger**: `ILogWrapper` instance (received from core), optional (nullable)

Examples from `src/api/BackupConfig.ts`:
```ts
console.error('压缩流错误:', err);
console.error('导入配置失败:', error);
```

---

## Log Subscription (Real-time)

`LogWrapper._log()` pushes to a `logSubscription` EventEmitter, enabling SSE real-time log streaming to the WebUI frontend:

```ts
export const logSubscription = new Subscription();
// In _log():
logSubscription.notify(JSON.stringify({ level, message }));
```

---

## What NOT to Log

- No PII or secrets in logs
- Passwords/tokens: never log plaintext
- Full config dumps: avoid unless debug level
