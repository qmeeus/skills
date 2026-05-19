---
name: jq
description: Query, filter, transform, and validate JSON using the jq CLI. Use when working with JSON files, API responses, config files, or any JSON data where parsing/filtering/transforming is needed.
---

# jq

`jq` is installed at `/usr/bin/jq` (v1.8.1). Use it for all JSON processing — never write custom JS/Python to parse JSON when a jq one-liner suffices.

## Quick start

```bash
# Extract a key
jq '.name' package.json

# Pipe input
echo '{"a":1}' | jq '.a'

# Pretty-print
curl -s https://api.example.com/data | jq '.'
```

## Common workflows

### Extract value from JSON file

```bash
jq '.dependencies.react' package.json
```

### Filter array

```bash
jq '.[] | select(.status == "ok")' data.json
```

### Build new object (remap)

```bash
jq '{name: .user.name, email: .user.email}' data.json
```

### Count items

```bash
jq 'length' data.json
jq '[.[] | select(.active)] | length' data.json
```

### Format / validate

```bash
# Validate (exit code 0/1)
jq empty file.json

# Pretty-print with color
jq . file.json

# Compact output (remove whitespace)
jq -c '.' file.json
```

### Read raw strings (not JSON-encoded)

```bash
jq -r '.name' file.json
```

### Slurp multiple objects into array

```bash
jq -s '.' file1.json file2.json
```

## Options reference

| Flag | Purpose |
|---|---|
| `-r` | Raw output (unquoted strings) |
| `-c` | Compact (one line per output) |
| `-s` | Slurp (read all inputs into array) |
| `-f file` | Read filter from file |
| `--arg k v` | Pass string variable `$k` |
| `--argjson k v` | Pass JSON variable `$k` |

See [REFERENCE.md](REFERENCE.md) for advanced patterns.
