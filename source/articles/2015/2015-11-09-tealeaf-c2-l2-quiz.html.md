---
layout: post
title: Tealeaf C2-L2 Quiz
date: 2015-11-09 19:54 UTC
tags: ['tealeaf', 'ruby']
image: tealeaf.png
---

![Tealeaf Academy](tealeaf.png)

Once again, I am posting my answers to the [Tealeaf][tl] 2nd Course as a cheatsheet for 'my future self'. :)

[tl]: www.gotealeaf.com

1. Name all the 7 (or 8) routes exposed by the resources keyword in the routes.rb file. Also name the 4 named routes, and how the request is routed to the controller/action.

    | path name      | method         | path           | controller#action |
    |:---------------|:---------------|:---------------|:------------------|
    | posts          | GET            | /posts         | posts#index       |
    | new\_post\_path| GET            | /posts/new     | posts#new         |
    |                | POST           | /posts         | posts#create      |
    |edit\_post\_path| GET            | /posts/:id/edit| posts#edit        |
    |                | PATCH/PUT      | /posts/:id     | posts#update      |
    | post\_path     | GET            | /posts/:id     | posts#show        |
    |                | DELETE         | /posts/:id     | posts#destroy     |

2. What is REST and how does it relate to the resources routes?

    REST stands for REpresetational State Transfer. REST is an architecture style of designing networked applications. This architecture is used for the Word Wide Web. It relies on using a stateless, client-server, cacheable communications protocol. In case of Rails (and WWW in general), mostly HTTP protocol is used. Through HTTP protocol browsers (and other clients) send requests to network resources (servers) and perform CRUD (create/read/update/delete) actions on them.

    Rails follows REST architecture (is RESTful) in the sense that it defines objects (models) as RESTful resources and uses standard HTTP methods (GET, POST, PATCH, DELETE) and set of standardized routes to perform the CRUD actions on them.

3. What's the major difference between model backed and non-model backed form helpers?

    Non-model backed form helpers are essentially a slightly more concise Rails(Ruby) syntax to display HTML forms.

    Model backed form helpers are much smarter because they build different forms depending on the object they are tied to. For example, if it's a new object, an empty form is displayed and the form action is set to the 'create' route for that type of object (and the 'POST' method). If it's an existing object, the form fields are pre-filled with the current values for that object, and the form action is set to 'update' (and the 'PATCH' method).

    Plus, it highlights the fields with errors if they do not pass the validations.

    Model backed form helpers are tied to an object and only allow field names that correspond to attributes or virtual attributes (for example, collections associated with this objects) of that object.

4. How does form_for know how to build the <form> element?

    The method checks if the object exist in the database already or it's a new object and displays a corresponding form (in the way described in the previous answer).

5. What's the general pattern we use in the actions that handle submission of model-backed forms (ie, the create and update actions)?

    The general pattern is to:
      - put the data submitted through the form into an instance variable (an object)
      - try to save the object into the database via ActiveRecord
      - in the process the validations are run on that object
      - if it passes all the validations, then we record a notice in the session (via ```flash[:notice]``` saying that the object was saved (or updated) successfully and then redirect to corresponding page (that object's 'show' page or the 'index' page where all object are listed).
      - if not, we render the template containing the form once again and there display the errors that occurred during validation.

      Here is how it looks in the code:

    ```ruby
    def create
      @post = Post.new(params.require(:post).permit(:url, :title, :description))

      if @post.save
          flash[:notice] = "Your post was saved."
          redirect_to posts_path
      else
          render :new
      end
    end

    def update
      @post = Post.find(params[:id])

      if @post.update(params.require(:post).permit(:url, :title, :description))
          flash[:notice] = "Your post was updated."
          redirect_to post_path(@post)
      else
          render :edit
      end
    end
    ```

6. How exactly do Rails validations get triggered? Where are the errors saved? How do we show the validation messages on the user interface?

    Rails validations get triggered when we attempt to save the new/updated object into the database. If the object does not pass all validations, it does not get saved and the errors are attached directly to that object (no exceptions are raised by default and nothing gets written into the log).

    The list of errors can be access by calling the ```.errors``` method on the object. It returns an Errors object that contains a hash of error messages. The keys are the attribute (field) names and the values are arrays of errors related to each attribute.

    To parse the errors out ```.full_messages``` (or its synonym ```.to_a```) method can be used, which concatenates the field names with error messages and puts the resulting lines into an array. We can use ```.full_messages.each``` to iterate through the messages and display them.

    Furthermore, the form fields that did not pass validations, get a ```field_with_errors``` class attached, so we can change formatting for such fields through CSS (for example highlight them with red color).

    We can also access the errors for a particular field through ```.errors[:field_name]``` or ```.errors.on :field_name```. For longer forms it makes more sense to display the errors under each field rather that in one block above the form.

    More details [here][ve].

[ve]: http://guides.rubyonrails.org/active_record_validations.html#working-with-validation-errors

    Sample code:

    ```ruby
    <% if obj.errors.any? %>
      <div class="row">
        <div class="alert alert-error span8">
        <h5>Please fix the following errors:</h5>
        <ul>
        <% obj.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
        <% end %>
        </ul>
        </div>
      </div>
    <% end %>
    ```

7. What are Rails helpers?

    Helpers are methods that can be used by the views templates. Typically we extract into the helpers the logic and formatting of bits of information that are needed multiple times in different views. Thus they help keep the code DRY and avoid repetition.

8. What are Rails partials?

    Rails partials are used to extract parts of views like forms, titles, HTML representation of objects, etc. They can be reused (for example a 'form' partial can be used by new, create, edit and update actions of the same resource).

    If we extract an object HTML representation into a partial, we can use the partial multiple times in a loop to display each object. We can even use a very concise special Rails syntax (```render @objects```) with the same effect (where ```objects``` is a collection of objects of certain type and the name of the partial is ```_object.html.erb```).

9. When do we use partials vs helpers?

    When we need to extract larger chunks of HTML code from the view template.

10. When do we use non-model backed forms?

    To build the forms that are not tied to any ActiveRecord model.
