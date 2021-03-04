---
layout: post
title: "Rails with Devise: Sign in as app landing page"
description: Use Devise sign-in page as landing page of Rails application
summary: How to use Devise’s sign-in page as the landing page for a Rails 
         application.
tags: [rails, devise] 
---

A sensible [guideline](https://twitter.com/tylertringas/status/1250521285630836741) for web applications is to separate the marketing site from the web application. This can be done by having the marketing site reside on the apex domain (e.g. `example.com`) and the application on a subdomain (e.g. `app.example.com`). Keep the product decoupled from marketing, so changes on the one won’t adversely  affect the other. 

Since we have the marketing landing page separate from the application, it’d be a waste to have *another* non-functional page inside the application users will have to click through before they get where they want to be. We want the first thing users see inside the app to be functional. How could we do this using Rails? If you use Devise for authentication, and I recommend you do, there might just be a simple solution.

Let’s assume user authentication is required any activity inside the application. We’ll add the `authenticate_user!` function to the `ApplicationController` as follows:

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

This bit tripped me up for a second (or two). At this point, the root route after logging in is an infinite redirect, which doesn’t resolve. Enter Stack Overflow. I happened upon [this page](https://stackoverflow.com/questions/4954876/setting-devise-login-to-be-root-page) first, which has the same problem we’re stuck on. No bueno. Thanks to the kindness of user Jngai1297, we could click through to [this page](https://stackoverflow.com/questions/19855866/how-to-set-devise-sign-in-page-as-root-page-in-rails), where Rajdeep Singh lands the finish blow on our formidable foe. It turns out we have another adjustment to make to `ApplicationController`:

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

Now visiting the app will direct users to the sign-in page, and signing out will do likewise. With that, you should be able to resume building your kickass web app. Happy coding! :)

 