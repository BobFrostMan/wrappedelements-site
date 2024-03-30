+++
archetype = "chapter"
title = "Interactors"
weight = 4
+++
By default, wrapped elements will wait for all element become clickable before any active interactions (click, sendKeys, etc).
It is possible because of **Interactor** mechanism, that allows you to configure waits for elements directly on page object layer.

**Interactor** - is the entity that describes the behavior with web element.
It actually, an opportunity to specify your own waiter for specific web element.

Available values are next:
- IMMEDIATELY (no waiters applied)
- UNTIL_VISIBLE (waits for element to be displayed before physical interaction)
- UNTIL_CLICKABLE (waits for element to be displayed and be enabled)
- VERTICAL_SCROLL_UNTIL_VISIBLE (scrolls down until the element will be visible in the view port)
To use it just specify **waitUntil** value of **@WebElement** annotation.

For instance:
You need to wait for element to become just visible (and may not be clickable) on some ui element called 'heisenberg'.
```java
@WebElement(value = "//div[@class='heisenberg']", waitUntil = UNTIL_VISIBLE, timeout = 15)
ClickableElement heisenberg();
```
In this case, any time you perform any kind of physical interaction (click, sendKeys, etc.), on heisenberg() element, it will wait for heisenberg() to become visible first, with 15 seconds timeout.


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
You can create you own waiter and use it with **wrappedelements**.
It is possible by implementing **IElementInteractor** interface.

To do that:
1. Implement your own Interactor class by implementing **IElementInteractor** interface and register it in WrappedElements framework:
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
    WrappedElements.config().registerInteractor(new WaitUntilMyCustomConditionsMet());
```
3. Now you can specify it in page object class using **@WebElement** 'waitUntil' value.
```java
    @WebElement(value = "//div[@class='heisenberg']", waitUntil = "custom_wait")
    ClickableElement heisenberg();
```
That's it!
{{% notice style="tip" %}}
If you don't like to use plain strings...
Since everything is an interface you can do next trick.
Create your own common page interface:
```java
public interface CommonPage extends IPage {
    String CUSTOM_WAIT = "custom_wait";
}
```
And extend this interface with your exact page: 
```java
public interface MyBreakingBadPage extends CommonPage {
    @WebElement(value = "//div[@class='heisenberg']", waitUntil = CUSTOM_WAIT)
    ClickableElement heisenberg();
}
```
This will allow you to avoid plain string like "custom_wait" in annotation value. No classes with constants, no additional imports, no plain strings, just inheritance.

At makes your code cleaner at some point). 
{{% /notice %}}