---
title: Organizing Integration Test Helpers with Dominos
layout: post
extract: Get rid of those mixins.
---

The [domino](https://github.com/ngauthier/domino) gem by
[Nick Gauthier](https://github.com/ngauthier) provides a great way to
organize your integration test support methods.  Before, I used to
package test helpers as mixins.

    # spec/features/admin/user_locking_spec.rb
    feature 'user locking' do
      include UsersTestHelper
      
      scenario 'locking a user account' do
        user = create(:user)
        
        visit user_path(user)
        lock_user_button.click
        
        expect(page).to have_locked_notice
      end
    end
    
    # spec/support/integration/users_test_helper.rb
    module UsersTestHelper
      extend RSpec::Matchers::DSL

      def lock_user_button
        page.find('section.user [data-rel=lock]')
      end

      matcher :have_locked_notice do |*args|
        # check if there is an element matching .lock_notice
      end
    end
    
While those usally made for quite readable tests, there were two major
downsides:

* _Findability_ &mdash; It quickly became hard to tell where a certain
  helper method was defined as soon as more than one or two mixins
  were used in a test.
  
* _Naming clashes_ &mdash; Extracting private methods to clean up test
  helper methods came with the risk of introducing name collisions.
  
With dominos you can take a more object oriented approach:

    # spec/support/dominos/user_page.rb
    module Dom 
      UserSection < Domino
        selector 'section.user'
        
        def lock_button
          node.find('[data-rel=lock]')
        end
        
        def has_locked_notice?
          node.has_selector?('.lock_notice')
        end
      end
    end
    
The test from above now becomes:

    # spec/features/admin/user_locking_spec.rb
    feature 'user locking' do
      scenario 'locking a user account' do
        user = create(:user)
        
        visit user_path(user)
        Dom::UserSection.first.lock_button.click
        
        expect(Dom::UserSection.first).to have_locked_notice
      end
    end

While this arguably reads a bit more cryptic at first, there are a
couple of wins:

* It is now directly obvious where each methods is defined.
* Private methods can freely be extracted inside dominos.
* Custom matchers can be replaced by RSpec's built in predicate
  matchers.
* It is easy to adapt a component based mental model of your UI.

While there are other great features, the
[domino](https://github.com/ngauthier/domino) codebase only clocks in
at about 160 LOC including doc comments.  So its a rather lightweight
dependency to add to your testing stack.
