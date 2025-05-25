
### Types

#### Type Inference

Let TypeScript do the heavy lifting with type inference when possible:

```ts
// ✅ Good
function add(a: number, b: number) {
	return a + b // TypeScript infers return type as number
}

// ❌ Avoid
function add(a: number, b: number): number {
	return a + b
}
```

#### Generics

Use generics to create reusable components and functions. And treat type names
in generics the same way you treat any other kind of variable or parameter
(because a generic type is basically a parameter!):

```tsx
// ✅ Good
function createArray<Value>(length: number, value: Value): Array<Value> {
	return Array(length).fill(value)
}

// ❌ Avoid
function createStringArray(length: number, value: string) {
	return Array(length).fill(value)
}
```

#### Type Assertions

Avoid type assertions (`as`) when possible. Instead, use type guards or runtime
validation.

```tsx
// ✅ Good
function isError(maybeError: unknown): maybeError is Error {
	return (
		maybeError &&
		typeof maybeError === 'object' &&
		'message' in maybeError &&
		typeof maybeError.message === 'string'
	)
}

// ❌ Avoid
const error = caughtError as Error
```

#### Type Guards

Use type guards to narrow types and provide runtime type safety. Type guards are
functions that check if a value is of a specific type. The most common way to
create a type guard is using a type predicate.

```tsx
// ✅ Good - Using type predicate
function isError(maybeError: unknown): maybeError is Error {
	return (
		maybeError &&
		typeof maybeError === 'object' &&
		'message' in maybeError &&
		typeof maybeError.message === 'string'
	)
}

// ✅ Good - Using type predicate with schema validation
function isApp(app: unknown): app is App {
	return AppSchema.safeParse(app).success
}

// ✅ Good - Using type predicate with composition
function isExampleApp(app: unknown): app is ExampleApp {
	return isApp(app) && app.type === 'example'
}

// ❌ Avoid - Not using type predicate
function isApp(app: unknown): boolean {
	return typeof app === 'object' && app !== null
}
```

Type predicates use the syntax `parameterName is Type` to tell TypeScript that
the function checks if the parameter is of the specified type. This allows
TypeScript to narrow the type in code blocks where the function returns true.

```tsx
// Usage example:
const maybeApp: unknown = getSomeApp()
if (isExampleApp(maybeApp)) {
	// TypeScript now knows that maybeApp is definitely an ExampleApp
	maybeApp.type // TypeScript knows this is 'example'
}
```



#### Schema Validation

Use schema validation (like Zod) for runtime type checking and type inference
when working with something that crosses the boundary of your codebase.

```tsx
// ✅ Good
const OAuthData = z.object({
	accessToken: z.string(),
	refreshToken: z.string(),
	expiresAt: z.date(),
})
type OAuthData = z.infer<typeof OAuthDataSchema>

const oauthData = OAuthDataSchema.parse(rawData)

// ❌ Avoid
type OAuthData = {
	accessToken: string
	refreshToken: string
	expiresAt: Date
}
const oauthData = rawData as OAuthData
```

#### Unknown Type

Use `unknown` instead of `any` for values of unknown type. This forces you to
perform type checking before using the value.

```tsx
// ✅ Good
function handleError(error: unknown) {
	if (isError(error)) {
		console.error(error.message)
	} else {
		console.error('An unknown error occurred')
	}
}

// ❌ Avoid
function handleError(error: any) {
	console.error(error.message)
}
```

#### Type Coercion

Avoid implicit type coercion. Use explicit type conversion when needed. An
exception to this is working with truthiness.

```tsx
// ✅ Good
const number = Number(stringValue)
const string = String(numberValue)
if (user) {
	// ...
}

// ❌ Avoid
const number = +stringValue
const string = '' + numberValue
if (Boolean(user)) {
	// ...
}
```


### Naming Conventions

Learn and follow [Artem's](https://github.com/kettanaito)
[Naming Cheatsheet](https://github.com/kettanaito/naming-cheatsheet). Here's a
summary:

```tsx
// ✅ Good
const firstName = 'Kent'
const friends = ['Kate', 'John']
const pageCount = 5
const hasPagination = postCount > 10
const shouldPaginate = postCount > 10

// ❌ Avoid
const primerNombre = 'Kent'
const amis = ['Kate', 'John']
const page_count = 5
const isPaginatable = postCount > 10
const onItmClk = () => {}
```

Key principles:

1. Use English for all names
2. Be consistent with naming convention (camelCase, PascalCase, etc.)
3. Names should be Short, Intuitive, and Descriptive (S-I-D)
4. Avoid contractions and context duplication
5. Function names should follow the A/HC/LC pattern:
	- Action (get, set, handle, etc.)
	- High Context (what it operates on)
	- Low Context (optional additional context)

For example: `getUserMessages`, `handleClickOutside`, `shouldDisplayMessage`

### Events

#### Event Constants

Define event constants using a const object. Use uppercase with underscores for
event names.

```tsx
// ✅ Good
export const EVENTS = {
	USER_CODE_RECEIVED: 'USER_CODE_RECEIVED',
	AUTH_RESOLVED: 'AUTH_RESOLVED',
	AUTH_REJECTED: 'AUTH_REJECTED',
} as const

// ❌ Avoid
export const events = {
	userCodeReceived: 'userCodeReceived',
	authResolved: 'authResolved',
	authRejected: 'authRejected',
}
```

#### Event Types

Use TypeScript to define event types based on the event constants.

```tsx
// ✅ Good
export type EventTypes = keyof typeof EVENTS

// ❌ Avoid
export type EventTypes =
	| 'USER_CODE_RECEIVED'
	| 'AUTH_RESOLVED'
	| 'AUTH_REJECTED'
```

#### Event Schemas

Define Zod schemas for event payloads to ensure type safety and runtime
validation.

```tsx
// ✅ Good
const CodeReceivedEventSchema = z.object({
	type: z.literal(EVENTS.USER_CODE_RECEIVED),
	code: z.string(),
	url: z.string(),
})

// ❌ Avoid
type CodeReceivedEvent = {
	type: 'USER_CODE_RECEIVED'
	code: string
	url: string
}
```

> **Note**: This is primarily useful because in event systems, you're typically
> crossing a boundary of your codebase (network etc.).

#### Event Cleanup

Always clean up event listeners when they're no longer needed.

```tsx
// ✅ Good
authEmitter.on(EVENTS.USER_CODE_RECEIVED, handleCodeReceived)
return () => {
	authEmitter.off(EVENTS.USER_CODE_RECEIVED, handleCodeReceived)
}

// ❌ Avoid
authEmitter.on(EVENTS.USER_CODE_RECEIVED, handleCodeReceived)
// No cleanup
```

#### Event Error Handling

Make certain to cover error cases and emit events for those.

```tsx
// ✅ Good
try {
	// event handling logic
} catch (error) {
	authEmitter.emit(EVENTS.AUTH_REJECTED, {
		error: getErrorMessage(error),
	})
}

// ❌ Avoid
try {
	// event handling logic
} catch (error) {
	console.error(error)
}
```


#### Lodash set/get

These are very handy when we are very annoyed about Typescript being Typescript

```tsx
// ❌ Avoid
_set(config, 'headers.Authorization', `Bearer ${token}`)

// ❌ Avoid
_get(event, 'data.slots.tokens.from', null)


```


#### Dynamic fields

Avoid creating any dynamic function that deals with typescript enums

For example, we have
```tsx
export function getEnumOptions<T extends Enum<T>>(
	e: T,
	enumOptions: {
		t: TFunction<['translation', ...string[]]>
		helperText?: Record<string, string>
	},
): OptionType[] {
	const { helperText, t } = enumOptions
	const keys = Object.keys(e).filter((key) => isNaN(Number(key)))
	return keys.map((k) =>
		helperText
			? { label: t(k), value: e[k], helperText: t(helperText[e[k]]) }
			: { label: t(k), value: e[k] },
	)
}

```
Although it provides a nice way of adding new options to any dropdown automatically when we modify the enum itself,
We should avoid using this function at any cost.

Reason:
1. not type safe by nature, even if we try.
2. The IDE won't understand that some properties of the enum are being used which makes it prune to removal by non-context-aware-peeps
3. No way to know that the translation keys are being used

