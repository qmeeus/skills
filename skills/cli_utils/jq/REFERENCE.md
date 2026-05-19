# jq Advanced Reference

## Nested access

```bash
# Chaining
jq '.a.b.c' file.json

# Optional (no error if null)
jq '.a?.b?.c' file.json

# Array indexing
jq '.items[0]' file.json
jq '.items[-1]' file.json       # last
jq '.items[0:3]' file.json      # slice
```

## Conditional logic

```bash
# if/then/else
jq 'if .age >= 18 then "adult" else "minor" end' file.json

# select with multiple conditions
jq '.[] | select(.age > 18 and .active == true)' file.json

# OR condition
jq '.[] | select(.status == "error" or .status == "fail")' file.json
```

## Object / array construction

```bash
# Rename keys
jq '{full_name: .name, age_years: .age}' file.json

# Merge objects
jq '.user + .metadata' file.json

# Add computed fields
jq '[.[] | {name, total: (.price * .quantity)}]' file.json

# Group by key
jq 'group_by(.category) | map({key: .[0].category, items: .})' file.json
```

## Built-in functions

```bash
# Unique values
jq '[.[] | .category] | unique' file.json

# Sort
jq 'sort_by(.name)' file.json
jq 'sort_by(.price) | reverse' file.json

# Min / max
jq '[.[] | .price] | min' file.json
jq '[.[] | .price] | max' file.json

# Map
jq 'map(select(.active))' file.json
jq 'map({name, upper: .name | ascii_upcase})' file.json

# Keys of an object
jq 'keys' file.json

# Check has key
jq 'has("name")' file.json
```

## String interpolation

```bash
jq '"Hello \(.name), you are \(.age)"' file.json

# Split / join
jq '[.[] | .tags | split(",")] | add | unique' file.json
jq '[.[] | .name] | join(", ")' file.json
```

## Working with env vars and arguments

```bash
# Shell variable
jq --arg env "$NODE_ENV" '.environments[$env]' file.json

# JSON argument
jq --argjson limit 10 '.[: $limit]' file.json

# From file
jq --arg key "$(cat key.txt)" '.[$key]' file.json
```

## Streaming large files

```bash
# One object at a time (streaming)
jq -n --stream 'fromstream(1|truncate_stream(inputs))' huge.json

# Extract specific keys only
jq -c '{name, email}' huge.json
```

## YAML / TOML interop

When data comes in YAML or TOML, convert to JSON first then pipe to jq:

```bash
yq eval '.' file.yaml | jq '.key'
tomlq '.' file.toml | jq '.key'
```

## Error handling

```bash
# Try/catch
jq 'try .key catch "error"' file.json

# Suppress errors, output null for missing
jq '.key // "default"' file.json
```
