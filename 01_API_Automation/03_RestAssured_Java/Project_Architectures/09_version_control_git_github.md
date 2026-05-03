# 09: Version Control with Git and GitHub

### 1. 🚀 Tutorial Overview
Individual scripts are local, but a **Framework** is a collaborative asset. To move into a Continuous Integration (CI) model, we must first establish a "Source of Truth" using Version Control. This tutorial covers the transition from a local workspace to a global GitHub repository.

- **Objective:** Master the Git lifecycle (Init -> Stage -> Commit -> Push).
- **Core Concepts:** Local vs Remote repositories, Distributed Version Control.
- **Troubleshooting:** Resolving JDK 11 and Maven compatibility issues for CLI execution.

---

### 2. 📌 Core Theoretical Pillars

#### A. The CI/CD Workflow
In a professional ecosystem:
1. **Developer:** Pushes application code to a feature branch.
2. **Tester (SDET):** Pushes automation framework updates to the automation repository.
3. **DevOps/Jenkins:** Detects changes and triggers the build/test pipeline.
4. **Git/GitHub:** Acts as the central hub for all code synchronization.

#### B. The Three States of Git
Before code reaches GitHub, it moves through three local zones:
1. **Working Directory:** Files you are currently editing.
2. **Staging Area (Index):** A "buffer" where you prepare a snapshot of changes.
3. **Local Repository (.git):** The permanent history of your commits.

#### C. Environment Compatibility (JDK 11)
A common failure in CLI execution is a mismatch between the IDE's internal JDK and the System JDK. 
- **Requirement:** Ensure `JAVA_HOME` points to JDK 11+ and matches the version in `pom.xml`.
- **Verification:** `mvn -version` must show the correct JDK path.

---

### 3. 🛠️ Step-by-Step Implementation

#### Phase 1: Local Repository Initialization
Navigate to your project root in the terminal and execute:

```bash
# 1. Initialize Git in the project root
git init

# 2. Check current status
git status
```

#### Phase 2: Staging and Committing
Prepare your code for its first version.

```bash
# 1. Add all files to the staging area
git add .

# 2. Commit the changes with a descriptive message
git commit -m "Initial commit: RestAssured Layered Framework with Logging & Reporting"
```

#### Phase 3: Connecting to GitHub
1. Create a new repository on [GitHub](https://github.com).
2. Copy the Remote URL (HTTPS or SSH).

```bash
# 1. Add the remote origin
git remote add origin https://github.com/yourusername/your-repo-name.git

# 2. Rename the branch to main (standard)
git branch -M main

# 3. Push the code to the remote server
git push -u origin main
```

#### Phase 4: Managing Changes (Updates & Deletions)
If you delete a file or modify code, you must sync the remote:
```bash
git add .
git commit -m "Refined UserEndpoints and updated README"
git push origin main
```

---

### 4. 📊 Technical Component Encyclopedia

| Command | Action | Key Use Case |
| :--- | :--- | :--- |
| `git init` | Creates a `.git` folder. | Start of a new project. |
| `git add .` | Moves changes to "Staged". | Preparing a commit. |
| `git commit` | Records the snapshot. | Saving a version. |
| `git push` | Uploads to the cloud. | Sharing code / CI trigger. |
| `git pull` | Downloads from the cloud. | Syncing with team members. |

---

### 5. 🏭 Real-World Industrial Patterns

#### A. The .gitignore File
Professional frameworks never upload binary files or local configs. Always include a `.gitignore` file in your root:
```text
target/
.idea/
.settings/
.classpath/
*.log
reports/
```

#### B. Meaningful Commit Messages
Avoid "Updated code." Use the **Conventional Commits** pattern:
- `feat(user): add delete endpoint implementation`
- `fix(reporting): resolve NPE in ExtentManager`
- `docs: update CI/CD setup instructions`

---

### 🛡️ Content Preservation Note
- Never push sensitive data (API Keys, Passwords) to a public GitHub repository.
- Use `git remote -v` to verify which remote repository your local project is linked to.
