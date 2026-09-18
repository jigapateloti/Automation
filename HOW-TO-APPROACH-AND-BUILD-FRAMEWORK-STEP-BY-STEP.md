# How to Build an Automation Robot (SDK) from Scratch
### A Simple, Step-by-Step Guide for Beginners (Explained Like You're 12)

---

## 🧸 Introduction: What Are We Actually Doing?

Imagine you love playing a computer game or visiting a website, but your teacher asks you to click 100 buttons and fill out 50 forms every single day to make sure the website isn't broken.

Doing that by hand is super boring and tiring! 

So, what do you do? **You build a Software Robot!**

The robot:
1. Opens Google Chrome.
2. Goes to the website.
3. Finds the buttons and text boxes.
4. Types your name and clicks "Submit".
5. Takes a picture (screenshot) and shouts: *"Hey, it passed!"* or *"Oh no, button was missing, here is the error picture!"*

Right now, in this project, you have an **SDK** (a pre-made robot kit) and **GitHub Copilot** (an AI assistant). But if someone took the kit away and told you: 
> *"Build this entire robot from a blank folder for a brand new project!"*

This guide shows you the exact order of what to learn first, second, and third, so you never feel lost.

---

## 🗺️ The Master Learning Map: Which Step Comes First?

Do **not** jump straight into AI or complex test scripts! Follow this step-by-step staircase:

```
Step 6: Teach AI (Copilot) Your Robot's Rules
        ▲
Step 5: Build the "Page Crawler" (The Smart Scanner)
        ▲
Step 4: The Page Object Model (Organizing Your Rooms)
        ▲
Step 3: TestBase & Explicit Waits (Teaching the Robot Patience)
        ▲
Step 2: Selenium Basics (Finding Buttons & Clicking)
        ▲
Step 1: Java Basics (The LEGO Bricks)
```

---

## 🧱 STEP 1: Learn the Java LEGO Bricks (The Foundation)

Before you can build a robot, you need to know what the bricks are called and how they click together.

### 1. Variables (Storage Boxes)
A variable is just a labeled box where you keep something.
* `String name = "Jigar";` (Stores text)
* `int age = 12;` (Stores numbers)
* `boolean isButtonFound = true;` (Stores Yes or No)

### 2. Methods (Action Recipes)
A method is a recipe of actions with a name.
```java
void brushTeeth() {
    pickUpBrush();
    applyToothpaste();
    brush();
}
```
In automation, your recipe looks like:
```java
void login(String user, String pass) {
    typeUsername(user);
    typePassword(pass);
    clickLoginButton();
}
```

### 3. Classes and Objects (The Blueprint vs. The Toy)
* **Class** is the paper blueprint for a LEGO spaceship.
* **Object** is the real LEGO spaceship built on your desk using `new Spaceship()`.

### 4. Inheritance (Copying Good Stuff from Parents)
If `Dog` extends `Animal`, the Dog gets all the features of Animal (like breathing and eating) for free without writing that code again!
* In our framework: Every test and page class **extends TestBase** to get all browser tools for free!

---

## 🌐 STEP 2: Selenium WebDriver (Giving Hands and Eyes to Java)

Java by itself cannot open Google Chrome. It has no eyes and no hands.  
**Selenium WebDriver** is the pair of robotic hands and eyes you attach to Java.

### How Selenium Finds Things on a Web Page:
A web page is like a big house with many objects (doors, windows, light switches). Selenium needs an address to find them. That address is called an **XPath**.

Think of XPath like giving directions:
* *"Go to the kitchen, find the red cabinet, open drawer 2."*
* In code: `//input[@id='username']` (*"Find the input box whose id tag is 'username'"*)

### The 3 Core Actions:
1. **Find:** `WebElement button = driver.findElement(By.xpath("//button[@id='login']"));`
2. **Type:** `button.sendKeys("Hello");`
3. **Click:** `button.click();`

---

## ⏳ STEP 3: The "TestBase" & Patience (Explicit Waits)

**The #1 mistake beginners make:**  
Computers run at lightning speed. Web pages load slowly over the internet.  
If Java tries to click a button before the webpage finishes loading, it throws an error:  
💥 `NoSuchElementException` (meaning: *"I looked, but it wasn't there yet!"*)

### How a 12-Year-Old Solves It vs. How an Engineer Solves It:
* ❌ **Bad way (`Thread.sleep(5000)`):** *"Sit blindly with your eyes closed for 5 seconds."* (Waste of time if the page loaded in 1 second; breaks if it takes 6 seconds!)
* ✅ **Smart way (Explicit Wait):** *"Keep checking the door every quarter-second until it opens, or give up after 20 seconds."*

```java
// Inside your TestBase class:
public WebElement waitForElement(WebElement element) {
    WebDriverWait wait = new WebDriverWait(driver, 20);
    return wait.until(ExpectedConditions.visibilityOf(element));
}
```

Whenever you build a new project, **create your `TestBase.java` first**. Put all your waiting methods, browser opening methods, and browser closing methods inside it.

---

## 📄 STEP 4: The Page Object Model (POM) — Keeping Rooms Organized

Imagine you have a big house with a **Kitchen**, a **Bedroom**, and a **Garage**.

Would you keep your toothbrush in the garage? No!  
If the stove breaks, you go to the Kitchen to fix it.

In web automation, every webpage is a "Room" called a **Page Object**:
* `LoginPage.java` (Only contains elements and actions for the login screen)
* `DashboardPage.java` (Only contains elements and actions for the dashboard)
* `ReservationPage.java` (Only contains elements and actions for reservations)

### How a Page Object Class Looks:
```java
public class LoginPage extends TestBase {

    // 1. Point to the items in the room
    @FindBy(xpath = "//input[@id='username']")
    public WebElement usernameBox;

    @FindBy(xpath = "//button[@id='submit']")
    public WebElement loginBtn;

    // 2. Constructor: Tell Selenium to hook up these items
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    // 3. What you can DO in this room
    public void logIn(String user, String pass) {
        clearAndType(usernameBox, user);
        clickElement(loginBtn);
    }
}
```

Now, your test script stays super clean and easy to read:
```java
@Test
public void testUserCanLogin() {
    LoginPage login = new LoginPage(driver);
    login.logIn("jigar", "Secret123!");
    Assert.assertTrue(driver.getCurrentUrl().contains("dashboard"));
}
```

---

## 🕷️ STEP 5: How to Replicate the "Crawler" (The Magic Locator Scanner)

In this project, people talk about the **"Crawler"** or `LocatorInvestigator`.  
It sounds like high-tech magic, but it is actually a very simple robot detective!

### How Does the Crawler Work?
Imagine you give a detective a magnifying glass and send them into a room:

1. **Step 1 (Find all buttons/inputs):**  
   The detective says: *"Hey browser, give me a list of every single button, text box, link, and dropdown on this screen!"*
   ```java
   List<WebElement> allItems = driver.findElements(By.xpath("//input | //button | //select | //a"));
   ```

2. **Step 2 (Read their ID tags):**  
   For each item, the detective looks at its name tag:
   * Does it have an `id`? (e.g., `id="submit-button"`)
   * Does it have a `name`?
   * What text does it say? (e.g., `"Sign In"`)

3. **Step 3 (The Uniqueness Test):**  
   The detective tests the candidate address:
   ```java
   // Test if this address matches ONLY 1 item:
   int count = driver.findElements(By.xpath("//button[@id='submit-button']")).size();
   ```
   * If `count == 1`: The detective stamps it: **`UNIQUE [x]`** (Great! Safe to use!).
   * If `count > 1`: The detective stamps it: **`NOT UNIQUE`** (Danger! Multiple buttons have this address).
   * If the ID is a weird computer number like `mat-input-0`: The detective stamps it: **`DYNAMIC`** (Throw away, it changes every time).

4. **Step 4 (Writing the Java file):**  
   The detective writes down all the `UNIQUE [x]` items into a brand new `.java` file with `@FindBy(xpath = "...")` automatically!

### How to Replicate this in Another Project:
If another project doesn't have the SDK, you can write a tiny 40-line Java helper that runs the loop above and prints the unique XPaths to the console!

---

## 🤖 STEP 6: Teach AI (GitHub Copilot) How to Follow Your Rules

GitHub Copilot is like a very smart puppy. If you don't give it clear house rules, it will chew up your shoes and make a mess!

In our project, Copilot behaves well because of **`.github/copilot-instructions.md`**.

When you go to a **new project**, simply create a `.github/` folder and write your instructions:

### What to tell Copilot:
1. **Rule 1:** *"Always extend `TestBase`. Never use raw `Thread.sleep`."*
2. **Rule 2:** *"Only use XPath. Never use dynamic Angular IDs like `mat-input-*`."*
3. **Rule 3:** *"Every test must read data from Excel using `@DataProvider`."*
4. **Rule 4:** *"Before you say you are done, run `mvn compile test-compile` and prove it builds!"*

Once you give Copilot those rules, it will write perfect tests matching your style every single time.

---

## 📅 Your 5-Stage Step-by-Step Action Plan

When starting on a fresh project tomorrow, do things in this exact order:

| Stage | What You Do | Result |
|---|---|---|
| **Stage 1** | Create Maven project (`pom.xml`) with Selenium, TestNG, Apache POI, and WebDriverManager. | You can compile with `mvn compile`. |
| **Stage 2** | Create `config.properties` and a `ConfigReader.java` to read URLs. | Your URLs are not hardcoded. |
| **Stage 3** | Create `WebDriverFactory.java` and `TestBase.java` with browser launch and wait methods. | You can open Chrome and maximize window. |
| **Stage 4** | Create your first Page Object (`LoginPage.java`) with unique XPaths. | Your page actions are encapsulated. |
| **Stage 5** | Create `Test_Login.java` with `@Test` and assertions. Run it via `mvn test`. | Your first test passes in the browser! |

---

## 💡 Summary Checklist: Keep It Simple!

* If you get stuck on an error, ask: *"Is the element visible yet? Did I wait for it?"*
* Keep your locators unique (`UNIQUE [x]`).
* Keep your code in page objects, and keep your assertions in test classes.
* Read through `SELENIUM-AUTOMATION-FROM-SCRATCH-GUIDE.md` whenever you want the exact code to copy-paste.
