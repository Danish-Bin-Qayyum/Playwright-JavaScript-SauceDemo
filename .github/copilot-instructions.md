# Copilot Instructions for Playwright Test Automation

## Project Overview

This is a **Playwright-based test automation framework** for the SauceDemo React web application (https://www.saucedemo.com/). The project uses the **Page Object Model (POM)** design pattern for maintainability and follows multi-browser test execution across Chrome, Firefox, Safari, and Edge.

## Architecture & Key Components

### Directory Structure
- **`pages/`** - Page Object classes extending `BasePage` (core abstraction layer)
- **`pageobjects/`** - Centralized CSS selectors organized by page (one file per page)
- **`tests/`** - Test suites using custom fixtures (5 test scenarios covering checkout workflows)
- **`data/users.json`** - Test data: user credentials and expected text values
- **`testFixtures/fixture.js`** - Custom Playwright fixtures providing page instances to tests
- **`config.js`** - Constants: base URLs, product prices, external site URLs

### Design Pattern: Page Object Model (POM)

Each page has two files:
1. **`pages/PageName.js`** - Page class with methods (extends `BasePage`), imports selectors and config
2. **`pageobjects/pageName.js`** - Exported CSS selector constants (e.g., `export const username = '#user-name'`)

**Example flow:**
```javascript
// pageobjects/loginPage.js
export const username = '#user-name'
export const password = '#password'

// pages/loginPage.js
import { username, password } from '../pageobjects/loginPage'
class LoginPage extends BasePage {
  async fillCredentials(user, pass) {
    await this.waitAndFill(username, user)
    await this.waitAndFill(password, pass)
  }
}
```

### BasePage Methods (Foundation)
Located in [pages/basePage.js](pages/basePage.js), all page classes inherit:
- Navigation: `open(url)`, `getUrl()`, `getTitle()`
- Interaction: `waitAndClick()`, `waitAndFill()`, `waitAndHardClick()` (JavaScript execution)
- Verification: `isElementVisible()`, `isElementEnabled()`, `wait()`, `waitForPageLoad()`
- Debugging: `pause()`

## Test Execution & Configuration

### Run Commands (from npm scripts)
- **`npm run test:chrome`** - Single browser execution (default, parallel workers)
- **`npm run test:serial`** - Sequential execution (--workers=1) for debugging
- **`npm run test:smoke`** - Smoke tests only (@smoke tag)
- **`npm run test:one/two/three/four/five`** - Run individual test files by scenario
- **`npm run test:shard`** - Sharded execution (TC_01 shard 3/3)
- **`npm run html-report`** - Generate HTML report
- **`npm run allure:report`** - Generate Allure test report

### Playwright Configuration ([playwright.config.js](playwright.config.js))
- **Test directory:** `tests/`
- **Timeout:** 60s per test
- **Retries:** 0 (no automatic retry)
- **Viewport:** 1720x850px (standardized for consistency)
- **Reporters:** HTML, JUnit XML (`results.xml`), Allure
- **Failure artifacts:** Screenshots, videos, traces retained on failure
- **Browser-specific settings:** Firefox uses 200ms slowMo; Safari/Firefox run headless

## Test Data & User Accounts

Test data is in [data/users.json](data/users.json). Available test users:
- `standard_user` - Default happy path user
- `locked_out_user` - Validates error message "Sorry, this user has been locked out."
- `problem_user` - Triggers last name validation error
- `performance_glitch_user` - Normal checkout but may have performance delays
- Password (all users): `secret_sauce`

Additional data includes product descriptions, expected error messages, sort options (`az`, `za`, `lohi`, `hilo`), and cart counts.

## Test Fixtures & Custom Assertions

### Fixture Pattern ([testFixtures/fixture.js](testFixtures/fixture.js))
Tests extend custom Playwright `test` with page fixtures:
```javascript
import test from '../testFixtures/fixture'
test('My test', async ({ loginPage, productsPage }) => {
  // loginPage and productsPage are auto-instantiated
})
```
Available fixtures: `loginPage`, `productsPage`, `productDetailsPage`, `yourCartPage`, `checkoutYourInformationPage`, `checkoutOverviewPage`, `checkoutCompletePage`

### Custom Reporter ([CustomReporter.js](CustomReporter.js))
Logs test lifecycle events:
- Test start/end with status
- Step execution (if using `test.step()`)
- Errors to console (useful with CI/CD integration)

## Test Scenarios Overview

1. **TC_01_productPage.test.js** - Login + product page verification + social media link navigation + logout
2. **TC_02_checkoutWorkflow.test.js** - Complete happy-path checkout (login → add product → checkout → verify order)
3. **TC_03_checkoutWithSUandPGU.test.js** - Multi-user scenario: standard user adds product, performance_glitch_user completes checkout
4. **TC_04_checkoutWithPUandPGU.test.js** - problem_user adds product (fails on lastname), performance_glitch_user completes
5. **TC_05_checkoutWithPGUandSU.test.js** - locked_out_user error test, performance_glitch_user adds product, standard_user completes checkout

## Developer Workflows

### Adding a New Test
1. Create [tests/TC_XX_description.test.js](tests/) using existing tests as template
2. Use `@smoke` tag for smoke suite: `test('@smoke', async ({ ... })`
3. Apply `test.step()` for step logging (CustomReporter displays them)
4. Import fixtures from [testFixtures/fixture.js](testFixtures/fixture.js)

### Adding a New Page
1. Create `pages/NewPage.js` extending `BasePage`
2. Create `pageobjects/newPage.js` with selector exports
3. Add fixture in [testFixtures/fixture.js](testFixtures/fixture.js)
4. Import test data from [data/users.json](data/users.json) using `fs.readFileSync()` pattern

### Debugging
- Use `npm run test:serial` for sequential execution
- Call `await page.pause()` in test or `await this.pause()` in page class
- Check `playwright-report/` for HTML reports or `test-results/` for failure artifacts
- Trace files (`.zip`) contain full browser interaction replay

## Key Conventions & Patterns

1. **Selector Organization:** Keep CSS selectors in `pageobjects/` files, never hardcoded in tests
2. **Async/Await:** All page methods return Promises; tests use `async/await`
3. **Test Data:** Always load from `data/users.json` using `fs.readFileSync()`; never hardcode credentials
4. **Wait Strategies:** `waitAndClick()` and `waitAndFill()` implicitly wait; use `waitForPageLoad()` for navigation
5. **Error Messages:** Validate against `data/users.json` values (e.g., `errorMessageLockedOutUser`)
6. **Multi-Browser:** Tests run on multiple browsers via `--project=Chrome|Firefox|Safari|Edge`; use headless config in [playwright.config.js](playwright.config.js)

## Common Tasks

| Task | Approach |
|------|----------|
| Run specific test | `npm run test:five` (or edit config to target file) |
| Debug failed test | `npm run test:serial -- tests/TC_XX.test.js` + `await page.pause()` |
| View test report | `npm run html-report` (opens after execution) |
| Add new assertion | Extend `BasePage` with custom method (e.g., `async verifyUserName()`) |
| Change test data | Edit [data/users.json](data/users.json) |
| Add browser | Extend `projects` array in [playwright.config.js](playwright.config.js) |
