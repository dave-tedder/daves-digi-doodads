# Node Dev Server Gotchas in Claude Code

## concurrently Doesn't Work in Claude Code's Background Process Sandbox

The `concurrently` npm package (used for running Express + Vite simultaneously via `npm run dev:full`) exits child processes prematurely when run in Claude Code's background command sandbox. The parent process finishes but the children get cleaned up.

### Workaround

Start each server as a separate `nohup` process:

```
nohup node server/index.js > /tmp/project-server.log 2>&1 & echo "PID: $!"
nohup npx vite > /tmp/project-vite.log 2>&1 & echo "PID: $!"
```

Check logs with `cat /tmp/project-server.log` and verify with `curl http://localhost:PORT/api/health`.

### After Code Changes

Express doesn't hot-reload. After editing server files, kill and restart:

```
pkill -f "node server/index.js"; sleep 1; nohup node server/index.js > /tmp/project-server.log 2>&1 &
```

Vite handles frontend hot-reload automatically.

## `require()` Loads ESM Modules Natively in Node 22+

Starting in Node 22 (and confirmed in 25.6), `require()` can load ES modules directly, as long as the target module has no top-level `await` and the import is synchronous. This means a single shared helper file with `export` syntax can be imported from both CommonJS server code and ESM test/frontend code without dual packaging, a build step, or wrapper.

### Example

```js
// server/lib/foo.js — ES module
export const ALLOWED = new Set(['a', 'b']);
export function bar(x) { return ALLOWED.has(x); }
```

```js
// server/index.js — CommonJS, but require() works fine
const { bar, ALLOWED } = require('./lib/foo');
```

```js
// tests/foo.test.js — ESM, same file
import { bar, ALLOWED } from '../server/lib/foo.js';
```

All three load the same module. Node resolves the ESM, exposes the named exports on the require result, and both sides see the same symbols.

### When to use this

If you're tempted to rewrite a shared helper as CommonJS just because the rest of the server uses `require()`, test the direct import first:

```bash
node -e "const m = require('./server/lib/your-file.js'); console.log(Object.keys(m));"
```

If it prints the export names cleanly, ship the ESM file. No scope bloat, no dual implementation, no `.cjs`/`.mjs` gymnastics.

### The cosmetic warning

Running a test file via `node --test tests/*.test.js` against an ESM test file without a `"type": "module"` field in `package.json` produces a `MODULE_TYPELESS_PACKAGE_JSON` warning. The tests still run correctly — Node reparses as ESM when it detects `import` syntax. The fix (adding `"type": "module"` to package.json) forces you to rewrite any CommonJS files in the project, so don't bother unless you're converting the whole project.
