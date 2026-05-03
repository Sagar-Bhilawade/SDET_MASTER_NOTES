# 08: OS-Level Maven Configuration and CLI Execution

### 1. 🚀 Tutorial Overview
To move from local execution to a truly professional SDET workflow, you must be able to execute your framework from the **Command Line (CLI)**. This is the prerequisite for all CI/CD pipelines. This tutorial covers the installation of Maven on your OS (Windows) and the execution of tests via the terminal.

- **Objective:** Install Maven and execute the framework outside the IDE.
- **Workflow:** Download -> Environment Variables -> CLI Verification -> `mvn test`.
- **System:** Focused on Windows environment configuration.

---

### 2. 🛠️ Step-by-Step Implementation

#### Phase 1: Installing Apache Maven on Windows
1. **Download:** Go to [maven.apache.org](https://maven.apache.org/download.cgi) and download the **Binary zip archive**.
2. **Extract:** Unzip the folder to a permanent location (e.g., `C:\maven\apache-maven-3.x.x`).
3. **Environment Variables:**
   - **MAVEN_HOME:** Create a new System Variable named `MAVEN_HOME` and set it to the path where you extracted Maven.
   - **Path:** Edit the `Path` variable in System Variables and add `%MAVEN_HOME%\bin`.

#### Phase 2: Verifying the Installation
Open a fresh Command Prompt or PowerShell and run:
```bash
mvn -version
```
**Success Criteria:** You should see the Maven version, Java version, and OS details. If you see "'mvn' is not recognized," your Path variable is likely incorrect.

#### Phase 3: Executing Tests via CLI
Navigate to your project root (where `pom.xml` is located) in the terminal and run:

```bash
mvn test
```

**What happens behind the scenes?**
1. Maven reads `pom.xml`.
2. It compiles the code.
3. It triggers the Surefire plugin.
4. Surefire reads `testng.xml`.
5. The tests execute, and results appear in the console.

---

### 3. 🔍 Troubleshooting Common CLI Failures

#### A. Compilation/Version Mismatch
**Error:** `UnsupportedClassVersionError` or `javac` not found.
**Fix:** Ensure your `JAVA_HOME` environment variable is set to the same Java version specified in your `pom.xml` compiler plugin.

#### B. Dependency Download Failures
**Error:** `Could not resolve dependencies...`
**Fix:** Usually caused by network issues or restricted company firewalls. Ensure you have an internet connection and check your `settings.xml` if you are behind a proxy.

#### C. POM Not Found
**Error:** `The goal you specified requires a project to execute but there is no POM in this directory.`
**Fix:** Ensure you are in the directory that contains the `pom.xml` file. Use `ls` or `dir` to verify.

---

### 4. 📊 Technical Component Encyclopedia

| Command | Purpose | When to Use |
| :--- | :--- | :--- |
| `mvn clean` | Deletes the `target/` folder (old results/classes). | Before a fresh test run. |
| `mvn compile` | Compiles the source code without running tests. | To check for syntax errors. |
| `mvn test` | Runs the full test suite defined in `pom.xml`. | The standard execution command. |
| `mvn -DsuiteXmlFile=name.xml test` | Passes a dynamic property to the execution. | For filtered test runs. |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. The "Clean Test" Pattern
In CI/CD (Jenkins/Azure DevOps), we always use `mvn clean test`. The `clean` ensures that old reports or cached classes from previous failed builds don't interfere with the current run.

#### B. Headless Execution
When running via CLI, we often configure the framework to run in "Headless" mode (for UI tests) or ensure that all logging is directed to the console for the CI tool to capture in the build logs.

---

### 🛡️ Content Preservation Note
- Never share your `settings.xml` if it contains private credentials for a Nexus/Artifactory repository.
- Always restart your terminal after changing Environment Variables for the changes to take effect.
