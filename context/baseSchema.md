# Base Schema

The user must run this in the exact target Airtable base before each new script
and after every relevant schema change. Replace the previous captured output;
do not append competing snapshots.

```javascript
const compact = value => String(value).replace(/\s+/g, ' ').trim();
const dependencyKey = key => /(table|field).*ids?$/i.test(key);

function splitOptions(value, dependencies, path = '') {
  if (Array.isArray(value)) {
    return value.map((item, index) => splitOptions(item, dependencies, `${path}[${index}]`));
  }
  if (!value || typeof value !== 'object') return value;

  const output = {};
  for (const [key, item] of Object.entries(value)) {
    const itemPath = path ? `${path}.${key}` : key;
    if (dependencyKey(key) && (typeof item === 'string' || Array.isArray(item))) {
      dependencies.push(`${itemPath}=${JSON.stringify(item)}`);
      continue;
    }
    output[key] = splitOptions(item, dependencies, itemPath);
  }
  return output;
}

const lines = [];
for (const table of base.tables) {
  lines.push(`TABLE ${compact(table.name)} [${table.id}]`);

  for (const view of table.views) {
    lines.push(`  VIEW ${compact(view.name)} [${view.id}]`);
  }

  for (const field of table.fields) {
    lines.push(`  FIELD ${compact(field.name)} (${field.type}) [${field.id}]`);
    if (field.description) lines.push(`    DESCRIPTION ${compact(field.description)}`);

    if (field.options) {
      const dependencies = [];
      const options = splitOptions(field.options, dependencies);
      const formulas = [];
      const findFormulas = value => {
        if (Array.isArray(value)) return value.forEach(findFormulas);
        if (!value || typeof value !== 'object') return;
        for (const [key, item] of Object.entries(value)) {
          if (key === 'formula' && typeof item === 'string') formulas.push(item);
          else findFormulas(item);
        }
      };
      findFormulas(field.options);

      const optionText = JSON.stringify(options);
      if (optionText !== '{}') lines.push(`    OPTIONS ${optionText}`);

      for (const formula of formulas) {
        const fields = [...formula.matchAll(/\{([^}]+)\}/g)].map(match => match[1]);
        if (fields.length) dependencies.push(`formulaFields=${JSON.stringify([...new Set(fields)])}`);
      }
      if (dependencies.length) lines.push(`    DEPENDENCIES ${dependencies.join('; ')}`);
    }
  }
}

console.log(lines.join('\n'));
```

Paste the latest output below. Dependencies exposed by Airtable field options
are listed explicitly; formula dependencies are inferred from `{Field Name}`
references. Airtable may not expose every computed-field dependency through the
Scripting API, so verify any missing formula, lookup, rollup, or last-modified
configuration in the Airtable UI. If anything is missing or stale, update
Airtable and refresh this file before planning dependencies or writing code.

## Latest captured output
