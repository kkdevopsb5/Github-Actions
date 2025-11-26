# Simple CI Build and Tomcat Deployment Workflow

This workflow demonstrates the basic CI pipeline using GitHub Actions:

1. Checkout source code
2. Set up Java
3. Build the application using Maven
4. Deploy the generated WAR file to Apache Tomcat Manager

This is the foundation for understanding CI/CD before adding Sonar, Nexus, or advanced stages.

---

## What this Workflow Does

### 1. Checkout the Code

GitHub Actions first pulls your project code from your repository using:

```
uses: actions/checkout@v4
```

This step brings all your files (including pom.xml, src, etc.) into the runner machine so Maven can build the project.

---

### 2. Setup Java

We install Temurin JDK 17 on the GitHub runner:

```
uses: actions/setup-java@v4
with:
  distribution: temurin
  java-version: 17
  cache: maven
```

This ensures Maven can run and the project builds consistently on any machine.

---

### 3. Build the Application

We run Maven to compile the project and generate the WAR file:

```
mvn -B clean package
```

The WAR file will be stored under:

```
target/*.war
```

This WAR file is what we send to Tomcat.

---

### 4. Deploy to Tomcat

The workflow uploads the generated WAR file to Tomcat Manager using curl:

```
curl -u USER:PASS --upload-file file.war "http://HOST:8080/manager/text/deploy?path=/maven-web-application&update=true"
```

This requires three secrets stored in your repository:

| Secret Name     | Purpose                                                                        |
| --------------- | ------------------------------------------------------------------------------ |
| TOMCAT_URL      | Tomcat server URL (ex: [http://3.108.194.157:8080](http://3.108.194.157:8080)) |
| TOMCAT_USERNAME | Tomcat manager username                                                        |
| TOMCAT_PASSWORD | Tomcat manager password                                                        |

These secrets keep your credentials safe.

The workflow automatically picks the newest WAR:

```
WAR=$(ls target/*.war | head -n 1)
```

Then deploys it to Tomcat Manager.

---

## Complete Workflow File

Place this file inside:

```
.github/workflows/simple-deploy.yml
```

```yaml
name: Simple Build and Deploy
on:
  workflow_dispatch:

jobs:
  full:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17
          cache: maven

      - run: mvn -B clean package

      - name: Deploy WAR to Tomcat
        env:
          TOMCAT_URL: ${{ secrets.TOMCAT_URL }}
          TOMCAT_USER: ${{ secrets.TOMCAT_USERNAME }}
          TOMCAT_PASS: ${{ secrets.TOMCAT_PASSWORD }}
        run: |
          WAR=$(ls target/*.war | head -n 1)
          curl -sS -u "${TOMCAT_USER}:${TOMCAT_PASS}" --upload-file "$WAR" "${TOMCAT_URL}/manager/text/deploy?path=/maven-web-application&update=true"
```

---

## How to Run This Workflow

1. Add TOMCAT_URL, TOMCAT_USERNAME, TOMCAT_PASSWORD to GitHub Secrets
2. Push this YAML file into `.github/workflows/` folder
3. Go to **Actions** tab in GitHub
4. Select **Simple Build and Deploy** workflow
5. Click **Run workflow**

This will:

* Build your Java project
* Generate WAR
* Deploy to Tomcat

