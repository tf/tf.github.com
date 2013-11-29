---
title: Chef Solo Provisioning with Capistrano Roundsman and Berkshelf
layout: post
extract: Automating Server Setup without a lot of Infrastructure
---

[Chef](http://www.opscode.com/chef/) helps make server provisioning
reproducible and self documenting. Even more, a lot of the benefits
can be gained without maintaining the infrastructure surrounding a
chef server. Especially if you do not have an ops team carefully
curating a set of cookbooks representing your server landscape, a more
application-centric approach might work for you.

We tend to deploy our applications to
[phoenix servers](http://martinfowler.com/bliki/PhoenixServer.html)
which run as [Linux Containers (lxc)](http://linuxcontainers.org/)
using [libvirt](http://libvirt.org/). Each application contains an
application cookbook describing the required deployment environment.
Here is an excerpt from the typical directory layout of our projects.

    some_app/
      Berksfile
      Berksfile.lock
      Capfile
      deploy/
        tasks
          provision.rb
        cookbooks/
          main/
            attributes/
            recipes/
              default.rb
              nginx.rb
              mysql.rb
            templates/
            metadata.rb

Let's take a looks at the different ingredients one by one.

### The Application Cookbook

The `cookbooks` directory normally only contains a single cookbook.
It provides recipes to setup everything required to run the app: ruby
versions, databases, web servers. The recipes mostly contain
`include_recipe` directives and some lightweight resources. Here is an
excerpt from the mysql recipe:

    # deploy/cookbooks/main/recipes/mysql.rb
    include_recipe 'mysql::server'
    include_recipe 'mysql::client'
    include_recipe 'mysql::ruby'

    mysql_connection_info = { ... }

    mysql_database node[:application] do
      connection mysql_connection_info
      action :create
    end

    mysql_database_user node[:application] do
      connection mysql_connection_info
      password   node[:database_password]
      action     :grant
    end

Dependent cookbooks can simply be listed in the cookbook's
`metadata.rb` file:

    # deploy/cookbooks/main/metadata.rb
    name              "main"
    maintainer        "Codevise Solution"
    maintainer_email  "me@example.com"
    description       ""
    long_description  ""
    version           "0.0.0"

    depends "rvm"
    depends "mysql", "3.0.12"
    depends "database"
    depends "nginx"

    supports "ubuntu"

Sometimes other custom cookbooks shall be bundled in the application
repository. We then place them next to `main` in the `cookbooks`
directory. Most of the time though it's easier to automatically
resolve dependencies.

### Dependency Resolution

Just like bundler resolves gem dependencies,
[Berkshelf](http://berkshelf.com/) can be used to install required
cookbooks. In the `Berksfile` alternative sources to fetch cookbooks
from can be specified.

    # Berksfile
    site :opscode

    # Application cookbook
    cookbook 'main', :path => 'deploy/cookbooks/main'

    # Alternative sources
    cookbook 'rvm', :git => 'https://github.com/fnichol/chef-rvm'

Once you run `berks install`, all required cookbooks are downloaded
into a shared directory and a `Berksfile.lock` is placed in the
project root. Pretty familiar.

One could just as well use
[Librarian Chef](https://github.com/applicationsonline/librarian-chef)
to manage dependencies. But since as of recently Berkshelf appears to
be
[backed by Opscode](http://www.opscode.com/blog/2013/11/12/opscode-to-steward-berkshelf/),
I figure it is a good choice.

### Controlling Chef Solo

Finally we need a tool to actually run the recipes on the app's
deployment server. This is where the
[Capistrano Roundsman](https://github.com/iain/roundsman) gem comes
in. It bootstraps the server with a version of ruby and the chef gem,
uploads the cookbooks and invokes a chef solo run.

Note that the ruby version used to run chef needs not be the one used
by passenger to later run the app. We usually have a chef recipe
install rvm on server to gain flexibility.

The following capistrano tasks wires everything together. When we run
`cap provision`, we first tell berkshelf to unpack the required
cookbooks to a `tmp` directory. The `:cookbooks_directory` option
tells roundsman to pick up on those cookbooks. Finally the
`roundsman.run_list` command triggers the chef solo run.

By default, roundsman installs a rather outdated version of chef. This
can easily be fixed by setting the `:chef_version` option.

    # deploy/tasks/provision.rb
    require 'roundsman/capistrano'

    set :application, 'some_app'
    server 'someapp.example.com', :app
    set :user, 'ubuntu'

    set :cookbooks_directory, ['tmp/cookbooks']
    set :chef_version, '11.4.0'

    namespace :provision do
      desc 'Install cookbooks and provision server'
      task :default do
        install
        apply
      end

      desc 'Install cookbooks with berkshelf'
      task :install do
        run_local "bundle exec berks install --path #{cookbook_directory}"
      end

      desc 'Provision server'
      task :apply do
        roundsman.run_list 'recipe[main]'
      end
    end

    def run_local(command)
      system(command)
      if($?.exitstatus != 0) then
        puts 'exit code: ' + $?.exitstatus.to_s
        exit
      end
    end

Happy provisioning!
