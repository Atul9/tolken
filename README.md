# Tolken
Straightforward Rails database translations using psql jsonb

Tolken's API is more verbose than most similar gems. The idea is that you should be aware of when you're dealing with translatable fields and what language you're interested in in any given moment. In tolken a translatable field is just a Ruby hash which makes it easy to reason about. See *Usage* for details.

[![Gem Version](https://badge.fury.io/rb/tolken.svg)](https://badge.fury.io/rb/tolken)
[![Build Status](https://travis-ci.org/varvet/tolken.svg?branch=master)](https://travis-ci.org/varvet/tolken)
[![Maintainability](https://api.codeclimate.com/v1/badges/72c772179a8baa586f7f/maintainability)](https://codeclimate.com/github/varvet/tolken/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/72c772179a8baa586f7f/test_coverage)](https://codeclimate.com/github/varvet/tolken/test_coverage)
[![semver stability](https://api.dependabot.com/badges/compatibility_score?dependency-name=tolken&package-manager=bundler&version-scheme=semver)](https://dependabot.com/compatibility-score.html?dependency-name=tolken&package-manager=bundler&version-scheme=semver)

## Installation
Add this line to your application's Gemfile:

```ruby
gem "tolken"
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install tolken

## Usage
Make sure you're running Postgres and that the column you want to translate is of the `jsonb` type:

### Setup
```rb
class CreatePosts < ActiveRecord::Migration[5.2]
  def change
    create_table :posts do |t|
      t.jsonb :title, :jsonb, null: false
      t.timestamps
    end

    execute "CREATE INDEX posts_title_index ON posts USING gin (title)"
  end
end
```

Next you need to configure `I18n` if you haven't already (usually in *config/initializers/locale.rb*):

```rb
I18n.available_locales = %i[en sv de]
```

Tolken expects you to only use translations for the locales in `I18n.available_locales`.

### Persistence
Extend your model with `Tolken` and tell it what column(s) you want to translate:

```rb
class Post < ApplicationRecord
  extend Tolken
  translates :title
end
```

You can now work with `title` as follows:

```rb
post = Post.create(title: { en: "News", sv: "Nyheter" })

post.title # => { en: "News", sv: "Nyheter" }
post.title(:en) # => "News"
post.title(:sv) # => "Nyheter"
post.title("sv") # => "Nyheter"
post.title[:en] # => "News"
post.title["en"] # => "News"
post.title(:dk) # ArgumentError, "Invalid locale dk"
post.title[:dk] # => nil
post.title(:de) # => nil

post.title = { en: "News", sv: "Nyheter" }
post.title[:en] = "News"
```

### Validation
Tolken comes with support for the *presence* validator:

```rb
class Post < ApplicationRecord
  extend Tolken
  translates :title, presence: true
end

post = Post.create(title: { en: "News", sv: "" })
post.errors.messages # => { name_sv: ["can't be blank"] }
```

Tolken checks that all `I18n.available_locales` has present values.

### View Forms
While creating form inputs for your locale versions are as simple as adding `[<locale>]` to the html name attribute Tolken provides opt-in support for integrating with [SimpleForm](https://github.com/plataformatec/simple_form). To opt-in update your Gemfile:

```ruby
gem "tolken", require: "tolken/simple_form"
```

Now if you add a simple_form field for a translatable field SimpleForm will generate an input per language version:

```erb
<%= simple_form_for(@post) do |form| %>
  <%= form.input :title %>
  <%= form.submit %>
<% end %>
```

By default a text input field is generated. If you want another type you can override with:

```erb
<%= form.input :title, type: :text %>
```

Without simple form your inputs might look something like:

```html
<input name="post[title][en]" id="post_title_en">
<input name="post[title][sv]" id="post_title_sv">
```

Don't forget whitelist the nested locale params in your controller:

```rb
params.require(:post).permit(title: I18n.available_locales)
```

This will instead render one text area per language version.

The specs for [translates](spec/tolken/translates_spec.rb) is a good resource of additional usage examples.

## Development

### Native dependencies
You need to have postgresql installed on your system. This can be done on Mac OS with `brew install postgresql`.

### Ruby setup
After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing
Bug reports and pull requests are welcome on GitHub at https://github.com/varvet/tolken. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

If you use and like Tolken but find yourself adding features that you're missing in Tolken please consider submitting a pull request!

To create a pull request follow these steps:

    $ git clone git@github.com:varvet/tolken.git
    $ cd tolken
    $ git checkout my-feature-branch
    Make changes and commits, consider squashing commits that doesn't add anything by themselves
    $ rubocop
    $ rspec
    $ open coverage/index.html
    If test coverage is missing address it or add a description to your PR why it is lacking.
    Commit, if possible squash in original locations, changes required by rubocop and or failing tests.
    $ git checkout master
    $ git pull --rebase
    $ git checkout my-feature-branch
    $ git rebase master
    $ git push

## Alternative solutions

* [Traco](https://github.com/barsoom/traco) - use multiple columns in the same model (Barsoom)
* [Mobility](https://github.com/shioyama/mobility) - pluggable translation framework supporting many strategies, including translatable columns, translation tables and hstore/jsonb (Chris Salzberg)
* [hstore_translate](https://github.com/cfabianski/hstore_translate) - use PostgreSQL's hstore datatype to store translations, instead of separate translation tables (Cédric Fabianski)
* [json_translate](https://github.com/cfabianski/json_translate) - use PostgreSQL's json/jsonb datatype to store translations, instead of separate translation tables (Cédric Fabianski)
* [Trasto](https://github.com/yabawock/trasto) - store translations directly in the model in a Postgres Hstore column

## License
The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
