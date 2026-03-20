# HIGH: Injection Vulnerabilities

## SQL Injection - String Interpolation in Queries

| Pattern | Language | Issue |
|---------|----------|-------|
| `query\(.*\$\{` | JS/TS | Template literal in SQL query |
| `query\(.*\+\s*(req\|params\|body\|query\|input\|args\|ctx)` | JS/TS | String concatenation with user input in SQL |
| `execute\(f["']` | Python | f-string in SQL execute |
| `execute\(["'].*%s.*%\s` | Python | Unsafe % formatting in SQL (tuple args are safe, flag only `% variable`) |
| `cursor\.execute\(.*\.format\(` | Python | str.format() in SQL |
| `\.Raw\(.*fmt\.Sprintf` | Go | Sprintf in raw SQL query |
| `\.Exec\(.*\+\s` | Go | String concat in Exec SQL call |
| `\.Query\(.*\+\s` | Go | String concat in Query SQL call |
| `\.Where\(.*fmt\.Sprintf` | Go (GORM) | Sprintf in GORM Where |

## XSS - Unescaped User Content in HTML

| Pattern | Framework | Issue |
|---------|-----------|-------|
| `dangerouslySetInnerHTML` | React | Direct HTML injection. Check if source is sanitized (DOMPurify). If user input without sanitization: HIGH |
| `innerHTML\s*=` | Vanilla JS | DOM-based XSS |
| `\.html\(` | jQuery | HTML injection (verify source is user-controlled) |
| `v-html=` | Vue | Unescaped HTML binding |
| `\|safe` | Django/Jinja2 | Auto-escaping disabled |
| `\{\{\{.*\}\}\}` | Handlebars/Mustache | Unescaped output |
| `<%[-]?=` | EJS | Unescaped output (verify source is sanitized) |

## Command Injection

| Pattern | Language | Issue |
|---------|----------|-------|
| `exec\(.*\$\{` | JS/TS | Template literal in shell exec |
| `exec\(.*\+` | JS/TS | String concat in shell exec |
| `child_process.*exec\(` | Node.js | Shell exec (`exec` runs via shell; use `execFile` or `spawn` instead) |
| `os\.system\(` | Python | os.system always runs via shell (use subprocess with shell=False) |
| `subprocess\.(call\|run\|Popen)\(.*shell=True` | Python | Shell=True enables injection |
| `exec\.Command\(.*\+` | Go | String concat in command args |

## Path Traversal

| Pattern | Language | Issue |
|---------|----------|-------|
| `path\.join\(.*req\.(params\|query\|body)` | Node.js | User input in file path without validation |
| `os\.path\.join\(.*request\.(GET\|POST\|args\|form)` | Python | User input in file path |
| `filepath\.Join\(.*r\.(URL\|Form\|PathValue)` | Go | User input in file path |
| `sendFile\(.*req\.` | Express | Serving file based on user input |
| `send_file\(.*request\.` | Flask | Serving file based on user input |

## SSRF (Server-Side Request Forgery)

AI generates code that fetches user-provided URLs without validation. Attacker can read internal services, cloud metadata (169.254.169.254), or scan your network.

| Pattern | Language | Issue |
|---------|----------|-------|
| `fetch\(.*req\.(params\|query\|body)` | JS/TS | User-controlled URL in fetch |
| `axios\(.*req\.(params\|query\|body)` | JS/TS | User-controlled URL in axios |
| `axios\.get\(.*req\.` | JS/TS | User-controlled URL in axios.get |
| `requests\.(get\|post)\(.*request\.(GET\|POST\|args\|form\|json)` | Python | User-controlled URL in requests |
| `urllib\.request\.urlopen\(.*request\.` | Python | User-controlled URL in urllib |
| `http\.Get\(.*r\.(URL\|Form\|PathValue)` | Go | User-controlled URL in http.Get |

If any found: verify the URL is validated against an allowlist of domains. Blocklisting `localhost`/`127.0.0.1` alone is insufficient (bypassed with DNS rebinding, IPv6, decimal IPs).

## Unsafe Deserialization

| Pattern | Language | Issue |
|---------|----------|-------|
| `pickle\.load\(` | Python | Executes arbitrary code during deserialization. Never use on untrusted data |
| `yaml\.load\(` without `Loader=SafeLoader` | Python | Can execute arbitrary Python. Use `yaml.safe_load()` |
| `eval\(.*req\.(body\|query\|params)` | JS/TS | Executes arbitrary code from user input |
| `eval\(.*request\.(GET\|POST\|json\|form)` | Python | Executes arbitrary code from user input |
| `new Function\(.*req\.` | JS/TS | Dynamic code execution from user input |
| `unserialize\(` | PHP | Object injection via deserialization |

## Open Redirect

Attacker uses your domain for phishing: `yourapp.com/redirect?url=evil.com`.

| Pattern | Language | Issue |
|---------|----------|-------|
| `res\.redirect\(.*req\.(params\|query\|body)` | Express | User-controlled redirect URL |
| `redirect\(.*request\.(GET\|POST\|args)` | Python | User-controlled redirect URL |
| `http\.Redirect\(.*r\.(URL\|Form)` | Go | User-controlled redirect URL |
| `window\.location\s*=\s*` | JS (client) | Client-side redirect (verify source) |
