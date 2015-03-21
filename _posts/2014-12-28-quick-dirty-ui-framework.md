---
layout: post
title: A Quick UI Automation Framework
description: "Built in Ruby using Capybara"
modified: 2014-12-28
tags: [uiautomation, capybara, pageobjectpattern, webdriver]
---

 I built a quick and dirty UI test framework for testing web application UIs which could serve the purpose at hand. The frameworks has 3 layers, specs, pages and page_uis... Before I proceed... if you are already thinking about, how it is different from "Page Object Pattern", here is what I have to say. IMHO, Page Object Pattern is quite over-rated.

###The heart of the framework

The core component of the framework is a PageComponent class which lets model the UI element hierarchy as a tree in the page_ui layer. This PageComponent class can be a tree of other PageComponents represented as nested maps (hashes in Ruby) or a list of other PageComponents. A PageComponent knows it's parent hierarchy, so I leveraged CSS selector hierarchy to locate elements. By this, if I have a Search Form with the class "search-form" and a search input with the class "search-field", I could create a PageComponent with the locator ".search-form" and another PageComponent ".search-field" and make the first the parent of the second and the framework knows how to traverse the hierarchy. The simplicity of the framework is in the way these UI trees could be built by an over-ridden "[]=" and "[]" operators which takes care of creating PageComponents and establishing parent-child relationships. For the above example, all I have to do is write the below lines (in Ruby) and the framework knows how create the parent child hierarchy.

{% highlight css %}
self[:search_form] = ".search-form"
self[:search_form][:search_field] = ".search-field"
{% endhighlight %}

(The trade-off, of course, is that I can only use CSS for location, which I was fine with.)

With the page_ui layer taking care of encapsulating the UI model, the pages layer exposes the page services making use of the page_ui layer and the specs mostly interact with the pages for performing UI flows and making assertions on the same.

Another aspect of the framework is the lazy-location of elements. The PageComponent locates an element upon a cue of its method "f" and till that point, only stores the locator information.

I created a [sample project using Amazon home page](https://github.com/sreedevivedula/quick_dirty_ui_tf) with the framework's core PageComponent class to demonstrate the points mentioned above. I appreciate any feedback and comments!
