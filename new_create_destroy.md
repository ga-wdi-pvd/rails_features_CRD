# Rails Create & Destroy features

Check out the [precursor to this lesson](index_show.md)

## Framing
One of the most common things that is done on the web is to create new rows in a database. Only users aren't thinking they're creating new rows in a database. They're creating a new status on Facebook, creating a new tweet on the latest news, creating a new sale on craigslist.

In the current version of our Reminder.ly app, we have a solid `index` and `show` feature that allows us to view all our `todos`  as well as each individual `todo`. Now we want to be able to create new `todos`!

### The `new` route: I do - Reminder.ly

In order to give the user ability to create a `todo`, we first need to provide that user with an interface(aka. a form). For now, we'll put this form on a separate page. Let's update the resources so that the router can make sense of requests to `new` route.

```ruby
resources :todos, only: [:index, :show, :new]
```

Let's also add a link in the `index` feature to link to the page that will create our form. In `app/views/todos/index.html.erb`:

```html
<h2><%= link_to "New Todo", new_todo_path %></h2>
```

Let's click on the link? This unknown action error again... We know what do with you. In `app/controllers/todos_controller.rb`:

```ruby
def new
end
```

Let's take a look at how we did this form in Sinatra first:

```html
<h2>New Todo</h2>

<form action="/todos" method="post">
  <label>Body:</label>
  <input type='text' name="todo[body]"><br>
  <label>Author:</label>
  <input type='text' name="todo[author]"><br>
  <input type="submit">
</form>
```

Here's how we do it in rails:

```html
<%= form_for @todo do |f| %>
  <%= f.label :body %>
  <%= f.text_field :body %>

  <%= f.label :author %>
  <%= f.text_field :author %>

  <%= f.submit %>
<% end %>
```

Checkout [rails docs on forms](http://guides.rubyonrails.org/form_helpers.html) to find out everything you want to know about forms and more.

`form_for` is a helper in rails that allows us to build forms for models. The `f` in pipes is a FormBuilder object that incorporates the knowledge about the model(`@todo` in this case). Methods defined on the FormBuilder are used to generate fields bound to this model.

Let's inspect the form using our dev tools and we'll see all the same elements that we see in the Sinatra version of this form. Also note that the input `name` attributes are nested under `todo`

### The `new` route: You do: Tunr

Create a form for users to create new artists at a brand new route.

### Creating: I do - Reminder.ly

If we submit the form we'll get the following error:

![routing error](images/routing_error.png)

The router doesn't understand the request that was trying to come through, let's update `config/routes.rb`:

```ruby
resources :todos, only: [:index, :show, :new, :create]
```

Another error ... unknown action `create` ..this is starting to follow a pattern ...

In `app/controllers/todos_controller.rb`:

```ruby
def create
  @todo = Todo.create!(params[:todo])
  redirect_to @todo
end
```

Now we get this weird error:

![Forbidden Attributes](images/forbidden_attr.png)

Wat?! Why can't we create a `todo` using the hash available in params? This is a security feature of Rails: params could include extra fields that have been maliciously added to the form. This extra data could be harmful, therefore Rails requires us to whitelist fields that are allowed through form submissions.

### Strong Params

Whitelisting is done using **strong parameters** configuration.

This is a security feature of Rails.

Now we should update our todos controller a bit.

The way we do strong parameters is through a private method that will allow us to pull information from the form.

in `app/controllers/todos_controller.rb`:
```ruby
private
def todo_params
  params.require(:todo).permit(:body, :author)
end
```

> The `require` method ensures that a specific parameter is present. Throws an error otherwise. The `permit` method returns a copy of the parameters object, returning only the permitted keys and values.

> **Note** that we encapsulate the `todo_params` in a private method because we only want this available to this particular class and it shouldn't work outside the scope of the controller

The only thing we have left to do is update our controller actions that use params to create a todo.

in `app/controllers/todos_controller.rb`:

```ruby
def create
  @todo = Todo.create!(todo_params)
  redirect_to @todo
end
```

### Creating: You do - Tunr

- Make your form create artists when submitted.
- be sure to use strong parameters
- have the create action redirect to the artist you just created

### Deleting: I do - Reminder.ly

Another important feature in web apps is the ability to delete rows of data in a database.

Deleting in a rails app is quite simple. First let's add a route to listen for delete requests. In `config/routes.rb`:

```ruby
resources :todos, only: [:index, :show, :new, :create, :destroy]
```

Then let's provide the ability to delete todos in the index feature. In `app/views/todos/index.html.erb`:

```html
<ul>
  <% @todos.each do |todo| %>
    <li>
      <p><%= link_to todo.body, todo %></p>
      <p><%= link_to 'Delete Todo', todo_path(todo),
              method: :delete,
              data: { confirm: 'Are you sure?' } %></p>
    </li>
  <% end %>
</ul>
<h2><%= link_to "New Todo", new_todo_path %></h2>
```

Before this link will work, we need to update the controller to determine what happens when a `delete` requests hits the router. In `app/controllers/todos_controller.rb`:

```ruby
def destroy
  @todo = Todo.find(params[:id])
  @todo.destroy
  redirect_to todos_path
end
```

Awesome, we can now delete any todos we want!

### Deleting: You do - Tunr

- Add the ability to delete artists

Deleting artist will actually cause an error because you have songs that are associated to artist. We need to include a bit of code the in the Artist Model.

```ruby
class Artist < ActiveRecord::Base
  has_many :songs, dependent: :destroy
end
```

This bit of code will destroy all songs belonging to a particular artist when that artist is destroyed.

### Moar - Tunr

- With any remaining time, try to incorporate the same features for the `song` model.

### Bonus

- Incorporate a third model `playlists` and add a feature where a user can custom make a playlist.
