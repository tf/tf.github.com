---
title: YARD Documentation with Emacs
layout: post
extract: "Configuring `fill-paragraph`"
---

In Emacs, the
[`fill-paragraph` command](https://www.gnu.org/software/emacs/manual/html_node/emacs/Fill-Commands.html)
lets you reformat a chunk of text to break lines at a certain
width. Bound to `M-q` by default, it is a handy way to enforce
consistent line widths inside text documents.

Recently, I've been editing a lot of inline documentation in Ruby
code using YARD. A typical doc comment looks like this:

```ruby

# Reverses the contents of a String or IO object. 
# 
# @param [String, #read] contents the contents to reverse 
# @return [String] the contents reversed lexically 
def reverse(contents) 
  # ...
end
```

As descriptions got longer, I found myself wishing to reformat
paragraphs. While Emacs knows how to fill paragraphs inside Ruby
comments, hitting `M-q` still turned text into unreadable gibberish:

```ruby
# Reverses the contents of a String or IO object.  @param [String,
# #read] contents the contents to reverse @return [String] the
# contents reversed lexically
```

Turns out there is an easy way to tell Emacs how to recognize
paragraphs: `paragraph-separate` and `paragraph-start`.  Let's define
a function that sets these variables to YARD specific values. The
`concatenate` call is only there to improve readability.

```lisp
(defun yard-paragraph-boundaries ()
  (interactive)
  ; Paragraphs are separated by lines containing only a # character
  (setq paragraph-separate "[ \t]*#[ \t]*$")
  ; Paragraphs start with YARD tags or list items
  (setq paragraph-start
      (concatenate
       'string
       "^[ \t]*"         ; some whitespace
       "#[ \t]*"         ; a # character followed by whitespace
       "\\("
         "@[[:alpha:]]+" ; a YARD tag
       "\\|"             ; or
         "-"             ; a list item
       "\\)"
       "\\([ \t]+.*\\)?" ; an optional text
       "[ \t]*$")))      ; some more whitespace
```

Finally, we can call the function whenever `ruby-mode` is entered:

```lisp
(add-hook 'ruby-mode-hook
	  '(lambda ()
	    (yard-paragraph-boundaries)))
```

Now text inside YARD comments can be reformatted with a single key
stroke. Emacs even preserves custom indentation levels:

```ruby
# @param [String] A really long description text which spans
#   multiple lines and has a custom indentation, which is
#   preserved even when the paragraph is reformatted.
```
