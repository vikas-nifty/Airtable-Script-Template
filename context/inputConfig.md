# Input Config

The user must refresh this after configuring or changing an Airtable script
action. Every planned table, view, field, record ID, and non-secret runtime value
must be captured. Label multiple script actions separately because every action
has its own `input.config()`. Actions may repeat the same keys and values when
they share dependencies.

Run inside each script action:

```javascript
const inputConfig = input.config();
const secretNames = ['Configured Secret Name']; // Replace with exact secret names.
const maskSecret = value => {
  const secret = String(value ?? '');
  if (secret.length <= 6) return '*'.repeat(secret.length);
  return `${secret.slice(0, 3)}${'*'.repeat(secret.length - 6)}${secret.slice(-3)}`;
};
const secrets = Object.fromEntries(secretNames.map(name => [
  name,
  maskSecret(input.secret(name))
]));

console.log(JSON.stringify({ ...inputConfig, secrets }, null, 2));
```

The output contains masked secret previews only: the first three and last three
characters are visible and the middle is replaced with asterisks. Secrets six
characters or shorter are fully masked. This capture helper is the only approved
place to output a masked secret preview. Never paste an unmasked secret.

Paste the latest input output below, replacing the previous snapshot. It becomes
authoritative only after it matches the dependency map in `output/plan.md`. Do
not write code while a dependency is missing or ambiguous.

Scripts do not share variables or in-memory context.

## Latest captured output
