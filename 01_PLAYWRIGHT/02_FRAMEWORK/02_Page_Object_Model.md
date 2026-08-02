# Playwright - Page Object Model: Organizing Page Interactions

## What is Page Object Model (POM)?

Page Object Model is a design pattern where each page of your application gets its own class that encapsulates all interactions with that page.

Basic idea:
```
LoginPage class
  - Contains all login page elements (email input, password input, submit button)
  - Contains all login page actions (fillEmail, fillPassword, clickSubmit)
  - Tests use this class, never interact directly with page

If page changes → Update ONE class → All tests adapt
```

---

## Why It Matters (Interview Context)

Interviewers ask about POM because:
1. **Shows you understand code organization** (not just "writing tests")
2. **Demonstrates maintainability thinking** (how do you scale?)
3. **Proves you can teach others** (POM is a framework standard)
4. **Shows maturity** (POM is used in enterprise frameworks)

POM is THE pattern for enterprise test frameworks.

---

## Your 30-Second Interview Answer

> "Page Object Model is a class for each page that encapsulates all page interactions. Test interacts with page object, not directly with Playwright. At NielsenIQ, we have page objects for LoginPage, DashboardPage, PaymentPage - each handles its own elements and actions. If UI changes, we update the page object once, all 50+ related tests adapt automatically."

---

## Your 2-Minute Deep Answer

> "Page Object Model is a simple but powerful pattern.
>
> **The Problem Without POM:**
>
> ```typescript
> // ❌ Test file is full of page interaction code
> test('User logs in', async ({ page }) => {
>   await page.goto('/login');
>   await page.getByTestId('email').fill('user@example.com');
>   await page.getByTestId('password').fill('password123');
>   await page.getByRole('button', { name: 'Login' }).click();
>   await page.waitForURL('/dashboard');
> });
>
> test('User resets password', async ({ page }) => {
>   await page.goto('/login');
>   await page.getByRole('link', { name: 'Forgot Password' }).click();
>   await page.getByTestId('email').fill('user@example.com');  // Duplicated!
>   // ... more duplication
> });
>
> // Problem: Email selector (data-testid='email') appears in 50+ tests
> // Designer changes it to data-testid='user-email'
> // You update 50+ tests. ERROR-PRONE!
> ```
>
> **The Solution: Page Object Model**
>
> ```typescript
> // pages/LoginPage.ts
> export class LoginPage {
>   constructor(private page: Page) {}
>
>   // Elements (encapsulated)
>   private emailInput = this.page.getByTestId('email');
>   private passwordInput = this.page.getByTestId('password');
>   private loginButton = this.page.getByRole('button', { name: 'Login' });
>
>   // Actions (methods)
>   async navigateTo() {
>     await this.page.goto('/login');
>   }
>
>   async fillEmail(email: string) {
>     await this.emailInput.fill(email);
>   }
>
>   async fillPassword(password: string) {
>     await this.passwordInput.fill(password);
>   }
>
>   async clickLogin() {
>     await this.loginButton.click();
>     await this.page.waitForURL('/dashboard');
>   }
>
>   async login(email: string, password: string) {
>     await this.navigateTo();
>     await this.fillEmail(email);
>     await this.fillPassword(password);
>     await this.clickLogin();
>   }
> }
> ```
>
> **Now tests are clean:**
>
> ```typescript
> test('User logs in', async ({ page }) => {
>   const login = new LoginPage(page);
>   await login.login('user@example.com', 'password123');
>   // Done. Test is 2 lines instead of 10.
> });
>
> test('User resets password', async ({ page }) => {
>   const login = new LoginPage(page);
>   await login.navigateTo();
>   await login.clickForgotPassword();
>   // ...
> });
> ```
>
> **Designer changes email selector to data-testid='user-email':**
> ```typescript
> // Update ONE place
> private emailInput = this.page.getByTestId('user-email');  // Changed here
> // All 50+ tests automatically use new selector!
> ```
>
> **POM Best Practices at NielsenIQ:**
>
> **1. ONE PAGE OBJECT PER PAGE**
> ```
> LoginPage.ts - for login page
> DashboardPage.ts - for dashboard page
> PaymentPage.ts - for payment page
> BasePage.ts - for common functionality
> ```
>
> **2. ENCAPSULATE ELEMENTS**
> ```typescript
> // ✅ DO: Keep selectors private
> private emailInput = this.page.getByTestId('email');
>
> // ❌ DON'T: Expose selectors
> public emailInput = this.page.getByTestId('email');
> // (If test uses it directly, change doesn't propagate)
> ```
>
> **3. PUBLIC METHODS FOR ACTIONS**
> ```typescript
> // ✅ DO: Actions as methods
> async fillEmail(email: string) { ... }
> async clickSubmit() { ... }
>
> // ❌ DON'T: Expose raw page
> public page: Page  // If test uses page directly, we've lost encapsulation
> ```
>
> **4. HELPER METHODS FOR MULTI-STEP ACTIONS**
> ```typescript
> // ✅ DO: Combine related actions
> async login(email: string, password: string) {
>   await this.fillEmail(email);
>   await this.fillPassword(password);
>   await this.clickLogin();
> }
>
> // Test: await login.login('user@example.com', 'pass123');
> // vs:
> // await login.fillEmail('user@example.com');
> // await login.fillPassword('pass123');
> // await login.clickLogin();
> ```
>
> **5. RETURN THIS FOR METHOD CHAINING**
> ```typescript
> async fillEmail(email: string) {
>   await this.emailInput.fill(email);
>   return this;  // Allow chaining
> }
>
> // Test: await login.fillEmail('user@example.com')
>   .fillPassword('pass123')
>   .clickLogin();
> ```
>
> **6. USE COMPOSITION FOR RELATED PAGES**
> ```typescript
> export class ApplicationFlow {
>   private loginPage = new LoginPage(this.page);
>   private dashboardPage = new DashboardPage(this.page);
>
>   async loginAndNavigateToDashboard(email: string, password: string) {
>     await this.loginPage.login(email, password);
>     return this.dashboardPage;
>   }
> }
> ```
>
> **At NielsenIQ:**
> - ✅ LoginPage, DashboardPage, PaymentPage, SettingsPage
> - ✅ Each page has 10-15 methods (actions)
> - ✅ Elements are private, methods are public
> - ✅ Tests use page objects, never interact with page directly
> - ✅ Change selector → Update page object → Tests adapt
> - ✅ Result: 300+ tests, highly maintainable"

---

## Page Object Model Structure

```typescript
export class LoginPage {
  constructor(private page: Page) {}

  // Elements (Private - Encapsulated)
  private emailInput = this.page.getByTestId('email');
  private passwordInput = this.page.getByTestId('password');
  private loginButton = this.page.getByRole('button', { name: 'Login' });
  private forgotPasswordLink = this.page.getByRole('link', { name: 'Forgot Password' });

  // Navigation
  async navigateTo() {
    await this.page.goto('/login');
  }

  // Individual Actions (Public)
  async fillEmail(email: string) {
    await this.emailInput.fill(email);
    return this;
  }

  async fillPassword(password: string) {
    await this.passwordInput.fill(password);
    return this;
  }

  async clickLogin() {
    await this.loginButton.click();
    await this.page.waitForURL('/dashboard');
  }

  async clickForgotPassword() {
    await this.forgotPasswordLink.click();
  }

  // Composite Actions (Public)
  async login(email: string, password: string) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.clickLogin();
  }
}
```

---

## POM vs Non-POM Comparison

| Aspect | Without POM | With POM |
|--------|------------|----------|
| **Selector Change** | Update 50+ tests | Update 1 page object |
| **Test Readability** | 10+ lines per test | 2-3 lines per test |
| **Code Duplication** | High (selectors repeated) | None (encapsulated) |
| **Maintainability** | Poor (scattered logic) | Excellent (centralized) |
| **New Test Writing** | 30 minutes | 5 minutes |
| **Scaling** | Breaks at 100 tests | Handles 1000+ tests |

---

## Your Real Project Example (NielsenIQ)

**LoginPage.ts:**
```typescript
export class LoginPage {
  private emailInput = this.page.getByTestId('email-input');
  private passwordInput = this.page.getByTestId('password-input');
  private loginButton = this.page.getByRole('button', { name: 'Sign In' });
  private errorMessage = this.page.getByTestId('error-message');

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
    await this.page.waitForURL('/dashboard');
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

// tests/auth/login-valid-credentials.spec.ts
test('User logs in with valid credentials', async ({ page }) => {
  const login = new LoginPage(page);
  await login.login('user@example.com', 'password123');
  await expect(page).toHaveURL('/dashboard');
});

// tests/auth/login-invalid-password.spec.ts
test('User sees error on invalid password', async ({ page }) => {
  const login = new LoginPage(page);
  await login.login('user@example.com', 'wrongpass');
  const error = await login.getErrorMessage();
  expect(error).toContain('Invalid credentials');
});
```

**The Win:**
- Tests describe WHAT (user logs in)
- Page objects describe HOW (fill email, fill password, click)
- Change? Update LoginPage once, tests adapt

---

## Common Interview Questions & Answers

### Q1: "Why encapsulate page elements if you could just use page directly?"

**A:** "Encapsulation is the whole point of POM. If tests use page directly, there's no central place to update selectors. With encapsulation, designer changes the email selector → I update ONE line in LoginPage → 50+ tests adapt automatically."

### Q2: "Should every action be a separate method?"

**A:** "No. Group related actions. `login(email, password)` combines fill email + fill password + click login. Tests call `login()`, not three separate methods. Balance between granularity and simplicity."

### Q3: "How do you handle dynamic elements?"

**A:** "Page objects handle it. For example, a list of items where each item has a delete button. You'd write: `async deleteItem(itemName: string) { ... }` that finds the item by name, then finds and clicks its delete button. Test just calls `deleteItem()`, doesn't care about the selector logic."

### Q4: "Should page objects return 'this' for chaining?"

**A:** "Yes, it's cleaner. `await login.fillEmail().fillPassword().clickLogin()` reads better than three separate await statements. But don't force it - only return 'this' if it makes sense semantically."

### Q5: "What about pages with many elements?"

**A:** "Break them into logical sections. A DashboardPage might have `header`, `sidebar`, `contentArea` as private sub-objects. Or create sub-page objects: `HeaderSection`, `SidebarSection`. Keeps each class focused and readable."

---

## What You Should Be Ready to Explain

✅ Why POM eliminates code duplication  
✅ How encapsulation enables easy changes  
✅ Difference between element selectors and action methods  
✅ When to combine actions vs keep them separate  
✅ How POM scales to 300+ tests  
✅ Your POM structure at NielsenIQ  

---

## Weak Areas to Watch

❌ **Don't expose:** Page directly to tests  
→ **Reality:** Keep it private, expose methods only

❌ **Don't create:** One massive page object for a complex page  
→ **Reality:** Break into logical sections

❌ **Don't mix:** Test logic with page logic  
→ **Reality:** Tests describe WHAT, pages describe HOW

❌ **Don't duplicate:** Selectors across multiple page objects  
→ **Reality:** One selector, one place

---

## Key Takeaway for Interview

Page Object Model is not optional in enterprise frameworks. It's THE pattern for maintaining tests at scale.

Your POM quality determines if 300+ tests are maintainable or chaotic. Good POM = tests adapt to UI changes without breaking.

**At NielsenIQ:** Your POM design is why changing selectors takes minutes, not days.

**Your competitive edge:** You architect test infrastructure, not just write test scripts. ✨

---

## POM Checklist

✅ One page object per page  
✅ Elements are private (encapsulated)  
✅ Actions are public methods  
✅ Helper methods combine related actions  
✅ No test logic in page objects  
✅ No page reference exposed to tests  
✅ Methods return appropriate types or 'this' for chaining  
✅ Clear, descriptive method names  

---

**Remember:** Your page objects are API definitions for your application's UI. Design them for ease of use by tests. 🎯
