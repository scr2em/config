# Internationalization (i18n) Standards and Best Practices
Implementing proper internationalization is crucial for creating globally accessible applications. Here's a comprehensive guide for standardizing i18n in your projects.
## Core Principles
### 1. Separation of Content from Code
- **Use dedicated translation files** - Keep translations separate from application code
- **Avoid hardcoded strings** - Never embed translatable text directly in your components
- **Structure by language** - Organize translations by language code (e.g., `en.json`, `es.json`)

### 2. Translation File Structure
```json
{
  "common": {
    "buttons": {
      "submit": "Submit",
      "cancel": "Cancel",
      "save": "Save"
    },
    "validation": {
      "required": "This field is required",
      "email": "Please enter a valid email"
    }
  },
  "pages": {
    "home": {
      "title": "Welcome to our application",
      "description": "Get started by exploring our features"
    }
  }
}
```

### 3. Key Naming Conventions
- Use descriptive, hierarchical keys that follow a consistent pattern
- Structure keys by feature/component, not by the text content
- Use dot notation for accessing nested keys

```ts

// ✅ Good
t('common.buttons.submit')
t('pages.home.title')

// ❌ Avoid
t('submit_button')
t('welcome_message')

```

## Implementation Best Practices
### 1. Setup and Configuration
- Install a robust i18n library (e.g., `react-i18next`, `next-intl`, `formatjs`)
- Configure language detection and fallbacks
- Set up language switching mechanism

```ts
// Example configuration with react-i18next
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    resources: {
      en: { translation: require('./locales/en.json') },
      es: { translation: require('./locales/es.json') }
    },
    interpolation: {
      escapeValue: false // React already safes from XSS
    }
  });

export default i18n;

```


### 2. String Interpolation
- Use placeholders for dynamic content
- Handle pluralization correctly
- Consider gender-specific translations when relevant

```tsx
// ✅ Good
// In translation file
{
  "greeting": "Hello, {{name}}!",
  "items": "{{count}} item",
  "items_plural": "{{count}} items"
}

// In component
t('greeting', { name: userName })
t('items', { count: itemCount })

// ❌ Avoid
// Concatenation
"Hello, " + userName + "!"

```

### 3. Handling Pluralization
Use proper pluralization rules for each language:
```json
{
  "items_zero": "No items",
  "items_one": "One item",
  "items_two": "Two items",
  "items_few": "A few items",
  "items_many": "Many items",
  "items_other": "{{count}} items"
}
```

### 4. Date and Number Formatting
- Use locale-aware date/number formatting functions
- Standardize date/time formats across the application
- Consider time zones when displaying dates

```tsx
// ✅ Good
// In component
import { formatDate, formatNumber } from 'your-i18n-utils';

<span>{formatDate(dateValue, { format: 'long' })}</span>
<span>{formatNumber(amount, { style: 'currency', currency: 'USD' })}</span>

// ❌ Avoid
<span>{new Date(dateValue).toLocaleDateString()}</span>
<span>${amount.toFixed(2)}</span>

```

### 5. Context-Aware Translations
- Provide context for ambiguous terms
- Use translation contexts for words with multiple meanings

```json
{
  "file_verb": "To file a document",
  "file_noun": "A computer file"
}

```

```tsx
t('file_verb')  // For the action of filing
t('file_noun')  // For referring to a file
```

## Code Organization
### 1. Translation Loading
- Load translations asynchronously to reduce initial bundle size
- Implement lazy loading for non-critical languages
- Use code splitting for language resources

```tsx
// Dynamic import of language resources
const loadLanguageAsync = async (language) => {
  const module = await import(`./locales/${language}.json`);
  i18n.addResourceBundle(language, 'translation', module.default);
};
```

### 2. Reusable Translation Components
Create wrapper components for common translation patterns:

```tsx
// TranslatedText.tsx
function TranslatedText({ id, values = {}, defaultMessage = '' }) {
  const { t } = useTranslation();
  return <>{t(id, { ...values, defaultValue: defaultMessage })}</>;
}

// Usage
<TranslatedText id="common.buttons.submit" defaultMessage="Submit" />
```


### 2. RTL Support
- Add support for right-to-left (RTL) languages
- Test layouts in RTL mode
- Use CSS logical properties (`start`/`end` instead of `left`/`right`)

```css
/* ✅ Good */
.container {
  padding-inline-start: 1rem;
  margin-inline-end: 2rem;
}

/* ❌ Avoid */
.container {
  padding-left: 1rem;
  margin-right: 2rem;
}

```