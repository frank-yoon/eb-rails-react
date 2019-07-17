React on Rails (RoR) in AWS Elastic Beanstalk
=============================================



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



Install Grommet and create an app
---------------------------------

https://github.com/grommet/grommet-starter-new-app


### Install Grommet

`$ npm install grommet grommet-icons styled-components --save`


### Create a Grommet app

Update _app/assets/stylesheets/welcome_page.scss_ to include the Roboto font-face and fix some browser defaults:

```
@import url(https://fonts.googleapis.com/css?family=Roboto:300,400,500);
```
Update _app/javascript/components/HelloWorld.js_ with the following:

```
import React from 'react';
import {
  Box,
  Button,
  Collapsible,
  Heading,
  Grommet,
  Layer,
  ResponsiveContext,
} from 'grommet';
import { FormClose, Notification } from 'grommet-icons';

const theme = {
  global: {
    font: {
      family: 'Roboto',
      size: '14px',
      height: '20px',
    },
  },
};

const AppBar = (props) => (
  <Box
    tag='header'
    direction='row'
    align='center'
    justify='between'
    background='brand'
    pad={{ left: 'medium', right: 'small', vertical: 'small' }}
    elevation='medium'
    style={{ zIndex: '1' }}
    {...props}
  />
);

class App extends React.Component {
  state = { showSidebar: false, }
  render () {
    const { showSidebar } = this.state;
    return (
      <Grommet theme={ theme } full >
        <ResponsiveContext.Consumer>
          {
            size => (
              <Box fill>
              <AppBar>
                <Heading level='3' margin='none'>My App</Heading>
                <Button
                  icon={<Notification />}
                  onClick={() => this.setState(prevState => ({ showSidebar: !prevState.showSidebar }))}
                />
              </AppBar>
              <Box direction='row' flex overflow={{ horizontal: 'hidden' }}>
                <Box flex align='center' justify='center'>
                  app body
                </Box>
                {(!showSidebar || size !== 'small') ? (
                  <Collapsible direction="horizontal" open={showSidebar}>
                    <Box
                      flex
                      width='medium'
                      background='light-2'
                      elevation='small'
                      align='center'
                      justify='center'
                    >
                      sidebar
                    </Box>
                  </Collapsible>
                ): (
                  <Layer>
                    <Box
                      background='light-2'
                      tag='header'
                      justify='end'
                      align='center'
                      direction='row'
                    >
                      <Button
                        icon={<FormClose />}
                        onClick={() => this.setState({ showSidebar: false })}
                      />
                    </Box>
                    <Box
                      fill
                      background='light-2'
                      align='center'
                      justify='center'
                    >
                      sidebar
                    </Box>
                  </Layer>
                )}
              </Box>
            </Box>
          )}
        </ResponsiveContext.Consumer>
      </Grommet>
    );
  }
}

export default App;
```
