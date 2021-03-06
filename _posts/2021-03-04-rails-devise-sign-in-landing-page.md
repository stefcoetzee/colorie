---
layout: post
title: "Rails with Devise: Sign-in page as app landing page"
description: How to use Devise’s sign-in page as the landing page for a Rails 
             application.
summary: How to use Devise’s sign-in page as the landing page for a Rails 
         application.
tags: [rails, devise]
last_modified_at: 2021-03-06
---

A sensible [guideline](https://twitter.com/tylertringas/status/1250521285630836741) for web applications is to keep the product decoupled from marketing. 
This prevents changes on the one from adversely affecting the other.
A straightforward way to do this: deploy the marketing site on the apex domain (e.g. `example.com`) and the application on a subdomain (e.g. `app.example.com`). 

Since we have the marketing landing page separate from the application, it’d be a waste to have *another* non-functional page inside the application users have to click through. 
We want the first thing users see inside the app to be functional. 
How could we do this in Rails? 
If you use Devise for authentication, and I recommend that you do, there might just be a simple solution.

Let’s assume user authentication is required for any activity inside the application. 
We’ll add the [`authenticate_user!`](https://github.com/heartcombo/devise/blob/0cd72a56f984a7ff089246f87a8b259120545edd/lib/devise/controllers/helpers.rb#L99) function to `ApplicationController` as follows:

```ruby
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  before_action :authenticate_user!
  
  ...
end
```

Next, we’ll change the root route (try saying that three times fast) in `routes.rb`:

```ruby
# config/routes.rb

Rails.application.routes.draw do
  devise_for :users
    
  devise_scope :user do
    root 'devise/sessions#new'
  end
    
  ...
end
```

At this point, after logging in the root route spirals into an infinite redirect, which obviously doesn’t resolve.
This bit tripped me up for a second (or two). 
Enter Stack Overflow. 
I happened upon [this page](https://stackoverflow.com/questions/4954876/setting-devise-login-to-be-root-page) first, which has the same problem we’re stuck on. 
No bueno. 
Thanks to the kindness of user Jngai1297, we could click through to [this page](https://stackoverflow.com/questions/19855866/how-to-set-devise-sign-in-page-as-root-page-in-rails), where Rajdeep Singh lands the finish blow on our formidable foe. 
It turns out we have another adjustment to make to `ApplicationController`:

```ruby
# app/controllers/application_controller.rb

class ApplicationController < ActionController::Base
  before_action :authenticate_user!
    
  def after_sign_in_path_for(user)
    # your path goes here
    user_posts_path(user) # as an example
  end
    
  ...
end
```

Now visiting the app will direct users to the sign-in page, and signing out will do likewise.
To remove the “You are already signed in.” flash notification users receive when visiting the root route after signing in, adjust `devise.en.yml` like so:

```yaml
# config/locales/devise.en.yml

en:
  devise:
    ...
    failure:
      already_authenticated: ""
      ...
```

With that, you should be able to resume building your kickass web app. 
Happy coding! :)

 