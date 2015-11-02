---
layout: post
title: Tealeaf C2-L1 Quiz
date: 2015-11-02 17:07 UTC
tags: ['tealeaf', 'rails']
image: tealeaf.png
---

![Tealeaf Academy](tealeaf.png)

I decided to keep my responses to the Course 2 / Lesson 1 Quiz questions here, so this post serves as a cheat sheet for the terminology and the most important concepts learned during this part of the [Tealeaf][tl] course.

[tl]: http://www.gotealeaf.com

1. Why do they call it a relational database?

    Because such database consists of several 'relations' otherwise called 'tables'.

    Each table describes the relationship (thus the name 'relation') between an entity (a class of objects, for example, 'book') and it's attributes (say, 'title', 'page_count', 'ISBN', etc).

    A table consists of multiple rows and columns. A row (or 'tuple') represents an entity instance, that is a particular object of that entity type (for example, 'The 4-hour workweek' book by Tim Ferriss). A table has columns, one for each entity attribute. One of the columns is used as the table key, an attribute that uniquely identifies each entity instance.

    Tables (and corresponding entities) are linked to each other by using a 'foreign key' column in one table to store the key of another entity related to the first one. This link is also often called a 'relation', which, I guess, is the other reason for the 'relational database' term.

2. What is SQL?

    'Structured query language' is an DSL (domain specific language) used to manipulate and retrieve data from relational databases.

3. There are two predominant views into a relational database. What are they, and how are they different?

    The first view is into the database schema. A schema lists the tables that the database contains and the attributes of each table along with the value type for each attribute. A schema shows the structure of the database.

    The second is the view into the actual data that is stored in these tables. For each table this looks like a spreadsheet with attributes as columns and records (instances of corresponding object) as rows.

4. In a table, what do we call the column that serves as the main identifier for a row of data? We're looking for the general database term, not the column name.

    It is called the 'primary key'.

5. What is a foreign key, and how is it used?

    It is used to link two entity instances together. The first entity has the foreign key of the second entity as one of it's attributes (always with 'integer' type). Thus, the first entity is linked to (belongs to) the second one.

6. At a high level, describe the ActiveRecord pattern. This has nothing to do with Rails, but the actual pattern that ActiveRecord uses to perform its ORM duties.

    ORM stands for the 'object-relational model'. ActiveRecord is an example of ORM. It is used to map objects (created in the code using OOP principles) to database structures. ActiveRecord in most cases replaces SQL as the DSL to access databases from Ruby on Rails.

7. If there's an ActiveRecord model called "CrazyMonkey", what should the table name be?

    It should be like this: ```crazy_monkeys```. To make sure, run ```"CrazyMonkey".tableize```.

8. If I'm building a 1:M association between Project and Issue, what will the model associations and foreign key be?

    ```ruby
    class Project < ActiveRecord::Base
      has_many :issues
    end

    class Issue < ActiveRecord::Base
      belongs_to :project
    end
    ```

    The foreign key will be stored in the ```issues``` table and (by default) will be called ```project_id```. If you want to use a different foreign key, you can define it in the associations by using a ```foreign_key: '<custom_key>'``` option.

9. Given this code:

    ```ruby
    class Zoo < ActiveRecord::Base
      has_many :animals
    end
    ```

    **What do you expect the other model to be and what does database schema look like?**

    ```ruby
    class Animal < ActiveRecord::Base
      belongs_to :zoo
    end
    ```

    The schema will contain a table ```zoos``` with a primary key ```id``` and a table ```animals``` with a foreign key ```zoo_id```.

    **What are the methods that are now available to a zoo to call related to animals?**

    ```ruby
    # take a particular instance of an object of class Zoo
    zoo = Zoo.first

    # get the array of animal objects belonging to this particular zoo
    zoo.animals

    # add an animal to the collection of animals of a specific zoo
    zoo.animals << new_animal

    # number of animals in the collection of a particular zoo
    zoo.animals.size # or .count or .length

    # are there no animals in this zoo?
    zoo.animals.empty?

    # replace the whole collection of animals with another one
    zoo.animals = [...array of animal objects...]
    zoo.save

    # iterate through the collection of animals of a specific zoo
    zoo.animals.each
    ```

    **How do I create an animal called "jumpster" in a zoo called "San Diego Zoo"?**

    ```ruby
    zoo = Zoo.find(name: 'San Diego Zoo')
    zoo.animals << Animal.new(name: 'jumpster')
    ```

10. What is mass assignment? What's the non-mass assignment way of setting values?

    ```ruby
    # mass assignment
    Animal.create(name: 'Bob', type: 'giraffe', age: 8)
    # or
    a = Animal.new(name: 'Bob', type: 'giraffe', age: 8)
    a.save

    # normal non-mass assignment
    a = Animal.new
    a.name = 'Bob'
    a.type = 'giraffe'
    a.age = 8
    a.save
    ```

11. Suppose Animal is an ActiveRecord model. What does this code do? ```Animal.first```

    This code selects the first record from the ```animals``` database table and returns it as an instance of the ```Animal``` class, with all attributes of that table available as instance methods.

12. If I have a table called "animals" with a column called "name", and a model called Animal, how do I instantiate an animal object with name set to "Joe". Which methods makes sure it saves to the database?

    ```ruby
    # .create saves the new object right away
    Animal.create(name: 'Joe')

    # .new doesn't, you need to call .save afterwards
    a = Animal.new(name: 'Joe')
    a.save
    ```

13. How does a M:M association work at the database level?

    It works through so-called join table. Each row of the join table contains two foreign keys representing instances of both entities that form the M:M association.

14. What are the two ways to support a M:M association at the ActiveRecord model level? Pros and cons of each approach?

    First is called ```has_and_belongs_to_many <objects>```

    ```
    + easy to set up, less code to write because no join model needed
    - does not support additional attributes in the join table, only the two foreign keys
    - the join table is not defined directly in the migrations, it is implied and created automatically in the background. Table name is chosen according to Rails convention ('entity1s_entiti2s') and cannot be changed manually.
    - will be deprecated in the future
    ```

    The other one is called ```has_many <objects>, through: <join table>```.

    ```
    - a little more code to write, join model needs to be defined
    + supports additional attributes in the join table
    + defined clearly in the migrations. Table name can be chosen manually.
    + more flexible, easier to change later
    + will be the only option in the future, when the other one is deprecated
    ```

    **Always use ```has_many :through``` !!!**

15. Suppose we have a User model and a Group model, and we have a M:M association all set up. How do we associate the two?

    ```ruby
    class User < ActiveRecord::Base
      has_many :user_groups
      has_many :groups, through: :user_groups
    end

    class Group < ActiveRecord::Base
      has_many :user_groups
      has_many :users, through: :user_groups
    end

    class UserGroup < ActiveRecord::Base
      belongs_to :group
      belongs_to :user
    end

    # in the migration, the join table is defined like this:
    create_table user_groups, id: false do |t|
      t.integer :group_id, :user_id
      # other attributes if needed
    end
    ```
