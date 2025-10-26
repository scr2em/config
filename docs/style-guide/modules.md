### Modules

#### File Organization

In general, files that change together should be located close to each other. In
Breaking a single file into multiple files should be avoided unless absolutely
necessary.

Specifics around file structure depends on a multitude of factors:

- Framework conventions
- Project size
- Team size

Strive to keep the file structure as flat as possible.

#### Module Exports

Framework and other tool conventions sometimes require default exports, but
prefer named exports in all other cases.

```tsx
// ✅ Good
export function add(a: number, b: number) {
	return a + b
}

export function subtract(a: number, b: number) {
	return a - b
}

// ❌ Avoid
export default function add(a: number, b: number) {
	return a + b
}
```

#### Barrel Files

Do **not** use barrel files. If you don't know what they are, good. If you do
and you like them, it's probably because you haven't experienced their issues
just yet, but you will. Just avoid them.

#### Pure Modules

In general, strive to keep modules pure (read more about this in
[Pure Modules](https://kentcdodds.com/blog/pure-modules)). This will make your
application start faster and be easier to understand and test.

```tsx
// ✅ Good
let serverData
export function init(a: number, b: number) {
	const el = document.getElementById('server-data')
	const json = el.textContent
	serverData = JSON.parse(json)
}

export function getServerData() {
	if (!serverData) throw new Error('Server data not initialized')
	return serverData
}

// ❌ Avoid
let serverData
const el = document.getElementById('server-data')
const json = el.textContent
export const serverData = JSON.parse(json)
```

> **Note**: In practice, you can't avoid some modules having side-effects (you
> gotta kick off the app somewhere), but most modules should be pure.

#### Import Conventions

Import order has semantic meaning (modules are executed in the order they're
imported), but if you keep most modules pure, then order shouldn't matter. For
this reason, having your imports grouped can make things a bit easier to read.

```ts
// Group imports in this order:
import 'node:fs' // Built-in
import 'match-sorter' // external packages
import '#app/components' // Internal absolute imports
import '../other-folder' // Internal relative imports
import './local-file' // Local imports
```

#### Type Imports

Each module imported should have a single import statement:

```tsx
// ✅ Good
import { type MatchSorterOptions, matchSorter } from 'match-sorter'

// ❌ Avoid
import { type MatchSorterOptions } from 'match-sorter'
import { matchSorter } from 'match-sorter'
```

#### Import Location

All static imports are executed at the top of the file so they should appear
there as well to avoid confusion.

```tsx
// ✅ Good
import { matchSorter } from 'match-sorter'

function doStuff() {
	// ...
}

// ❌ Avoid
function doStuff() {
	// ...
}

import { matchSorter } from 'match-sorter'
```

#### Export Location

All exports should be inline with the function/type/etc they are exporting. This
avoids duplication of the export identifier and having to keep it updated when
changing the name of the exported thing.

```tsx
// ✅ Good
export function add(a: number, b: number) {
	return a + b
}

// ❌ Avoid
function add(a: number, b: number) {
	return a + b
}
export { add }
```

#### Module Type

Use ECMAScript modules for everything. The age of CommonJS is over.

✅ Good **package.json**:

```json
{
	"type": "module"
}
```

Use **exports** field in **package.json** to explicitly declare module entry
points.

✅ Good **package.json**:

```json
{
	"exports": {
		"./utils": "./src/utils.js"
	}
}
```

#### Import Aliases

Use import aliases to avoid long relative paths. Use the standard `imports`
config field in **package.json** to declare import aliases.

✅ Good **package.json**:

```json
{
	"imports": {
		"#app/*": "./app/*",
		"#tests/*": "./tests/*"
	}
}
```

```tsx
import { add } from '#app/utils/math.ts'
```

> **Note**: Latest versions of TypeScript support this syntax natively.

#### Include file extensions

The ECMAScript module spec requires file extensions to be included in import
paths. Even though TypeScript doesn't require it, always include the file
extension in your imports. An exception to this is when importing a module which
has `exports` defined in its **package.json**.

```tsx
// ✅ Good
import { redirect } from 'react-router'
import { add } from './math.ts'

// ❌ Avoid
import { add } from './math'
```
