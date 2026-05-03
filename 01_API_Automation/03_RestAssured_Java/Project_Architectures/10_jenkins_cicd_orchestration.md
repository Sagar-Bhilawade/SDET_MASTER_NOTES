# 10: Jenkins CI/CD Orchestration

### 1. 🚀 Tutorial Overview
The final stage of the automation journey is **Continuous Execution**. We use **Jenkins**, the leading open-source automation server, to pull our code from GitHub and execute it on a schedule or on every code push. This removes the manual "mvn test" step and ensures the framework is truly integrated into the SDLC.

- **Objective:** Automate test execution using a Jenkins Pipeline.
- **Workflow:** Jenkins -> GitHub -> Maven -> Test Reports.
- **Goal:** Create a "Push-Button" or "Trigger-Based" execution model.

---

### 2. 📌 Core Theoretical Pillars

#### A. The Jenkins Agent Model
Jenkins acts as a "Controller" that manages "Jobs". A Job is a set of instructions:
1. Go to this GitHub URL.
2. Download the latest code.
3. Run `mvn clean test`.
4. Archive the Extent Report.

#### B. Global Tool Configuration (GTC)
For Jenkins to run your code, it needs to know where your system tools are located. You must configure the paths for:
- **JDK:** The Java Development Kit.
- **Maven:** The build tool.
- **Git:** The version control client.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Launching Jenkins
Usually, Jenkins is started as a Windows Service or via a `.war` file:
```bash
java -jar jenkins.war
```
Access Jenkins via `http://localhost:8080`.

#### Phase 2: Global Tool Configuration
Navigate to `Manage Jenkins` -> `Global Tool Configuration`.
1. **JDK:** Add JDK, name it "Java11", and provide the `JAVA_HOME` path.
2. **Git:** Ensure the path to `git.exe` is correct.
3. **Maven:** Add Maven, name it "Maven3", and provide the `MAVEN_HOME` path.

#### Phase 3: Creating a Maven Project (Job)
1. Select `New Item` -> `Maven project`.
2. **Source Code Management:** Select **Git** and enter your GitHub repository URL.
3. **Build Triggers:** (Optional) Select `Poll SCM` to check for changes every hour.
4. **Build:**
   - **Root POM:** `pom.xml`
   - **Goals and options:** `clean test`

#### Phase 4: Execution and Verification
1. Click `Build Now`.
2. Open the **Console Output** to see the real-time logs (exactly like your local terminal).
3. **Post-Build Actions:** Configure Jenkins to archive `reports/*.html` so you can view results directly in the browser.

---

### 4. 📊 Technical Component Encyclopedia

| Component | Responsibility | Config Key |
| :--- | :--- | :--- |
| **Jenkins Dashboard**| Managing and monitoring all test jobs. | UI |
| **GTC** | Mapping Jenkins to system binary paths. | Manage Jenkins |
| **Build History** | Tracking the pass/fail status of every run. | Left Sidebar |
| **Console Output** | Debugging failed builds via CLI logs. | Build Level |

---

### 5. 🏗️ Advanced CI/CD Patterns

#### A. Webhooks for Instant Testing
Instead of "Polling", you can configure a **GitHub Webhook**. This sends a signal to Jenkins the exact second you `git push`, triggering an immediate "Sanity Test."

#### B. Pipeline as Code (Jenkinsfile)
Instead of a "Freestyle" project, senior SDETs use a `Jenkinsfile`. This is a text file inside your project that defines the entire build pipeline in Groovy script.
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'mvn clean compile' }
        }
        stage('Test') {
            steps { sh 'mvn test' }
        }
    }
}
```

---

### 🛡️ Content Preservation Note
- Ensure the Jenkins user has permission to access the local folder or remote GitHub repository.
- If using Private Repositories, you must add "Credentials" (Username/Password or Token) in Jenkins.
