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