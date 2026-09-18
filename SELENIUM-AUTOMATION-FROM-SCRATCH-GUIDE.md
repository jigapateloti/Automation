# Selenium Automation Framework: From Scratch Setup & Core Java Learning Guide

> **Target Audience:** QA Engineers and Developers looking to understand the mechanics of enterprise Selenium frameworks, build a complete framework from a blank slate without relying solely on black-box SDKs, and master the Java and AI-assisted concepts required to replicate this architecture across any project.

---

## Table of Contents

1. [Executive Summary: The Big Picture](#1-executive-summary-the-big-picture)
2. [Anatomy of THIS Project: What Every File Does](#2-anatomy-of-this-project-what-every-file-does)
3. [The Core Java Concepts You Must Learn](#3-the-core-java-concepts-you-must-learn)
   - [3.1 Object-Oriented Programming (OOP) Applied to Automation](#31-object-oriented-programming-oop-applied-to-automation)
   - [3.2 Java Collections Framework](#32-java-collections-framework)
   - [3.3 Exception Handling in WebDriver](#33-exception-handling-in-webdriver)
   - [3.4 Annotations and Java Reflection](#34-annotations-and-java-reflection)
   - [3.5 File I/O and Property Files](#35-file-io-and-property-files)
   - [3.6 Concurrency and ThreadLocal](#36-concurrency-and-threadlocal)
   - [3.7 String Manipulation and Dynamic Locators](#37-string-manipulation-and-dynamic-locators)
4. [Step-by-Step: How to Build This Framework from Scratch](#4-step-by-step-how-to-build-this-framework-from-scratch)
   - [Step 1: Initialize Maven Project & Structure](#step-1-initialize-maven-project--structure)
   - [Step 2: Configure Dependencies in `pom.xml`](#step-2-configure-dependencies-in-pomxml)
   - [Step 3: Build Configuration Management](#step-3-build-configuration-management)
   - [Step 4: Build Thread-Safe WebDriverFactory](#step-4-build-thread-safe-webdriverfactory)
   - [Step 5: Build the `TestBase` Foundation](#step-5-build-the-testbase-foundation)
   - [Step 6: Build the Excel Test Data Engine (Apache POI)](#step-6-build-the-excel-test-data-engine-apache-poi)
   - [Step 7: Build the Page Object Model (POM) Layer](#step-7-build-the-page-object-model-pom-layer)
   - [Step 8: Write Test Cases with TestNG DataProviders](#step-8-write-test-cases-with-testng-dataproviders)
   - [Step 9: Add Listeners, Retries, and Screenshots](#step-9-add-listeners-retries-and-screenshots)
   - [Step 10: Configure Suite XML & CI/CD Pipelines](#step-10-configure-suite-xml--cicd-pipelines)
5. [Replicating the AI / GitHub Copilot Engine in Other Projects](#5-replicating-the-ai--github-copilot-engine-in-other-projects)
   - [5.1 Custom Copilot Instructions (`.github/copilot-instructions.md`)](#51-custom-copilot-instructions-githubcopilot-instructionsmd)
   - [5.2 Prompt Templates (`.github/prompts/`)](#52-prompt-templates-githubprompts)
   - [5.3 The DOM Crawler & Locator Investigator Concept](#53-the-dom-crawler--locator-investigator-concept)
   - [5.4 Azure DevOps / Jira MCP Integration](#54-azure-devops--jira-mcp-integration)
6. [Complete Self-Contained Starter Boilerplate](#6-complete-self-contained-starter-boilerplate)
7. [Structured 6-Week Learning Roadmap](#7-structured-6-week-learning-roadmap)
8. [Common Real-World Scenarios & Production Gotchas](#8-common-real-world-scenarios--production-gotchas)
   - [8.1 Modern SPAs & Single-Page Loading](#81-modern-spas-angular--react--vue--single-page-loading)
   - [8.2 Overlays, Loading Spinners & Intercepted Clicks](#82-overlays-loading-spinners--elementclickinterceptedexception)
   - [8.3 Custom Angular Dropdowns (`mat-select` / `ng-select`)](#83-custom-angular-dropdowns-mat-select-or-ng-select)
   - [8.4 File Uploads without Native Dialogs](#84-file-uploads-without-native-dialogs)
   - [8.5 Shadow DOM & iFrames](#85-shadow-dom--iframes)
9. [Comprehensive Troubleshooting & Error Dictionary](#9-comprehensive-troubleshooting--error-dictionary)

---

## 1. Executive Summary: The Big Picture

When you work on this project, GitHub Copilot assists you with generating page objects, structuring tests, and handling assertions. You also use a custom dependency (`functional-test-automation-sdk`). However, an SDK is simply pre-packaged Java code.

Underneath the hood, every enterprise Selenium framework (including this one) consists of **6 foundational pillars**:

```
+--------------------------------------------------------------------------+
|                            Test Layer                                    |
|   (TestNG Classes, @Test, @DataProvider, Allure step(), Assertions)     |
+------------------------------------+-------------------------------------+
                                     |
+------------------------------------v-------------------------------------+
|                     Page Object Model (POM) Layer                        |
|   (uiActions Classes, @FindBy, PageFactory, Encapsulated Actions)       |
+------------------------------------+-------------------------------------+
                                     |
+------------------------------------v-------------------------------------+
|                            TestBase Engine                               |
|   (Lifecycle hooks, Explicit Waits, Browser Actions, JS Helpers)        |
+------------------------------------+-------------------------------------+
                                     |
+------------------+-----------------+-------------------+-----------------+
|  Data Engine     | Driver Factory  | Config Engine     | Reporting Engine|
|  (Apache POI,    | (WebDriver,     | (Properties,      | (TestNG Listener|
|   Excel Reader)  |  ThreadLocal)   |  Environments)    |  Allure, Extent)|
+------------------+-----------------+-------------------+-----------------+
```

If you start a brand-new project at another company where no SDK exists, you can construct this exact system using standard open-source tools.

---

## 2. Anatomy of THIS Project: What Every File Does

Understanding the current project layout helps you see where each responsibility lives:

### Root Level
- **`pom.xml`**: The Maven project descriptor. Defines Java version (Java 1.8), plugins (`maven-compiler-plugin`, `maven-surefire-plugin`), AspectJ weaver for Allure reporting, and repositories (Azure Artifacts feed and local git fallback).
- **`configuration/config.properties`**: Environment-specific settings. Contains base URLs (`dev`, `tst`, `stg`, `nonprod`, `prod`), proxy settings, directory paths for reports/screenshots, and Excel data file names.
- **`regression_suite.xml`**: TestNG suite XML used in CI/CD to run the full regression batch. Configures listeners and test classes.
- **`SingleTest.xml`**: TestNG XML for isolating and verifying a single test class during development or debugging without running the full suite.
- **`crawler_suite.xml`**: TestNG XML configured to execute `LocatorInvestigator` for DOM inspection and locator generation.
- **`azure-pipelines.yml` / `azure-pipelines_stg.yml`**: Cloud CI/CD automation pipelines that check out the code, configure proxies, trigger Maven builds, execute tests via headless browsers, and publish Allure reports.
- **`GETTING-STARTED.md` & `SDK-USER-GUIDE.md`**: Onboarding and reference manuals explaining environment variables, proxy configurations, and SDK capabilities.

### Application Code (`src/main/java/com/automation/poletop/`)
- **`testbase/PoletopTestBase.java`**: Domain-specific base class. It extends the SDK's `TestBase` and adds business-level navigation helpers (e.g., `navigateToReservation(id)`, `navigateToAddPermit(id)`).
- **`uiActions/*.java`**: Page Object classes (e.g., `PoletopLoginPage`, `PoletopDashboardPage`, `ReservationDetailsPage`). Each class represents a single web page or modal, declares elements via `@FindBy(xpath = "...")`, initializes them via `PageFactory.initElements()`, and exposes user actions (`enterUsername()`, `clickSubmit()`).
- **`utility/*.java`**: Helper utilities for reading CSV files (`CSVUtils`, `CSVReporter`), handling temporary disposable emails (`MailinatorSupport`), and updating Excel files.

### Test Code (`src/test/java/com/poletop/automation/`)
- **`testCases/Test_*.java`**: Executable test classes (e.g., `Test_113913_LoginLogoutWithUsername`). Each class extends `PoletopTestBase` or `TestBase`, supplies data via `@DataProvider`, checks `runMode` (`"N"` skips), logs steps with Allure `step()`, and validates business outcomes using TestNG `Assert`.
- **`tools/LocatorInvestigator.java`**: A developer utility that navigates to a live page and invokes the SDK's crawler to scan the DOM and produce resilient locator reports.
- **`resources/testData/*.xlsx`**: Excel workbooks containing test data rows per environment (`POLETOP_DEV_TestData.xlsx`, `POLETOP_STG_TestData.xlsx`, etc.).

### AI / Copilot Configuration (`.github/`)
- **`.github/copilot-instructions.md`**: System instructions giving GitHub Copilot context on coding standards, locator priority rules, Java 8 constraints, mandatory compilation steps, and execution guidelines.
- **`.github/prompts/*.prompt.md`**: Specialized prompt templates (e.g., `create-test.prompt.md`, `fix-broken-locator.prompt.md`, `ado-sync-test.prompt.md`) that guide Copilot to perform repeatable, deterministic actions.

---

## 3. The Core Java Concepts You Must Learn

To build or customize an automation framework without getting stuck, you need to understand specific core Java concepts and how they directly map to Selenium:

### 3.1 Object-Oriented Programming (OOP) Applied to Automation

| OOP Concept | How It Is Used in This Framework | Concrete Example |
|-------------|----------------------------------|------------------|
| **Inheritance** | Sharing common setup, teardown, and WebDriver helper methods across all pages and test classes. | `public class PoletopLoginPage extends TestBase`<br>`public class Test_Login extends PoletopTestBase` |
| **Encapsulation** | Hiding raw `WebElement` interactions inside page object methods so tests do not manipulate raw locators. | Instead of `driver.findElement(By.id("btn")).click();` in the test, call `loginPage.clickLoginButton();` |
| **Polymorphism** | Writing code against the generic `WebDriver` interface while instantiating specific browser drivers. Method overloading for waits. | `WebDriver driver = new ChromeDriver();`<br>`waitForElementPresent(driver, element);`<br>`waitForElementPresent(driver, element, 30);` |
| **Abstraction** | Hiding complex synchronization, JavaScript execution, and retry mechanics behind simple method calls. | `clearAndType(element, text)` hides clearing, typing, waiting for visibility, and logging. |

### 3.2 Java Collections Framework
Selenium heavily relies on collections:
- **`List<WebElement>`**: Returned by `driver.findElements(By.xpath(...))`. Used to iterate over tables, dropdown options, search results, or verify element counts.
  ```java
  List<WebElement> rows = driver.findElements(By.xpath("//table//tbody/tr"));
  int totalCount = rows.size();
  for (WebElement row : rows) {
      log.info(row.getText());
  }
  ```
- **`Set<String>`**: Returned by `driver.getWindowHandles()`. Because window handles are unique strings, Java uses a `Set` to track browser tabs.
  ```java
  Set<String> handles = driver.getWindowHandles();
  for (String handle : handles) {
      driver.switchTo().window(handle);
  }
  ```
- **`Map<String, String>`**: Used to store key-value configuration pairs or dynamically map Excel column names to cell values.

### 3.3 Exception Handling in WebDriver
In Selenium, tests fail via specific runtime exceptions. You must understand how to handle and prevent them:
- **`NoSuchElementException`**: The locator did not match any element in the DOM at the instant of execution. *Solution: Use Explicit Waits (`WebDriverWait`) instead of immediate searches.*
- **`TimeoutException`**: An explicit wait condition was not satisfied within the timeout window.
- **`StaleElementReferenceException`**: The element was attached to the DOM when found, but the DOM re-rendered (e.g., Angular/React refresh) before the action was performed. *Solution: Re-find the element or use PageFactory retry wrappers.*
- **`ElementClickInterceptedException`**: Another element (like a spinner or overlay) received the click. *Solution: Wait for overlay disappearance or click via `JavascriptExecutor`.*

### 3.4 Annotations and Java Reflection
Frameworks use Java annotations to trigger code behind the scenes via Reflection:
- **TestNG Annotations**: `@Test`, `@BeforeClass`, `@AfterClass`, `@BeforeMethod`, `@DataProvider`. TestNG scans your compiled class files using reflection, identifies annotated methods, and executes them in lifecycle order.
- **Selenium PageFactory**: `@FindBy(xpath = "...")`. When you call `PageFactory.initElements(driver, this)`, Selenium uses reflection to find all fields with `@FindBy`, creates a dynamic proxy for each `WebElement`, and evaluates the locator lazily upon access.

### 3.5 File I/O and Property Files
- **`java.util.Properties` & `FileInputStream`**: Reading key-value pairs from `config.properties` without hardcoding URLs or credentials.
  ```java
  Properties prop = new Properties();
  FileInputStream fis = new FileInputStream("configuration/config.properties");
  prop.load(fis);
  String url = prop.getProperty("stg_base_url");
  ```
- **Apache POI**: Reading binary Microsoft Excel files (`.xlsx`). Key classes:
  - `XSSFWorkbook` (represents the `.xlsx` workbook)
  - `XSSFSheet` (represents an individual sheet tab)
  - `Row` and `Cell` (iterating through tabular rows and cells)
  - Converting Excel rows into a two-dimensional array: `Object[][]` for TestNG `@DataProvider`.

### 3.6 Concurrency and ThreadLocal
When running tests in parallel across multiple threads:
- A shared `static WebDriver driver;` will fail because multiple threads will write to and control the same browser window simultaneously.
- **`ThreadLocal<WebDriver>`** provides each thread with its own independent driver instance:
  ```java
  public class DriverFactory {
      private static ThreadLocal<WebDriver> tlDriver = new ThreadLocal<>();

      public static void setDriver(WebDriver driver) { tlDriver.set(driver); }
      public static WebDriver getDriver() { return tlDriver.get(); }
      public static void quitDriver() {
          if (tlDriver.get() != null) {
              tlDriver.get().quit();
              tlDriver.remove();
          }
      }
  }
  ```

### 3.7 String Manipulation and Dynamic Locators
- Formatting dynamic XPaths:
  ```java
  String rowXpath = String.format("//table//tr[td[text()='%s']]//button[text()='Edit']", reservationId);
  WebElement editBtn = driver.findElement(By.xpath(rowXpath));
  ```
- Resilient Union XPaths using `|`:
  ```xpath
  //input[@id='gigya-loginID'] | //input[@name='loginID'] | //input[@type='email']
  ```

---

## 4. Step-by-Step: How to Build This Framework from Scratch

Follow these 10 steps to create a production-grade Selenium framework from scratch:

```
[1. Project Init] -> [2. pom.xml Config] -> [3. Config Reader] -> [4. Driver Factory]
       |
[5. TestBase Core] <- [6. Apache POI Data Engine] <- [7. Page Objects (POM)]
       |
[8. Test Cases] -> [9. Listeners & Screenshots] -> [10. TestNG XML & CI/CD]
```

---

### Step 1: Initialize Maven Project & Structure

Create a standard Maven folder layout:

```
MyAutomationProject/
├── pom.xml
├── configuration/
│   └── config.properties
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/mycompany/automation/
│   │   │       ├── base/           # Driver setup & TestBase
│   │   │       ├── pages/          # Page Object classes (POM)
│   │   │       └── utils/          # Excel, Config, Screenshot helpers
│   │   └── resources/
│   │       └── log4j2.xml
│   └── test/
│       ├── java/
│       │   └── com/mycompany/automation/
│       │       └── tests/          # TestNG test scenarios
│       └── resources/
│           └── testData/           # Excel files (.xlsx)
├── testng.xml
└── .github/
    ├── copilot-instructions.md
    └── prompts/
```

---

### Step 2: Configure Dependencies in `pom.xml`

In a project without a custom SDK, declare all required open-source libraries directly:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mycompany.automation</groupId>
    <artifactId>Enterprise_Selenium_Framework</artifactId>
    <version>1.0.0</version>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <selenium.version>3.141.59</selenium.version> <!-- or 4.x -->
        <testng.version>7.4.0</testng.version>
        <poi.version>5.2.3</poi.version>
        <webdrivermanager.version>5.5.3</webdrivermanager.version>
        <log4j.version>2.20.0</log4j.version>
        <allure.version>2.24.0</allure.version>
        <aspectj.version>1.9.5</aspectj.version>
    </properties>

    <dependencies>
        <!-- Selenium WebDriver -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>${selenium.version}</version>
        </dependency>

        <!-- TestNG Test Runner -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>${testng.version}</version>
        </dependency>

        <!-- WebDriverManager: Auto-downloads ChromeDriver, GeckoDriver -->
        <dependency>
            <groupId>io.github.bonigarcia</groupId>
            <artifactId>webdrivermanager</artifactId>
            <version>${webdrivermanager.version}</version>
        </dependency>

        <!-- Apache POI: Excel Reader -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>${poi.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>${poi.version}</version>
        </dependency>

        <!-- Logging: Log4j2 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
        </dependency>

        <!-- Allure TestNG Reporting -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>${allure.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <suiteXmlFiles>
                        <suiteXmlFile>testng.xml</suiteXmlFile>
                    </suiteXmlFiles>
                    <argLine>
                        -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar"
                    </argLine>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjweaver</artifactId>
                        <version>${aspectj.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```

---

### Step 3: Build Configuration Management

Create `configuration/config.properties`:
```properties
browser=chrome
environment=stg
stg_base_url=https://myapp-stg.example.com/
dev_base_url=https://myapp-dev.example.com/
prod_base_url=https://myapp.example.com/
testDataPath=src/test/resources/testData/TestData.xlsx
explicitWaitSeconds=20
```

Create `ConfigReader.java`:
```java
package com.mycompany.automation.utils;

import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public class ConfigReader {
    private static Properties prop;

    public static Properties initProperties() {
        if (prop == null) {
            prop = new Properties();
            try (FileInputStream fis = new FileInputStream("configuration/config.properties")) {
                prop.load(fis);
            } catch (IOException e) {
                throw new RuntimeException("Could not load configuration/config.properties: " + e.getMessage());
            }
        }
        return prop;
    }

    public static String getProperty(String key) {
        if (prop == null) initProperties();
        return prop.getProperty(key);
    }
}
```

---

### Step 4: Build Thread-Safe WebDriverFactory

Create `WebDriverFactory.java`:
```java
package com.mycompany.automation.base;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;

import io.github.bonigarcia.wdm.WebDriverManager;

public class WebDriverFactory {

    private static ThreadLocal<WebDriver> driverThreadLocal = new ThreadLocal<>();

    public static WebDriver createDriver(String browser, boolean headless) {
        WebDriver driver;
        if (browser.equalsIgnoreCase("chrome")) {
            WebDriverManager.chromedriver().setup();
            ChromeOptions options = new ChromeOptions();
            if (headless) {
                options.addArguments("--headless=new", "--disable-gpu", "--window-size=1920,1080");
            }
            options.addArguments("--start-maximized", "--ignore-certificate-errors");
            driver = new ChromeDriver(options);
        } else if (browser.equalsIgnoreCase("firefox")) {
            WebDriverManager.firefoxdriver().setup();
            driver = new FirefoxDriver();
        } else if (browser.equalsIgnoreCase("edge")) {
            WebDriverManager.edgedriver().setup();
            driver = new EdgeDriver();
        } else {
            throw new IllegalArgumentException("Unsupported browser: " + browser);
        }

        driverThreadLocal.set(driver);
        return driver;
    }

    public static WebDriver getDriver() {
        return driverThreadLocal.get();
    }

    public static void quitDriver() {
        if (driverThreadLocal.get() != null) {
            driverThreadLocal.get().quit();
            driverThreadLocal.remove();
        }
    }
}
```

---

### Step 5: Build the `TestBase` Foundation

`TestBase` acts as the superclass for every test and page object. It controls driver initialization, hooks, and explicit wait helpers:

```java
package com.mycompany.automation.base;

import java.io.IOException;
import java.lang.reflect.Method;
import java.util.Properties;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.testng.annotations.*;

import com.mycompany.automation.utils.ConfigReader;
import com.mycompany.automation.utils.ExcelReader;

public class TestBase {

    public static final Logger log = LogManager.getLogger(TestBase.class.getName());
    public static Properties Prop;
    protected WebDriver driver;
    protected String baseURL;

    @BeforeClass
    @Parameters({"environment", "browserName"})
    public void setUp(@Optional("") String environment, @Optional("") String browserName) {
        Prop = ConfigReader.initProperties();

        String env = (environment != null && !environment.isEmpty())
                ? environment : System.getProperty("environment", Prop.getProperty("environment", "stg"));
        String browser = (browserName != null && !browserName.isEmpty())
                ? browserName : System.getProperty("browserName", Prop.getProperty("browser", "chrome"));

        baseURL = Prop.getProperty(env + "_base_url");
        boolean headless = Boolean.parseBoolean(System.getProperty("headless", "false"));

        driver = WebDriverFactory.createDriver(browser, headless);
        driver.manage().window().maximize();
        driver.get(baseURL);
        waitUntillPageLoad();
    }

    @AfterClass(alwaysRun = true)
    public void tearDown() {
        WebDriverFactory.quitDriver();
    }

    // --- Core Synchronization & Helper Methods ---

    public WebElement waitForElementPresent(WebDriver driver, WebElement element, int timeoutSeconds) {
        WebDriverWait wait = new WebDriverWait(driver, timeoutSeconds);
        return wait.until(ExpectedConditions.visibilityOf(element));
    }

    public WebElement waitForElementPresent(WebDriver driver, WebElement element) {
        return waitForElementPresent(driver, element, 20);
    }

    public WebElement fluentWaitUntilElementToBeClickable(WebDriver driver, WebElement element, int timeoutSeconds) {
        WebDriverWait wait = new WebDriverWait(driver, timeoutSeconds);
        return wait.until(ExpectedConditions.elementToBeClickable(element));
    }

    public void clearAndType(WebElement element, String text) {
        waitForElementPresent(driver, element);
        element.clear();
        element.sendKeys(text);
        log.info("Typed text: " + text);
    }

    public void clickElement(WebElement element) {
        fluentWaitUntilElementToBeClickable(driver, element, 20);
        element.click();
        log.info("Clicked element");
    }

    public void waitUntillPageLoad() {
        WebDriverWait wait = new WebDriverWait(driver, 30);
        wait.until(d -> ((JavascriptExecutor) d).executeScript("return document.readyState").equals("complete"));
    }

    // --- Excel Data Provider Bridge ---
    public Object[][] getData(String sheetName) throws IOException {
        String filePath = Prop.getProperty("testDataPath");
        return ExcelReader.getSheetData(filePath, sheetName);
    }
}
```

---

### Step 6: Build the Excel Test Data Engine (Apache POI)

Create `ExcelReader.java` to turn any sheet in an `.xlsx` file into an `Object[][]`:

```java
package com.mycompany.automation.utils;

import java.io.FileInputStream;
import java.io.IOException;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public class ExcelReader {

    public static Object[][] getSheetData(String filePath, String sheetName) throws IOException {
        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheet(sheetName);
            if (sheet == null) {
                throw new IllegalArgumentException("Sheet '" + sheetName + "' not found in " + filePath);
            }

            int totalRows = sheet.getLastRowNum(); // 0-based, row 0 is header
            int totalCols = sheet.getRow(0).getLastCellNum();

            Object[][] data = new Object[totalRows][totalCols];
            DataFormatter formatter = new DataFormatter();

            for (int r = 1; r <= totalRows; r++) {
                Row row = sheet.getRow(r);
                for (int c = 0; c < totalCols; c++) {
                    if (row != null && row.getCell(c) != null) {
                        data[r - 1][c] = formatter.formatCellValue(row.getCell(c)).trim();
                    } else {
                        data[r - 1][c] = "";
                    }
                }
            }
            return data;
        }
    }
}
```

---

### Step 7: Build the Page Object Model (POM) Layer

Page objects represent pages or UI components. They encapsulate elements and interactions:

```java
package com.mycompany.automation.pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import com.mycompany.automation.base.TestBase;

public class LoginPage extends TestBase {

    @FindBy(xpath = "//input[@id='username' or @name='user']")
    public WebElement usernameField;

    @FindBy(xpath = "//input[@id='password' or @name='pwd']")
    public WebElement passwordField;

    @FindBy(xpath = "//button[@id='submit' or normalize-space(.)='Log In']")
    public WebElement loginButton;

    @FindBy(xpath = "//div[contains(@class,'alert-danger')]")
    public WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public void login(String username, String password) {
        clearAndType(usernameField, username);
        clearAndType(passwordField, password);
        clickElement(loginButton);
        waitUntillPageLoad();
    }

    public String getErrorMessage() {
        waitForElementPresent(driver, errorMessage);
        return errorMessage.getText();
    }
}
```

---

### Step 8: Write Test Cases with TestNG DataProviders

Test classes extend `TestBase`, retrieve test data, verify `runMode`, execute actions with Allure `step()`, and perform assertions:

```java
package com.mycompany.automation.tests;

import static io.qameta.allure.Allure.step;
import java.io.IOException;

import org.testng.Assert;
import org.testng.SkipException;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

import com.mycompany.automation.base.TestBase;
import com.mycompany.automation.pages.LoginPage;

public class Test_LoginScenario extends TestBase {

    LoginPage loginPage;

    @DataProvider(name = "loginData")
    public Object[][] getLoginData() throws IOException {
        return getData("LoginCredentials");
    }

    @Test(dataProvider = "loginData", priority = 1)
    public void testLogin(String testCaseName, String username, String password, String expectedResult, String runMode) {
        step("Checking RunMode for scenario: " + testCaseName);
        if ("N".equalsIgnoreCase(runMode)) {
            throw new SkipException("Skipping test case: " + testCaseName + " due to RunMode=N");
        }

        loginPage = new LoginPage(driver);

        step("Entering credentials and submitting");
        loginPage.login(username, password);

        step("Validating result");
        if (expectedResult.equalsIgnoreCase("SUCCESS")) {
            Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"), "Expected user to land on dashboard");
        } else {
            String error = loginPage.getErrorMessage();
            Assert.assertTrue(error.contains("Invalid"), "Expected error message to display for invalid login");
        }
    }
}
```

---

### Step 9: Add Listeners, Retries, and Screenshots

Create a TestNG listener to automatically capture a screenshot on test failure:

```java
package com.mycompany.automation.utils;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.testng.ITestListener;
import org.testng.ITestResult;

import com.mycompany.automation.base.WebDriverFactory;

public class TestListener implements ITestListener {

    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = WebDriverFactory.getDriver();
        if (driver != null) {
            TakesScreenshot ts = (TakesScreenshot) driver;
            File src = ts.getScreenshotAs(OutputType.FILE);
            String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
            String dest = "test-output/screenshots/" + result.getName() + "_" + timestamp + ".png";
            try {
                Files.createDirectories(Paths.get("test-output/screenshots/"));
                Files.copy(src.toPath(), Paths.get(dest));
                System.out.println("Screenshot captured on failure: " + dest);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

---

### Step 10: Configure Suite XML & CI/CD Pipelines

Create `testng.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Regression Suite" parallel="none">
    <listeners>
        <listener class-name="com.mycompany.automation.utils.TestListener"/>
    </listeners>
    <test name="Smoke Tests">
        <classes>
            <class name="com.mycompany.automation.tests.Test_LoginScenario"/>
        </classes>
    </test>
</suite>
```

Execute tests via CLI:
```bash
mvn test -Denvironment=stg -DbrowserName=chrome
```

---

## 5. Replicating the AI / GitHub Copilot Engine in Other Projects

In this project, GitHub Copilot operates effectively because it is constrained by a rigid set of instructions, templates, and DOM analysis tools. Here is how you replicate that AI setup in any new project:

### 5.1 Custom Copilot Instructions (`.github/copilot-instructions.md`)
Create a `.github/copilot-instructions.md` file in the root of your new repository. It should define:
1. **Framework identity**: Language (e.g. Java 8), base class name (`TestBase`), Page Object directory (`src/main/java/...`), Test directory (`src/test/java/...`).
2. **Locator rules**:
   - Only stable XPaths matching exactly 1 element.
   - Prohibit dynamic IDs (e.g., Angular `mat-input-*` or auto-incremented numbers).
   - Priority hierarchy: `@id` -> `@data-testid` -> `@name` -> `@aria-label` -> `@placeholder` -> `normalize-space(.)`.
3. **Mandatory validation workflow**:
   - Copilot must run `mvn compile test-compile` before considering a task complete.
   - Copilot must execute the test with `mvn test -Dtest=...`.
   - Copilot must never leave a test broken.

### 5.2 Prompt Templates (`.github/prompts/`)
Create reusable markdown files in `.github/prompts/` to invoke standardized routines via `#prompt-name`:
- **`#create-test`**: Step-by-step instructions to create a test class, wire up `@DataProvider`, add `runMode` check, and add Allure steps.
- **`#fix-failed-test`**: Diagnostic checklist: Check screenshot -> Check stack trace -> Inspect DOM locator -> Verify Excel data -> Re-run.
- **`#fix-broken-locator`**: Guidance on identifying DOM mutations and writing resilient union XPaths.

### 5.3 Deep Dive: How the DOM Crawler Scans and Formulates Unique XPaths

The SDK's `ElementCrawler` and `PageObjectGenerator` automate the discovery and formulation of locators through a 5-step engineering pipeline:

```
[1. DOM Discovery] ──> [2. Attribute Extraction] ──> [3. Candidate Formulation]
                                                               │
[5. Java File (.java)] <── [4. Validation / Filtering] <───────┘
```

#### Step 1: DOM Discovery (Querying Interactive Elements)
Instead of parsing raw HTML text with regular expressions, the crawler interacts directly with the live browser DOM via WebDriver:
```java
// Query only interactive and semantic elements:
List<WebElement> elements = driver.findElements(
    By.xpath("//input | //button | //select | //a | //textarea | //h1 | //h2 | //table")
);
```

#### Step 2: Attribute & Metadata Extraction
For each element, the crawler calls `element.getAttribute(...)` to retrieve:
- Identification: `id`, `name`, `data-testid`, `formcontrolname` (Angular)
- Accessibility: `aria-label`, `placeholder`, `title`
- Text & Type: `element.getText()`, `type`, `href`
- Visibility: `element.isDisplayed()`

#### Step 3: Candidate Formulation with Priority Hierarchy
The crawler constructs candidate XPaths following a strict stability hierarchy:
1. `@id` (discarding dynamic patterns like `mat-input-` or random digits) -> `//input[@id='gigya-loginID']`
2. `@data-testid` -> `//button[@data-testid='submit-btn']`
3. `@formcontrolname` -> `//input[@formcontrolname='email']`
4. `@name` + `@type` -> `//input[@name='username' and @type='text']`
5. `@aria-label` -> `//button[@aria-label='Close dialog']`
6. `@placeholder` -> `//input[@placeholder='Search']`
7. `normalize-space(.)` -> `//button[normalize-space(.)='Login']`

#### Step 4: DOM Uniqueness Verification (`findElements().size() == 1`)
Every candidate XPath is tested immediately in the browser session:
```java
int matchCount = driver.findElements(By.xpath(candidateXPath)).size();
if (matchCount == 1) {
    status = "UNIQUE [x]";       // Candidate is safe to use in Page Object
} else if (matchCount > 1) {
    status = "NOT UNIQUE (" + matchCount + " found)"; // Discarded or flagged
} else {
    status = "STALE";            // Element not present in current DOM state
}
```
If elements share identical identifiers across multiple responsive designs or environments, the crawler builds **Resilient Union XPaths** using `|`:
```xpath
//input[@id='gigya-loginID'] | //input[@name='loginID'] | //input[@name='username' and @type='text']
```

#### Step 5: Code Generation
Unique locators (`UNIQUE [x]`) are converted into camelCase field names and emitted into a Java file with `@FindBy(xpath = "...")` annotations, ready to be committed to `uiActions/`.

### 5.4 Azure DevOps / Jira MCP Integration
To link AI directly to test cases:
1. Configure an MCP server (e.g., `@azure-devops/mcp` or Atlassian Jira MCP) in `%USERPROFILE%\.copilot\mcp-config.json`.
2. Provide a Personal Access Token (PAT) with Work Item / Test Case read permissions.
3. Copilot can then fetch formal test steps directly from work item IDs (e.g., ADO-113913) and map each step and expected result into Java test code with assertions.

---

## 6. Complete Self-Contained Starter Boilerplate

If you need to stand up a new project quickly, copy the directory layout and files below:

```
new-automation-project/
├── pom.xml
├── testng.xml
├── configuration/
│   └── config.properties
├── src/
│   ├── main/java/com/demo/automation/
│   │   ├── base/
│   │   │   ├── TestBase.java
│   │   │   └── WebDriverFactory.java
│   │   ├── pages/
│   │   │   └── LoginPage.java
│   │   └── utils/
│   │       ├── ConfigReader.java
│   │       ├── ExcelReader.java
│   │       └── TestListener.java
│   └── test/java/com/demo/automation/tests/
│       └── Test_Login.java
└── src/test/resources/testData/
    └── TestData.xlsx
```

Every code snippet in **Section 4** is completely functional, self-contained, and ready to compile using standard open-source dependencies.

---

## 7. Structured 6-Week Learning Roadmap

Follow this progression to transition from a beginner to an expert automation framework architect:

```
Week 1: Core Java Foundations
├── Classes, Objects, Constructors, Methods
├── OOP: Inheritance (Base classes), Encapsulation (Page Objects), Polymorphism
└── Collections: List<WebElement>, Set<String>, Map<String, String>

Week 2: Exception Handling & File I/O
├── try-catch-finally, throws, Custom Exceptions
├── Selenium Exceptions: NoSuchElementException, StaleElementReferenceException
└── FileInputStream, java.util.Properties, reading config files

Week 3: Selenium WebDriver Mastery & Locators
├── WebDriver Architecture & Browser options
├── XPath: Axes (parent, following-sibling, ancestor), functions (text(), contains(), normalize-space())
└── Synchronization: Implicit vs Explicit (WebDriverWait) vs FluentWait (No Thread.sleep)

Week 4: TestNG & Data-Driven Automation
├── Annotations: @Test, @BeforeClass, @AfterClass, @BeforeMethod, @DataProvider
├── Data-driven testing with Apache POI (ExcelReader reading into Object[][])
└── Assertions: Hard Assert vs Soft Assert, Verify vs Assert

Week 5: Framework Design (POM) & Advanced Architecture
├── Page Object Model (POM) with PageFactory and @FindBy
├── ThreadLocal<WebDriver> for parallel execution
└── TestNG Listeners for failure screenshots and Allure/Extent reporting

Week 6: CI/CD & AI-Assisted Automation
├── Maven lifecycle: compile, test-compile, test, Surefire plugin
├── Headless execution in Azure DevOps / GitHub Actions pipelines
└── AI-assisted testing: Setting up .github/copilot-instructions.md and MCP integration
```

---

## Summary Checklist for New Projects

When asked to build or evaluate a new automation framework, verify these items:
- [ ] Maven dependencies are clean and conflict-free.
- [ ] URLs and credentials live in external config files, never hardcoded in tests.
- [ ] Drivers are managed via a centralized factory using `ThreadLocal` for concurrency.
- [ ] All pages extend a common base class containing wait wrappers (no raw `Thread.sleep`).
- [ ] Locators are unique, stable XPaths (no dynamic/auto-generated IDs).
- [ ] Tests are data-driven via Excel (`@DataProvider`), checking `runMode`.
- [ ] Failures capture screenshots automatically through TestNG listeners.
- [ ] Clear guidelines exist in `.github/copilot-instructions.md` so AI tools assist effectively.

---

## 8. Common Real-World Scenarios & Production Gotchas

When applying this framework to other web applications, you will face complex UI behaviors. Here is how to handle each without getting stuck:

### 8.1 Modern SPAs (Angular / React / Vue) & Single-Page Loading
In SPAs like Poletop, navigating between pages does not reload the HTML page (the URL hash `/#/dashboard` changes via JavaScript).
* **The Pitfall:** `driver.get()` or clicks finish before the JavaScript framework finishes rendering elements.
* **The Solution:** Never rely on static sleeps. Wait for Angular / React / Document ready state or a key container element:
  ```java
  public void waitForAngularReady() {
      WebDriverWait wait = new WebDriverWait(driver, 20);
      wait.until(d -> ((JavascriptExecutor) d).executeScript(
          "return window.getAllAngularTestabilities ? " +
          "window.getAllAngularTestabilities().findIndex(x => !x.isStable()) === -1 : " +
          "document.readyState === 'complete'"
      ));
  }
  ```

### 8.2 Overlays, Loading Spinners & `ElementClickInterceptedException`
Angular and Material UI display transparent overlays (`.cdk-overlay-backdrop`, `.spinner-loading`) during API calls:
* **The Solution:** Always wait for the overlay to disappear before clicking:
  ```java
  public void waitForOverlayToDisappear() {
      By overlay = By.xpath("//div[contains(@class,'spinner') or contains(@class,'cdk-overlay-backdrop')]");
      WebDriverWait wait = new WebDriverWait(driver, 15);
      wait.until(ExpectedConditions.invisibilityOfElementLocated(overlay));
  }
  ```

### 8.3 Custom Angular Dropdowns (`mat-select` or `ng-select`)
Custom frontend dropdowns are not HTML `<select>` tags — `new Select(element)` will throw `UnexpectedTagNameException`.
* **The Solution:** Use a 2-step click interaction:
  ```java
  public void selectCustomDropdown(WebElement dropdownTrigger, String optionText) {
      fluentWaitUntilElementToBeClickable(driver, dropdownTrigger, 10).click();
      String optionXpath = String.format("//mat-option[normalize-space(.)='%s'] | //div[contains(@class,'ng-option') and normalize-space(.)='%s']", optionText, optionText);
      WebElement option = driver.findElement(By.xpath(optionXpath));
      fluentWaitUntilElementToBeClickable(driver, option, 10).click();
  }
  ```

### 8.4 File Uploads without Native Dialogs
Selenium cannot interact with OS file dialogs (Windows File Explorer).
* **The Solution:** Target the hidden `<input type='file'>` directly using `sendKeys()` with the absolute path:
  ```java
  public void uploadFile(WebElement fileInput, String relativeFilePath) {
      File file = new File(relativeFilePath);
      fileInput.sendKeys(file.getAbsolutePath());
  }
  ```

### 8.5 Shadow DOM & iFrames
* **iFrames:** Must switch into before searching for elements, and switch back:
  ```java
  driver.switchTo().frame("frameNameOrId");
  // perform actions inside iframe
  driver.switchTo().defaultContent();
  ```
* **Shadow DOM:** Elements inside a shadow root cannot be found via standard XPath:
  ```java
  // Use JavascriptExecutor to expand shadow root:
  WebElement host = driver.findElement(By.cssSelector("custom-shadow-host"));
  SearchContext shadowRoot = (SearchContext) ((JavascriptExecutor) driver)
      .executeScript("return arguments[0].shadowRoot", host);
  WebElement shadowElement = shadowRoot.findElement(By.cssSelector(".inner-btn"));
  shadowElement.click();
  ```

---

## 9. Comprehensive Troubleshooting & Error Dictionary

Keep this reference table handy when debugging failures in your new project:

| Error / Exception | Root Cause | Exact Fix |
|---|---|---|
| `NoSuchElementException` | Element not in DOM yet, or locator changed | Increase explicit wait (`WebDriverWait`), check if element is inside an iframe, verify locator in browser DevTools Console via `$x("xpath")`. |
| `TimeoutException` | Element never reached expected condition within seconds | Check if preceding action (button click) actually triggered the change; check if a blocker spinner/overlay is visible. |
| `StaleElementReferenceException` | DOM re-rendered or refreshed after element was found | Re-initialize the page object or re-locate element (`driver.findElement`) right before the action. |
| `ElementClickInterceptedException` | Another element (banner, header, toast, spinner) is on top | Wait for overlay invisibility (`invisibilityOfElementLocated`), scroll element into view, or click via JavaScript: `((JavascriptExecutor)driver).executeScript("arguments[0].click();", elem);`. |
| `NullPointerException` on `driver` | `PageFactory.initElements()` wasn't called, or driver was not initialized in `@BeforeClass` | Verify your page object constructor calls `this.driver = driver; PageFactory.initElements(driver, this);`. |
| `NullPointerException` in `getData()` | Excel file name wrong, sheet name does not exist, or path misconfigured | Verify sheet name spelling matches the Excel tab; verify `config.properties` has correct `testDataPath`. |
| Test skipped unexpectedly | `runMode` is set to `"N"` | Check the Excel row's `RunMode` column; set to `"Y"`. |
| `SessionNotCreatedException` | ChromeDriver version does not match Chrome browser version | Update Chrome or use `io.github.bonigarcia:webdrivermanager` to automatically download the matching version. |
| Corporate Proxy Connection Error | Maven or WebDriver cannot download driver binaries or access web pages | Add `-Dhttps.proxyHost=yourproxy -Dhttps.proxyPort=port` to your Maven command or configure `pom.xml` / `sdk-config.yaml`. |

