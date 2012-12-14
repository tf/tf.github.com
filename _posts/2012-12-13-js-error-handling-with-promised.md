---
title: JavaScript Error Handling with Promises
layout: post
---

Define a hierarchy of error classes:

    awesome.Error = Class.extend({
      init: function(message) {
        this.message = message;
      }
    });
    awesome.FetchError = awesome.Error.extend();

Now when when an error occurs, we simply reject the returned promise
passing a new instance of one or our error classes:
    
    awesome.fetch = function () {
      return new Promise(function(deferred) {
        ...
        deferred.reject(new awesome.FetchError('Could not fetch.'));
      });
    }

A consumer can handle errors in a declarative manner:

    return awesome.fetch()
      .pipe(function(result) {
        return new consumer.Wrapper(result);
      }
      .failWith(awesome.FetchError, function(error) {
        return new consumer.UpdateError({innerError: error});
      })
      .fail(function()  {
        // Something else went wrong
      });