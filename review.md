# Index, Show, Create, Delete Review

## Initial Setup

```
$ git clone git@github.com:ga-wdi-exercises/tunr_features.git -b review
$ cd tunr_features
$ bundle install
$ rake db:migrate db:create db:seed
$ rails s
```

We just:

- cloned the repository
- installed dependencies
- created the database
- created tables and columns in the database
- loaded in test data
- started the rails server

## Index

In order to list all of the Artists in the browser, we need to define a route:

```rb
# config/routes.rb

Rails.application.routes.draw do
  resources :artists
end
```

This maps all of the CRUD actions to methods in the artistscontroller.

You can tell by looking at the output of `rake routes`:

```
     Prefix Verb   URI Pattern                 Controller#Action
    artists GET    /artists(.:format)          artists#index
            POST   /artists(.:format)          artists#create
 new_artist GET    /artists/new(.:format)      artists#new
edit_artist GET    /artists/:id/edit(.:format) artists#edit
     artist GET    /artists/:id(.:format)      artists#show
            PATCH  /artists/:id(.:format)      artists#update
            PUT    /artists/:id(.:format)      artists#update
            DELETE /artists/:id(.:format)      artists#destroy
```

### Create the Artists Controller

```rb
# app/controllers/artists_controller.rb
class ArtistsController < ApplicationController

end
```

### Define the Index method

```rb
# app/controllers/artists_controller.rb
class ArtistsController < ApplicationController
  def index
    @artists = Artist.all
  end
end
```

### Display each artist

```erb
<!-- app/views/artists/index.html.erb -->
<!--          ^ the       ^ the action
	      controller
	      name
-->

<ul>
<% @artists.each do |artist| %>
  <li><%= artist.name %></li>
<% end %>
</ul>
```

This should display each artist at <http://localhost:3000/artists>

## Show


```
<ul>
<% @artists.each do |artist| %>
  <li><%= link_to artist.name, artist_path(artist)%></li>
<% end %>
</ul>
```

- `link_to` generates an anchor tag
- `artist_path` is a method created from `resources :artists` in config/routes

### Define the show action

```rb
class ArtistsController < ApplicationController
  def index
    @artists = Artist.all
  end
  def show
    @artist = Artist.find(params[:id])
  end
end
```

### Create the template

```
<!-- app/views/artists/show.html.erb -->

<%= artist.name %>
```

## Create

### Define the action

```rb
class ArtistsController < ApplicationController
  def index
    @artists = Artist.all
  end
  def show
    @artist = Artist.find(params[:id])
  end
  def new
    @artist = Artist.new
  end
end
```

### Create a form

```
<!-- app/views/new.html.erb -->

<%= form_for @artist do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.label :photo_url %>
  <%= f.text_field :photo_url %>
  <%= f.label :nationality %>
  <%= f.text_field :nationality %>
  <%= f.submit %>
<% end %>
```

### Define the action

```rb
class ArtistsController < ApplicationController
  def index
    @artists = Artist.all
  end
  def show
    @artist = Artist.find(params[:id])
  end
  def new
    @artist = Artist.new
  end
  def create
    @artist = Artist.create artist_params
    redirect_to artist_path @artist
  end

  private
  def artist_params
    params.require(:artist).permit(:name, :photo_url, :nationality)
  end
end
```

Using `strong_params` prevents a vulnerability called [mass assignment](http://brakemanscanner.org/docs/warning_types/mass_assignment/)

## Delete

### Update the show view

```
<!-- app/views/artists/show.html.erb -->

<%= artist.name %>

<%= link_to artist_path(@artist), method: :delete %>
```

### Define the action

```rb
class ArtistsController < ApplicationController
  def index
    @artists = Artist.all
  end
  def show
    @artist = Artist.find(params[:id])
  end
  def new
    @artist = Artist.new
  end
  def create
    @artist = Artist.create artist_params
    redirect_to artist_path @artist
  end
  def destroy
    @artist = Artist.find(params[:id])
    @artist.destroy
    redirect_to artists_path
  end

  private
  def artist_params
    params.require(:artist).permit(:name, :photo_url, :nationality)
  end
end
```

All the code from today is here: https://github.com/ga-wdi-exercises/tunr_features/compare/review...review-solution