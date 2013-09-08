---
title: Automating MySQL Setup for Jenkins Jobs
layout: post
extract: A Jenkins Plugin.
---

Creating databases is one of those repetitive tasks when setting up
continuous integration for a project.  At work we use Jenkins.  So
we've developed the
[jenkins-mysql-job-databases-plugin](http://github.com/codevise/jenkins-mysql-job-databases-plugin)
to automate the required steps.

The plugin creates a database and a user for the job, granting access
only to the job's database.  User credentials and database name are
exported to environment variables, which is a great fit with the
[ci_config_generator](http://github.com/codevise/ci_config_generator)
gem we saw in a
[previous post](/2013/09/05/generating-ignored-config-files-for-ci.html).
In a Rails project, for example, you could commit the following file
to wire everything up:

    # database.yml.ci
    test:
      adapter: mysql2
      encoding: utf8
      database: %{MYSQL_DATABASE}
      username: %{MYSQL_USER}
      password: %{MYSQL_PASSWORD}

The plugin itself is
[written in Ruby](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+plugin+development+in+Ruby).
Since it is
[unclear](https://github.com/jenkinsci/jenkins.rb/issues/67) how to
handle global configuration options in Ruby plugins, the credentials
of the MySQL user that creates databases and grants access are hard
coded in the plugin at the moment.  Apart from this caveat, the plugin
has worked well for us so far.
