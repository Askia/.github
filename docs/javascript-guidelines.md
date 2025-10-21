## JavaScript/TypeScript

### JS1: Modularise and Structure Imports
Use barrel files for simpler imports.

```javascript
// ❌ Unstructured
import { SurveyCard } from './SurveyCard';
import { SummaryChart } from './SummaryChart';
// ✅ Barrel file
// index.ts
export * from './SurveyCard';
export * from './SummaryChart';
// usage
import { SurveyCard, SummaryChart } from '@components/survey';
```

### JS2: Use Exported Constants Rather Than Inheritance
Avoid inheritance for constants.

```javascript
// ❌ Inheritance for config
class BaseSurvey {
  static MAX_QUESTIONS = 50;
}
// ✅ Shared constant
export const MAX_QUESTIONS = 50;
```

### JS3: Enums over Constants
Use enums for type safety.

```javascript
// ❌ Loose constants
const SINGLE = 'single';
const MULTI = 'multi';
// ✅ Enums
enum QuestionType {
  SingleChoice = 'single',
  MultipleChoice = 'multi',
  Text = 'text',
}
```

**Useful Resources**:
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
