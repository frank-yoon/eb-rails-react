React on Rails (RoR) in AWS Elastic Beanstalk, with Material-UI
===============================================================



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



Generate a new React-Rails website
----------------------------------


### Generate a Rails controller, route, and view for your welcome page:

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ruby-rails-tutorial.html#ruby-rails-tutorial-generate

`$ rails generate controller WelcomePage welcome`

Add the following route to _config/routes.rb_:

```
Rails.application.routes.draw do
  get 'welcome_page/welcome'
  root 'welcome_page#welcome'
```


### Generate a new React component:

https://github.com/reactjs/react-rails

`$ rails g react:component HelloWorld greeting:string`

Render it in a Rails view, such as _app/views/welcome_page/welcome.html.erb_:

```
<!-- erb: paste this in view -->
<%= react_component("HelloWorld", { greeting: "Hello from react-rails." }) %>
```



Install the React UI framework, Material-UI
-------------------------------------------

https://material-ui.com/getting-started/installation

The following steps are for documentation, as this repository is already comprised of a Material-UI installation.


### Install Material-UI

`$ npm install @material-ui/core`


### Install the prebuilt SVG Material icons

`$ npm install @material-ui/icons`


### Roboto Font

Add the following import (link markup) to _app/assets/stylesheets/welcome_page.scss_:

`@import url(https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap);`


### Font Icons

Add the following import (link markup) to _app/assets/stylesheets/welcome_page.scss_:

`@import url(https://fonts.googleapis.com/icon?family=Material+Icons);`



Install the Material-UI theme, Onepirate
----------------------------------------

https://github.com/mui-org/material-ui/tree/master/docs/src/pages/premium-themes/onepirate

The following steps are for documentation, as this repository is already comprised of a Onepirate implementation.


### Install Onepirate

* Replace the contents of _eb-rails-react⁩/app/javascript⁩/components/_ with that of _material-ui/docs/src/pages/premium-themes/onepirate/_.

* Copy the _material-ui⁩/docs/⁨static/⁨themes/onepirate_⁩ directory to _eb-rails-react⁩/public/themes/_.
