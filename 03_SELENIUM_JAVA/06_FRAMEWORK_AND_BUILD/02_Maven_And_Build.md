# Maven - Build Management for Test Frameworks

## What This Covers

Maven's role in a Java automation framework — dependencies, lifecycle, profiles, and running suites from the command line.

---

## Why It Matters (Interview Context)

Maven questions check whether you understood the build or just ran `mvn test` because someone told you to.

The specific things interviewers probe: dependency scopes, how you run different suites, and how environment switching works without code changes.

---

## Your 30-Second Interview Answer

> "Maven handles dependencies and the build lifecycle. For a test framework the parts that matter are the surefire plugin pointing at TestNG suite XMLs, profiles for switching environment or suite from the command line, and `test` scope on testing dependencies so they don't leak into a production artifact. Running `mvn test -Psmoke -Denv=staging` is how Jenkins triggers different runs without any code change."

---

## Your 2-Minute Deep Answer

> "**Core dependencies:**
>
> ```xml
> <dependencies>
>     <dependency>
>         <groupId>org.seleniumhq.selenium</groupId>
>         <artifactId>selenium-java</artifactId>
>         <version>4.27.0</version>
>     </dependency>
>     <dependency>
>         <groupId>org.testng</groupId>
>         <artifactId>testng</artifactId>
>         <version>7.10.2</version>
>         <scope>test</scope>
>     </dependency>
>     <dependency>
>         <groupId>io.rest-assured</groupId>
>         <artifactId>rest-assured</artifactId>
>         <version>5.5.0</version>
>         <scope>test</scope>
>     </dependency>
>     <dependency>
>         <groupId>org.apache.poi</groupId>
>         <artifactId>poi-ooxml</artifactId>
>         <version>5.3.0</version>
>     </dependency>
> </dependencies>
> ```
>
> **Dependency scopes** — a standard question:
> - `compile` — default, available everywhere
> - `test` — only for test compilation and execution, excluded from the packaged artifact
> - `provided` — needed to compile but supplied at runtime by the container
> - `runtime` — needed at runtime but not to compile, like a JDBC driver
>
> Testing libraries should be `test` scope so they don't end up in a production artifact.
>
> ---
>
> **Surefire — running TestNG suites:**
>
> ```xml
> <plugin>
>     <groupId>org.apache.maven.plugins</groupId>
>     <artifactId>maven-surefire-plugin</artifactId>
>     <version>3.5.2</version>
>     <configuration>
>         <suiteXmlFiles>
>             <suiteXmlFile>${suiteFile}</suiteXmlFile>
>         </suiteXmlFiles>
>         <systemPropertyVariables>
>             <env>${env}</env>
>             <browser>${browser}</browser>
>         </systemPropertyVariables>
>     </configuration>
> </plugin>
> ```
>
> The parameterized `${suiteFile}` is what makes this flexible — Jenkins picks the suite rather than the pom hardcoding it.
>
> **Surefire versus Failsafe** is worth knowing: Surefire runs unit tests in the `test` phase and fails the build immediately. Failsafe runs integration tests in `integration-test` and defers failure to `verify`, so teardown in `post-integration-test` still runs. For UI suites Failsafe is arguably more correct, though Surefire is what most Selenium frameworks actually use.
>
> ---
>
> **Profiles — environment and suite switching:**
>
> ```xml
> <profiles>
>     <profile>
>         <id>smoke</id>
>         <properties>
>             <suiteFile>src/test/resources/suites/smoke.xml</suiteFile>
>         </properties>
>     </profile>
>     <profile>
>         <id>regression</id>
>         <activation><activeByDefault>true</activeByDefault></activation>
>         <properties>
>             <suiteFile>src/test/resources/suites/regression.xml</suiteFile>
>         </properties>
>     </profile>
> </profiles>
> ```
>
> ```bash
> mvn test -Psmoke -Denv=staging -Dbrowser=firefox
> ```
>
> One command controls which tests run, against which environment, in which browser — no code change. That's what makes the same framework serve a per-commit smoke job and a nightly regression job.
>
> ---
>
> **The lifecycle** — asked directly sometimes:
>
> `validate → compile → test → package → verify → install → deploy`
>
> Each phase runs all preceding phases. `mvn package` runs tests first, which is why `-DskipTests` exists.
>
> **`clean` is a separate lifecycle**, which is why you see `mvn clean test` — clean isn't part of the default sequence.
>
> ---
>
> **Version management:**
>
> ```xml
> <properties>
>     <selenium.version>4.27.0</selenium.version>
>     <testng.version>7.10.2</testng.version>
>     <maven.compiler.source>17</maven.compiler.source>
>     <maven.compiler.target>17</maven.compiler.target>
> </properties>
> ```
>
> Versions as properties keeps upgrades to one place. And `dependency:tree` is the command worth knowing — it shows transitive dependencies, which is how you diagnose version conflicts when two libraries pull incompatible versions of the same thing. That's a real problem in Java projects and the tree is how you find it.
>
> ---
>
> **Maven versus Gradle, if it comes up:**
>
> Maven's XML is verbose but declarative and predictable — anyone can read a pom. Gradle's Groovy or Kotlin DSL is more concise and flexible, with better incremental build performance. For test frameworks Maven is still the more common convention and the predictability is worth something; Gradle is the better choice if the build needs genuine custom logic."

---

## Command Reference

| Command | Does |
|---------|------|
| `mvn clean test` | Clean, compile, run tests |
| `mvn test -Psmoke` | Run with the smoke profile |
| `mvn test -Denv=staging` | Pass a system property |
| `mvn test -Dtest=LoginTest` | Single test class |
| `mvn clean install -DskipTests` | Build without running tests |
| `mvn dependency:tree` | Diagnose version conflicts |

---

## Your Real Project Example (NielsenIQ)

**How to frame it:**

> "Maven managed the build with profiles controlling which suite ran and against which environment. Jenkins invoked `mvn test -Psmoke -Denv=staging` for the deployment-triggered job and the regression profile for the scheduled run — so the same framework served both jobs with no code change, just a different command line.
>
> The surefire configuration pointed at a parameterized suite file rather than a hardcoded one, which is what makes that flexibility possible. Hardcode the suite in the pom and you need a code change to run a different set of tests, which defeats the point.
>
> Scopes mattered for keeping the artifact clean — TestNG and RestAssured as `test` scope so they don't leak into anything packaged.
>
> The command I'd single out as most useful in practice is `dependency:tree`. Version conflicts from transitive dependencies are a genuine problem in Java projects — two libraries pulling incompatible versions of the same thing produces errors that point nowhere useful. The tree is how you actually find it."

---

## Common Interview Questions & Answers

### Q1: "What does Maven do in your framework?"

**A:** "Dependency management and build lifecycle. For test frameworks specifically, the surefire plugin points at TestNG suite XMLs, and profiles let Jenkins switch which suite runs and against which environment from the command line — so one framework serves a smoke job and a regression job without a code change."

### Q2: "What are dependency scopes?"

**A:** "`compile` is the default and available everywhere. `test` is only for test compilation and execution and is excluded from the packaged artifact — that's where TestNG and RestAssured belong. `provided` means the container supplies it at runtime. `runtime` means needed at runtime but not to compile, like a JDBC driver."

### Q3: "How do you run different test suites?"

**A:** "Maven profiles setting a `suiteFile` property that surefire reads. `mvn test -Psmoke` runs the smoke suite, `-Pregression` the full one. Combined with `-Denv=staging` and `-Dbrowser=firefox`, one command controls what runs, where, and in which browser."

### Q4: "Surefire or Failsafe?"

**A:** "Surefire runs in the `test` phase and fails the build immediately. Failsafe runs in `integration-test` and defers failure to `verify`, so post-integration teardown still executes. For UI suites Failsafe is arguably more correct because of that teardown guarantee, though Surefire is what most Selenium frameworks actually use."

### Q5: "How do you resolve a dependency version conflict?"

**A:** "`mvn dependency:tree` to see what's pulling which version transitively. Then either exclude the unwanted transitive dependency or pin the version explicitly in `dependencyManagement`. It's worth knowing because the resulting runtime errors — usually `NoSuchMethodError` — point nowhere near the actual cause."

---

## Weak Areas to Watch

❌ **Don't hardcode the suite file in the pom**
→ Defeats the point of profiles.

❌ **Don't use `compile` scope for test libraries**
→ They leak into the packaged artifact.

❌ **Don't forget `clean` is a separate lifecycle**
→ Explains `mvn clean test`.

❌ **Don't ignore `dependency:tree`**
→ It's the tool for the hardest Maven problem.

---

## Key Takeaway for Interview

Maven questions check whether you understood the build or just ran the command.

The capability that matters:

> **"Profiles plus a parameterized suite file mean one command controls what runs, where, and in which browser. Jenkins runs smoke on deployment and regression on a schedule from the same framework, with no code change."**

And the practical depth:

> **"`dependency:tree` for version conflicts. Two libraries pulling incompatible versions produces a `NoSuchMethodError` that points nowhere near the cause — the tree is how you actually find it."**

---

## Checklist

✅ Test libraries in `test` scope
✅ Surefire pointed at a parameterized suite file
✅ Profiles for suite and environment switching
✅ Versions centralized in `<properties>`
✅ Can state the lifecycle phases
✅ Know Surefire vs Failsafe
✅ Comfortable with `dependency:tree`

---

**Remember:** If changing which tests run requires a code change, the build isn't doing its job. 🎯
