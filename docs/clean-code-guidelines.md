
# Clean Code Guidelines

This document outlines best practices for engineers to ensure clean, maintainable, and secure code during development. It serves as a reference and tool for code reviews, providing a shared vocabulary and value system for building scalable applications.

## 📚 Table of Contents

- [Comments](#comments)
- [Environment](#environment)
- [Functions](#functions)
- [General](#general)
- [JavaScript/TypeScript](#javascripttypescript)
- [Names](#names)
- [Tests](#tests)
- [Security](#security)
- [Commits](#commits)

## Comments

Use comments to explain *why* code exists, not *what* it does. Good comments clarify intent, assumptions, and edge cases. Avoid redundant or outdated comments.

### C1: Inappropriate Information
Avoid comments with metadata (e.g., author, date). Use version control instead.

```javascript
// ❌ Author: Jane Doe, Last modified: 2022-01-01
// ✅ Use git blame or log instead
```

### C2: Obsolete Comment
Update or remove outdated comments.

```javascript
// ❌ Sends data to v1 API
sendSurveyToV2API(payload); // Misleading comment
// ✅ Removed obsolete comment
sendSurveyToV2API(payload);
```

### C3: Redundant Comment
Avoid restating what code clearly expresses.

```javascript
// ❌ Increment respondent count
survey.respondentCount += 1;
// ✅ Self-explanatory
survey.respondentCount += 1;
```

### C4: Poorly Written Comment
Write clear, concise comments when needed.

```javascript
// ❌ Fixes stuff
updateSurveyState();
// ✅ Sets survey status to "complete" if all questions answered
updateSurveyState();
```

### C5: Commented-Out Code
Remove commented-out code. Use version control instead.

```javascript
// ❌ const legacySurveyUrl = 'https://old.surveysystem.com/api/v1/submit';
// ✅ Removed – use git if needed
```

**Useful Resources**:
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
- [Better Comments VSCode Extension](https://marketplace.visualstudio.com/items?itemName=aaron-bond.better-comments)

## Environment

Standardize development and runtime environments to avoid “it works on my machine” issues.

### E1: Build Requires More Than One Step
Ensure builds start with one command (e.g., `pnpm dev`).

```bash
# ❌ Multiple steps
pnpm i
pnpm dev
# ✅ One-liner
pnpm dev
```

Document additional setup (e.g., environment files) in `README.md`.

### E2: Tests Require More Than One Step
Run tests with one command (e.g., `pnpm test`).

```bash
# ❌ Multiple commands
pnpm build
pnpm run:test
# ✅ Single command
pnpm test
```

### E3: Add/Update Documentation
Every repository must include an up-to-date `README.md` with setup and usage instructions.

**Useful Resources**:
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [ESLint](https://eslint.org/)
- [Prettier](https://prettier.io/)
- [EditorConfig](https://editorconfig.org/)

## Functions

Functions should be small, pure, and follow the single responsibility principle.

### F1: Too Many Arguments
Use objects instead of multiple arguments.

```javascript
// ❌ Too many arguments
createSurvey('Customer Feedback', 'en', true, '2025-01-01');
// ✅ Single object
createSurvey({
  title: 'Customer Feedback',
  language: 'en',
  isAnonymous: true,
  startDate: '2025-01-01'
});
```

### F2: Output Arguments
Prefer return values over mutating inputs.

**C#**:
```csharp
// ❌ Mutates input
void MarkSurveyAsComplete(Survey survey) {
    survey.Status = "Complete";
}
// ✅ Returns new object
Survey MarkSurveyAsComplete(Survey survey) {
    return new Survey { Id = survey.Id, Status = "Complete" };
}
```

**C++**:
```cpp
// ❌ Mutates reference
void markSurveyAsComplete(Survey& survey) {
    survey.status = "complete";
}
// ✅ Returns modified copy
Survey markSurveyAsComplete(const Survey& survey) {
    return { .id = survey.id, .Status = "Complete" };
}
```

**JavaScript**:
```javascript
// ❌ Mutates input
function markSurveyAsComplete(survey) {
  survey.status = 'complete';
}
// ✅ Returns new object
function markSurveyAsComplete(survey) {
  return { ...survey, status: 'complete' };
}
```

### F3: Flag Arguments
Split functions instead of using booleans.

```javascript
// ❌ Flag alters behavior
submitSurvey(survey, true); // true = isFinal
// ✅ Purpose-specific functions
submitDraftSurvey(survey);
submitFinalSurvey(survey);
```

Alternatively, use enums:
```javascript
submitSurvey(survey, status.draft);
submitSurvey(survey, status.final);
```

### F4: Dead Function
Delete unused functions.

```javascript
// ❌ Never called
function submitLegacySurvey() {
  // legacy API call
}
// ✅ Deleted – use git to restore
```

**Useful Resources**:
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
- [Extract Method Object](https://refactoring.guru/refactoring/techniques/simplifying-method-calls/extract-method-object)

## General

High-level practices for simplicity, consistency, and maintainability.

### G1: Avoid Mixing Concerns in a Single File
Separate logic, presentation, and configuration.

**JavaScript**:
```javascript
// ❌ HTML, logic, styling mixed
<script>/* 200+ lines */</script>
<style>/* 150+ lines */</style>
// ✅ Extracted
<script src="./SurveyForm.logic.ts" />
<style src="./SurveyForm.styles.css" />
```

**C#**:
```csharp
// ❌ Mixed concerns
public class SurveyController {
    public IActionResult CreateSurvey(...) {
        // Logic + DB access
    }
}
// ✅ Separated layers
```

**C++**:
```cpp
// ❌ Mixed concerns in main.cpp
// ✅ Separate modules: ConfigLoader.cpp, Processor.cpp, UIRenderer.cpp
```

### G2: Obvious Behaviour Is Unimplemented
Ensure function names match behavior.

```javascript
// ❌ Misleading name
function getFirstPageAnswers() {
  return getAnswers(); // all pages
}
// ✅ Matches intent
function getFirstPageAnswers() {
  return getAnswersByPage(1);
}
```

### G3: Incorrect Behaviour at the Boundaries
Handle edge cases properly.

```javascript
// ❌ Crashes if empty
renderQuestion(survey.questions[0]);
// ✅ Guards edge case
if (survey.questions.length > 0) {
  renderQuestion(survey.questions[0]);
}
```

### G4: Overridden Safeties
Don’t suppress warnings or ignore type errors.

```javascript
// ❌ Dangerous
// @ts-ignore
submitAnswer(123);
// ✅ Fix typing
submitAnswer(String(answerId));
```

### G5: Duplication
Avoid repeated logic.

```javascript
// ❌ Repeated validation
if (!answer) showError('Required');
// ✅ Shared util
if (!isAnswerValid(answer)) showError('Required');
```

### G6: Code at Wrong Level of Abstraction
Separate concerns.

```javascript
// ❌ Low-level logic in high-level class
SurveyManager.formatEmailBody()
// ✅ Extracted utility
EmailFormatter.formatSurveySummary()
```

### G7: Base Classes Depending on Their Derivatives
Avoid circular dependencies.

```javascript
// ❌ BaseComponent imports SurveyFormComponent
// ✅ Use hooks, events, or factories
```

### G8: Too Much Information
Keep interfaces small.

```javascript
// ❌ Exposes everything
surveyForm.answers
// ✅ Use accessor
surveyForm.getAnsweredQuestions()
```

### G9: Dead Code
Remove unused components.

```javascript
// ❌ Unused component
function SurveyPreviewLegacy() {}
// ✅ Removed
```

### G10: Vertical Separation
Declare variables/functions close to their use.

```javascript
// ❌ Scattered
function submit() { ... }
const endpoint = '/submit';
const maxRetries = 3;
// ✅ Grouped
const endpoint = '/submit';
const maxRetries = 3;
function submit() { ... }
```

### G11: Inconsistency
Use consistent naming patterns.

```javascript
// ❌ Mixed verbs
sendFeedback();
submitAnswer();
// ✅ Aligned
submitFeedback();
submitAnswer();
```

### G12: Clutter
Remove unused imports or empty constructors.

```javascript
// ❌ Clutter
import fs from 'fs';
class SurveyBuilder {
  constructor() {}
}
// ✅ Cleaned
class SurveyBuilder {}
```

### G13: Artificial Coupling
Avoid unrelated enums/constants in classes.

```javascript
// ❌ Enum in unrelated class
class SurveyUploader {
  enum SurveyType { Quick, Full }
}
// ✅ Dedicated module
```

### G14: Feature Envy
Keep logic with its data.

```javascript
// ❌ External dependency
if (surveyForm.state === 'submitted') { ... }
// ✅ Encapsulated
if (surveyForm.isSubmitted()) { ... }
```

### G15: Selector Arguments
Use intent-revealing names.

```javascript
// ❌ Ambiguous
sendNotification('email');
// ✅ Explicit
sendEmailNotification();
```

### G16: Obscured Intent
Use clear names.

```javascript
// ❌ Confusing
const x = handle(a, b);
// ✅ Clear
const submission = submitSurvey(form, user);
```

### G17: Misplaced Responsibility
Put logic where it belongs.

**JavaScript**:
```javascript
// ❌ UI handles business logic
SurveyComponent.submit = () => updateSurveyStatus(id, 'complete');
// ✅ Delegate to service
SurveyService.markAsComplete(id);
```

**C#**:
```csharp
// ❌ Controller with PDF generation
public class SurveyController {
    public FileResult GeneratePdfReport() {
        var pdf = PDFGenerator.BuildFromSurvey(survey);
        return File(pdf, "application/pdf");
    }
}
// ✅ Dedicated service
public class ReportService {
    public byte[] GeneratePdf(Survey survey) { ... }
}
```

**C++**:
```cpp
// ❌ UI with DB logic
class SurveyView {
    void fetchFromDatabase();
};
// ✅ Data layer
class SurveyRepository {
    Survey load(int id);
};
```

### G18: Inappropriate State
Avoid static methods where polymorphism is better.

```javascript
// ❌ Static hides extension
static calculateScore()
// ✅ Instance method
calculateScore()
```

### G19: Use Explanatory Variables
Improve readability with intermediate variables.

```javascript
// ❌ Cryptic
if (q.length > 10 && !q.includes('?')) {}
// ✅ Clear
const maxLength = 10;
const isTooLong = q.length > maxLength;
const isMissingQuestionMark = !q.includes('?');
if (isTooLong && isMissingQuestionMark) { ... }
```

### G20: Function Names Should Say What They Do
Be descriptive.

```javascript
// ❌ Vague
handleRate()
// ✅ Clear
calculateCompletionRate()
```

### G21: Understand the Algorithm
Understand and document algorithms.

```javascript
// ❌ Copied without understanding
answers.sort(customSort);
// ✅ Documented
answers.sort(sortBySubmissionTimestamp);
```

### G22: Make Logical Dependencies Physical
Avoid hardcoded assumptions.

```javascript
// ❌ Tightly coupled
if (survey.type === 'feedback') {...}
// ✅ Config-driven
surveyHandlers[survey.type]?.handle();
```

### G23: Prefer Polymorphism to If/Else or Switch/Case
Use polymorphic behavior.

```javascript
// ❌ Switch on type
switch (question.type) {
  case 'text': renderText(); break;
  case 'choice': renderChoice(); break;
}
// ✅ Polymorphic
question.render();
```

### G24: Follow Standard Conventions
Stick to team conventions.

```javascript
// ❌ Weird name
FormSubmitz.ts
// ✅ Conventional
SubmitForm.ts
```

### G25: Replace Magic Numbers with Named Constants
Define numbers clearly.

```javascript
// ❌ Hardcoded
if (title.length > 100)
// ✅ Constant
const MAX_TITLE_LENGTH = 100;
if (title.length > MAX_TITLE_LENGTH)
```

### G26: Be Precise
Use precise types and conditions.

```javascript
// ❌ Vague
let result: any;
// ✅ Precise
let result: SurveyResult | null;
```

### G27: Structure over Convention
Use language features over comments.

```javascript
// ❌ Comment-based
// Override in subclasses
class Survey {
  render() {}
}
// ✅ Enforced
abstract class Survey {
  abstract render(): void;
}
```

### G28: Encapsulate Conditionals
Name logic clearly.

```javascript
// ❌ Inline
if (survey.isArchived && !survey.isDeleted)
// ✅ Named
if (shouldArchiveSurvey(survey))
```

### G29: Avoid Negative Conditionals
Use positive conditions.

```javascript
// ❌ Negative
if (!user.isNotAdmin)
// ✅ Positive
if (user.isAdmin)
```

### G30: Functions Should Do One Thing
Split multi-purpose functions.

```javascript
// ❌ Mixed concerns
function processSurveys() { ... }
// ✅ Separated
const surveys = fetchSurveys();
const active = filterActive(surveys);
renderSurveyList(active);
```

### G31: Hidden Temporal Couplings
Avoid requiring method call order.

```javascript
// ❌ Must call init()
form.init();
form.submit();
// ✅ Explicit flow
createForm().submit();
```

### G32: Don't Be Arbitrary
Structure code with purpose.

```javascript
// ❌ Random utils
utils.ts
// ✅ Domain-specific
survey/validate.ts
auth/permissions.ts
```

### G33: Encapsulate Boundary Conditions
Hide index logic.

```javascript
// ❌ Inline
if (i === list.length - 1)
// ✅ Helper
if (isLastItem(i, list))
```

### G34: Functions Should Descend Only One Level of Abstraction
Separate concerns.

```javascript
// ❌ Mixed concerns
function renderSurvey() {
  fetchSurvey();
  formatTimestamp();
  document.body.innerHTML = ...
}
// ✅ One level
function renderSurvey() {
  const data = getSurveyData();
  displaySurvey(data);
}
```

### G35: Keep Configurable Data at High Levels
Use configurable defaults.

```javascript
// ❌ Hardcoded
retryRequest(3);
// ✅ Configurable
retryRequest(env.DEFAULT_RETRY_LIMIT);
```

### G36: Check for Null Before Accessing
Avoid null reference errors.

```javascript
// ❌ Fragile
survey.user.account.preferences.theme
// ✅ Safe
survey?.user?.account?.preferences?.theme
```

**Useful Resources**:
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
- [The Twelve-Factor App](https://12factor.net/)
- [Google Engineering Practices](https://google.github.io/eng-practices/)

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

## Names

Use clear, consistent names to reflect domain concepts.

### N1: Choose Descriptive Names
Express intent clearly.

```javascript
// ❌ Vague
function doSubmit() {}
// ✅ Clear
function submitSurvey() {}
```

### N2: Use Abstraction Appropriate Names
Match conceptual level.

```javascript
// ❌ Misleading
class PageLayoutManager {}
// ✅ Domain-relevant
class SurveyBuilder {}
```

### N3: Follow Standard Nomenclature
Use industry norms.

```javascript
// ❌ Ambiguous
class SurveyThing {}
// ✅ Conventional
class SurveyComponent {}
```

### N4: Names Should Be Unambiguous
Avoid generic terms.

```javascript
// ❌ Vague
const data = getData();
// ✅ Specific
const surveyQuestions = getSurveyQuestions();
```

### N5: Scope-Length Names
Short names for short scope; longer for broader scope.

```javascript
// ❌ Long loop var
for (questionIndexInTheLoop of questions) {}
// ✅ Appropriate
for (i of questions) {}
const surveyCompletionThreshold = 0.8;
```

### N6: Avoid Encodings in Names
Avoid prefixes like `strName`.

```javascript
// ❌ Encoded
const strName: string = 'John';
// ✅ Clear
const name = 'John';
```

### N7: Describe Side-Effects
Make side effects obvious.

```javascript
// ❌ Sounds safe
getDraft(); // actually saves!
// ✅ Reflects side effect
saveDraft();
```

**Useful Resources**:
- [Clean Code JavaScript](https://github.com/ryanmcdermott/clean-code-javascript)
- [Object Names and IDs](https://www.objectmentor.com/resources/naming.html)

## Tests

Tests ensure maintainability and safe refactoring.

### T1: Comprehensive Tests
Cover all critical paths.

**C++ (GoogleTest)**:
```cpp
TEST(CalculateScoreTest, HandlesPositiveInput) {
    EXPECT_EQ(calculateScore(10), 100);
}
TEST(CalculateScoreTest, ThrowsOnNegativeInput) {
    EXPECT_THROW(calculateScore(-1), std::invalid_argument);
}
```

**C# (xUnit)**:
```csharp
[Theory]
[InlineData(10, 100)]
[InlineData(0, 0)]
public void CalculateScore_ValidInput_ReturnsExpected(int input, int expected) {
    Assert.Equal(expected, ScoreUtils.CalculateScore(input));
}
[Fact]
public void CalculateScore_NegativeInput_Throws() {
    Assert.Throws<ArgumentException>(() => ScoreUtils.CalculateScore(-1));
}
```

**JavaScript**:
```javascript
test('calculateScore handles positive inputs', () => {
  expect(calculateScore(10)).toBe(100);
});
test('calculateScore handles negative inputs', () => {
  expect(() => calculateScore(-1)).toThrow();
});
```

### T2: Use Coverage Tools
Identify gaps with tools:
- **C++**: gcov, llvm-cov
- **C#**: Coverlet
- **JS/TS**: Vitest, c8, nyc

### T3: Don't Skip Simple Tests
Test simple logic.

**C++**:
```cpp
TEST(StringUtilsTest, TrimWhitespace) {
    EXPECT_EQ(trim("  hello "), "hello");
}
```

**C#**:
```csharp
[Fact]
public void Trim_RemovesLeadingAndTrailingSpaces() {
    Assert.Equal("hello", StringUtils.Trim("  hello  "));
}
```

**JavaScript**:
```javascript
test('trimAnswer removes spaces', () => {
  expect(trimAnswer('  hello  ')).toBe('hello');
});
```

### T4: Use Tests to Document Ambiguities
Clarify behavior with tests.

**C++**:
```cpp
TEST(UserValidationTest, AcceptsEmptyUsername) {
    EXPECT_TRUE(validateUsername(""));
}
```

**C#**:
```csharp
[Fact(Skip = "Clarify if empty username allowed")]
public void EmptyUsername_ShouldBeValid() {
    Assert.True(Validator.IsUsernameValid(""));
}
```

**JavaScript**:
```javascript
test('empty username is valid — to confirm', () => {
  expect(validateUsername("")).toBe(true);
});
```

### T5: Test Boundary Conditions
Cover edge cases.

**C++**:
```cpp
TEST(ValidateAgeTest, BoundaryConditions) {
    EXPECT_TRUE(validateAge(18));
    EXPECT_FALSE(validateAge(17));
}
```

**C#**:
```csharp
[Theory]
[InlineData(18, true)]
[InlineData(17, false)]
public void ValidateAge_BoundaryCheck(int age, bool expected) {
    Assert.Equal(expected, Validator.ValidateAge(age));
}
```

**JavaScript**:
```javascript
test('validateAge handles boundaries', () => {
  expect(validateAge(18)).toBe(true);
  expect(validateAge(17)).toBe(false);
});
```

### T6: Test Thoroughly Around Bug Fixes
Add regression tests.

**C++**:
```cpp
TEST(ParseResponseTest, HandlesNullInput) {
    EXPECT_NO_THROW(parseResponse(nullptr));
}
```

**C#**:
```csharp
[Fact]
public void ParseResponse_NullInput_DoesNotThrow() {
    var result = Parser.ParseResponse(null);
    Assert.NotNull(result);
}
```

**JavaScript**:
```javascript
test('parseResponse handles null', () => {
  expect(() => parseResponse(null)).not.toThrow();
});
```

### T7: Analyze Failure and Coverage Patterns
Investigate flaky tests and poor coverage.

### T8: Maintain Fast Tests
Keep tests isolated.

**C++**:
```cpp
TEST(MathTest, AddsNumbers) {
    EXPECT_EQ(add(1, 2), 3);
}
```

**C#**:
```csharp
[Fact]
public void Add_ReturnsCorrectSum() {
    Assert.Equal(3, MathUtils.Add(1, 2));
}
```

**JavaScript**:
```javascript
test('add returns correct sum', () => {
  expect(add(1, 2)).toBe(3);
});
```

### T9: Avoid DB or Network Calls in Unit Tests
Use mocks:
- **C++**: GoogleMock
- **C#**: Moq
- **JS**: `vi.fn()`, `jest.fn()`

### T10: Avoid Special Code for Tests
Don’t add test-specific logic in production.

```javascript
// ❌ Bad
if (TESTING_ENV) {
  doSomethingElseThanProductionCode();
}
```

## Security

Follow OWASP Top 10 guidelines to ensure secure code.

### S1: Sanitize User Input
Prevent injection attacks.

**JavaScript**:
```javascript
// ❌ Unsafe DOM injection
question.innerHTML = userAnswer;
// ✅ Sanitized
const sanitized = DOMPurify.sanitize(userAnswer);
question.innerHTML = sanitized;
```

**C#**:
```csharp
// ❌ Unsafe SQL
var query = $"SELECT * FROM Users WHERE Name = '{userInput}'";
// ✅ Parameterized
var query = "SELECT * FROM Users WHERE Name = @name";
command.Parameters.AddWithValue("@name", userInput);
```

**C++**:
```cpp
// ❌ Unsafe output
std::cout << userInput;
// ✅ Sanitized
std::string safeInput = sanitize(userInput);
std::cout << safeInput;
```

### S2: Enforce HTTPS and Secure Cookies
Use secure cookie flags.

**Node.js (Express)**:
```javascript
// ❌ Vulnerable cookie
res.cookie('session', token);
// ✅ Secure
res.cookie('session', token, {
  secure: true,
  httpOnly: true,
  sameSite: 'strict',
});
```

### S3: Avoid Hardcoding Secrets
Use environment variables.

**JavaScript**:
```javascript
// ❌ Hardcoded
const API_KEY = 'sk-test-1234';
// ✅ Environment variable
const apiKey = process.env.MAILER_API_KEY;
```

**C#**:
```csharp
// ✅ Configuration
var apiKey = Configuration["Mailer:ApiKey"];
```

### S4: Apply the Principle of Least Privilege
Restrict access by roles.

**JavaScript**:
```javascript
// ❌ No role check
deleteSurvey(surveyId);
// ✅ RBAC
if (user.role !== 'admin') {
  throw new Error('Forbidden');
}
deleteSurvey(surveyId);
```

**C#**:
```csharp
[Authorize(Roles = "Admin")]
public IActionResult DeleteSurvey(int id) { ... }
```

### S5: Protect Against CSRF
Use CSRF tokens.

```html
<!-- ❌ No token -->
<form method="POST" action="/delete-survey">
<!-- ✅ With token -->
<form method="POST" action="/delete-survey">
  <input type="hidden" name="_csrf" value="{{csrfToken}}">
</form>
```

### S6: Validate Data Server-Side
Always validate inputs.

**JavaScript**:
```javascript
// ❌ Client-only validation
// ✅ Server-side
if (!isValidAnswer(answer)) {
  throw new Error('Invalid submission');
}
```

**C#**:
```csharp
RuleFor(x => x.Answer)
  .NotEmpty()
  .MaximumLength(500);
```

### S7: Escape Output Based on Context
Escape data appropriately.

```javascript
// ❌ Unescaped
element.setAttribute('title', userInput);
// ✅ Escaped
element.setAttribute('title', escapeAttribute(userInput));
```

### S8: Secure Deserialization
Validate before deserializing.

**C#**:
```csharp
// ❌ Risky
var obj = JsonConvert.DeserializeObject<User>(rawInput);
// ✅ Validated
var obj = JsonConvert.DeserializeObject<User>(rawInput);
if (!IsValid(obj)) throw new SecurityException("Invalid data");
```

### S9: Disable Unnecessary Features
Reduce attack surface.

```javascript
// ❌ Dangerous
eval(userInput);
// ✅ Avoid or use safe evaluators
```

### S10: Implement Rate Limiting
Prevent brute force attacks.

**Node.js**:
```javascript
import rateLimit from 'express-rate-limit';
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
}));
```

### S11: Set Secure HTTP Headers
Use security headers.

**Node.js**:
```javascript
import helmet from 'helmet';
app.use(helmet());
```

### S12: Use Dependency Scanning
Keep dependencies updated.
- **JS/TS**: `npm audit`, `pnpm audit`
- **C#**: `dotnet list package --vulnerable`
- **C++**: Use CodeQL, OSS-Fuzz

### S13: Avoid Insecure Randomness
Use secure random generators.

**JavaScript**:
```javascript
// ❌ Insecure
const token = Math.random().toString(36);
// ✅ Secure
const token = crypto.randomUUID();
```

### S14: Secure File Uploads
Validate uploads:
- MIME type
- File size
- Filename sanitization
- Secure storage

### S15: Implement Logging and Alerting
Log security events without sensitive data.

**Useful Resources**:
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP Developer Guide](https://owasp.org/www-project-developer-guide/)

## Commits

Write clear, consistent commit messages.

### CMT1: Commit Small, Logical Changes
Commit one logical change per commit.

```bash
# ❌ Everything in one
"fix layout, add validation, update chart labels"
# ✅ Individual commits
"fix: align survey layout header"
"feat: add matrix question type support"
```

### CMT2: Write Clear, Meaningful Messages
Use imperative mood.

```bash
# ❌ Bad
"update stuff"
# ✅ Good
"fix: prevent multiple draft submissions"
```

### CMT3: Use Conventional Commit Format
Standardize messages.

```bash
# ✅ Examples
feat(survey): support conditional question flows
fix(api): handle invalid tokens gracefully
feat!: drop support for IE11
```

### CMT4: Reference Issues or Tickets
Link to tickets.

```bash
# ✅
fix: avoid double submission (TEMP-345)
```

### CMT5: Use the Body for Context
Add details for complex changes.

```bash
# ✅
fix(auth): refresh token handling
When backend sends 401, client attempts silent refresh.
```

### CMT6: Don’t Commit Debug Code
Remove debug code.

```javascript
// ❌ Bad
debugger;
console.log('data', response);
// ✅ Clean
```

### CMT7: Squash Before Merging
Clean up history.

```bash
# ✅ Squashed
feat: implement survey progress bar
```

### CMT8: Combine Ticket Numbers with Description
Use descriptive branch names.

```bash
# ✅
feat/IN-2222-categorical-images
```

**Useful Resources**:
- [Conventional Commits](https://www.conventionalcommits.org/)
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
- [Gitmoji](https://gitmoji.dev/)
- [Git Commit Best Practices](https://justinjoyce.dev/git-commit-message-best-practices/)