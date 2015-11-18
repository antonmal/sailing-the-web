---
layout: post
title: Tealeaf C1-L3 Quiz
date: 2015-11-18 11:26 UTC
tags: tealeaf
image: tealeaf.png
---

![Tealeaf Academy](tealeaf.png)

1. What's the difference between rendering and redirecting? What's the impact with regards to instance variables, view templates?

    When a Rails app gets an HTTP request, the router (```routes.rb```) determines which controller+action should generate the HTTP response. Within that action we define the instance variables and then either render a view template or redirect the request to another URL.

    In Rails rendering means sending back an HTTP response with HTML/JS/JSON content generated based on a view template. The instance variables that we defined in the action are available for the view template.

    Redirecting means telling the browser to make a new request to a different url. Because the HTTP protocol is stateless, instance variables defined in the current action are lost during redirect and are no longer available when the new request comes.

2. If I need to display a message on the view template, and I'm redirecting, what's the easiest way to accomplish this?

    The ```FlashHash``` object can be used. You can store the info you need in this hash. It will be available in the current and the next action and then cleared out automatically. Common practice is to use this hash to store messages that need to be displayed after redirect. You can use ```flash[:notice] = "..."``` to store a success or info message. And ```flash[:error]``` to store an error message. Since these particular messages are stored in ```flash``` so often, you can use setter and getter methods instead: ```flash.notice```, ```flash.error = "..."```. Then in your views, you need to access the ```flash``` object to display the messages.

3. If I need to display a message on the view template, and I'm rendering, what's the easiest way to accomplish this?

    Similarly to the situation when we are redirecting, we can use ```flash``` to store the message. Since by default the ```flash``` hash is cleared only after the next action completes, it will display the message both for the current action (on the view we render), and for the next action (when the next request comes). Usually, you want to display the message only once in this case. To accomplish this, you should use ```flash.now``` instead of just ```flash```.

4. Explain how we should save passwords to the database.

    We should never store passwords directly in a database. Instead, we should store an encrypted password hash (digest) that is generated with a library like ```bcrypt-ruby``` gem in a way that does not allow decrypting the hash back into the password. Instead, when we need to authorize a user, we ask him to enter his password, then encrypt it and compare the resulting hash with the one we have in our database.

    Mode specifically, we can use the ```has_secure_password :password_field``` method in the model. In that case, the database for this model should contain the ```password_field_digest``` attribute, where the password hash will be stored. This will give the corresponding object the following methods:
    * ```.password=``` setter method (that will encrypt the password into a hash)
    * and the ```.authenticate(password)``` that takes the ```password```, turns it into a hash and compares that with the hash stored in our database. If they are equal, the entered password is correct, and the method returns ```true```; otherwise it returns ```false```.  

5. What should we do if we have a method that is used in both controllers and views?

    It depends on what that method does:
    * If it is about business logic or data manipulation, then the best place for it is in the model file (```/models/object_name.rb```). In general, this is the preferred approach ('Fat model, thin controller').
    * If it is a helper method used primarily by a controller, then it can be defined in the controller file itself. If it is used by multiple controllers, it should go into ```application_controller.rb```. If it is also needed in the views, we can make it available by adding this line: ```helper_method: :your_method_name```.

6. What is memoization? How is it a performance optimization?

    Memoization looks like this: ```@foo ||= <statement...>```, which means: 'if ```@foo``` variable has not been defined yet, then evaluate the expression behind the ```||=``` operator (send a request to a database, do some data manipulation, calculations, etc), otherwise use the current value of that variable.'. It optimizes performance because the expression is evaluated only once (and the evaluation often takes a lot of time and resources, especially if we send requests to the database).

    In other words, memoization allows us to enhance performance because we are caching the result of a method call instead of calling it every single time. This is a good technique whenever the result is the same every time. Instead of running the method and hitting the database every time per request, we can store the first result as an instance variable. By doing so, it will initially hit the database only once to get the stored values and will optimize our performance since we can refer to the instance variable instead of calling the method again.

7. If we want to prevent unauthenticated users from creating a new comment on a post, what should we do?

    We should check if the user is logged in before executing the ```new``` action of the ```comments``` controller, like this:

    ```ruby
    # comments_controller.rb

    class CommentsController < ApplicationController
      before_action :require_user, except: [:show, :index]

      def new
        ###
      end

      def create
        ###
      end
    end


    # application_controller.rb

    helper_method :current_user, :logged_in?, :require_user

    def current_user
      @current_user ||= User.find(session[:user_id]) if session[:user_id]
    end

    def logged_in?
      !!current_user
    end

    def require_user
      unless logged_in?
        redirect_to root_path, alert: "You need to be logged in to do this."
      end
    end
    ```

    If the new comment form is only a part of a view (for example, the 'post show' view), we can hide the form from the logged-out users like this:

    ```ruby
    <% if logged_in? %>
      <%= form_for @comment do |f| %>
        ###
      <% end %>
    <% end %>
    ```

8. Suppose we have the following table for tracking "likes" in our application. How can we make this table polymorphic? Note that the "user_id" foreign key is tracking who created the like.

    | id | user\_id | photo\_id | video\_id | post\_id |
    |----|----------|-----------|-----------|----------|
    | 1  |	4		    |           | 12        |          |
    | 2  |	7			  |           |           | 3        |
    | 3  |	2	      | 6	  	    |           |          |

    We should use this table instead:

    | id | user\_id | likeable\_type | likeable\_id |
    |----|----------|----------------|--------------|
    | 1  |	4		    | "Video"        | 12           |
    | 2  |	7			  | "Photo"        | 3            |
    | 3  |	2	      | "Post"  	     | 6            |

9. How do we set up polymorphic associations at the model layer? Give example for the polymorphic model (eg, Vote) as well as an example parent model (the model on the 1 side, eg, Post).

    ```ruby
    # /models/vote.rb
    class Vote < ActiveRecord::Base
      belongs_to :votable, polymorphic: true
    end

    # /models/post.rb
    class Post < ActiveRecord::Base
      has_many :votes, as: :votable
    end
    ```

10. What is an ERD diagram, and why do we need it?

    ERD diagram is a 'Entity Relationship Diagram', which shows entities (tables) in our database, attributes of each table, and, most importantly, relationships between different entities. A relationship is shown via a line connecting a the primary key attribute of one entity to a foreign key attribute of another entity. In on of the common ERD styles, arrow(s) are attached to the lines on the ```many``` side of the relationship to describe the different types of relationships (```1:1```, ```1:M```, ```M:M```).

    ERD diagram makes it easy to understand the relationships between the entities at a glance. It is much easier (and makes more sense) to work out the database structure by drawing the ERD diagram iteratively, rather than changing the models / database many times. Once the ERB is drawn, writing the migrations and models code is very easy.
