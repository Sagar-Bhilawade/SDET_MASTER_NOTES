# TestNG - Data Providers & Data-Driven Testing

## What This Covers

Running the same test with multiple data sets — `@DataProvider`, `@Parameters`, and reading test data from Excel, CSV, or JSON.

Data-driven testing is one of the framework types interviewers ask you to name, so this connects to the framework architecture answer too.

---

## Why It Matters (Interview Context)

Two things get checked:

1. **Mechanics** — can you write a data provider?
2. **Judgement** — do you know when data-driven testing helps and when it just multiplies maintenance?

The second is the senior layer. Twenty data rows through a UI test is twenty times the execution cost, and the same validation at API level would run in a fraction of the time with more stability.

---

## Your 30-Second Interview Answer

> "`@DataProvider` supplies a two-dimensional array and the test runs once per row. `@Parameters` pulls single values from the XML suite — I'd use that for environment config like browser, not for test data. For external data we read Excel with Apache POI into the data provider. The judgement part is that data-driven UI tests multiply execution cost, so validation logic that doesn't need a browser belongs at API level where twenty cases cost seconds rather than minutes."

---

## Your 2-Minute Deep Answer

> "**`@DataProvider` — the main mechanism:**
>
> ```java
> @DataProvider(name = "loginData")
> public Object[][] loginData() {
>     return new Object[][] {
>         {"valid@example.com", "correct", true},
>         {"valid@example.com", "wrong",   false},
>         {"unknown@example.com", "any",   false},
>         {"", "password",                 false},
>     };
> }
>
> @Test(dataProvider = "loginData")
> public void loginScenarios(String email, String password, boolean shouldSucceed) {
>     LoginPage login = new LoginPage(driver);
>     login.enterCredentials(email, password);
>     // ...
> }
> ```
>
> Each row is one test invocation, reported separately — so a failure tells you which data set broke, not just that the test failed.
>
> **Data provider in a separate class:**
> ```java
> @Test(dataProvider = "loginData", dataProviderClass = TestDataProviders.class)
> ```
> Worth doing once you have more than a handful — keeps test classes focused.
>
> **Parallel data provider:**
> ```java
> @DataProvider(name = "loginData", parallel = true)
> ```
> Rows run concurrently. Only safe if your test is genuinely independent per row and your driver management is thread-safe — which is the ThreadLocal question.
>
> ---
>
> **`@Parameters` — configuration, not test data:**
>
> ```java
> @BeforeMethod
> @Parameters({"browser", "environment"})
> public void setUp(String browser, @Optional("staging") String environment) {
>     driver = DriverFactory.create(browser);
> }
> ```
>
> ```xml
> <parameter name="browser" value="chrome"/>
> ```
>
> This is for environment configuration — browser, base URL, environment name. It supplies a single value per run, so it's the wrong tool for test data. `@Optional` provides a default when the parameter isn't in the XML, which prevents a hard failure when someone runs a suite without it.
>
> ---
>
> **Reading external data — Apache POI for Excel:**
>
> ```java
> public static Object[][] readExcel(String path, String sheetName) throws IOException {
>     try (FileInputStream fis = new FileInputStream(path);
>          Workbook workbook = new XSSFWorkbook(fis)) {
>
>         Sheet sheet = workbook.getSheet(sheetName);
>         int rows = sheet.getPhysicalNumberOfRows();
>         int cols = sheet.getRow(0).getPhysicalNumberOfCells();
>
>         Object[][] data = new Object[rows - 1][cols];   // skip header
>         for (int i = 1; i < rows; i++) {
>             for (int j = 0; j < cols; j++) {
>                 data[i - 1][j] = getCellValue(sheet.getRow(i).getCell(j));
>             }
>         }
>         return data;
>     }
> }
> ```
>
> **The gotcha with POI:** cell types. A cell containing `12345` comes back as a numeric `Double`, so you get `"12345.0"` when you expect `"12345"`. You need a `getCellValue` helper that switches on cell type and formats numerics properly. That bites everyone once.
>
> `XSSFWorkbook` for `.xlsx`, `HSSFWorkbook` for the older `.xls`.
>
> **Honestly though** — I'd question Excel as a data source in a new framework. It's binary, so diffs in code review are useless and merge conflicts are unresolvable. CSV or JSON are version-controllable and diffable. Excel persists mostly because non-technical stakeholders can edit it, which is a real reason but worth naming as a trade-off rather than a default.
>
> ---
>
> **When data-driven testing is worth it:**
>
> ✅ **Validation rules with many boundary cases** — but at API level, not UI
> ✅ **The same flow across genuinely different user roles or configurations**
> ✅ **Calculation correctness across input ranges**
>
> **When it isn't:**
>
> ❌ **Twenty rows through a full UI flow.** That's twenty browser sessions to test logic that doesn't involve the browser. The same coverage at API level costs seconds.
> ❌ **Data sets that only differ cosmetically.** Three rows testing the same code path is three times the maintenance for no additional coverage.
>
> The framing I'd use: data-driven testing multiplies whatever the test costs. If the test is cheap, that's fine. If it's a ten-second UI flow, twenty rows is over three minutes — and it's usually validating something a 200ms API test would cover better.
>
> That's the same test-level argument as everywhere else, and it applies with more force here because the multiplication makes the wrong choice expensive fast."

---

## Mechanism Comparison

| | `@DataProvider` | `@Parameters` |
|---|---|---|
| Source | Java method | XML suite file |
| Volume | Many rows | One value per run |
| Use for | Test data | Environment config |
| Type-safe | Yes | Strings only |
| Parallel | `parallel = true` | N/A |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "We used data providers for validation scenarios and `@Parameters` from the XML for environment configuration — browser and target environment — so the same suite ran against different setups from Jenkins without code changes.
>
> Test data came from Excel via Apache POI, which was the convention on that stack. The gotcha I'd flag is POI cell types: a numeric cell comes back as a Double, so an ID like 12345 becomes '12345.0' and your lookup fails for reasons that aren't obvious. You need a helper that switches on cell type and formats properly.
>
> If I were building that framework now I'd push for CSV or JSON instead. Excel is binary, so code review diffs are useless and merge conflicts are unresolvable — two people editing test data in the same sprint is a genuine problem. The counter-argument is that non-technical stakeholders can edit Excel, which is real, but it's a trade-off worth stating rather than a default.
>
> The judgement point I'd make is that data-driven testing multiplies whatever the test costs. Twenty rows through a ten-second UI flow is over three minutes, usually to validate logic that doesn't involve the browser at all. That's exactly the case for pushing validation down to the API layer — same coverage, a fraction of the cost, and more stable."

---

## Common Interview Questions & Answers

### Q1: "How do you do data-driven testing in TestNG?"

**A:** "`@DataProvider` returning a two-dimensional Object array — each row becomes a separate test invocation, reported individually so you know which data set failed. For external sources we read Excel via Apache POI into the provider. `@Parameters` from the XML is a different thing — single values for environment config, not test data."

### Q2: "Difference between `@DataProvider` and `@Parameters`?"

**A:** "`@DataProvider` supplies many rows from a Java method and is type-safe. `@Parameters` pulls single string values from the XML suite file. I use data providers for test data and parameters for configuration — browser, environment. Using parameters for test data doesn't work because you only get one value per run."

### Q3: "What's tricky about reading Excel with Apache POI?"

**A:** "Cell types. A numeric cell returns a Double, so an ID of 12345 becomes '12345.0' and your lookups fail for non-obvious reasons. You need a helper that switches on cell type and formats numerics correctly. Also `XSSFWorkbook` for .xlsx versus `HSSFWorkbook` for the older .xls format."

### Q4: "Would you use Excel for test data in a new framework?"

**A:** "Probably not. It's binary, so code review diffs show nothing and merge conflicts can't be resolved — that's a real problem when two people edit test data in the same sprint. CSV or JSON are version-controllable and diffable. The argument for Excel is that non-technical stakeholders can edit it, which is legitimate, but it's a trade-off rather than a default."

### Q5: "Is more data-driven coverage always better?"

**A:** "No, because it multiplies whatever the test costs. Twenty rows through a ten-second UI flow is over three minutes, and it's usually validating logic that has no UI dependency. The same twenty cases at API level run in seconds and are more stable. Data-driven testing is right — mostly at the wrong level when people apply it to UI tests."

---

## Weak Areas to Watch

❌ **Don't use `@Parameters` for test data**
→ One value per run. Wrong tool.

❌ **Don't ignore POI cell types**
→ Numeric cells become Doubles; '12345.0' breaks lookups.

❌ **Don't default to Excel**
→ Binary, undiffable, unmergeable. Name the trade-off.

❌ **Don't run large data sets through UI tests**
→ Multiplies cost for validation that belongs at API level.

❌ **Don't enable `parallel = true` without thread-safe drivers**
→ That's the ThreadLocal problem.

---

## Key Takeaway for Interview

Mechanics are table stakes; the judgement is what's assessed.

The line that shows it:

> **"Data-driven testing multiplies whatever the test costs. Twenty rows through a ten-second UI flow is three minutes, usually validating logic that never touches the browser — the same cases at API level cost seconds and are more stable."**

And the practical detail that proves you've used POI:

> **"Numeric cells come back as Doubles, so an ID of 12345 becomes '12345.0'. You need a cell-type helper, and everyone learns that the hard way once."**

---

## Checklist

✅ `@DataProvider` for test data, `@Parameters` for config
✅ Providers in a separate class once there are several
✅ `@Optional` defaults on parameters
✅ POI cell-type helper implemented
✅ CSV/JSON preferred over Excel where possible
✅ Large data sets pushed to API-level tests
✅ `parallel = true` only with thread-safe driver management

---

**Remember:** Data-driven testing at the wrong level multiplies the wrong cost. 🎯
