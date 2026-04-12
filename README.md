# Projects-trainers
To put the projects on which thee trainers are supposed to do

Voici le projet complet à donner à ton enseignant 👇

***

# 📦 Projet à réaliser — Java + Maven/Gradle + SonarQube

**Deadline : fin de semaine prochaine**
**Validation requise avant de préparer le cours**

***

## 🎯 Contexte

Tu dois créer une petite application Java, la builder avec Maven **et** Gradle, écrire des tests unitaires, puis analyser la qualité du code avec SonarQube. C'est exactement ce que les apprenants devront faire dans leur pipeline CI/CD.

***

## 📁 L'application à créer

Une **calculatrice simple en Java** avec les opérations : addition, soustraction, multiplication, division.

### Structure attendue

```
java-sonar-project/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/devops/calculator/
│   │           └── Calculator.java
│   └── test/
│       └── java/
│           └── com/devops/calculator/
│               └── CalculatorTest.java
├── pom.xml          ← Maven
├── build.gradle     ← Gradle
└── sonar-project.properties
```

### Calculator.java

```java
package com.devops.calculator;

public class Calculator {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }

    public double divide(int a, int b) {
        if (b == 0) {
            throw new IllegalArgumentException("Division par zéro impossible");
        }
        return (double) a / b;
    }
}
```

### CalculatorTest.java

```java
package com.devops.calculator;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {

    Calculator calc = new Calculator();

    @Test
    public void testAdd() {
        assertEquals(10, calc.add(3, 7));
    }

    @Test
    public void testSubtract() {
        assertEquals(5, calc.subtract(10, 5));
    }

    @Test
    public void testMultiply() {
        assertEquals(12, calc.multiply(3, 4));
    }

    @Test
    public void testDivide() {
        assertEquals(2.5, calc.divide(5, 2));
    }

    @Test
    public void testDivideByZero() {
        assertThrows(IllegalArgumentException.class, () -> calc.divide(5, 0));
    }
}
```

***

## ⚙️ Partie 1 — Maven (pom.xml)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <groupId>com.devops</groupId>
  <artifactId>calculator</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <sonar.projectKey>java-sonar-project</sonar.projectKey>
    <sonar.host.url>http://localhost:9000</sonar.host.url>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- Plugin pour les tests -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.1.2</version>
      </plugin>

      <!-- Plugin SonarQube -->
      <plugin>
        <groupId>org.sonarsource.scanner.maven</groupId>
        <artifactId>sonar-maven-plugin</artifactId>
        <version>3.10.0.2594</version>
      </plugin>

      <!-- Plugin couverture de code JaCoCo -->
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.10</version>
        <executions>
          <execution>
            <goals><goal>prepare-agent</goal></goals>
          </execution>
          <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

***

## ⚙️ Partie 2 — Gradle (build.gradle)

```groovy
plugins {
    id 'java'
    id 'jacoco'
    id 'org.sonarqube' version '4.4.1.3373'
}

group = 'com.devops'
version = '1.0.0'

java {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
}

test {
    useJUnitPlatform()
    finalizedBy jacocoTestReport
}

jacocoTestReport {
    dependsOn test
    reports {
        xml.required = true
    }
}

sonar {
    properties {
        property 'sonar.projectKey', 'java-sonar-project'
        property 'sonar.host.url', 'http://localhost:9000'
        property 'sonar.coverage.jacoco.xmlReportPaths',
                 'build/reports/jacoco/test/jacocoTestReport.xml'
    }
}
```

***

## 🔍 Partie 3 — SonarQube

### Lancer SonarQube avec Docker (sur son compte AWS EC2)

```bash
# Lancer SonarQube
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:community

# Attendre 1-2 min puis ouvrir :
# http://<IP-EC2>:9000
# Login par défaut : admin / admin
```

### Analyser avec Maven

```bash
mvn clean verify sonar:sonar \
  -Dsonar.token=TON_TOKEN_SONAR
```

### Analyser avec Gradle

```bash
./gradlew test sonarqube \
  -Dsonar.token=TON_TOKEN_SONAR
```

***

## ✅ Critères de validation du projet

| Critère | Attendu |
|---|---|
| Application compile | ✅ sans erreur |
| Tests unitaires | ✅ 5 tests passent |
| Couverture de code | ≥ 80% |
| Analyse Maven → SonarQube | ✅ rapport visible |
| Analyse Gradle → SonarQube | ✅ rapport visible |
| Zéro bug critique SonarQube | ✅ Quality Gate passé |
| Code poussé sur GitHub | ✅ repo public ou privé partagé |

***

## 📬 Ce qu'il doit te rendre

1. **Lien GitHub** du projet
2. **Screenshot SonarQube** — Quality Gate ✅
3. **Screenshot couverture de code** ≥ 80%
4. **Un court résumé** (5-10 lignes) : ce qu'il a compris, ce qui était difficile

***


