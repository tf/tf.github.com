---
title: Handling Errors with Promises
layout: post
extract: Asynchronous Error Handling
---

In this post we look at some techniques for dealing with asynchronous
errors using promises.  The code examples apply Any Promise A+ compliant library will do: RNVP
(a.k.a Ember promises), Q, or Angular's $q. Note though that jQuery's
Deferred still misses some crucial pieces as we will see.

### The Basics

A promise provides a `then` method which takes two functions as its
parameters: one which is called if the promise is fulfilled, and
another which is called if the promise is rejected.

### Chained Promises and Error Propagation



    function VoucherRedemption(features, voucher, xhr) {
      this.perform = function() {
        return xhr.post(redemptionPath(voucher)).then(function(response) {
          return features.enableById(response.featureId);
        });
      };
    }
    
### Integration with uncaught Exceptions
    
### Error Hierarchies

    vouchers.Error = lib.Object.extend({
      init: function(message) {
        this.message = message;
      }
    });
    vouchers.Error.prototype = new Error();
    
    vouchers.InvalidVocuherError = vouchers.Error.extend();
    vouchers.AlreadyRedeemedError = vouchers.Error.extend();

Translation

    function VoucherRedemption(features, voucher, xhr) {
      this.perform = function() {
        return xhr.post(redemptionPath(voucher)).then(
          function(response) {
            return features.enableById(response.featureId);
          },
          function(error) {
            if (error instanceof HTTPError && error.status === 404) {
              return new voucher.InvalidVoucherError();
            }
            else if (error instanceof HTTPError && error.status === 409) {
              return new voucher.AlreadyRedeemedError();
            }
          }
        );
      };
    }
    
    
