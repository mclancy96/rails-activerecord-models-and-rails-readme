# Active Record Models and Rails

## Active Record's Role

Active Record is the built-in ORM that Rails utilizes to manage the model
aspects of an application. What is an ORM? An ORM is an Object Relational
Mapping system, essentially this is the module that enables your application to
manage data in a method driven structure. This means that you are able to run
queries, add records, and perform all of the traditional database processes by
leveraging methods as opposed to writing SQL manually. For example, below is the
traditional way that we would query a database of 'posts' using SQL:

```sql
SELECT * FROM posts
```

Compared with leveraging Active Record:

```ruby
Post.all
```

By using Active Record, you are also able to perform advanced query tasks, such
as method chaining and scoping, which typically require less code and make for a
more readable query.

## Active Record Models

So if we have a database table, why do we need a model file? By using model
files, we are able to create an organized layer of abstraction for our data. An
important thing to remember is that at the end of the day the model file is a Ruby class. Your models will inherit from the `ApplicationRecord` class. This may seem a bit unintuitive at first, but `ApplicationRecord` itself inherits from `ActiveRecord::Base`, which is the core class that provides all of Active Record's powerful features. This means that by inheriting from `ApplicationRecord`, your models automatically gain access to all the database-related methods and behaviors provided by Active Record, such as querying, validations, and associations.

The use of `ApplicationRecord` as an intermediate class is a Rails convention that allows you to add shared logic for all your models in one place if needed, while still leveraging everything Active Record offers. However, you can still treat your model like a regular Ruby class, allowing you to create methods, data attributes, and everything else that you would want to do in a class file.

A typical model file will contain code such as but not limited to the following:

- [Custom scopes][scopes]
- Model instance methods
- Default settings for database columns
- [Validations][validations]
- [Model-to-model relationships][relationships]
- [Callbacks][callbacks]
- Custom algorithms

_If any/all of the items above aren't familiar to you yet, don't worry. We'll
cover them in future lessons. It's important to have an idea of what can be
included in a model file, even at this early stage._

## Creating an Active Record Model

As a professional Rails developer, you will be expected to build applications by
leveraging a [BDD][bdd] process, so we will walk through how to
build each feature with a test-first approach so that the tests can lead our
development. However, please focus on the implementation code so that you can
get a firm understanding of how to build a model, database table, etc.

In order to get started, we will first create an RSpec test. We've provided a
basic skeleton of a Rails application using RSpec in this repo.

To generate this app, we installed the Rails gem, then ran the following
commands. **Note**: you don't have to run these commands yourself; they're just
here to demonstrate how this app was created.

```bash
# the -T flag tells the Rails project generator not to
# include TestUnit, the default testing framework:
bin/rails new rails-activerecord-models-and-rails-readme -T

# The Rails project generator created this directory for us:
cd rails-activerecord-models-and-rails-readme

# We modified the Gemfile to include
# gem 'rspec-rails', '~> 6.0'
# in the :development, :test group, then ran:

bundle install

# Finally, we created the initial RSpec config:
bin/rails g rspec:install
```

Let's create a new file: `spec/models/post_spec.rb`. In that file, place the following code:

```ruby
require 'rails_helper'

describe Post do

end
```

If we run `bin/rspec`, it will throw an error since we don't have any
code in the application for our `Post` model yet. To fix this, create a new file
in the `app/models` directory called `post.rb`, and add the following code:

```ruby
class Post
end
```

This will get the tests passing, but it still has some weird errors because we
need to create a schema file. You can do that by running `bin/rails db:migrate`.
(There is no need to create a database with `bin/rails db:create` first. The test
suite will create a test database for us when we run our tests.) This will
create the schema file and clear the warning. Now update the `Post` spec to test
for a `Post` being created. It should look something like this:

```ruby
describe Post do
  it 'can be created' do
    post = Post.create!(title: "My title", description: "The post description")
    expect(post).to be_valid
  end
end
```

Running this test gives us the error of: `undefined method 'create!' for Post:Class`. To implement this feature, let's create the database table for our
posts. Create a new directory in the `db/` directory called `migrate`, and add a
new file called `001_create_posts.rb`. To that file, add the following code:

```ruby
class CreatePosts < ActiveRecord::Migration[7.1]
  def change
    create_table :posts do |t|
      t.string :title
      t.text :description

      t.timestamps null: false
    end
  end
end
```

This is a basic migration that will create a `posts` table that will have title
and description columns, along with the built in timestamps. For a refresher on
migrations, see [this documentation][migrations]. This migration follows the
standard naming convention. When you want to create a table, the migration's
class name should reflect that; hence, `CreatePosts`. This is then reiterated by
the `:posts` argument passed to the `create_table` method. The filename itself
needs to be unique, and when you generate a migration automatically through a
model or scaffold generator you will notice that the migration file name is
prepended with a timestamp value to make sure that we can run migrations in the
order they were written.

The timestamp also plays a role in making sure that only new migrations run when
we run `bin/rails db:migrate`. The `db/schema.rb` file is updated with a version number
corresponding to the timestamp of the last migration you ran. When you run
`bin/rails db:migrate` again, only migrations whose timestamps are greater than the
schema's version number will run. So, the numbers at the beginning of the filenames
of your migrations are required so ActiveRecord can be sure to run each of your
migrations just once and in the proper order.

After running `bin/rails db:migrate` we can see that our `db/schema.rb` file has been
updated with our new posts table. However, if we run our tests again we will
still see them failing due to the same error: `` undefined method `create!' for Post:Class ``. This is because we left out one very important piece of code from

the `Post` model. In order to leverage built-in methods such as `.create!`, we
need to have the Post class inherit from `ApplicationRecord`. Update the
`post.rb` model file to match the following:

```ruby
  class Post < ApplicationRecord
  end
```

Now all of the tests are passing and we can create a new post correctly. Even
though we know this is working because our tests are passing, let's still test
this in the console. Open up the Rails console by running `bin/rails console`.
Running the console will load the entire Rails environment and give you command
line access to the app and the database. The console is a powerful tool that you
can leverage in order to test out scripts, methods, and database queries.

Once the session has started, run the following command to ensure it recognizes
our new Post model:

```ruby
Post.all
```

If everything is set up properly, you will see that it returns an empty Active
Record object. Let's test creating a record using the console:

```ruby
Post.create!(title: "My title", description: "The post description")
```

Now run the query:

```ruby
Post.last
```

It returned our newly-created post!

With our `Post` model working, let's add a new feature that returns a summary of
a post. As usual, start off by creating a spec for the feature:

```ruby
it 'has a summary' do
  post = Post.create!(title: "My title", description: "The post description")
  expect(post.post_summary).to eq("My title - The post description")
end
```

If we run this, we'll get a failure since we do not have a `post_summary` method
for `Post`. Add that to the model file:

```ruby
def post_summary
end
```

This now results in a failure since the method currently doesn't return anything. Update the `post_summary` method as follows:

```ruby
def post_summary
  self.title + " - " + self.description
end
```

Now if you run the tests, all of them are passing and our `Post` model has an
instance method that returns a post summary. You can test this out in the Rails
console as well by running a query on the record we created, such as:

```ruby
Post.last.post_summary
```

It should return the summary value of the last post we created: `"My title - The post description"`.

As you may have noticed, we did not have to create a controller, route, view,
etc. in order to get the `Post` model working. The data aspect of the
application can work separately from the view and data flow logic. This level of
abstraction makes it efficient to test data behavior without having it strongly
coupled to how it is rendered to the user. With that being said, it is
considered a best practice to have your controller and view files follow the
proper naming convention so that the MVC associations are readable. For example,
to build out the controller and view code for our `Post` model, we would create
the following structure:

- Create a `posts_controller.rb` file that calls on the `Post` model
- Create a `views/posts/` directory that stores the views related to the `Post` model

Also, if you are coming from other programming languages you may be wondering
how exactly we are able to connect to the database automatically without having
to create connection strings. The reason for this simplicity resides in the
`config/database.yml` file that was generated when we created our application
and ran `bin/rails db:create`. In that file, you will see that the development, test,
and production databases are all configured. From that stage, the
`ActiveRecord::Base.connection` method connects your application to the
database, which is another benefit of having our model classes inherit from the
`ApplicationRecord` class.

Being able to work in different environments is one of the strong points of
Rails, and the database.yml file takes advantage of this feature by having
different database options for each type of environment. If you take a look at
the file, you can see that you can establish different database adapters, pools,
timeout values, etc. for each environment specifically. This allows for you to
have a setup such as using SQLite locally and Postgres in production, along with
having a segmented database environment for your testing suite. Some of these
items are components that you won't need until you get into more advanced
applications. However, it's good to know where these items are located in the
file system for when you get to that point. Essentially, this file includes a
lot of stuff you will rarely have to handle, but just remember that if anything
requires database configuration it will be here.

## Summary

We covered quite a bit of material in this lesson. You should now have a firm
understanding of Active Record models in Rails. Active Record is a powerful tool
that enables developers to focus on the logic of their applications while
streamlining processes such as connecting to the database, running queries, and
much more.

[scopes]: http://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html
[validations]: http://api.rubyonrails.org/classes/ActiveModel/Validations/ClassMethods.html
[relationships]: http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html
[callbacks]: http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html
[bdd]: http://rspec.info/
[migrations]: http://edgeguides.rubyonrails.org/active_record_migrations.html
