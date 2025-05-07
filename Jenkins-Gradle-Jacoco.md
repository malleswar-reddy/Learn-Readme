# WebFluxTodo CI/CD Pipeline

This repository contains a Jenkins pipeline for building, testing, and packaging the `WebFluxTodo` project, a multi-module Spring Boot application using WebFlux. The pipeline automates the process of checking out code, building the project, running tests, generating code coverage reports, and archiving artifacts.

## Prerequisites

- **Jenkins Server**:
  - Install Jenkins (version 2.426.3 or later recommended).
  - Install the following plugins:
    - Git Plugin
    - Gradle Plugin
    - HTML Publisher Plugin
    - JaCoCo Plugin (optional, for code coverage visualization).
  - Configure a JDK named `JDK` in `Manage Jenkins > Global Tool Configuration`.
- **Repository**:
  - The repository is hosted at `https://github.com/malleswar-reddy/WebFluxTodo.git`.
  - Ensure the `devlop` branch exists (or update to `develop` if renamed).
- **Gradle**:
  - The repository includes a Gradle wrapper (`gradlew`). Ensure it’s executable (`chmod +x gradlew`).
- **Java**: Java 17 is required, as specified in the project’s `build.gradle`.

## Pipeline Overview

The Jenkins pipeline (`Jenkinsfile`) performs the following stages:

1. **Checkout**:
   - Clones the `devlop` branch from `https://github.com/malleswar-reddy/WebFluxTodo.git`.
2. **Build**:
   - Runs `./gradlew clean build -x test --no-daemon` to build the project without running tests.
3. **Test**:
   - Runs tests for `CommonService` and `UserManagement` modules in parallel using `./gradlew :Module:test --no-daemon`.
4. **Coverage Report**:
   - Generates JaCoCo code coverage reports using `./gradlew jacocoTestReport --no-daemon`.
5. **Package**:
   - Creates executable JARs using `./gradlew bootJar --no-daemon`.
   - Archives JARs in Jenkins for later use.
6. **Post-Build**:
   - Publishes JUnit test reports for `UserManagement`.
   - Publishes JaCoCo coverage reports for `CommonService`.

## Setup Instructions

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/malleswar-reddy/WebFluxTodo.git
   cd WebFluxTodo
   ```

2. **Configure Jenkins**:
   - Create a new pipeline job in Jenkins:
     - Go to `New Item`, select `Pipeline`, and name it (e.g., `WebFluxTodo`).
     - In the **Pipeline** section, set:
       - **Definition**: Pipeline script from SCM.
       - **SCM**: Git.
       - **Repository URL**: `https://github.com/malleswar-reddy/WebFluxTodo.git`.
       - **Branch Specifier**: `*/devlop`.
       - **Script Path**: `Jenkinsfile`.
     - Save the configuration.

3. **Run the Pipeline**:
   - Click `Build Now` to trigger the pipeline.
   - Monitor the console output for progress and errors.
   - View test and coverage reports in the Jenkins UI under the job’s build history.

## Example

Below is an example of the expected pipeline execution in Jenkins:

```
Started by user Malleswar Reddy
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/WebFluxTodo
[Pipeline] stage (Checkout)
Fetching changes from https://github.com/malleswar-reddy/WebFluxTodo.git
Checking out branch devlop
[Pipeline] stage (Build)
Executing: ./gradlew clean build -x test --no-daemon
BUILD SUCCESSFUL
[Pipeline] stage (Test)
[Pipeline] parallel
[Branch: CommonService Tests] Executing: ./gradlew :CommonService:test --no-daemon
[Branch: UserManagement Tests] Executing: ./gradlew :UserManagement:test --no-daemon
Tests completed successfully
[Pipeline] stage (Coverage Report)
Executing: ./gradlew jacocoTestReport --no-daemon
JaCoCo report generated
[Pipeline] stage (Package)
Executing: ./gradlew bootJar --no-daemon
Archiving artifacts: **/build/libs/*.jar
[Pipeline] post
Publishing UserManagement Test Report
Publishing CommonService JaCoCo Coverage Report
Finished: SUCCESS
```

## Troubleshooting

- **Branch Not Found**:
  - Ensure the `devlop` branch exists. Check with:
    ```bash
    git ls-remote https://github.com/malleswar-reddy/WebFluxTodo.git
    ```
  - If using `develop`, create and push it:
    ```bash
    git checkout -b develop
    git push origin develop
    ```
- **Missing Reports**:
  - Verify that `UserManagement/build/reports/tests/test/index.html` and `CommonService/build/reports/jacoco/test/html/index.html` are generated.
  - Run `./gradlew test jacocoTestReport` locally to debug.
- **Gradle Issues**:
  - Ensure `gradlew` is executable (`chmod +x gradlew`).
  - Check Gradle version compatibility (use the wrapper’s version).
- **Jenkins Configuration**:
  - Confirm the JDK is named `JDK` in Global Tool Configuration.
  - Install required plugins (Git, Gradle, HTML Publisher).

## References

- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Gradle Documentation](https://docs.gradle.org/current/userguide/userguide.html)
- [JaCoCo Plugin for Jenkins](https://plugins.jenkins.io/jacoco/)
- [HTML Publisher Plugin](https://plugins.jenkins.io/htmlpublisher/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [WebFluxTodo Repository](https://github.com/malleswar-reddy/WebFluxTodo.git)

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.