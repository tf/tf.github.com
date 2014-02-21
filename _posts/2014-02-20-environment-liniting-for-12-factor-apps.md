---
title: Introducing Env Lint
layout: post
extract: Checking environment variables for 12 Factor Apps
---

We've been following the
[12 factor approach](http://12factor.net/config) of configuring our
Rails apps through environment variables.  Tweaking options through
the `env:set` task provided by
[recap](https://github.com/tomafro/recap) feels much more light weight
than manually fiddling with `yml` files on the server.  Moreover, the
[dotenv](https://github.com/bkeepers/dotenv) gem makes controlling
your development environment very comfortable. While the `.env` file
itself is not checked into version control, there is a `.env.example`
file which acts as a starting point for new developers and documents
the available options.

Still, there were some pain points left:

* Whenever new variables were added to the app's configuration, the
  process of updating the staging and production environments turned
  out to be rather error prone. Even having carefully compared the
  lists of defined variables, only after booting the newly deployed
  app could one be sure that nothing was missing.
  
* This was even harder as small spelling errors in variable names
  either in the code or on the command line were easy to miss.
  
* Finally, the `.env.example` file would quickly become outdated,
  incomplete or simply incorrect.
  
To resolve all of these issues we came up with the
[env_lint gem](https://github.com/tf/env_lint), which is comprised of
three components:

* A wrapper around `ENV` called `LintedEnv` which fails loudly
  whenever variables are not defined or misspelled variable names are
  used.
  
* A capistrano task to verify all required environment variables are
  defined on the production machine --- even before deploying a new
  revision of the app.
  
* A rake task which helps developers make sure that their local
  environment is in shape.
  
For all of this the application's `.env.example` file becomes the
primary source of truth. Only variables explained there can be used in
the code or passed to tasks like `env:set`. The gem also parses the
description comments to generate helpful error messages.  A typical
`.env.example` file looks like this:

    # Host name used in absolute urls 
    HOST_NAME=my.app.com
     
    # Redirect users to the https site
    # ENFORCE_SSL=true

Commented out assignments signify optional variables. `LintedEnv`
makes sure you always supply a default value when fetching one of
these.  

    linted_env.fetch(:enforce_ssl)
    # => raises an error telling you to supply a default

All in all, it's almost impossible for your example file to rot. And
when it's time to deploy, a simple

    $ cap production env:lint

makes sure your app won't crash after restart only because an
environment variable is missing.

More details can be found in the
[README](https://github.com/tf/env_lint).

Finally, the
[ci-config-generator](/2013/09/05/generating-ignored-config-files-for-ci.html)
is nice addition to this team, letting you define a `.env.ci` file
which contains a concrete, version controlled configuration which is
verified in your continuous integration runs.


