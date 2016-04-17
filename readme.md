# Rails Index/Show Features

## Learning Objectives

## Framing && Revisit MVC pattern

![rMVC](http://i.stack.imgur.com/Sf2OQ.png)

Over the next few lessons we'll be learning about the "Rails way" to do things. We're going to implement features that you are familiar with from Sinatra(index, create etc..) in Rails. We'll be going through a significant amount of conventions throughout the lesson and they're all really important! There's also an [appendix]() for this at the bottom.

The general format for today will be similar to the sinatra lesson. I do- An app called Reminder.ly, you do- TUNR. At this point, you might be asking your self why are we doing this.. again??

- You are already familiar with this domain model
- Many web apps or features of web apps are based around this simple domain model.

> this will be the first todo app we'll be seeing, but it's the quintessential app for learning a new framework because its the perfect contrived example of a single model CRUD application.

If at any point in your code, you have the words `todos` or `reminderly` in your application, you're probably doing something incorrectly.

## SETUP: I do - Reminder.ly

The first thing that we should do is determine our domain model.

### ERD
|TODOS|
|---|---|
|body|text|
|author|text|

Starting a new rails app:

```bash
$ rails new reminderly -d postgresql
$ cd reminderly
```

> This command will create a new rails app. The `-d postgresql` tells rails to create a new app using postgresql instead of sqlite3 by default.

In order to talk about `todos`, we need to define them in the context of our rails app. We do this like we did in Sinatra, by creating model files. We can create these files using generators.

### Generators - An Aside

In rails, there are terminal commands that generate massive amounts of code for you. If you find yourself using these commands, you should know what each file being created does. Then use them at your leisure. That said we're going to learn some basic ones today.

### Models

Run this terminal command in your rails directory to create models:

$ rails generate model todo

> We're telling the terminal to run a rails command that generates a model called todo. Notice the convention for model name being singular and snake case.

You can see that it created some files. Don't worry about those testing files for now, they won't influence our app today. The two files that are important:

in `app/models/todo.rb`:

```ruby
class Todo < ActiveRecord::Base
end
```

in `<sometimestamp>_create_todos.rb`:

```ruby
class CreateTodos < ActiveRecord::Migration
  def change
    create_table :todos do |t|

      t.timestamps null: false
    end
  end
end
```

### Migrations

Migrations are a convenient way to alter your database schema over time in a consistent and easy way. They use a Ruby DSL so that you don't have to write SQL by hand, allowing your schema and changes to be database independent.

You can think of each migration as being a new 'version' of the database. A schema starts off with nothing in it, and each migration modifies it to add or remove tables, columns, or entries. Active Record knows how to update your schema along this timeline, bringing it from whatever point it is in the history to the latest version. Active Record will also update your db/schema.rb file to match the up-to-date structure of your database.

No more writing `schema.sql` files! or running psql commands to load databases!

### Update Migration
We need to update the migration to reflect our domain model. In `db/migrate/<timestamp>_create_todos.rb`:

```ruby
class CreateTodos < ActiveRecord::Migration
  def change
    create_table :todos do |t|
      t.string :body
      t.string :author
      t.timestamps null: false
    end
  end
end

```

### Rails Console.
If you ever need a sand box in your web app's developement environment:

```bash
rails c
```

This will allow you to enter a REPL with all your model definitions and connections to the database. Very powerful tool for testing out different things you're curious about for your application.

### `seeds.rb`



## SETUP: You do - Tunr - Model & Migration

Create models and migrations for Tunr. Don't forget your associations in your model definitions! You did this correctly if you can create an `artist` and a `song` in the rails console.

## SETUP: You do - Tunr - Seed

Visit [tunr_repo](https://github.com/ga-wdi-exercises/tunr_rails/tree/solution/db)

At this site you'll see 3 files: `artist_data.rb`, `song_data.rb` and `seeds.rb`. Make sure you have the same 3 files in your `db` directory. Then run the following command in the terminal:

```bash
$ rake db:seed
```

You know you this did this right if you run the rails console and enter `Artist.all.length` and get back `5`

## EDD revisted: I do - Reminder.ly

The index feature is generally one that wants to display a collection of items. Before we define the feature, we need to first code a path to listen for. In `config/routes.rb`:

```ruby
Rails.application.routes.draw do
  get 'todos' => 'todos#index'
end
```

If we start our server and go to the path `http://localhost:3000/todos`, we'll immediately see an error.  

![Uninitialized Constant](images/uninit_controller.png)

Let's fix that error:

```bash
$ touch app/controllers/todos_controller.rb
```

In `app/controllers/todos_controller.rb`:

```ruby
class TodosController < ApplicationController
end
```

A new error?!

![Unknown Action](images/unknown_action.png)

Let's fix that by creating the action it could not find. In `app/controllers/todos_controller.rb`:

```ruby
class TodosController < ApplicationController
  def index
  end
end
```

Yet another error ...

![Template Missing](images/template_missing.png)

Lets disect this error message.

- `Missing template todos/index`
- `Searched in: * "/Users/andrewkim/wdi/temp/reminderly/app/views"`

We need to create a folder called `todos`, and a file called `index` in that folder. Let's do that now:

```bash
$ mkdir app/views/todos
$ touch app/views/todos/index.html.erb
```

In `app/views/todos/index.html.erb` we'll simply put the word `hello world`. Finally we can see our page rendered.

## EDD re revisited: You do - Tunr
- Get "hello world" to show up in the `index` view for the `tunr` application.

## The Index Feature: I do - Reminder.ly

hello world is nice, but really we want to see all of our todos. The first thing we need to do is query for todos in our controller. In `app/controllers/todos_controller.rb`:

```ruby
class TodosController < ApplicationController
  def index
    @todos = Todo.all
  end
end
```

When our router(`config/routes.rb`) is configured this way:

```ruby
get 'todos' => 'todos#index'
```

It recognizes requests at `todos` and points those requests to execute the index method(action) in the `TodosController`. Once there, it will query for all the `todo`'s in our database. If nothing is explicitly rendered, it will implicitly render a corresponding index view. That view will be located in a folder with the same name as the controller.

Now all that's left is to update the view. In `app/views/todos/index.html.erb`:

```html
<ul>
  <% @todos.each do |todo| %>
    <li>
      <p><%= todo.body %></p>
    </li>
  <% end %>
</ul>
```

## The Index Feature: You do - Tunr

Create the index route for artists.

## The Show Feature: I do - Reminder.ly

Similarly to the `index` feature, we need to set up our routes so that the app can listen for the `show` feature. In `config/routes.rb`:

```ruby
get 'todos' => 'todos#index'
get 'todos/:id' => 'todos#show'
```

We're going to have to write this at least 5 more times for this model looks like. That seems not dry. Turns out, rails has a short hand way of writing this:

```ruby
resources :todos, only: [:index, :show]
```

Because rails developers were consistently writing out routes for the 7 RESTful routes, they decided to make a short cut for it, `resources`. The argument for `resources` should always be snake case and plural.

We can inspect all of the routes in the terminal by running the command `$ rake routes`

![Rake Routes](images/rake_routes.png)

Take note of the prefix column. We're going to leverage the prefix to create paths that we can link to using the ...

### `link_to` method

There's a great helper method rails gives us, to link to things in our rails views, `link_to`. The `link_to` method takes two or more arguments. The first argument is what text you want the link to show up as. The second argument is where it will go. You can also checkout other options you can pass in [here](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to). Check out the way we wrote this in Sinatra:

```html
<ul>
  <% @todos.each do |todo| %>
    <li>
      <p><a href="todos/<%= todo.id %>"><%= todo.body %></a></p>
    </li>
  <% end %>
</ul>
```

We can write this in rails like this:

```html
<ul>
  <% @todos.each do |todo| %>
    <li>
      <p><%= link_to todo.body, todo_path(todo) %></p>
    </li>
  <% end %>
</ul>
```

In the latter snippet we're using the `todo` prefix with `_path` to link to the show route.

These two snippets are functionally equivalent. The latter is the rails way. In fact, Rails is so smart that if we pass in the model itself to `link_to`, it'll work as well:

```html
<%= link_to todo.body, todo %>
```

If we actually click one of these links, we'll see a familiar error, `unknown action`. Let's add the action now, along with the query to the database that we'll need. In `app/controllers/todos_controller.rb`:

```ruby
def show
  @todo = Todo.find(params[:id])
end
```

Just like in Sinatra, rails passes data from requests through `params`.

Refresh and ... the same template error! Let fix that by creating the view for this action.

```bash
$ touch app/views/todos/show.html.erb
```

Let's update the show view to link back to the index and show both the body and author of the reminder. In `app/views/todos/show.html.erb`:

```html
<h2><%= link_to "Todos", todos_path %></h2>
<p><%= @todo.body %></p>
<p><%= @todo.author %></p>
```

You'll notice here, we're using the `todos` prefix for the index route in the `link_to`. It's very important we're very exact when we write the path in a `link_to` helper. One 's' can change the route completely.

Awesome! We now have a fully fledged index and show feature for our application.

## The Show Feature: You do - Tunr

- Create a show feature for each artist.
- Make sure the show feature links to the index of artists.
- Change existing routes to leverage `resources`.
