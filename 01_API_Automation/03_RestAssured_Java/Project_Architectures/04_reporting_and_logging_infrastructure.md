# 04: Reporting and Logging Infrastructure

### 1. 🚀 Tutorial Overview
A framework is only as good as its visibility. Without proper reporting and logging, debugging failed CI/CD pipelines becomes a nightmare. This tutorial covers the integration of **Extent Reports** for high-level visual reporting and **Log4j2** for granular execution logs.

- **Objective:** Implement automated HTML reports and thread-safe logging.
- **Reporting:** Extent Reports v5+ with TestNG Listeners.
- **Logging:** Apache Log4j2 with custom configuration.

---

### 2. 📌 Core Theoretical Pillars

#### A. TestNG Listeners
Listeners are "observers" that wait for specific events (Test Start, Test Success, Test Failure) and trigger actions. We use them to automatically update the Extent Report without cluttering the test methods.

#### B. The Log4j2 Architecture
Unlike `System.out.println`, Log4j2 allows:
1. **Log Levels:** Filtering output (Info, Debug, Error, Fatal).
2. **Appenders:** Sending logs to both the Console and a `.log` file simultaneously.
3. **Performance:** Asynchronous logging that doesn't slow down test execution.

#### C. HTML Report Generation
Extent Reports provide a dashboard view of test execution, including pass/fail percentages, execution time, and system environment details.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Dependency Configuration
Add the reporting and logging libraries to `pom.xml`.

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Extent Reports -->
    <dependency>
        <groupId>com.aventstack</groupId>
        <artifactId>extentreports</artifactId>
        <version>5.0.9</version>
    </dependency>
    <!-- Log4j2 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.19.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>        <version>2.19.0</version>
    </dependency>
</dependencies>
```

#### Phase 2: Extent Report Manager (Listener)
Create a class that implements `ITestListener`.

```java
// api.utils.ExtentReportManager.java
public class ExtentReportManager implements ITestListener {
    public ExtentSparkReporter sparkReporter;
    public ExtentReports extent;
    public ExtentTest test;

    public void onStart(ITestContext testContext) {
        String repName = "Test-Report-" + new SimpleDateFormat("yyyy.MM.dd.HH.mm.ss").format(new Date()) + ".html";
        sparkReporter = new ExtentSparkReporter(".\\reports\\" + repName);

        sparkReporter.config().setDocumentTitle("RestAssuredAutomationProject");
        sparkReporter.config().setReportName("Pet Store Users API");
        sparkReporter.config().setTheme(Theme.DARK);

        extent = new ExtentReports();
        extent.attachReporter(sparkReporter);
        extent.setSystemInfo("Application", "Pet Store Users API");
        extent.setSystemInfo("Operating System", System.getProperty("os.name"));
        extent.setSystemInfo("User Name", System.getProperty("user.name"));
        extent.setSystemInfo("Environment", "QA");
    }

    public void onTestSuccess(ITestResult result) {
        test = extent.createTest(result.getName());
        test.assignCategory(result.getMethod().getGroups());
        test.createNode(result.getName());
        test.log(Status.PASS, "Test Passed");
    }

    public void onTestFailure(ITestResult result) {
        test = extent.createTest(result.getName());
        test.createNode(result.getName());
        test.assignCategory(result.getMethod().getGroups());
        test.log(Status.FAIL, "Test Failed");
        test.log(Status.FAIL, result.getThrowable().getMessage());
    }

    public void onFinish(ITestContext testContext) {
        extent.flush();
    }
}
```

#### Phase 3: Log4j2 Configuration
Create `log4j2.xml` in `src/test/resources`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
    <File name="MyFile" fileName="logs/automation.log">
      <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n"/>
    </File>
  </Appenders>
  <Loggers>
    <Root level="info">
      <AppenderRef ref="Console"/>
      <AppenderRef ref="MyFile"/>
    </Root>
  </Loggers>
</Configuration>
```

#### Phase 4: Integrating Logs into Tests
Initialize the logger in the test class.

```java
// api.tests.UserTests.java
public class UserTests {
    public Logger logger;

    @BeforeClass
    public void setupData() {
        // ... previous setup ...
        
        // Initialize Log4j2 logger
        logger = LogManager.getLogger(this.getClass());
    }

    @Test(priority = 1)
    public void testPostUser() {
        logger.info("********** Creating User **********");
        Response response = UserEndpoints.createUser(userPayload);
        response.then().log().all();
        Assert.assertEquals(response.getStatusCode(), 200);
        logger.info("********** User Created Successfully **********");
    }
}
```

#### Phase 5: TestNG XML Configuration
The listener must be registered in the `testng.xml` file.

```xml
<!-- testng.xml -->
<suite name="PetSuite">
    <listeners>
        <listener class-name="api.utils.ExtentReportManager" />
    </listeners>
    <test name="PetTest">
        <classes>
            <class name="api.tests.UserTests" />
        </classes>
    </test>
</suite>
```

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Pattern Used |
| :--- | :--- | :--- |
| **ExtentSparkReporter** | Styling and physical file generation. | Decorator |
| **ITestListener** | Hooking into TestNG lifecycle. | Observer |
| **Appenders** | Directing log flow (File/Console). | Strategy |
| **Log Levels** | Controlling verbosity (INFO/DEBUG/ERROR). | Filter |

---

### 5. 🏗️ Framework Best Practices

#### A. Reusability Across Modules
The `ExtentReportManager` and `log4j2.xml` are global. Once implemented, they work for `UserTests`, `PetTests`, and `StoreTests` without any code changes in the utilities.

#### B. Log-Driven Debugging
Always log the start and end of a test, and if an assertion fails, log the specific reason. This allows you to identify the bottleneck in seconds by looking at the `.log` file.

---

### 🛡️ Content Preservation Note
- Ensure `log4j2.xml` is in the classpath (`src/test/resources`).
- Always call `extent.flush()` in the `onFinish` method, otherwise the report will not be generated.
