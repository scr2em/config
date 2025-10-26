## React

### Avoid useEffect

[You Might Not Need `useEffect`](https://react.dev/learn/you-might-not-need-an-effect)

Instead of using `useEffect`, use ref callbacks, event handlers with
`flushSync`, css, `useSyncExternalStore`, etc.

```tsx
// This example was ripped from the docs:
// ✅ Good
function ProductPage({ product, addToCart }) {
	function buyProduct() {
		addToCart(product)
		showNotification(`Added ${product.name} to the shopping cart!`)
	}

	function handleBuyClick() {
		buyProduct()
	}

	function handleCheckoutClick() {
		buyProduct()
		navigateTo('/checkout')
	}
	// ...
}

useEffect(() => {
	setCount(count + 1)
}, [count])

// ❌ Avoid
function ProductPage({ product, addToCart }) {
	useEffect(() => {
		if (product.isInCart) {
			showNotification(`Added ${product.name} to the shopping cart!`)
		}
	}, [product])

	function handleBuyClick() {
		addToCart(product)
	}

	function handleCheckoutClick() {
		addToCart(product)
		navigateTo('/checkout')
	}
	// ...
}
```

There are a lot more examples in the docs. `useEffect` is not banned or
anything. There are just better ways to handle most cases.

Here's an example of a situation where `useEffect` is appropriate:

```tsx
// ✅ Good
useEffect(() => {
	const controller = new AbortController()

	window.addEventListener(
		'keydown',
		(event: KeyboardEvent) => {
			if (event.key !== 'Escape') return

			// do something based on escape key being pressed
		},
		{ signal: controller.signal },
	)

	return () => {
		controller.abort()
	}
}, [])
```

### Don't Sync State, Derive It

[Don't Sync State, Derive It](https://kentcdodds.com/blog/dont-sync-state-derive-it)

```tsx
// ✅ Good
const [count, setCount] = useState(0)
const isEven = count % 2 === 0

// ❌ Avoid
const [count, setCount] = useState(0)
const [isEven, setIsEven] = useState(false)

useEffect(() => {
	setIsEven(count % 2 === 0)
}, [count])
```

### Do not render falsiness

In JSX, do not render falsy values other than `null`.

```tsx
// ✅ Good
<div>
	{contacts.length ? <div>You have {contacts.length} contacts</div> : null}
</div>

// ❌ Avoid
<div>
	{contacts.length && <div>You have {contacts.length} contacts</div>}
</div>
```

### Use ternaries

Use ternaries for simple conditionals. When automatically formatted, they should
be plenty readable, even on multiple lines. Ternaries are also the only
conditional in the spec (currently) which are expressions and can be used in
return statements and other places expressions are used.

```tsx
// ✅ Good
const isAdmin = user.role === 'admin'
const access = isAdmin ? 'granted' : 'denied'

function App({ user }: { user: User }) {
	return (
		<div className="App">
			{user.role === 'admin' ? <Link to="/admin">Admin</Link> : null}
		</div>
	)
}
```

## Testing

### Test User Interactions

Test components based on how users actually interact with them, not
implementation details:

> The more your tests resemble the way your software is used, the more
> confidence they can give you. -
> [Kent C. Dodds](https://x.com/kentcdodds/status/977018512689455106)

```tsx
// ✅ Good
test('User can add items to cart', async () => {
	render(<ProductList />)
	await userEvent.click(screen.getByRole('button', { name: /add to cart/i }))
	await expect(screen.getByText(/1 item in cart/i)).toBeInTheDocument()
})

// ❌ Avoid
test('Cart state updates when addToCart is called', () => {
	const { container } = render(<ProductList />)
	const addButton = container.querySelector('[data-testid="add-button"]')
	fireEvent.click(addButton)
	expect(
		container.querySelector('[data-testid="cart-count"]'),
	).toHaveTextContent('1')
})
```

### Avoid Unnecessary Mocks

Only mock what's absolutely necessary. Most of the time, you don't need to mock
any of your own code or even dependency code.

```tsx
// ✅ Good
function Greeting({ name }: { name: string }) {
	return <div>Hello {name}</div>
}

test('Greeting displays the name', () => {
	render(<Greeting name="Kent" />)
	expect(screen.getByText('Hello Kent')).toBeInTheDocument()
})

// ❌ Avoid
test('Greeting displays the name', () => {
	const mockName = 'Kent'
	vi.mock('./greeting.tsx', () => ({
		Greeting: () => <div>Hello {mockName}</div>,
	}))
	render(<Greeting name={mockName} />)
	expect(container).toHaveTextContent('Hello Kent')
})
```

### Mock External Services

Use MSW (Mock Service Worker) to mock external services. This allows you to test
your application's integration with external APIs without actually making
network requests.

```tsx
// ✅ Good
import { setupServer } from 'msw/node'
import { http } from 'msw'

const server = setupServer(
	http.get('/api/user', async ({ request }) => {
		return HttpResponse.json({
			name: 'Kent',
			role: 'admin',
		})
	}),
)

test('User data is fetched and displayed', async () => {
	render(<UserProfile />)
	await expect(await screen.findByText('Kent')).toBeInTheDocument()
})

// ❌ Avoid
test('User data is fetched and displayed', async () => {
	vi.spyOn(global, 'fetch').mockResolvedValue({
		json: () => Promise.resolve({ name: 'Kent', role: 'admin' }),
	})
	render(<UserProfile />)
	await expect(await screen.findByText('Kent')).toBeInTheDocument()
})
```

### Use Test Function

Use the `test` function instead of `describe` and `it`. This makes tests more
straightforward and easier to understand.

```tsx
// ✅ Good
test('User can log in with valid credentials', async () => {
	render(<LoginForm />)
	await userEvent.type(
		screen.getByRole('textbox', { name: /email/i }),
		'kent@example.com',
	)
	await userEvent.type(
		screen.getByRole('textbox', { name: /password/i }),
		'password123',
	)
	await userEvent.click(screen.getByRole('button', { name: /login/i }))
	await expect(await screen.findByText('Welcome back!')).toBeInTheDocument()
})

// ❌ Avoid
describe('LoginForm', () => {
	it('should allow user to log in with valid credentials', async () => {
		render(<LoginForm />)
		await userEvent.type(
			screen.getByRole('textbox', { name: /email/i }),
			'kent@example.com',
		)
		await userEvent.type(
			screen.getByRole('textbox', { name: /password/i }),
			'password123',
		)
		await userEvent.click(screen.getByRole('button', { name: /login/i }))
		await expect(await screen.findByText('Welcome back!')).toBeInTheDocument()
	})
})
```

### [Avoid Nesting Tests](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing)

Keep your tests flat. Nesting makes tests harder to understand and maintain.

```tsx
// ✅ Good
test('User can log in', async () => {
	render(<LoginForm />)
	await userEvent.type(
		screen.getByRole('textbox', { name: /email/i }),
		'kent@example.com',
	)
	await userEvent.type(
		screen.getByRole('textbox', { name: /password/i }),
		'password123',
	)
	await userEvent.click(screen.getByRole('button', { name: /login/i }))
	await expect(await screen.findByText('Welcome back!')).toBeInTheDocument()
})

// ❌ Avoid
describe('LoginForm', () => {
	describe('when user enters valid credentials', () => {
		it('should show welcome message', async () => {
			render(<LoginForm />)
			await userEvent.type(
				screen.getByRole('textbox', { name: /email/i }),
				'kent@example.com',
			)
			await userEvent.type(
				screen.getByRole('textbox', { name: /password/i }),
				'password123',
			)
			await userEvent.click(screen.getByRole('button', { name: /login/i }))
			await expect(await screen.findByText('Welcome back!')).toBeInTheDocument()
		})
	})
})
```

### Avoid shared setup/teardown variables

```tsx
// ✅ Good
test('renders a greeting', () => {
	render(<Greeting name="Kent" />)
	expect(screen.getByText('Hello Kent')).toBeInTheDocument()
})

// ❌ Avoid
let utils
beforeEach(() => {
	utils = render(<Greeting name="Kent" />)
})

test('renders a greeting', () => {
	expect(utils.getByText('Hello Kent')).toBeInTheDocument()
})
```

> **Note**: Most of the time your individual tests can avoid the use of
> `beforeEach` and `afterEach` altogether and it's only global setup that needs
> it (like mocking out `console.log` or setting up a mock server).

### Avoid Testing Implementation Details

Test your components based on how they're used, not how they're implemented.

```tsx
// ✅ Good
function Counter() {
	const [count, setCount] = useState(0)
	return <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
}

test('Counter increments when clicked', async () => {
	render(<Counter />)
	const button = screen.getByRole('button')
	await userEvent.click(button)
	expect(getByText('Count: 1')).toBeInTheDocument()
})

// ❌ Avoid
test('Counter increments when clicked', () => {
	const { container } = render(<Counter />)
	const button = container.querySelector('button')
	fireEvent.click(button)
	const state = container.querySelector('[data-testid="count"]')
	expect(state).toHaveTextContent('1')
})
```

### Keep Assertions Specific

Make your assertions as specific as possible to catch the exact behavior you're
testing.

```tsx
// ✅ Good
test('Form shows error for invalid email', async () => {
	render(<LoginForm />)
	await userEvent.type(
		screen.getByRole('textbox', { name: /email/i }),
		'invalid-email',
	)
	await userEvent.click(screen.getByRole('button', { name: /login/i }))
	await expect(
		await screen.findByText(/enter a valid email/i),
	).toBeInTheDocument()
})

// ❌ Avoid
test('Form shows error for invalid email', async () => {
	const { container } = render(<LoginForm />)
	await userEvent.type(
		screen.getByRole('textbox', { name: /email/i }),
		'invalid-email',
	)
	await userEvent.click(screen.getByRole('button', { name: /login/i }))
	expect(container).toMatchSnapshot()
})
```

### Follow the Testing Trophy

Prioritize your tests according to the Testing Trophy:

1. Static Analysis (TypeScript, ESLint)
2. Unit Tests (Pure Functions)
3. Integration Tests (Component Integration)
4. E2E Tests (Critical User Flows)

```tsx
// ✅ Good
// 1. Static Analysis
function add(a: number, b: number): number {
	return a + b
}

// 2. Unit Tests
test('add function adds two numbers', () => {
	expect(add(1, 2)).toBe(3)
})

// 3. Integration Tests
test('Calculator component adds numbers', async () => {
	render(<Calculator />)
	await userEvent.click(screen.getByRole('button', { name: '1' }))
	await userEvent.click(screen.getByRole('button', { name: '+' }))
	await userEvent.click(screen.getByRole('button', { name: '2' }))
	await userEvent.click(screen.getByRole('button', { name: '=' }))
	expect(getByText('3')).toBeInTheDocument()
})

// 4. E2E Tests (using Playwright)
await page.goto('/calculator')
await expect(page.getByRole('button', { name: '0' })).toBeInTheDocument()
await page.getByRole('button', { name: '1' }).click()
await page.getByRole('button', { name: '+' }).click()
await page.getByRole('button', { name: '2' }).click()
await page.getByRole('button', { name: '=' }).click()
await expect(page.getByRole('button', { name: '3' })).toBeInTheDocument()

// ❌ Avoid
// Don't write E2E tests for everything
test('every button click updates display', () => {
	render(<Calculator />)
	// Testing every possible button combination...
})
```

### Use Appropriate Queries

Follow the query priority order and avoid using container queries:

```tsx
// ✅ Good
screen.getByRole('textbox', { name: /username/i })

// ❌ Avoid
screen.getByTestId('username')
container.querySelector('.btn-primary')
```

### Use Query Variants Correctly

Only use query\* variants for checking non-existence:

```tsx
// ✅ Good
expect(screen.queryByRole('alert')).not.toBeInTheDocument()

// ❌ Avoid
expect(screen.queryByRole('alert')).toBeInTheDocument()
```

### Use find\* Over waitFor for Elements

Use find\* queries instead of waitFor for elements that may not be immediately
available:

```tsx
// ✅ Good
const submitButton = await screen.findByRole('button', { name: /submit/i })

// ❌ Avoid
const submitButton = await waitFor(() =>
	screen.getByRole('button', { name: /submit/i }),
)
```

### Avoid Testing Implementation Details

Test components based on how users interact with them, not implementation
details:

```tsx
// ✅ Good
test('User can add items to cart', async () => {
	render(<ProductList />)
	await userEvent.click(screen.getByRole('button', { name: /add to cart/i }))
	await expect(screen.getByText(/1 item in cart/i)).toBeInTheDocument()
})

// ❌ Avoid
test('Cart state updates when addToCart is called', () => {
	const { container } = render(<ProductList />)
	const addButton = container.querySelector('[data-testid="add-button"]')
	fireEvent.click(addButton)
	expect(
		container.querySelector('[data-testid="cart-count"]'),
	).toHaveTextContent('1')
})
```

### Use userEvent Over fireEvent

Use userEvent over fireEvent for more realistic user interactions:

```tsx
// ✅ Good
await userEvent.type(screen.getByRole('textbox'), 'Hello')

// ❌ Avoid
fireEvent.change(screen.getByRole('textbox'), { target: { value: 'Hello' } })
```

## Misc

### File naming

Use kebab-case for file names.

```tsx
// ✅ Good
import { HighlightButton } from './highlight-button'

// ❌ Avoid
import { HighlightButton } from './HighlightButton'
```

It makes things work consistently on Windows and Unix-based systems.
