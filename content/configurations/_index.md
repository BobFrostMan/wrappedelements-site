+++
archetype = "chapter"
title = "Configuration"
weight = 2
+++

## Configuration
**Wrappedelements** designed to be configurable and extendable framework. 
The entry point of framework is a static **WrappedElements** class, that provides next main functions:
- Page Object interfaces initialization
- WebDriver providence for all Page Object interfaces 
- Allows to config WrappedElements general settings

This section is an overview for the configuration part.

## Initial WrappedElements setup
The main object that allows you to interact with Web or Mobile UI is WebDriver object.
General configuration provided by config() method of **WrappedElements** class.
So the initial step for **Wrappedelements** is define a driver creation function:
```java
WrappedElements.config()
        .driverCreator(() -> {
            //any expression that creates WebDriver object
            //For instance ChromeDriver
            ChromeOptions options = new ChromeOptions();
            options.setImplicitWaitTimeout(Duration.ofSeconds(10));
            return new ChromeDriver();
        });
```

## Setting up framework for Mobile platforms
It's quite common case - run ui mobile tests on different platforms.
To do that, you need to add to your code annotations that will find elements specifically for you mobile platform:

Page example:
```java
    @WebElement(value = "linkText=someLinkText", name = "Web element")
    @AndroidElement(value = "//span/a", name = "Android element")
    @IOSElement(value = "just_some_ios_element_id", name = "IOS element")
    ClickableElement multiplatformElement();
```
In this case, configuration may look like this:
```java
    WrappedElements.config().setPlatform(IKnowPlatforms.ANDROID);
```
And element locators will be picked up from **@AndroidElement** annotation.
Specify IOS platform, and framework will run tests with locators in **@IOSElement**