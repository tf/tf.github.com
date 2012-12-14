---
title: Provisioning with Chef-Solo, Capistrano and Roundsman
layout: post
---

Neat automatically removes the last columns's gutter. However, if you
are queueing more than one row of columns within the same parent
element, you need to specify which columns are considered last in
their row to preserve the layout. Use the omega mixin to achieve this.

All the default settings in Neat can be modified, as long as these
overrides occur before importing Neat (failing to do so will cause the
framework to use the default values). The most straighforward way to
achieve this is creating a _grid-settings.scss file in the root of
your stylesheets folder, then importing it before Neat.

### Why exactly

We built [Neat](http://google.com) with the aim of promoting clean and semantic markup; it
relies entirely on Sass mixins and does not pollute the HTML with
presentation classes and extra wrapping divs. It is less explicit—as
in requires fewer mixins—than most other Sass-based frameworks and
comes with built-in table-cell layout support.

1. This is the first item.
2. But there is a second.

And we also want to make use of unordered lists:

* Those look like this.
* While this is the second item.

Finally we will give `code` examples:

    var json = example.get();
    return true;

And the the text continues.