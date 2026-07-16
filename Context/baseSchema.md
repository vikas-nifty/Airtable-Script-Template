# Base Schema

Use this file to capture a fresh compact schema for the Airtable base before planning a new script. Replace the previous captured output when the base changes or a new goal begins.

Command to output a compact schema:

```javascript
const schema = base.tables.map(table => ({
  name: table.name,
  id: table.id,
  fields: table.fields.map(field => ({
    name: field.name,
    type: field.type,
    id: field.id
  }))
}));

console.log(JSON.stringify(schema, null, 2));
```

Paste the latest output below. This file is considered authoritative. The schema must be verified before creating the dependency map or writing code. If a required table or field is missing, ask the user to create it and refresh this file first.
