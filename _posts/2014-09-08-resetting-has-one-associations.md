---
title: Resetting Has One Assocations
layout: post
extract: Lazy reloading of cached associations
---

_Tested with Rails 4.0_

Just a quick note since I could not find this information
anywhere. Making sure a `has_many` association is reloaded on next
access is easy. Just call the
[`reset` method](http://api.rubyonrails.org/classes/ActiveRecord/Associations/CollectionProxy.html#method-i-reset)
provided by the collection proxy:

    class Post
      has_many :comments
    end

    post.comments.reset
    post.comments # fetches comments from database

But how to accomplish the same for `has_one` associations?

    class Post
      has_one :last_comment, -> { order 'created_at' }, class_name: "Comment"
    end

Calling `post.last_comment` just returns a comment (or `nil`). Of
course, you can pass `true` as the `force_reload` parameter:

    post.last_comment(true) # fetches comment from database

But what if there is no need to reload the association now, only next
time it is used?  Turns out the underlying association object can
be accessed via the `associations` method:

    post.associations(:last_comment).reset

Now the cache is cleared and the next call to `post.last_comment` will
fetch the record from the database.
