+++
archetype = "default"
title = "Getting Started"
weight = 2
+++
## Setup project
Add wrappedelements and selenium-java dependency to assembler build file.
It's optional to add any additional logger dependency to activate built-in logging.
### Maven
```maven
<dependency>
    <groupId>io.github.bobfrostman</groupId>
    <artifactId>wrappedelements</artifactId>
    <version>${wrappedelements.version}</version>
</dependency>
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>${selenium.version}</version>
</dependency>
```
### Gradle
```gradle
implementation group: 'io.github.bobfrostman', name: 'wrappedelements', version: '${wrappedelements.version}'
implementation group: 'org.seleniumhq.selenium', name: 'selenium-java', version: '${selenium.version}'
```
### Ivy
```
<dependency org="io.github.bobfrostman" name="wrappedelements" rev="${wrappedelements.version}"/>
<dependency org="org.seleniumhq.selenium" name="selenium-java" rev="${selenium.version}"/>
```
## Getting started
Create your own Page Object entities as interfaces (extend the IPage interface)
```java
public interface LoginPage extends IPage {

    @WebElement(value = "//form//input")
    ClickableElement loginInput();

    @WebElement(value = "(//form//input)[2]")
    ClickableElement passwordInput();

    @WebElement(value = "//input[@type='submit']")
    ClickableElement loginButton();

} 
```
**ClickableElement** is a wrapper class around standard selenium **WebElement**. It's the basic web element class that provides additional functionality and allows you to use built-in waiters.
It provides an opportunity to adjust waiters and timeout conditions on Page Object level by adjusting **@WebElement** annotation values.

For instance, you can adjust:
- Wait condition - by changing **waitUntil** value
- Element wait timeout - by changing **timeout** value (in seconds)
- Element name (for logging purposes) - by changing **name** annotation value

Basically **ClickableElement** used instead of classical selenium **WebElement** and **@WebElement** annotation used instead of selenium **@FindBy** annotation.
It provides additional flexibility to your Page Object Model during test automation process. 

First, you need to register selenium webdriver instance to be used by wrappedelements framework.
To do that provide a driver creation function, and init your page with wrappedelements do next:
```java
    @BeforeClass
    public void setUp() {
        //register webdriver creation function (whatever you want)
        WrappedElements.config()
                .driverCreator(() -> {
                    WebDriverManager.chromedriver().setup();
                    ChromeOptions options = new ChromeOptions();
                    options.setImplicitWaitTimeout(Duration.ofSeconds(10));
                    return new ChromeDriver();
                });
        //Initialize your page with WrappedElements
        loginPage = WrappedElements.initPage(LoginPage.class);
    }
```

Enjoy writing your clean tests without any explicit waits!
```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.chrome.*;
import org.testng.Assert;
import org.testng.annotations.*;
import io.github.bobfrostman.config.WrappedElements;
import wrappedelements.page.LoginPage;

public class TestNGSmokeTest {

    private LoginPage loginPage;

    @BeforeClass
    public void setUp() {
       //register webdriver creation function (whatever you want)
        WrappedElements.config()
                .driverCreator(() -> {
                    WebDriverManager.chromedriver().setup();
                    ChromeOptions options = new ChromeOptions();
                    options.setImplicitWaitTimeout(Duration.ofSeconds(10));
                    return new ChromeDriver();
                });
        //Initialize your page with WrappedElements
        loginPage = WrappedElements.initPage(LoginPage.class);
    }

    @BeforeMethod
    public void beforeMethod() {
        loginPage.driver().get("https://www.saucedemo.com/");
    }

    //Simple login test feel free to try it. Quite pretty, isn't it?
    @Test
    public void simpleUIInteractionTest() {
        loginPage.loginInput().sendKeys("standard_user");
        loginPage.passwordInput().sendKeys("secret_sauce");
        loginPage.loginButton().click();

        Assert.assertEquals(loginPage.driver().getCurrentUrl(), "https://www.saucedemo.com/inventory.html");
    }

    @AfterMethod
    public void tearDown() {
       //WebDriver object will be created or recreated automatically if it's closed using driverCreatorFunction
       //on next driver.get(String) function invocation
        if (loginPage.driver() != null) {
            loginPage.driver().quit();
        }
    }
}
```