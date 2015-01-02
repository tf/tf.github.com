---
title: Nested Helper Classes vs Private Helper Methods
layout: post
extract: Preventing namespace collisions
---

While criticism towards Rails helpers has almost become folklore
(procedural, global scope, junk drawers), I still find myself reaching
for them as a low ceremony way of making templates more concise and
building simple view layer APIs.

Obviously, the greatest downside of helper modules is how dangerous
extracting private methods becomes.

    module UsersHelper
      def user_section(user)
        content_tag(:section, user.name, class: css_class(user))
      end

      private

      def css_class(user)
        ['user', user.role].join(' ')
      end
    end

    module PostsHelper
      def post_section
        content_tag(:section, post.text, class: css_class(post))
      end

      private

      # collides with UsersHelper#css_class
      def css_class(post)
        'post'
      end
    end

While I always liked extracting classes to keep helper methods short,
only recently did I realize how a nuance in Ruby's constant look-up
mechanism helps prevent namespace collisions as above.

Let's look at a variant of the above helpers that uses nested classes
instead of private methods.

    module UsersHelper
      def user_section(user)
        Section.new(user).render(self)
      end

      class Section < Struct.new(:user)
        def render(template)
          template.content_tag(:section, user.name, class: css_class)
        end

        private

        def css_class
          ['user', user.role].join(' ')
        end
      end
    end

    module PostsHelper
      def post_section(post)
        Section.new(post).render(self)
      end

      class Section < Struct.new(:post)
        def render(template)
          template.content_tag(:section, post.text, class: css_class)
        end

        private

        def css_class
          'post'
        end
      end
    end

At first glance, one could expect the same problem as above. Instead
of both defining an equally named private method, both modules now
contain a nested class with the same name. Once both modules get
included into the view object only one can win, right? That's sure the
way it looks when we try to reference one of the nested classes from a
template:

      # app/views/users/index.html.erb
      <%= Section == PostsHelper::Section # => true %>

But in contrast to method invocations, which are resolved by the
object handling the call, constant look-up uses the lexical scope at
the point where the constant is referenced. So referring to a
`Section` class inside `UsersHelper` still resolves to
`UsersHelper::Section`, no matter whether there is a colliding
`PostsHelper::Section` class when including helper modules into the
view context.

Nested helper classes thus provide a safe space for refactoring
towards small methods. Helper methods end up containing only wiring
code, keeping implementation details from leaking into tests or
templates.

