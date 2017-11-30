# wisej-selenium

Selenium WebDriver test automation support for Wisej .NET development platform.

wisej-selenium is based on [QxWebDriver-NET](http://github.com/tfreitasleal/QxWebDriver-NET) with additional classes to support [Wisej development platform](http://wisej.com).

THIS FILE WAS ADAPTED FROM QxWebDriver-NET AND IS A WORK IN PROGRESS.

## Project Status

wisej-selenium is a work in progress. The API is subject to change without notice, not all of Wisej widgets/controls are supported. Don't let that stop you from playing with it, giving us feedback on the [issues tab](http://github.com/iceteagroup/wisej-selenium/issues) and sending pull requests. Thanks!

## Project Goal

The goal of this project is to provide an API that facilitates writing [WebDriver](http://seleniumhq.org/docs/03_webdriver.html)-based interaction tests for [Wisej](http://wisej.com) applications by abstracting away the implementation details of qooxdoo/Wisej JavaScript widgets.

## [Wisej.Web.Ext.Selenium](http://github.com/iceteagroup/wisej-selenium/tree/master/Wisej.Web.Ext.Selenium/Wisej.Web.Ext.Selenium)

This is the WisejWebDriver project that embeds QxWebDriver and extends it with classes that try to match Wisej UI classes.

## [Wisej.Web.Ext.Selenium.MSTest](http://github.com/iceteagroup/wisej-selenium/tree/master/Wisej.Web.Ext.Selenium/Wisej.Web.Ext.Selenium.MSTest)

This project is a set of helper classes to simplify the manipulation of Wisej widgets. Most methods handle assertions as needed (eg. close Form and assert it's closed).

## Widget Interfaces

WisejWebDriver provides a set of widget classes similar to WebDriver's support classes, each of which implements _Qooxdoo.WebDriver.UI.IWidget_ or one or more of the interfaces inheriting from it, such as _ITouchable_, _ISelectable_ or _IScrollable_. These interfaces allow complex actions to be performed by relatively few API calls.

Widgets are obtained by calling _QxWebDriver.FindWidget(by)_. where _by_ is any locator strategy that finds a DOM element which is part of a qooxdoo/Wisej widget. _FindWidget_ will determine the qooxdoo/Wisej class of the widget, its inheritance hierarchy and the interfaces it implements, and use this information to decide which _IWidget_ implementation to return.

Similar to _IWebElement.FindElement_, _IWidget.FindWidget_ will restrict the search to children of the current widget.

### Examples

The best way to learn about the various widget interfaces is to check out the demo integration tests in [Wisej.SeleniumDemo](http://github.com/iceteagroup/wisej-selenium/tree/master/Wisej.SeleniumDemo).

## Locating Widget Elements

WebDriver's built-in _"By"_ strategies like _By.XPath_ or _By.ClassName_ generally don't work too well with the entangled and dynamic DOM structures generated by qx.Desktop applications. The _By.Qxh_ strategy (Qx for qooxdoo, h for hierarchy) provides an alternative approach that searches for elements by using JavaScript to traverse the application's widget hierarchy.

For example, the [qooxdoo Feed Reader](http://demo.qooxdoo.org/current/feedreader/)'s UI hierarchy looks like this (easily determined by opening the Feed Reader in the [Inspector](http://www.qooxdoo.org/Inspector/)):

    qx.ui.root.Application
    - qx.ui.container.Composite
      - feedreader.view.desktop.Header
      - feedreader.view.desktop.ToolBar
        - qx.ui.toolbar.Button
      [...]

A Qxh locator that finds a toolbar button with the label _Reload_ could look like this:

    By by = By.Qxh("child[0]/feedreader.view.desktop.ToolBar/[@label=Reload]");

As you can see, the syntax is similar to XPath, consisting of location steps separated by slashes. While searching, each location step selects a widget which will be used as the root for the rest of the search.

### Supported Steps

*   **foo.bar.Baz** A string containing dots will be interpreted as the class name of a widget. Uses _instanceof_ internally so inheriting widgets will be found as well.
*   **child[N]** Signifies the Nth child of the object returned by the previous step.
*   **[@attrib{=value}]** Selects a child that has a property _attrib_ which has a value of _value_. "Property" here covers both qooxdoo as well as generic JavaScript properties. As for the values, only string comparisons are possible, but you can specify a RegExp which the property value will be matched against. _toString()_ is used to compare non-String values.
*  __\*__ is a wildcard operator. The wildcard can span zero to multiple levels of the object hierarchy. This saves you from always specifying absolute locators, e.g. the example above could be rewritten as


    By by = By.Qxh("*/[@label=Reload]");

This will recursively search from the first level below the search root for an object with a _label_ property that matches the regular expression _/Reload/_. As you might expect, these recursive searches take more time than other searches, so it is good practice to be as specific in your locator as possible.

Note that the Qxh strategy will only return the **first match** for the locator expression, so it can't be used with _WebDriver.FindElements_.

### Relative Locators

The root node where a _By.Qxh_ locator will begin searching is determined by its context: When used with _QxWebDriver.FindWidget_, the children of the qooxdoo application's root widget will be matched against the first step.
When used with _IWidget.FindWidget_, the widget itself will be the root node for the search.

### Inline Applications

Inline applications extending [qx.application.Inline](http://demo.qooxdoo.org/current/apiviewer/#qx.application.Inline) can have multiple root widgets. To locate a child widget, first find the DOM node to which the inline root is attached, then search within it:

    WebElement root = driver.FindElement(By.Id("inlineRoot"));
    // Within a WebElement, we can't use FindWidget, so we use FindElement...
    WebElement buttonEl = root.FindElement(By.Qxh("qx.ui.container.Composite/qx.ui.form.Button"));
    // ...and get a IWidget for it
    IWidget button = (IWidget) driver.GetWidgetForElement(buttonEl);

## Extending WisejWebDriver

_FindWidget_ uses a _IWidgetFactory_ to determine which widget class to instantiate. An alternative class implementing _IWidgetFactory_ and probably extending _DefaultWidgetFactory_ can be passed to the _QxWebDriver_ constructor to support custom widgets.

## Browser Support

In theory, WisejWebDriver should work with any WebDriver that implements _IJavascriptExecutor_. Drivers for Chrome, Firefox, Edge and Opera do work all right although the Edge driver only supports one running instance. The current Internet Explorer driver is said to work but in practice is too hard to get it working.
