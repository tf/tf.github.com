---
title: Sass Mixins and Icon Fonts
layout: post
extract: Keeping style specific information to your stylesheet.
---

Sass mixins are a great tool for building abstractions and writing
intention revealing CSS code.  One application lies in using them with
icons fonts.  Instead of littering your HTML with style specific icon
classes, we can define mixins to apply icons in a declarative manner:

    a.edit {
      @include pencil-icon;
    }

The basic idea of the mixin definition is the following:

    @mixin icon($font-name, $char) {
      &:before {
        content: $char;
        font-family: $font-name;
      }
    }

    @mixin pencil-icon { @include icon("entypo", "\270E"); }
    @mixin warning-icon { @include icon("entypo", "\26A0"); }

This approach also makes it easy to swap icon fonts later on by
limiting duplication.

### Taking it further

Extracting other patterns concerning icon placement and styling, an
expressive language can be built up.  The following example uses some
simple mixins defined in
[this gist](https://gist.github.com/tf/6485675#file-background_icon_mixins-scss)
to display icons in the background of navigation links.

    nav a {
      @include background-icon($font-size: 20px);
    }

    a.edit {
      @include pencil-icon;
    }

    a.errors {
      @include warning-icon;
      @include background-icon-color(#d00);
      @include background-icon-animation(pulse);
    }

A careful selection of such mixins might make a great gem in the vein
of [Bourbon](http://bourbon.io) or other mixin libraries.
