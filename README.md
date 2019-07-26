React on Rails (RoR) in AWS Elastic Beanstalk
=============================================

Deploy Ruby on Rails (RoR) to AWS Elastic Beanstalk.



Create a Ruby environment in AWS Elastic Beanstalk
--------------------------------------------------

https://us-east-1.console.aws.amazon.com/elasticbeanstalk/home?region=us-east-1#/newEnvironment?applicationName=Elastic%20Beanstalk%20Ruby%20Sample%20App

Configure the _Create a web server environment_ page as follows:

* Platform
  * Preconfigured platform: Ruby
* Base configuration
  * Application code: Sample application

Click _Create environment_ – AWS will begin creating a Ruby environment, including a basic credentials file.

Note the version of Ruby (e.g. 2.6).



Install Rails and create a project
----------------------------------

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ruby-rails-tutorial.html


### Switch to the correct version of Ruby

An easy way to accomplish this is to install and use the Ruby Version Manager.

`$ rvm use 2.6.3`


### Install Rails

`$ gem install rails`


### Create a new Rails project

`$ rails new eb-rails`

This step is for documentation, as this repository is already comprised of a Rails project.



Add React
---------

https://github.com/reactjs/react-rails

If uploading and deploying from this repository, then skip to [Upload and Deploy to AWS Elastic Beanstalk](##Upload-and-Deploy-to-AWS-Elastic-Beanstalk).

Add *webpacker* and *react-rails* to your gemfile:

```
gem 'webpacker'
gem 'react-rails'
```

Now run the installers:

```
$ bundle install
$ rails webpacker:install       # OR (on rails version < 5.0) rake webpacker:install
$ rails webpacker:install:react # OR (on rails version < 5.0) rake webpacker:install:react
$ rails generate react:install
```

Link the JavaScript pack in Rails view using javascript_pack_tag helper:

```
<!-- application.html.erb in Head tag below turbolinks -->
<%= javascript_pack_tag 'application' %>
```



Upload and Deploy to AWS Elastic Beanstalk
------------------------------------------

Add the following .ebextensions config script to update Bundler: `gem_install_bundler.config`


```
# Updates Bundler so that Rails can be uploaded and deployed to AWS Elastic Beanstalk.
# https://github.com/shakacode/react_on_rails/blob/master/docs/additional-reading/elastic-beanstalk.md

files:
  # Runs before `./10_bundle_install.sh`:
  "/opt/elasticbeanstalk/hooks/appdeploy/pre/09_gem_install_bundler.sh" :
    mode: "000775"
    owner: root
    group: users
    content: |
      #!/usr/bin/env bash
      set -xe

      EB_APP_STAGING_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k app_staging_dir)
      EB_SCRIPT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k script_dir)
      EB_SUPPORT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k support_dir)

      . $EB_SUPPORT_DIR/envvars
      # Source the application's Ruby, i.e., 2.6; otherwise, the version will be 2.3, which will give this error: `bundler requires Ruby version >= 2.3.0`
      . $EB_SCRIPT_DIR/use-app-ruby.sh

      cd $EB_APP_STAGING_DIR
      echo "Installing compatible bundler"
      gem install bundler -v 2.0.2
```

Add the following .ebextensions config script to update Node.js and install Yarn for deployment of React: `react-rails.config`

```
# Updates Node and installs Yarn so that React-Rails can be uploaded and deployed to AWS Elastic Beanstalk.
# https://github.com/shakacode/react_on_rails/blob/master/docs/additional-reading/elastic-beanstalk.md

files:
  # Runs before `./10_bundle_install.sh`:
  "/opt/elasticbeanstalk/hooks/appdeploy/pre/09_react-rails.sh" :
    mode: "000775"
    owner: root
    group: users
    content: |
      #!/usr/bin/env bash
      set -xe

      # Install Node.js
      echo "Installing Node.js"
      curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash -
      yum -y install nodejs

      # Install Yarn
      echo "Installing Yarn"
      wget https://dl.yarnpkg.com/rpm/yarn.repo -O /etc/yum.repos.d/yarn.repo;
      yum -y install yarn;

      # Yarn install
      yarn
```

Add the following .ebextensions config script to configure NGINX to serve React /packs: `nginx.config`

```
# Modifies webapp_healthd.conf to allow React-Rails requests for /packs.

files:
  # Runs before `./10_bundle_install.sh`:
  "/opt/elasticbeanstalk/hooks/appdeploy/pre/09_nginx_modify.sh" :
    mode: "000775"
    owner: root
    group: users
    content: |
      #!/usr/bin/env bash
      set -xe

      if ! grep -q "location[[:space:]]*/packs" /etc/nginx/conf.d/webapp_healthd.conf; then
        sed -i 's|\([[:space:]]*\)\(location[[:space:]]*/assets\)|\1location /packs { alias /var/app/current/public/packs; gzip_static on; gzip on; expires max; add_header Cache-Control public; }\n\n\1\2|' /etc/nginx/conf.d/webapp_healthd.conf
      fi

container_commands:
  nginx_reload:
    command: "sudo service nginx reload"
```

This repository already includes `gem_install_bundler.config`, `react-rails.config`, and `nginx.config`.


### Include credentials

Include a _master key_ via `config/master.key` or by configuring it in your AWS Elastic Beanstalk Ruby environment property RAILS_MASTER_KEY. If including the master key via `config/master.key`, don't commit the file; instead, put the master key in a password manager. By default, both Rails and this repository _.gitignore_ `config/master.key`.


### Create a source bundle

Create a source bundle containing the files created by Rails. The following command creates a source bundle named `rails-default.zip`.

```
$ cd eb-rails
$ zip ../rails-default.zip -r * .[^.]* -x@.exclude.lst
```


### Upload and Deploy

Upload and deploy the source bundle to the AWS Elastic Beanstalk Ruby environment.



Troubleshooting
---------------

#### Error:

```
Command failed on instance. Return code: 1 Output: (TRUNCATED)...tring with `rails credentials:edit` /var/app/ondeck/config/environment.rb:5:in `<main>' /opt/rubies/ruby-2.6.3/bin/bundle:23:in `load' /opt/rubies/ruby-2.6.3/bin/bundle:23:in `<main>' Tasks: TOP => environment (See full trace by running task with --trace). Hook /opt/elasticbeanstalk/hooks/appdeploy/pre/11_asset_compilation.sh failed. For more detail, check /var/log/eb-activity.log using console or EB CLI.
```

#### Solution:

Include a _master key_. See [Include credentials](###Include-credentials).

If a master key has never been created for this project, then create the credentials by running the following command:

`$ rails credentials:edit`

On successful execution, the command will create two files:

* config/credentials.yml.enc
* config/master.key

Remember, for security reasons, do not commit _master.key_ to the repo.
