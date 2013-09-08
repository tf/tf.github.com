---
title: Generating Configuration Files for CI
layout: post
extract: Rake tasks for easing Continuous Integration setup.
---

When setting up continuous integration for a project, manually editing
host specific configuration has always annoyed me.  The
[ci_config_generator](http://github.com/codevise/ci_config_generator)
gem sets out to free you from fiddling with config files in your ci
job's workspace directory.

Instead, it lets you create templates for ignored config files.  For
each of those `.yml` files, simply commit an accompanying `.yml.ci`
file and run the `ci:config:generate` rake task at the beginning of
your ci job.  While copying files, the task interpolates environment
variables:

    # zencoder.yml.ci
    test:
      output_bucket: "com.example.test"
      api_key: "%{API_KEY}"

That way, passwords and other secret bits of data do not need to be
stored in the repository, but can be configures as build parameters
via your ci servers admin interface.

As an additional benefit, new developers on your team can follow a
self documented and continuously tested path to obtain a passing test
suite.
