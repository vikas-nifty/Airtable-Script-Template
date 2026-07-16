# Base Schema

Run this in the target Airtable base before planning a new goal or after schema changes:

```javascript
const schema = base.tables.map(table => ({
  name: table.name,
  id: table.id,
  fields: table.fields.map(field => ({
    name: field.name,
    id: field.id,
    type: field.type
  })),
  views: table.views.map(view => ({
    name: view.name,
    id: view.id,
    type: view.type
  }))
}));

console.log(JSON.stringify(schema, null, 2));
```

Paste the latest output below. Treat it as authoritative. If a required table,
view, field, type, or relationship is missing, update Airtable and refresh this
file before planning dependencies or writing code.
