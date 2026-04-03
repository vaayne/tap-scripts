# AGENTS.md

Site scripts for [tap](https://github.com/vaayne/tap). Each push to `main` auto-syncs all scripts to [tap.vaayne.com](https://tap.vaayne.com) via GitHub Actions.

## Structure

```
sites/
  {site}/
    {action}.js       # One script per action
.github/workflows/
  sync.yml            # Auto-sync to D1 on push
```

## Adding a New Script

### 1. Create the file

```
sites/{site}/{action}.js
```

Use lowercase, hyphen-separated names. One `.js` file per action.

### 2. Add the `@meta` block

Every script must start with a `/* @meta ... */` JSON block:

```js
/* @meta
{
  "name": "{site}/{action}",
  "description": "Short description of what this script does",
  "domain": "www.example.com",
  "args": {
    "query": {"required": true, "description": "Search query"},
    "count": {"required": false, "description": "Number of results (default 10)"}
  },
  "readOnly": true,
  "example": "tap site {site}/{action} query=\"hello world\""
}
*/
```

**Required fields:**
- `name` — `{site}/{action}`, must match the file path
- `description` — what the script does
- `domain` — target website domain

**Optional fields:**
- `args` — parameter definitions with `required` and `description`
- `readOnly` — `true` (default) for read-only scripts, `false` for write actions (e.g., fork, post)
- `capabilities` — `["network"]` if using fetch directly (most scripts need this)
- `example` — example CLI invocation

### 3. Write the script body

```js
async function(args) {
  // Validate required args
  if (!args.query) return {error: 'query is required'};

  // Fetch data (fetch + DOMParser are available)
  const resp = await fetch('https://example.com/api?q=' + encodeURIComponent(args.query));
  if (!resp.ok) return {error: 'HTTP ' + resp.status};

  // Parse and return structured data
  const data = await resp.json();
  return {
    count: data.items.length,
    items: data.items.map(item => ({
      title: item.title,
      url: item.url,
    })),
  };
}
```

**Available APIs:**
- `fetch()` — HTTP requests (with `credentials: 'include'` for cookie-authenticated sites)
- `DOMParser` — parse HTML strings into DOM for querying
- `args` — user-supplied parameters as key-value pairs

**Conventions:**
- Return structured JSON objects, not raw HTML
- Return `{error: 'message'}` on failures
- Use `count` / `max` params to limit results
- Default to sensible limits (e.g., 10-20 results)
- Always validate required args before fetching

### 4. Test locally

```bash
# Run with tap CLI
tap site {site}/{action} query="test"

# Or use local override for development
mkdir -p ~/.config/tap/sites/{site}/
cp sites/{site}/{action}.js ~/.config/tap/sites/{site}/
tap site {site}/{action} query="test"
```

### 5. Commit and push

```bash
git add sites/{site}/{action}.js
git commit -m "✨ feat: add {site}/{action} script"
git push
```

The sync workflow runs automatically and pushes the script to production.

## Modifying Existing Scripts

Edit the `.js` file, test locally, commit and push. The workflow syncs the full catalog on every push.

## Code Style

- Plain JavaScript (no TypeScript, no modules, no build step)
- Single async function per file
- Return structured JSON, never raw strings
- Handle errors gracefully with `{error: 'message'}`
- Keep scripts focused — one action per file

## Commits

Emoji-prefixed Conventional Commits: `✨ feat:`, `🐛 fix:`, `♻️ refactor:`, `📝 docs:`.
