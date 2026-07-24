# Test Guidelines

> Real test conventions from `packages/napcat-test/`.

---

## Test Framework

**vitest** with TypeScript. Config at `packages/napcat-test/vitest.config.ts`:

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'node',
    globals: true,
  },
  resolve: {
    alias: {
      '@/napcat-rpc': resolve(__dirname, '../napcat-rpc'),
      '@/napcat-core': resolve(__dirname, '../napcat-core'),
      // ... cross-package path aliases
    },
  },
});
```

---

## Test File Structure

All test files live in `packages/napcat-test/`:

```
napcat-test/
  schema.test.ts
  rpc.test.ts
  imageSize.test.ts
  groupTodoTransformer.test.ts
  sha1Stream.test.ts
  vitest.config.ts
```

Each `.test.ts` imports from the package under test via `@/` path aliases.

---

## Test Pattern: describe / it / expect

Standard vitest BDD style:

```ts
// packages/napcat-test/schema.test.ts
import { describe, expect, test } from 'vitest';
import { TypeCompiler } from '@sinclair/typebox/compiler';

describe('NapCat Schemas Compilation', () => {
  test('should compile OB11MessageDataSchema without duplicate id error', () => {
    expect(() => TypeCompiler.Compile(OB11MessageDataSchema)).not.toThrow();
  });
});
```

---

## Schema & Config Validation Tests

Schema tests use `@sinclair/typebox` for JSON Schema compilation and validation:

```ts
// Compilation check
expect(() => TypeCompiler.Compile(MySchema)).not.toThrow();

// Coercion + validation
let data: unknown = structuredClone(payload);
data = Value.Parse(MySchema, data);
const compiler = TypeCompiler.Compile(MySchema);
expect(compiler.Check(data)).toBe(true);

// Config defaults
expect(loaded.network.httpServers[0]?.host).toBe('127.0.0.1');
```

---

## Mock Pattern: vi.fn()

```ts
// packages/napcat-test/rpc.test.ts
import { vi } from 'vitest';

const callback = vi.fn(async (x: number) => x * 3);
const result = await proxy.asyncWithCallback(callback);

expect(callback).toHaveBeenCalledWith(5);
expect(result).toBe(115);
```

---

## Unit Test Pattern (Pure Functions)

For modules like image-size:

```ts
// packages/napcat-test/imageSize.test.ts
import { describe, it, expect } from 'vitest';
import { detectImageTypeFromBuffer, ImageType } from '@/napcat-image-size/src';

describe('detectImageTypeFromBuffer', () => {
  it('should detect PNG image type', () => {
    expect(detectImageTypeFromBuffer(testBuffers.png)).toBe(ImageType.PNG);
  });

  it('should return UNKNOWN for empty buffer', () => {
    expect(detectImageTypeFromBuffer(testBuffers.empty)).toBe(ImageType.UNKNOWN);
  });
});
```

---

## Integration Test Pattern (RPC)

For multi-layer RPC tests that test proxy ↔ server communication:

```ts
const { client, server } = createRpcPair({
  counter: 0,
  increment() { return ++this.counter; },
});

expect(await client.increment()).toBe(1);
expect(server.counter).toBe(2);
```

---

## Running Tests

```bash
pnpm test        # runs vitest
pnpm test:ui     # vitest UI mode
```

No coverage thresholds configured currently.
