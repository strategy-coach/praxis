## Prompt Caching: Don’t Pay for the Same Prompt Again and Again

When we build AI agents or workflows, we often repeat the same instructions in every request. Examples include:

* Tool definitions (what actions are available)
* System rules (how the model should behave)
* Stable instructions or memory that rarely change

Without caching, the model is charged for these repeated parts every single time. This increases costs, slows down responses, and can cause problems with token rate limits.

**Prompt caching** fixes this. Instead of sending the same “prefix” again and again, you store it once in the model. After that, the model only processes the new, changing input.

The pattern is simple:

1. Put tools, system rules, and stable instructions at the start.
2. Mark the end of the static block with a cache breakpoint.
3. After the breakpoint, only send the dynamic input (what changes each step).

### Benefits

* **Much lower cost**: because most of the repeated input is charged only once.
* **Faster responses**: less text for the model to re-process.
* **Stable accuracy**: results stay the same, since only the prefix handling changes.
* **Better rate-limit control**: cached tokens don’t count toward some input token limits.

### How the Costs Work

* **Cache write (first time):** a little more expensive than normal (about 1.25× cost for a 5-minute cache, or 2× for a 1-hour cache).
* **Cache read (next times):** very cheap (about 0.1× the normal input cost).

This means the first call costs more, but every call after that is much cheaper.

### How to Use Prompt Caching

1. Find the static parts of your prompt (tools, system rules, stable memory).
2. Put them at the beginning.
3. Add a cache breakpoint after them.
4. Put all the dynamic input after the breakpoint.
5. Track cache usage with metrics (e.g., tokens written vs. tokens read).
6. Set the cache time-to-live (TTL): shorter for frequently changing instructions, longer for stable ones.

### Things to Watch Out For

* If your cached instructions change, you need to refresh the cache.
* If TTL is too short, you save less. If it’s too long, you may reuse outdated instructions.

### Why It Matters

Prompt caching is a **fundamental pattern** in AI interactions. Whenever you have agents or workflows that repeat instructions, caching can save a lot of money and make the system faster.

In short: **don’t pay again for what hasn’t changed.** Cache it once, and reuse it.

### Visual Flow

```
[ Static Prefix ] --> [ Cache Breakpoint ] --> [ Dynamic Input ]
      |                      |                        |
      |                      |                        |
   (Tools, Rules,     (Cache written once)     (Only new content
   Stable Memory)                              sent each time)

Result: Lower cost, faster responses, same quality
```

## 1) Provider-agnostic pseudocode (most basic)

```txt
# 1) Build the static prefix once (cacheable)
TOOLS:
  - name: click, description: "Click a selector", input: { selector: string }
  - name: type_text, description: "Type into a selector", input: { selector: string, text: string }

SYSTEM RULES:
  - "You are an automation agent. Use tools when possible."
  - "Be concise. Use JSON outputs."

STABLE INSTRUCTIONS / MEMORY:
  - "<team conventions, safety rules, canonical few-shots, DOM policy, etc.>"

# ---- CACHE BREAKPOINT (everything above is cacheable) ----

# 2) Dynamic input (changes each step)
USER INPUT:
  - "Open the calendar and pick Aug-13."
```

Implementation idea: send the block above the breakpoint once and mark it as cached (using your provider’s cache feature). On later calls, send **only** the dynamic input and reference the cached prefix as instructed by your provider.

## 2) TypeScript helper (SDK-agnostic)

This shows a tiny wrapper that:

* Computes a stable **cache key** from the static prefix (e.g., a hash)
* Sends the prefix once (cache write) and reuses it (cache read)
* Keeps dynamic messages separate

```ts
type Tool = {
  name: string;
  description: string;
  input_schema: Record<string, unknown>;
};

type CachedPrefix = {
  tools: Tool[];
  system: string[];           // rules, behavior
  stableInstructions: string; // long, rarely-changing block
};

// Simple hash for demo only; replace with a real hash (e.g., SHA-256).
function simpleHash(s: string): string {
  let h = 0;
  for (let i = 0; i < s.length; i++) h = ((h << 5) - h + s.charCodeAt(i)) | 0;
  return String(h >>> 0);
}

function buildPrefixString(p: CachedPrefix): string {
  return JSON.stringify({
    tools: p.tools,
    system: p.system,
    stable: p.stableInstructions,
  });
}

type CacheStore = Map<string, { expiresAt: number }>;
const inMemoryCache: CacheStore = new Map();

function cacheWriteIfNeeded(prefix: CachedPrefix, ttlMs: number): { key: string; wrote: boolean } {
  const key = simpleHash(buildPrefixString(prefix));
  const entry = inMemoryCache.get(key);
  const now = Date.now();
  if (!entry || entry.expiresAt <= now) {
    // In a real implementation, call your LLM provider's "cache write" API here.
    inMemoryCache.set(key, { expiresAt: now + ttlMs });
    return { key, wrote: true };
  }
  return { key, wrote: false };
}

type LlmClient = {
  // You will adapt this to your provider. Some take a cache key / handle, some take a special flag.
  createMessage(input: {
    model: string;
    max_tokens: number;
    cache_key?: string;      // provider-specific
    use_cached_prefix?: boolean;
    dynamic_messages: Array<{ role: "user" | "assistant" | "tool"; content: string }>;
  }): Promise<{ text: string; tokens_used?: number }>;
};

async function runStep(
  client: LlmClient,
  prefix: CachedPrefix,
  dynamicUserText: string,
  ttlMs = 5 * 60_000 // 5 minutes
) {
  const { key, wrote } = cacheWriteIfNeeded(prefix, ttlMs);

  // NOTE: On providers with explicit caching, you may need to:
  // (a) send the prefix with a "cache_control" tag on first call,
  // (b) then on subsequent calls, omit the prefix and pass the cache handle/key.

  const res = await client.createMessage({
    model: "my-llm-model",
    max_tokens: 512,
    cache_key: key,            // or provider-specific field
    use_cached_prefix: true,   // your adapter toggles this into the provider’s mechanism
    dynamic_messages: [
      { role: "user", content: dynamicUserText }
    ]
  });

  // Basic logging so juniors can see savings in action:
  console.log(
    wrote ? "[cache write]" : "[cache read]",
    "cache_key=", key,
    "response_tokens=", res.tokens_used ?? "n/a"
  );

  return res.text;
}

// Example usage:
const prefix: CachedPrefix = {
  tools: [
    { name: "click", description: "Click a selector", input_schema: { selector: "string" } as any },
    { name: "type_text", description: "Type into a selector", input_schema: { selector: "string", text: "string" } as any }
  ],
  system: [
    "You are an automation agent. Prefer tools.",
    "Return JSON."
  ],
  stableInstructions: "<long rules, few-shots, DOM policy, etc.>"
};

// First call likely writes cache; later calls reuse it:
await runStep(myClient, prefix, "Open the calendar and pick Aug-13.");
await runStep(myClient, prefix, "Create a new event for tomorrow at 10am.");
```

> Notes
>
> * Replace `myClient.createMessage` with your provider’s SDK.
> * If your provider supports explicit cache markers, your adapter can:
>
>   * **Write:** send the `tools/system/stableInstructions` once with a cache flag.
>   * **Read:** omit the prefix and send `cache_key` + dynamic messages only.

## 3) HTTP-style JSON (easy to port to `fetch()`)

Two-step pattern: **write** the cache once, then **read** it many times.

### 3a. Cache write (first call)

```http
POST /llm/messages
Content-Type: application/json

{
  "model": "my-llm-model",
  "max_tokens": 512,

  "tools": [
    { "name": "click", "description": "Click a selector", "input_schema": { "type": "object", "properties": { "selector": { "type": "string" } }, "required": ["selector"] } },
    { "name": "type_text", "description": "Type into a selector", "input_schema": { "type": "object", "properties": { "selector": { "type": "string" }, "text": { "type": "string" } }, "required": ["selector","text"] } }
  ],

  "system": [
    { "type": "text", "text": "You are an automation agent. Prefer tools." },
    { "type": "text", "text": "<long stable rules / few-shots / policies>", "cache_control": { "type": "ephemeral" } }
  ],

  "messages": [
    { "role": "user", "content": "Start session." }
  ]
}
```

* Everything before the `cache_control` marker is the **static prefix**.
* This call “writes” the cache. Providers differ in exact syntax; adapt the flag name if needed.

### 3b. Cache read (subsequent calls)

```http
POST /llm/messages
Content-Type: application/json

{
  "model": "my-llm-model",
  "max_tokens": 512,

  "use_cached_prefix": true,    // provider-specific flag or pass a cache handle returned earlier

  "messages": [
    { "role": "user", "content": "Open the calendar and pick Aug-13." }
  ]
}
```

* Do **not** resend the prefix.
* Just tell the provider to reuse the cached prefix and send the **dynamic** input.

## Tiny “metrics” checklist (so savings are visible)

* Log cache mode per request: `"cache write"` vs. `"cache read"`.
* Track:

  * **Input tokens written** (first call) vs. **input tokens read** (later calls).
  * **Latency p95** with and without caching.
  * **Cost per step/test** trend after enabling caching.
* Add a simple “cache hit rate” counter during load tests.

### Quick recap

* Put all repeated instructions at the **front**.
* Mark a **cache breakpoint** once.
* After that, send **only the changing input**.
* Measure to confirm you’re saving money and time.
