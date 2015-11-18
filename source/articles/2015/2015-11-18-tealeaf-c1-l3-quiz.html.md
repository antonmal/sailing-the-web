---
layout: post
title: Tealeaf C1-L3 Quiz
date: 2015-11-18 11:26 UTC
tags: tealeaf
image: tealeaf.png
---

![Tealeaf Academy](tealeaf.png)

Quiz: Lesson 3

1. What's the difference between rendering and redirecting? What's the impact with regards to instance variables, view templates?

    Rendering means sending HTTP response that contains HTML (or other) code that the client (browser) can display or otherwise use. In other words, we render one of the view templates and send it back to the browser to be displayed. Instance variables created when we received the current request are available in view templates.

    Redirecting means telling the browser that it needs to make another request (to a different url). Instance variables created when the current request was received are lost, because they are only available within the current request; when the new request comes (after redirect), all instance variables will need to be re-created again.

2. If I need to display a message on the view template, and I'm redirecting, what's the easiest way to accomplish this?

    The ```FlashHash``` object can be used. You can store the info you need in this hash. It will be available in the current and the next action and then cleared out automatically. Common practice is to use this hash to store messages that need to be displayed after redirect. You can use ```flash[:notice] = "..."``` to store a success or info message. And ```flash[:error]``` to store an error message. Since these particular messages are stored in ```flash``` so often, you can use setter and getter methods instead: ```flash.notice```, ```flash.error = "..."```. Then in your views, you need to access the ```flash``` object to display the messages.

3. If I need to display a message on the view template, and I'm rendering, what's the easiest way to accomplish this?

    Similarly to the situation when we are redirecting, we can use ```flash``` to store the message. Since by default the ```flash``` hash is cleared only after the next action completes, it will display the message both for the current action (on the view we render), and for the next action (when the next request comes). Usually, you want to display the message only once in this case. To accomplish this, you should use ```flash.now``` instead of just ```flash```.

4. Explain how we should save passwords to the database.

    We should never store passwords directly in a database. Instead, we should store an encripted password hash (digest) that is generated with a library like ```bcrypt-ruby``` gem in a way that does not allow decripting the hash back into the password. Instead, when we need to authorize a user, we ask him to enter his password, then encript it and compare the resulting hash with the one we have in our database.

    Mode specifically, we can use the ```has_secure_password :password_field``` method in the model. In that case, the database for this model should contain the ```password_field_digest``` attribute, where the password hash will be stored. This will give the corresponding object the following methods:

        - ```.password=``` setter method (that will encript the password into a hash)
        - and the ```.authenticate(password)``` that takes the ```password```, turns it into a hash and compares that with the hash stored in our database. If they are equal, the entered password is correct, and the method returns ```true```; otherwise it returns ```false```.

5. What should we do if we have a method that is used in both controllers and views?

    We should declare the method in the ```application_controller.rb```, this will make it available to all controllers. Then, we should make the method available to all views by adding this line to the same file: ```helper_method: :your_method_name```.

6. What is memoization? How is it a performance optimization?

    Memoization look like this: ```@foo ||= Foo.find(params[:id])```, which means: 'if ```@foo``` variable has not been defined yet, then set it equal to a specific value from the database, otherwise use the current value of that variable (and do not go to the database again)'. It optimizes performance because it prevents unnecessary requests to the database (which take a lot of resources/time).

    More generally, memoization allows us to enhance performance because we are caching the result of a method call instead of calling it every single time. This is a good technique whenever the result is the same every time. Instead of running the method and hitting the database every time per request, we can store the first result as an instance variable. By doing so, it will initially hit the database only once to get the stored valued and will optimize our performance since we can refer to the instance variable instead of calling the method again.

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
    class Vote < ActiveRecord::base
      belongs_to :votable, polymorphic: true
    end

    # /models/post.rb
    class Post < ActiveRecord::Base
      has_many :votes, as: :votable
    end
    ```

10. What is an ERD diagram, and why do we need it?

    ERD diagram is a 'Entity Relationsip Diagram', which shows entities (tables) in our database, attributes of each table, and, most importantly, relationships between different entities. A relationship is shown via a line connecting a the primary key attribute of one entity to a foreign key attribute of another entity. In on of the common ERD styles, arrow(s) are attached to the lines on the ```many``` side of the relationship to describe the different types of relationships (```1:1```, ```1:M```, ```M:M```).

    ERD diagram makes it easy to understand the relationships between the entities at a glance. It is much easier (and makes more sense) to work out the database structure by drawing the ERD diagram iteratively, rather than changing the models / database many all the time. Once the ERB is built, writing the migrations and models code is very easy.
