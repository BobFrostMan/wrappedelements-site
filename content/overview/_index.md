+++
archetype = "chapter"
title = "Overview"
weight = 1
+++
## A few words before dig in
The main drawback of web/mobile ui test automation is slow implementation and it's quite high support cost.
wrapedelements designed to minimise drawbacks.

Usually Page Object pattern in java implemented as Page object classes.

You may be surprised how much flexibility you will get if you implement Page Object classes and components as interfaces.

## Small comparison with other frameworks wrappers

Wrappedelements provides some juicy features to do that that other selenium webdriver wrappers frameworks don't have.

| Features           							                                        | wrappedelements	         | htmlelements             | Selenide                 |
|-------------------------------------------------------------------|--------------------------|--------------------------|--------------------------|
| Supported drivers                         				                    | Chrome, FF, Safari, Edge | Chrome, FF, Safari, Edge | Chrome, FF, Safari, Edge |
| Waiters and timeouts configuration in page object classes		       | Yes            	         | 	No     		               | 	No		                    |
| No web element synchronization code in tests				                  | Yes	          	          | 	No		                    | 	No		                    |
| Dynamic locators support       	      				                        | Yes                      | 	Yes		                   | 	No           	          |
| Automatic web driver recreation (if dead)				                     | Yes	          	          | 	No		                    | 	No	                     |
| Custom components reusable						                                  | Yes	          	          | 	Yes		                   | 	No	       	             |
| Page classes reusable for different platforms (Android, IOS, Web) | Yes	                     | 	No                      | 	No                      |
| Built-in web elements logging 				                                | Yes	                     | 	No                      | 	No                      |

# Features 
## Clear and simple Page object classes
Imagine page object model description as an interface!
That makes page object classes clear and avoids additional code.
```java
import io.github.bobfrostman.annotation.WebElement;
import io.github.bobfrostman.element.clickable.ClickableElement;
import io.github.bobfrostman.page.IPage;

public interface LoginPage extends IPage {

    @WebElement(value = "//form//input")
    ClickableElement loginInput();

    @WebElement(value = "(//form//input)[2]")
    ClickableElement passwordInput();

    @WebElement(value = "//input[@type='submit']")
    ClickableElement loginButton();

}
```
#### Question: What is ClickableElement? What is the difference between ClickableElement and standard selenium WebElement
**ClickableElement** is a wrapper class around standard selenium **WebElement**. It's the basic web element class that provides additional functionality and allows you to use built-in waiters.
It provides an opportunity to adjust waiters and timeout conditions on Page Object level by adjusting **@WebElement** annotation values.

For instance, you can adjust:
- Wait condition - by changing **waitUntil** value
- Element wait timeout - by changing **timeout** value (in seconds)
- Element name (for logging purposes) - by changing **name** annotation value
#### Question: How can I create instance of my page object interface?
Quite simple:
```java
LoginPage loginPage = WrappedElements.initPage(LoginPage.class);
```

## No waiters on test level that leaves code clean and clear
All element's interaction waiters encapsulated on page object layer. No additional waiters in element interaction code, - enjoy the usage:)
```java  
@Test
public void simpleUIInteractionTest() {
    loginPage.loginInput().sendKeys("standard_user");
    loginPage.passwordInput().sendKeys("secret_sauce");
    loginPage.loginButton().click();
    ...
}
```
By default wrapped elements will wait for all element become clickable before any active interactions (click, sendKeys, etc).
It is possible because of 'Interactor' mechanism, that allows you to configure waits for elements directly on page object layer.

#### Question: How can I set some different waiter before elements interaction?
Feel free to use 'waitUntil' value of @WebElement annotation.

For instance:
You need to wait for element to become just visible (and may not be clickable) on some ui element called 'heisenberg'.
```java
@WebElement(value = "//div[@class='heisenberg']", waitUntil = UNTIL_VISIBLE, timeout = 15)
ClickableElement heisenberg();
```
Any time you perform any kind of physical interaction (click, sendKeys, etc.), on heisenberg() element, it will wait for heisenberg() to become visible first, with 15 seconds timeout.

Available values are next:
- IMMEDIATELY (no waiters applied)
- UNTIL_VISIBLE (waits for element to be displayed before physical interaction)
- UNTIL_CLICKABLE (waits for element to be displayed and be enabled)
- VERTICAL_SCROLL_UNTIL_VISIBLE (scrolls down until the element will be visible in the view port)

#### Question: What if I want to set some custom waiter for all elements by default?
To do that:
1. Implement your own Interactor class by implementing IElementInteractor interface and register it in WrappedElements framework:
```java
import io.github.bobfrostman.wrapper.interactor.IElementInteractor;

public class WaitUntilMyCustomConditionsMet implements IElementInteractor {

    @Override
    public String name() {
        return "custom_wait";
    }
    
    Override
    public boolean isReadyForInteraction(String methodName, By by, WebDriver webDriver) {
        //my interaction logic that says if method name is "click" or "sendKeys" or anything else, then wait for my custom conditions
    }
}
```
2. Register you interactor in WrappedElements class before your tests run with code:
```java
    WrappedElements.config().defaultElementInteractor(new WaitUntilMyCustomConditionsMet());
```
That's it. Your custom waiter will be applied by default for all elements that doesn't have 'waitUntil' value specified explicitly.

#### Question: What if I want to set some custom waiter for element?
You can create you own waiter and use it with wrappedelements.
It is possible by implementing your IElementInteractor interface.
Interactor - is the entity that describes the behavior with web element.

To do that:
1. Implement your own Interactor class by implementing IElementInteractor interface and register it in WrappedElements framework:
```java
import io.github.bobfrostman.wrapper.interactor.IElementInteractor;

public class WaitUntilMyCustomConditionsMet implements IElementInteractor {

    @Override
    public String name() {
        return "custom_wait";
    }
    
    @Override
    public boolean isReadyForInteraction(String methodName, By by, WebDriver webDriver) {
        //my interaction logic that says if method name is "click" or "sendKeys" or anything else, then wait for my custom conditions
    }
}
```
2. Register you interactor in WrappedElements class before your tests run with code:
```java
    WrappedElements.config().registerInteractor(new WaitUntilMyCustomConditionsMet());
```
3. Now you can specify it in page object class using **@WebElement** 'waitUntil' value.
```java
    @WebElement(value = "//div[@class='heisenberg']", waitUntil = "custom_wait")
    ClickableElement heisenberg();
```

## Easy element's locator definition for different platforms
It's quite common case - run ui mobile tests on different platforms.
Enjoy running same tests with different locators for different platforms without changing test logic:

Page example:
```java
    @WebElement(value = "linkText=someLinkText", name = "Web element")
    @AndroidElement(value = "//span/a", name = "Android element")
    @IOSElement(value = "just_some_ios_element_id", name = "IOS element")
    ClickableElement multiplatformElement();
```
To run tests on android just invoke:
```java
    WrappedElements.config().setPlatform(IKnowPlatforms.ANDROID);
```
And element locators will be picked up from **@AndroidElement** annotation.
Specify IOS platform, and framework will run tests with locators in **@IOSElement**

## Dynamic locators support
WrappedElements supports dynamical element locators, we know that sometimes you need some specific element that may be found only dynamically.

Declaration example:
```java
    @WebElement(value = "//div[text() = '%s']|//span[contains(text(), '%s')]")
    ClickableElement elementWithText(String text);
    
    @WebElement(value = "//div[text() = '%d']|//span[contains(text(), '%d')]")
    ClickableElement elementWithDigits(int number);
    
    @WebElement(value = "//li/a[contains(text(), '${first_name} ${ last_name }')]")
    ClickableElement elementWithNamedParameter(@Parameter("last_name") String lastName, @Parameter("first_name") String firstName);
```
Usage example:
```java
String name = page.elementWithDigits(0).getText();
String lastName = page.elementWithDigits(1).getText();
String someOtherDynamicalText = page.elementWithNamedParameter(lastName, name).getText();
elementWithText(someOtherDynamicalText).click();
```
## Components design
It's quite common case when you have some component on the page that duplicated on different page, and may contain elements that duplicates from page to page.
For instance: in your header you have a basket button that contains from basket element button and products count label.

To create BasketButton component you need:
1. Extend WrappedComponent interface:
```java
public interface BasketButton extends WrappedComponent {

    @WebElement(value = ".//span[@class=\"shopping_cart_badge\"]", waitUntil = IMMEDIATELY)
    ClickableElement notificationsCountLabel();

    @WebElement(value = ".//a[@class=\"shopping_cart_link\"]", waitUntil = VERTICAL_SCROLL_UNTIL_VISIBLE)
    ClickableElement basketButton();

}
```
2. Specify locator for your basket button on the page use **@WebComponent** annotation
```java
public interface Header extends WrappedComponent {

    @WebComponent("//div[@class='basket_button_locator']")
    BasketButton basket();
    
}
```
In such way **wrappedelelements** will search basketButton() and notificationsCountLabel() by relative locator from from BasketButton component.

Such approach allows you to reuse component with different locators on different pages.
WrappedComponents also supports @IOSComponent and @AndroidComponent annotations, so you can define same component with different locators for different platforms

## List elements support
Common case when you need to find a list of components or elements on the page, **wrappedelements** also supports such functionality

```java
import io.github.bobfrostman.wrapper.element.impl.ClickableElement;

public interface InventoryPage extends IPage {
    @WebComponent("//*[@class='inventory_item']")
    List<InventoryItem> inventoryItems();

    @WebElement(".inventory_item")
    List<ClickableElement> inventoryItems();
}
```
Note that it's recommended to use xpath locator for list of components.
## Built-in elements interaction logger
**Wrappedelements** has a build-in slf4-api logging interface, so you can see elements interaction's logs by adding different logging libraries to your dependencies:
For instance to use logback logging just add next dependencies to your pom.xml
```
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>${your_logback_version}</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>${your_logback_version}</version>
</dependency>
```