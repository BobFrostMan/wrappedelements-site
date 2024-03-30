+++
archetype = "chapter"
title = "Web Element Wrapper"
weight = 3.1
+++

Web element is the main basic unit for page object. It's basically any clickable, visible or intractable element on a web page.

The main class that represents web element in **wrappedelements** framework is **ClickableElement**
**ClickableElement** is a wrapper class around standard selenium **WebElement**. It's the basic web element class that provides additional functionality and allows you to use built-in waiters.
It provides an opportunity to adjust waiters and timeout conditions on Page Object level by adjusting **@WebElement** annotation values.

For instance, you can adjust:
- Web element locator - by set **value** for annotation (it can be anything, id, name, tagname, xpath, css)
- Wait condition - by changing **waitUntil** value
- Element wait timeout - by changing **timeout** value (in seconds)
- Element name (for logging purposes) - by changing **name** annotation value

How it looks like in code?
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
First, your page object class is actually an interface! And it should extend IPage interface for **wrappedelements** to detect this interface as a PageObject.

All interface methods that return **ClickableElement** represents web elements.
You can also use you own custom methods with default modificator as well as a usual method.

We are using **ClickableElement** paired with **@WebElement**, **@AndroidElement**, **@IOSElement** annotations.
**@WebElement** will work for common web ui, while **@AndroidElement** will be used for Android platform and **@IOSElement** for IOS platform.

These annotations signalise that your method should be wrapped with element.
If you have a mirror like mobile application you can use these annotations in combination like this:
```java
    @AndroidElement(value = "//my-awesome-android-element-locator")
    @IOSElement(value = "//my-awesome-ios-element-locator")
    @WebElement(value = "//my-awesome-web-element-locator")
    ClickableElement myAwesomeElement();
```
In such case your test level will remain same:
```java
    page.myAwesomeElement().click();
```
While different locators will be used for different platform types.
To run tests on android just invoke these piece of code before your tests run:
```java
    WrappedElements.config().setPlatform(IKnowPlatforms.ANDROID);
```
And element locators will be picked up from **@AndroidElement** annotation.
Specify IOS platform, and framework will run tests with locators in **@IOSElement**

#### Question: Why I don't see any explicit waits? How this will work in real life?
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
By default, wrapped elements will wait for all element become clickable before any active interactions (click, sendKeys, etc).
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

### Dynamic locators
**wrappedElements** supports dynamical element locators. What is it?

We know that sometimes you need some specific element that may be found only dynamically.

For instance, you want to interact with different elements in tests based on text value this element has. 

Due to that fact tha all elements declared as methods, it means that you can pass a part of locator as a method parameter to find required element.
```java
    @WebElement(value = "//li/a[contains(text(), '${category_name}')]")
    ClickableElement category(@Parameter("category_name") String categoryName);
```
Then usage will be:
```java
    page.category("Drinks & Beverages").click();
```

Another a bit more complex example of dynamic locator usage
Declaration example:
```java
    //you can use %s format to automatically format locator with string parameter values
    @WebElement(value = "//div[text() = '%s']|//span[contains(text(), '%s')]")
    ClickableElement elementWithText(String text);
    
    //or %d for int numbers
    @WebElement(value = "//div[text() = '%d']|//span[contains(text(), '%d')]")
    ClickableElement elementWithDigits(int number);
    
    //or use named parameter with @Parameter annotation usage and ${placeholder} in value annotation
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
With such implementation, you can search required web elements based on your needs, and no other additional *findElement(By)* invocations.
