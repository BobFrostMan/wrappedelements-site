+++
archetype = "chapter"
title = "Components"
weight = 3.2
+++

## Components design
It's quite common case when you have some component on the page that duplicated on different page, and may contain elements that duplicates from page to page.
For instance: in your header you have a basket button that contains from basket element button and products count label.

To create BasketButton component you need:
1. Extend **WrappedComponent** interface:
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

{{% notice style="info" %}}
Note that, it's recommended to use xpath locator for list of components.
{{% /notice %}}

```java
import io.github.bobfrostman.wrapper.element.impl.ClickableElement;

public interface InventoryPage extends IPage {
    @WebComponent("//*[@class='inventory_item']")
    List<InventoryItem> inventoryItems();

    @WebElement(".inventory_item")
    List<ClickableElement> inventoryItems();
}
```
