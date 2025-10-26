## JavaScript

This section will include TypeScript guidelines as well.

### Variables

#### References

Use `const` by default. Only use `let` when you need to reassign. Never use
`var`.

Remember that `const` does not mean "constant" in the sense of "unchangeable".
It means "constant reference". So if the value is an object, you can still
change the properties of the object.

#### Naming conventions

Use descriptive, clear names that explain the value's purpose. Avoid
single-letter names except in small loops or reducers where the value is obvious
from context.

```tsx
// ✅ Good
const workshopTitle = 'Web App Fundamentals'
const instructorName = 'Kent C. Dodds'
const isEnabled = true
const sum = numbers.reduce((total, n) => total + n, 0)
const names = people.map((p) => p.name)

// ❌ Avoid
const t = 'Web App Fundamentals'
const n = 'Kent C. Dodds'
const e = true
```

Follow [the naming cheatsheet](https://github.com/kettanaito/naming-cheatsheet)
by [Artem Zakharchenko](https://github.com/kettanaito) for more specifics on
naming conventions.

#### Constants

For truly constant values used across files, use uppercase with underscores:

```tsx
const BASE_URL = 'https://epicweb.dev'
const DEFAULT_PORT = 3000
```

### Objects

#### Literal syntax

Use object literal syntax for creating objects. Use property shorthand when the
property name matches the variable name.

```tsx
// ✅ Good
const name = 'Kent'
const age = 36
const person = { name, age }

// ❌ Avoid
const name = 'Kent'
const age = 36
const person = { name: name, age: age }
```

#### Computed property names

Use computed property names when creating objects with dynamic property names.

```tsx
// ✅ Good
const key = 'name'
const obj = {
	[key]: 'Kent',
}

// ❌ Avoid
const key = 'name'
const obj = {}
obj[key] = 'Kent'
```

#### Method shorthand

Use object method shorthand:

```tsx
// ✅ Good
const obj = {
	method() {
		// ...
	},
	async asyncMethod() {
		// ...
	},
}

// ❌ Avoid
const obj = {
	method: function () {
		// ...
	},
	asyncMethod: async function () {
		// ...
	},
}
```

> **Note**: Ordering of properties is not important (and not specified by the
> spec) and it's not a priority for this style guide either.

#### Accessors

Don't use them. When I do this:

```ts
console.log(person.name)
person.name = 'Bob'
```

All I expect to happen is to get the person's name and pass it to the `log`
function and to set the person's name to `'Bob'`.

Once you start using property accessors (getters and setters) then those
guarantees are off.

```ts
// ✅ Good
const person = {
	name: 'Hannah',
}

// ❌ Avoid
const person = {
	get name() {
		// haha! Now I can do something more than just return the name! 😈
		return this.name
	},
	set name(value) {
		// haha! Now I can do something more than just set the name! 😈
		this.name = value
	},
}
```

This violates the principle of least surprise.

### Arrays

#### Literal syntax

Use Array literal syntax for creating arrays.

```tsx
// ✅ Good
const items = [1, 2, 3]

// ❌ Avoid
const items = new Array(1, 2, 3)
```

#### Filtering falsey values

Use `.filter(Boolean)` to remove falsey values from an array.

```tsx
// ✅ Good
const items = [1, null, 2, undefined, 3]
const filteredItems = items.filter(Boolean)

// ❌ Avoid
const filteredItems = items.filter(
	(item) => item !== null && item !== undefined,
)
```

#### Array methods over loops

Use Array methods over loops when transforming arrays with pure functions. Use
`for` loops when imperative code is necessary. Never use `forEach` because it's
never more readable than a `for` loop and there's not situation where the
`forEach` callback function could be pure and useful. Prefer `for...of` over
`for` loops.

```tsx
// ✅ Good
const items = [1, 2, 3]
const doubledItems = items.map((n) => n * 2)

// ❌ Avoid
const doubledItems = []
for (const n of items) {
	doubledItems.push(n * 2)
}
```

```tsx
// ✅ Good
for (const n of items) {
	// ...
}

// ❌ Avoid
for (let i = 0; i < items.length; i++) {
	const n = items[i]
	// ...
}

// ❌ Avoid
items.forEach((n) => {
	// ...
})
```

```tsx
// ✅ Good
for (const [i, n] of items.entries()) {
	console.log(`${n} at index ${i}`)
}

// ❌ Avoid
for (const n of items) {
	const i = items.indexOf(n)
	console.log(`${n} at index ${i}`)
}
```

#### Favor simple chains over `.reduce`

Favor simple `.filter` and `.map` chains over complex `.reduce` callbacks unless
performance is an issue.

```tsx
// ✅ Good
const items = [1, 2, 3, 4, 5]
const doubledGreaterThanTwoItems = items.filter((n) => n > 2).map((n) => n * 2)

// ❌ Avoid
const doubledItems = items.reduce((acc, n) => {
	acc.push(n * 2)
	return acc
}, [])
```

#### Spread to copy

Prefer the spread operator to copy an array:

```tsx
// ✅ Good
const itemsCopy = [...items]
const combined = [...array1, ...array2]

// ❌ Avoid
const itemsCopy = items.slice()
const combined = array1.concat(array2)
```

#### Non-mutative array methods

Prefer non-mutative array methods like `toReversed()`, `toSorted()`, and
`toSpliced()` when available. Otherwise, create a new array. Unless performance
is an issue or the original array is not referenced (as in a chain of method
calls).

```tsx
// ✅ Good
const reversedItems = items.toReversed()
const mappedFilteredSortedItems = items
	.filter((n) => n > 2)
	.map((n) => n * 2)
	.sort((a, b) => a - b)

// ❌ Avoid
const reversedItems = items.reverse()
```

#### Use `with`

Use `with` to create a new object with some properties replaced.

```tsx
// ✅ Good
const people = [{ name: 'Kent' }, { name: 'Sarah' }]
const personIndex = 0
const peopleWithKentReplaced = people.with(personIndex, { name: 'John' })

// ❌ Avoid (mutative)
const peopleWithKentReplaced = [...people]
peopleWithKentReplaced[personIndex] = { name: 'John' }
```

#### TypeScript array generic

Prefer the Array generic syntax over brackets for TypeScript types:

```tsx
// ✅ Good
const items: Array<string> = []
function transform(numbers: Array<number>) {}

// ❌ Avoid
const items: string[] = []
function transform(numbers: number[]) {}
```

Learn more about the reasoning behind the Array generic syntax in the
[Array Types in TypeScript](https://tkdodo.eu/blog/array-types-in-type-script)
article by [Dominik Dorfmeister](https://github.com/tkdodo).

### Destructuring

#### Destructure objects and arrays

Use destructuring to make your code more terse.

```tsx
// ✅ Good
const { name, avatar, 𝕏: xHandle } = instructor
const [first, second] = items

// ❌ Avoid
const name = instructor.name
const avatar = instructor.avatar
const xHandle = instructor.𝕏
```

Destructuring multiple levels is fine when formatted properly by a formatter,
but can definitely get out of hand, so use your best judgement. As usual, try
both and choose the one you hate the least.

```tsx
// ✅ Good (nesting, but still readable)
const {
	name,
	avatar,
	𝕏: xHandle,
	address: [{ city, state, country }],
} = instructor

// ❌ Avoid (too much nesting)
const [
	{
		name,
		avatar,
		𝕏: xHandle,
		address: [
			{
				city: {
					latitude: firstCityLatitude,
					longitude: firstCityLongitude,
					label: firstCityLabel,
				},
				state: { label: firstStateLabel },
				country: { label: firstCountryLabel },
			},
		],
	},
] = instructor
```

### Strings

#### Interpolation

Prefer template literals over string concatenation.

```tsx
// ✅ Good
const name = 'Kent'
const greeting = `Hello ${name}`

// ❌ Avoid
const greeting = 'Hello ' + name
```

#### Multi-line strings

Use template literals for multi-line strings.

```tsx
// ✅ Good
const html = `
<div>
	<h1>Hello</h1>
</div>
`.trim()

// ❌ Avoid
const html = '<div>' + '\n' + '<h1>Hello</h1>' + '\n' + '</div>'
```

### Functions

#### Function declarations

Use function declarations over function expressions. Name your functions
descriptively.

This is important because it allows the function definition to be hoisted to the
top of the block, which means it's callable anywhere which frees your mind to
think about other things.

```tsx
// ✅ Good
function calculateTotal(items: Array<number>) {
	return items.reduce((sum, item) => sum + item, 0)
}

// ❌ Avoid
const calculateTotal = function (items: Array<number>) {
	return items.reduce((sum, item) => sum + item, 0)
}

const calculateTotal = (items: Array<number>) =>
	items.reduce((sum, item) => sum + item, 0)
```

#### Limit single-use functions

Limit creating single-use functions. By taking a large function and breaking it
down into many smaller functions, you reduce benefits of type inference and have
to define types for each function and make additional decisions about the number
and format of arguments. Instead, extract logic only when it needs to be reused
or when a portion of the logic is clearly part of a unique concern.

```tsx
// ✅ Good
function doStuff() {
	// thing 1
	// ...
	// thing 2
	// ...
	// thing 3
	// ...
	// thing N
}

// ❌ Avoid
function doThing1(param1: string, param2: number) {}
function doThing2(param1: boolean, param2: User) {}
function doThing3(param1: string, param2: Array<User>, param3: User) {}
function doThing4(param1: User) {}

function doStuff() {
	doThing1()
	// ...
	doThing2()
	// ...
	doThing3()
	// ...
	doThing4()
}
```

#### Default parameters

Prefer default parameters over short-circuiting.

```tsx
// ✅ Good
function createUser(name: string, role = 'user') {
	return { name, role }
}

// ❌ Avoid
function createUser(name: string, role: string) {
	role ??= 'user'
	return { name, role }
}
```

#### Early return

Return early to avoid deep nesting. Use guard clauses:

```tsx
// ✅ Good
function getMinResolutionValue(resolution: number | undefined) {
	if (!resolution) return undefined
	if (resolution <= 480) return MinResolution.noLessThan480p
	if (resolution <= 540) return MinResolution.noLessThan540p
	return MinResolution.noLessThan1080p
}

// ❌ Avoid
function getMinResolutionValue(resolution: number | undefined) {
	if (resolution) {
		if (resolution <= 480) {
			return MinResolution.noLessThan480p
		} else if (resolution <= 540) {
			return MinResolution.noLessThan540p
		} else {
			return MinResolution.noLessThan1080p
		}
	} else {
		return undefined
	}
}
```

#### Async/await

Prefer async/await over promise chains:

```tsx
// ✅ Good
async function fetchUserData(userId: string) {
	const user = await getUser(userId)
	const posts = await getUserPosts(user.id)
	return { user, posts }
}

// ✅ Fine, because wrapping in try/catch is annoying
function sendAnalytics(event: string) {
	return fetch('/api/analytics', {
		method: 'POST',
		body: JSON.stringify({ event }),
	}).catch(() => null)
}

// ❌ Avoid
function fetchUserData(userId: string) {
	return getUser(userId).then((user) => {
		return getUserPosts(user.id).then((posts) => ({ user, posts }))
	})
}

// ❌ Avoid
async function sendAnalytics(event: string) {
	try {
		return await fetch('/api/analytics', {
			method: 'POST',
			body: JSON.stringify({ event }),
		})
	} catch {
		// ignore
		return null
	}
}
```

#### Inline Callbacks

Anonymous inline callbacks should be arrow functions:

```tsx
// ✅ Good
const items = [1, 2, 3]
const doubledGreaterThanTwoItems = items.filter((n) => n > 2).map((n) => n * 2)

// ❌ Avoid
const items = [1, 2, 3]
const doubledGreaterThanTwoItems = items
	.filter(function (n) {
		return n > 2
	})
	.map(function (n) {
		return n * 2
	})
```

#### Arrow Parens

Arrow functions should include parentheses even with a single parameter:

<!-- prettier-ignore -->
```tsx
// ✅ Good
const items = [1, 2, 3]
const doubledGreaterThanTwoItems = items.filter((n) => n > 2).map((n) => n * 2)

// ❌ Avoid
const items = [1, 2, 3]
const doubledGreaterThanTwoItems = items.filter(n => n > 2).map(n => n * 2)
```

This makes it easier to add/remove parameters without having to futz around with
parentheses.

### Properties

#### Use dot-notation

When accessing properties on objects, use dot-notation unless you can't
syntactically (like if it's dynamic or uses special characters).

```tsx
const user = { name: 'Brittany', 'data-id': '123' }

// ✅ Good
const name = user.name
const id = user['data-id']
function getUserProperty(user: User, property: string) {
	return user[property]
}

// ❌ Avoid
const name = user['name']
```

### Comparison Operators & Equality

#### Triple equals

Use triple equals (`===` and `!==`) for comparisons. This will ensure you're not
falling prey to type coercion.

That said, when comparing against `null` or `undefined`, using double equals
(`==` and `!=`) is just fine.

```tsx
// ✅ Good
const user = { id: '123' }
if (user.id === '123') {
	// ...
}
const a = null
if (a === null) {
	// ...
}
if (b != null) {
	// ...
}

// ❌ Avoid
if (a == null) {
	// ...
}
if (b !== null && b !== undefined) {
	// ...
}
```

#### Rely on truthiness

Rely on truthiness instead of comparison operators.

```tsx
// ✅ Good
if (user) {
	// ...
}

// ❌ Avoid
if (user === true) {
	// ...
}
```

#### Switch statement braces

Using braces in switch statements is recommended because it helps clarify the
scope of each case and it avoids variable declarations from leaking into other
cases.

```tsx
// ✅ Good
switch (action.type) {
	case 'add': {
		const { amount } = action
		add(amount)
		break
	}
	case 'remove': {
		const { removal } = action
		remove(removal)
		break
	}
}

// ❌ Avoid
switch (action.type) {
	case 'add':
		const { amount } = action
		add(amount)
		break
	case 'remove':
		const { removal } = action
		remove(removal)
		break
}
```

#### Avoid unnecessary ternaries

```tsx
// ✅ Good
const isAdmin = user.role === 'admin'
const value = input ?? defaultValue

// ❌ Avoid
const isAdmin = user.role === 'admin' ? true : false
const value = input != null ? input : defaultValue
```

### Blocks

#### Use braces for multi-line blocks

Use braces for multi-line blocks even when the block is the body of a single
statement.

```tsx
// ✅ Good
if (!user) return
if (user.role === 'admin') {
	abilities = ['add', 'remove', 'edit', 'create', 'modify', 'fly', 'sing']
}

// ❌ Avoid
if (user.role === 'admin')
	abilities = ['add', 'remove', 'edit', 'create', 'modify', 'fly', 'sing']
```

### Control Statements

#### Use statements

Unless you're using the value of the condition in an expression, prefer using
statements instead of expressions.

```tsx
// ✅ Good
if (user) {
	makeUserHappy(user)
}

// ❌ Avoid
user && makeUserHappy(user)
```

### Comments

#### Use comments to explain "why" not "what"

Comments should explain why something is done a certain way, not what the code
does. The names you use for variables and functions are "self-documenting" in a
sense that they explain what the code does. But if you're doing something in a
way that's non-obvious, comments can be helpful.

```tsx
// ✅ Good
// We need to sanitize lineNumber to prevent malicious use on win32
// via: https://example.com/link-to-issue-or-something
if (lineNumber && !(Number.isInteger(lineNumber) && lineNumber > 0)) {
	return { status: 'error', message: 'lineNumber must be a positive integer' }
}

// ❌ Avoid
// Check if lineNumber is valid
if (lineNumber && !(Number.isInteger(lineNumber) && lineNumber > 0)) {
	return { status: 'error', message: 'lineNumber must be a positive integer' }
}
```

#### Use TODO comments for future improvements

Use TODO comments to mark code that needs future attention or improvement.

```tsx
// ✅ Good
// TODO: figure out how to send error messages as JSX from here...
function getErrorMessage() {
	// ...
}

// ❌ Avoid
// FIXME: this is broken
function getErrorMessage() {
	// ...
}
```

#### Use FIXME comments for immediate problems

Use FIXME comments to mark code that needs immediate attention or improvement.

```tsx
// ✅ Good
// FIXME: this is broken
function getErrorMessage() {
	// ...
}
```

> **Note**: The linter should lint against FIXME comments, so this is useful if
> you are testing things out and want to make sure you don't accidentally commit
> your work in progress.

#### Use @ts-expect-error for TypeScript workarounds

When you need to work around TypeScript limitations (or your own knowledge gaps
with TypeScript), use `@ts-expect-error` with a comment explaining why.

```tsx
// ✅ Good
// @ts-expect-error no idea why this started being an issue suddenly 🤷‍♂️
if (jsxEl.name !== 'EpicVideo') return

// ❌ Avoid
// @ts-ignore
if (jsxEl.name !== 'EpicVideo') return
```

#### Use JSDoc for public APIs

Use JSDoc comments for documenting public APIs and their types.

```tsx
// ✅ Good
/**
 * This function generates a TOTP code from a configuration
 * and this comment will explain a few things that are important for you to
 * understand if you're using this function
 *
 * @param {OTPConfig} config - The configuration for the TOTP
 * @returns {string} The TOTP code
 */
export function generateTOTP(config: OTPConfig) {
	// ...
}
```

#### Avoid redundant comments

Don't add comments that just repeat what the code already clearly expresses.

```tsx
// ✅ Good
function calculateTotal(items: Array<number>) {
	return items.reduce((sum, item) => sum + item, 0)
}

// ❌ Avoid
// This function calculates the total of all items in the array
function calculateTotal(items: Array<number>) {
	return items.reduce((sum, item) => sum + item, 0)
}
```

### Semicolons

#### Don't use unnecessary semicolons

Don't use semicolons. The rules for when you should use semicolons are more
complicated than the rules for when you must use semicolons. With the right
eslint rule
([`no-unexpected-multiline`](https://eslint.org/docs/latest/rules/no-unexpected-multiline))
and a formatter that will format your code funny for you if you mess up, you can
avoid the pitfalls. Read more about this in
[Semicolons in JavaScript: A preference](https://kentcdodds.com/blog/semicolons-in-javascript-a-preference).

<!-- prettier-ignore -->
```tsx
// ✅ Good
const name = 'Kent'
const age = 36
const person = { name, age }
const getPersonAge = () => person.age
function getPersonName() {
	return person.name
}

// ❌ Avoid
const name = 'Kent';
const age = 36;
const person = { name, age };
const getPersonAge = () => person.age;
function getPersonName() {
	return person.name
}
```

The only time you need semicolons is when you have a statement that starts with
`(`, `[`, or `` ` ``. Instances where you do that are few and far between. You
can prefix that with a `;` if you need to and a code formatter will format your
code funny if you forget to do so (and the linter rule will bug you about it
too).

```tsx
// ✅ Good
const name = 'Kent'
const age = 36
const person = { name, age }

// The formatter will add semicolons here automatically
;(async () => {
	const result = await fetch('/api/user')
	return result.json()
})()

// ❌ Avoid
const name = 'Kent'
const age = 36
const person = { name, age }

// Don't manually add semicolons
;(async () => {
	const result = await fetch('/api/user')
	return result.json()
})()
```
