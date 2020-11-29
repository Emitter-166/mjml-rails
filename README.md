# MJML-Rails

[![Build Status](https://api.travis-ci.org/sighmon/mjml-rails.svg?branch=master)](http://travis-ci.org/sighmon/mjml-rails) [![Gem Version](https://badge.fury.io/rb/mjml-rails.svg)](https://badge.fury.io/rb/mjml-rails)

**MJML-Rails** allows you to render HTML e-mails from an [MJML](https://mjml.io) template.

**Note**: As of MJML 4.3.0 you can no longer use `<mj-text>` directly inside an `<mj-body>`, wrap it in `<mj-section><mj-column>`.

An example template might look like:

```erb
<!-- ./app/views/user_mailer/email.html.mjml -->
<mjml>
  <mj-head>
    <mj-preview>Hello World</mj-preview>
  </mj-head>
  <mj-body>
    <mj-section>
      <mj-column>
        <mj-text>Hello World</mj-text>
        <%= render partial: "info" %>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>
```

And the partial:

```erb
<!-- ./app/views/user_mailer/_info.html.erb -->
<mj-text>This is <%= @user.username %></mj-text>
```

* Notice you can use ERB and partials inside the template.

Your `user_mailer.rb` might look like this:

```ruby
# ./app/mailers/user_mailer.rb
class UserMailer < ActionMailer::Base
  def user_signup_confirmation
    mail(to: "user@example.com", from: "app@example.com") do |format|
      format.text
      format.mjml
    end
  end
end
```

## Example application

* [MJML with Rails 6](https://github.com/dyanagi/example_mjml_rails): Renders HTML emails with MJML layout, view, and partial.

## Installation

Add it to your Gemfile.

```ruby
gem 'mjml-rails'
```

Run the following command to install it:

```console
bundle install
```

Add the MJML parser to your project with your favourite package manager:

```console
# with npm
npm install mjml

# with yarn
yarn add mjml
```

MJML-Rails falls back to a global installation of MJML but it is strongly recommended to add MJML directly to your project.

You'll need at least Node.js version 6 for MJML to function properly.

If you're using ```:haml``` or any other Rails template language, create an initializer to set it up:

```ruby
# config/initializers/mjml.rb
Mjml.setup do |config|
  config.template_language = :erb # :erb (default), :slim, :haml, or any other you are using
end
```

**Note:** If you’re using Haml/Slim layouts, please don’t put `<mjml>` in comments in your partial. Read more: [#34](https://github.com/sighmon/mjml-rails/issues/34).

If there are configurations you'd like change:
- render errors: defaults to `true` (errors raised)
- minify: defaults to `false` (not minified)
- beautify: defaults to `true` (beautified)
- validation_level: defaults to `strict` (abort on any template error, see [MJML validation documentation](https://github.com/mjmlio/mjml/tree/master/packages/mjml-validator#validating-mjml) for possible values)

```ruby
# config/initializers/mjml.rb
Mjml.setup do |config|
  # ignore errors silently
  config.raise_render_exception = false

  # optimize the size of your email
  config.beautify = false
  config.minify = true

  # render illformed MJML templates, not recommended
  config.validation_level = "soft"
end
```

If you’d like to specify a different MJML binary to run other than `4.`:

```ruby
# config/initializers/mjml.rb
Mjml.setup do |config|
  config.mjml_binary_version_supported = "3.3.5"
end
# If you set a different MJML binary version, you need to re-discover the binary
Mjml::BIN = Mjml.discover_mjml_bin
```

**Note:** If you set a different MJML binary you’ll see warnings in your logs of `already initialized constant Mjml::BIN`. Read more: [#39](https://github.com/sighmon/mjml-rails/issues/39#issuecomment-429151908)

### MJML v3.x & v4.x support

Version 4.x of this gem brings support for MJML 4.x

Version 2.3.x and 2.4.x of this gem brings support for MJML 3.x

If you'd rather still stick with MJML 2.x then lock the mjml-rails gem:

```ruby
gem 'mjml-rails', '2.2.0'
```

For MJML 3.x lock the mjml-rails gem:

```ruby
gem 'mjml-rails', '2.4.3'
```

And then to install MJML 3.x

```console
npm install -g mjml@3.3.5
```

### How to guides

[Hugo Giraudel](https://twitter.com/hugogiraudel) wrote a post on [using MJML in Rails](http://dev.edenspiekermann.com/2016/06/02/using-mjml-in-rails/).

## Using Email Layouts

*Note*: [Aleksandrs Ļedovskis](https://github.com/aleksandrs-ledovskis) kindly updated the gem for better Rails Email Layouts support - it should be a non-breaking change, but check the updated file naming below if you experience problems.

Mailer:
```ruby
# mailers/my_mailer.rb
class MyMailer < ActionMailer::Base
  layout "default"

  def foo_bar(user)
    @recipient = user

    mail(to: user.email, from: "app@example.com") do |format|
      format.html # This will look for "default.html.erb" and then "default.html.mjml"
    end
  end
end
```

Note: If `default.html.erb` exists, email will be rendered as ERB, and MJML tags will not be compiled.

Email layout:
```html
<!-- views/layouts/default.html.mjml -->
<mjml>
  <mj-body>
    <%= yield %>
  </mj-body>
</mjml>
```

Email view:
```html
<!-- views/my_mailer/foo_bar.html.mjml (or foo_bar.html.erb) -->
<%= render partial: "to" %>

<mj-section>
  <mj-column>
    <mj-text>
      Something foo regarding bar!
    </mj-text>
  </mj-column>
</mj-section>
```

Email partial:
```html
<!-- views/my_mailer/_to.html.mjml (or _to.html.erb) -->
<mj-section>
  <mj-column>
    <mj-text>
      Hello <%= @recipient.name %>,
    </mj-text>
  </mj-column>
</mj-section>
```

## Sending Devise user emails

If you use [Devise](https://github.com/plataformatec/devise) for user authentication and want to send user emails with MJML templates, here's how to override the [devise mailer](https://github.com/plataformatec/devise/blob/master/app/mailers/devise/mailer.rb):
```ruby
# app/mailers/devise_mailer.rb
class DeviseMailer < Devise::Mailer
  def reset_password_instructions(record, token, opts={})
    @token = token
    @resource = record
    # Custom logic to send the email with MJML
    mail(
      template_path: 'devise/mailer',
      from: "some@email.com",
      to: record.email,
      subject: "Custom subject"
    ) do |format|
      format.text
      format.mjml
    end
  end
end
```

Now tell devise to user your mailer in `config/initializers/devise.rb` by setting `config.mailer = 'DeviseMailer'` or whatever name you called yours.

And then your MJML template goes here: `app/views/devise/mailer/reset_password_instructions.mjml`

Devise also have [more instructions](https://github.com/plataformatec/devise/wiki/How-To:-Use-custom-mailer) if you need them.

## Deploying with Heroku

To deploy with [Heroku](https://heroku.com) you'll need to setup [multiple buildpacks](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app) so that Heroku first builds Node for MJML and then the Ruby environment for your app.

Once you've installed the [Heroku Toolbelt](https://toolbelt.heroku.com/) you can setup the buildpacks from the commandline:

`$ heroku buildpacks:set heroku/ruby`

And then add the Node buildpack to index 1 so it's run first:

`$ heroku buildpacks:add --index 1 heroku/nodejs`

Check that's all setup by running:

`$ heroku buildpacks`

Next you'll need to setup a `package.json` file in the root, something like this:

```json
{
  "name": "your-site",
  "version": "1.0.0",
  "description": "Now with MJML email templates!",
  "main": "index.js",
  "directories": {
    "doc": "doc",
    "test": "test"
  },
  "dependencies": {
    "mjml": "^4.0.0"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/your-repo/your-site.git"
  },
  "keywords": [
    "mailer"
  ],
  "author": "Your Name",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/sighmon/mjml-rails/issues"
  },
  "homepage": "https://github.com/sighmon/mjml-rails"
}
```

Then `$ git push heroku master` and it should Just WorkTM.

## Bug reports

If you discover any bugs, feel free to create an issue on GitHub. Please add as much information as possible to help us fixing the possible bug. We also encourage you to help even more by forking and sending us a pull request.

[github.com/sighmon/mjml-rails/issues](https://github.com/sighmon/mjml-rails/issues)

## Maintainers

* Simon Loffler [github.com/sighmon](https://github.com/sighmon)
* Steven Pickles [github.com/thatpixguy](https://github.com/thatpixguy)
* [The Rails community](https://github.com/sighmon/mjml-rails/pulls?q=is%3Apr+is%3Aclosed). :-)

## Other similar gems

* [srghma/mjml-premailer](https://github.com/srghma/mjml-premailer)
* [kolybasov/mjml-ruby/](https://github.com/kolybasov/mjml-ruby/)

## License

MIT License. Copyright 2018 Simon Loffler. [sighmon.com](http://sighmon.com)

Lovingly built on [github.com/plataformatec/markerb](https://github.com/plataformatec/markerb)
