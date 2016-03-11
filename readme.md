# Rails Models and Migrations

## Learning Objectives

* Create a new rails application with postgres as the default.
* Use `rake` to create, edit, and update, and seed the db.
* Use Rails generators to create migrations.
* Use Rails console to inspect and manipulate models.
* Use Rails migrations to create tables and modify columns.
* Undo a migration with `rake db:rollback`.
* Create migrations that associate one model with another.
* Identify the impacts of editing existing migrations.
* Use `timestamps` to timestamp crud actions.
* Use shorthand syntax to create migrations from the command line.

---

## Framing

The files we’ll be working with in this lesson are highlighted in blue below:

![](https://dl.dropboxusercontent.com/s/2zho5ekuhxpp5lx/Screenshot%202015-07-26%2010.26.28.png?dl=0)

## We Do: Set Up a Rails Application (10 minutes / 0:10)

Run the below command in Terminal to create a new Rails application...  

```bash
$ rails new tunr -d postgresql
```
> `-d postgresql` is important! It tells our application that we will be using postgres as our database. Otherwise, Rails uses SQLite as a database by default.  

This creates 92 new files in a directory called `tunr`. We can easily keep track of what files have changed if we use version control...  

```bash
$ cd tunr
$ git init
$ git add .
$ git commit -m "initial commit"
```
> How can you tell if 92 new files were created? Try running `find . -type f | wc -l` in the Terminal.  

Let's explore some of the files that were created...  

### `tunr/config/database.yml`

Specifies the database connection options. By default, rails uses the development options detailed in this file.  

> `.yml` stands for "YAML Ain't Markup Language". It's often used as a configuration file in Rails (and elsewhere).  

To create the database, run the following in the Terminal...

```bash
$ rake db:create
```

How can we tell this actually created the database? Run the following in the terminal. If you see a `psql` prompt then you're good to go...

```bash
$ rails dbconsole
```

On top of that, we can see what environment we're currently in by entering the Rails Console and viewing the output of `ENV["RAILS_ENV"]`...

```bash
$ rails c
```
```rb
ENV["RAILS_ENV"]
```

> You can see that the output is `development`. ENV["RAILS_ENV"] is a way we get/set our environment and allows for different types of configurations. The other two types of environment are `test` (a developer-free environment where a QA team can test the application) and `production` (the live application).  

#### What is Rake?

**Rake** is a ruby implementation of Make. **Make** is a popular tool that allows developers to put repetitive command line tasks into a single file and run several tasks at the same time
with a single command.  

Rails uses rake to...
* Create the database (`rake db:create`)
* Create / Edit database tables (`rake db:migrate`)
* Drop the database (`rake db:drop`)
* Seed the database (`rake db:seed`)

[Learn more about Rake here](https://github.com/ruby/rake#description).  

## You Do: Models (10 minutes / 0:20)

**[5 minutes]** Create Artist and Song models the same way you did in last week's Active Record class. They should...
* Have the appropriate file name.
* Have the appropriate model name.
* Inherit from Active Record.
* Indicate the appropriate relationships. This app will follow the below ERD...

![tunr erd](http://i.imgur.com/JzWriwJ.png)

You will be placing these files into the `app/models` directory of your Rails application.  

How do you know it worked? If we hop into the rails console and type in Artist...  

```bash
$ rails c
```

You should then see something like...  

```text
Loading development environment (Rails 4.2.4)
2.2.3 :001> Artist
 => Artist(Table doesn't exist)
```
> If we type in any other capitalized word it will throw an error `uninitialized Constant`  
>  
> The console may respond by telling you to enter `Artist.connection` or `Song.connection`. Go ahead and type that in, then continue to test.  

## We Do: Migrations (15 minutes / 0:35)

At the end of the last exercise, we got an error in the Rails Console telling us that a table did not yet exist. Let's take care of that with migrations...  

Think of a set of migrations as a recipe for a database schema, with each migration representing a step in that recipe.
* Our schema file defines the structure of our application's database.
* A schema starts off with nothing in it, and each migration modifies it to add/remove tables, columns or entries.
* Active Record knows how to update your schema along this timeline, bringing it from whatever point it is in the history to the latest version.
* Active Record will also update your `db/schema.rb` file to match the up-to-date structure of your database.  

> An extensive description of Rails migrations can be found via the Rails docs [here](http://edgeguides.rubyonrails.org/active_record_migrations.html).  

In the terminal...

```bash
$ rails g migration create_artists
```

> `rails g` is short for `rails generate`.  
>
> Note the title we gave our migration file. Here we've indicated that we're creating a table called `artists`. The two words are separated by an underscore. This is a rails convention.  

This creates a migration file `db/migrate/20150726145027_create_artists.rb`. Let's see what it looks like...  

The numbers at the beginning of a migration file are a timestamp.
* Rails uses this timestamp to determine which migration should be run and in what order, so if you're copying a migration from another application or generate a file yourself, be aware of its position in the order.  

> [Source](http://edgeguides.rubyonrails.org/active_record_migrations.html#creating-a-migration)  

```rb
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
    end
  end
end
```
The above file, as is, defines a new table called `artists`.
* Now we need to define what columns are going to appear in this table. We'll do this inside of the `create_table` method...

```rb
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
      t.string :name
      t.string :photo_url
      t.string :nationality
    end
  end
end
```

How are we defining columns in `create_table`?
* We use `t` to indicate the table we are creating inside of the code block.
* We then use methods like `.string` or `.integer` to represent the data types of the columns we are creating.
* These methods take in a symbol argument (e.g., `:photo_url`) that represents the name of the column.  

> For a full list of SQL data types, click [here](http://www.w3schools.com/sql/sql_datatypes_general.asp). For more on using the `change` method, click [here](http://edgeguides.rubyonrails.org/active_record_migrations.html#using-the-change-method)  

You can create the `artists` table and run this migration by entering the following into the terminal...

```bash
$ rake db:migrate
```

After running this, a file called `db/schema.rb` is generated. What are it's contents?

> You should NEVER have to update the `schema.rb`. Running `rake db:migrate` will update the schema for you using migration files.  

## You Do: View `db/schema.rb` (5 minutes / 0:40)

**[1 minute]** Read through this new file.

## You Do: Use Rails Console (5 minutes / 0:45)

**[2 minutes]** Enter the Rails Console and create two Artists.
* You can do this using the Active Record methods you learned in earlier classes.  

> Remember, you can enter the Rails Console by typing `rails c` into the Terminal.  

## You Do: Songs Migration (10 minutes / 0:55)

**[5 minutes]** Create a migration file for a new `songs` table. Songs should have columns for `title` `album` and `preview_url`, all of which are strings.  

To associate one model with another in a migration file, you can include one of the following methods in the `create_table` method. They all do the same thing assuming you have set up your associations properly in your models...  
* `t.belongs_to :artist`
* `t.references :artist`
* `t.integer :artist_id`

## Break (10 minutes / 1:05)

## I Do: Passing Arguments Into `rails g migration` (10 minutes / 1:15)

We can use the Terminal to create an entire migration file -- column names and data types included -- in one command.

```
$ rails g migration create_songs title:string album:string preview_url:string artist:references
```
> There are lots of little short cuts that `g` or `generate` can give you. [Model Generators](http://edgeguides.rubyonrails.org/active_record_migrations.html#model-generators) are just one example.  

The above command creates the following file...  

```rb
# db/migrate/20150726150324_create_songs.rb

class CreateSongs < ActiveRecord::Migration
  def change
    create_table :songs do |t|
      t.string :title
      t.string :album
      t.string :preview_url
      t.references :artist, index: true, foreign_key: true
    end
  end
end
```

What do we see?
* `references` is used when the column represents a foreign key for another table. It appears
wherever `belongs_to` appears in the model definition.
* `t.references :artist` is equivalent to `t.integer :artist_id`  

> What is [`index: true`?](http://rny.io/rails/postgresql/2013/08/20/postgresql-indexing-in-rails.html)  

## You Do: Rails Console (5 minutes / 1:20)

**[2 minutes]** Use Rails Console to create at least three songs that are associated with the previous two artists.  
> Make a mental note of the commands you use to do this.  

## We Do: Seeds (10 minutes / 1:30)

Seeds allow us to quickly create dummy data. Why would we do that?
* In order to test out the interfaces and functionalities we build out, we need some content/data to manipulate in order to see how it looks and feels on our application.

**[3 minutes]** Let's update our seeds (`db/seeds.rb`) file now.
* Write out the same Active Record commands you used in the Rails Console to create three Artists and three Songs.
* Make sure to include the following two lines of code at the very top of your `db/seeds.rb`...

```rb
Artist.destroy_all
Song.destroy_all
```

To run `db/seeds.rb` all we need to do is run `$ rake db:seed` in the Terminal.  

After running the seeds, go into the `rails console` and play with the objects you created.  

> Here's some [sample seed data](https://gist.github.com/amaseda/54223fc96f914500ff15) if you need it.  

## We Do: Adding Timestamps (10 minutes / 1:40)

Rails can automatically timestamp when objects are created and updated. Let’s create a new migration to add timestamp columns to the artists table.

```bash
$ rails g migration add_timestamps_to_artists
```

In that migration place the following content:

```rb
# 20150726150946_add_timestamps_to_artists.rb
class AddTimestampsToArtists < ActiveRecord::Migration
  def change
    add_column :artists, :created_at, :datetime
    add_column :artists, :updated_at, :datetime
  end
end
```

`add_column`. **What's that?**
* It's one of many custom methods that can be used in the `change` method in migrations.
* It takes 3 arguments.
  1. The table you want to add the column to.  
  2. The column you'd like to add to that table.  
  3. The column's data type.  

## You Do: Add Timestamps to the Songs Table (5 minutes / 1:45)

**[3 minutes]** Apply what we just learned to the `songs` table.

## Break (10 minutes / 1:55)

## I Do: Undoing Things

### The Wrong Way (10 minutes / 2:05)

Lets assume you have this migration...

```rb
class CreateArtists < ActiveRecord::Migration
  def change
    create_table :artists do |t|
      t.string :name
      t.string :poto_url
      t.string :nationality
    end
  end
end
```

We really messed this migration up by misspelling `:photo_url`. But oh no, we've already run our migrations! What do we do?  

We could directly modify this migration file and run `rake db:migrate`.
* But if we then look at `db/schema.rb`, however, we see that nothing has changed. It still says `poto_url`.

What about resetting the entire database via the Terminal?  

```bash
$ rake db:drop
$ rake db:create
$ rake db:migrate
```

If we run these commands, we're dropping our entire database and creating a brand new one.
* Why is this potentially super dangerous?

Another (wrong) way we can do this is by using `rake db:rollback`.
* To undo a single migration, run `rake db:rollback` in the Terminal.
* Be careful though -- this might destroy data! Whatever columns or tables that were created by that migration will now be gone.  

> Running `rake db:rollback` will only undo the migration with the most recent timestamp. Every subsequent rollback will undo the most recent timestamped migration that hasn't been undone yet.  

It is considered **OK** to rollback migrations, edit them and re-migrate in a development environment, but **NOT** in a production environment.
* If you are working on an application with other developers, avoid using `rake db:rollback` after code has been pushed.  

A common theme here is that if you're not working alone in development, destroying data is bad.
* It's sometimes not obvious what actions you take may or may not destroy user data. Know that `rake db:rollback` and `rake db:drop` have the potential of doing it.

### You Do: Create a Migration and Roll It Back (5 minutes / 2:10)

* Create a migration that adds a column to artist called `genre` that has a string as the data type
* Run the migrations by running `$ rake db:migrate`
* Inspect `db/schema.rb`. Look at its contents.
* Now run rake db:rollback
* Inspect `db/schema.rb` again and note that `genre` is not longer a column in the table.
* Spend two minutes experimenting for yourself how `$ rake db:rollback` works


### I Do: The Right Way (10 minutes / 2:20)

Let's assume we did mess up our initial migration like the code above with the misspelled `poto_url`. Instead of the methods listed above, the right way is to create an additional migration that changes the name of the column in our table.  

In the terminal...  

```bash
$ rails g migration change_column_in_artists
```

This will generate a new migration. Lets fill its contents now in `db/migrate/20151105201357_change_column_in_artists.rb`...

```rb
class ChangeColumnInArtists < ActiveRecord::Migration
  def change
    rename_column :artists, :poto_url, :photo_url
  end
end
```

`rename_column` is another method we can use inside `change`.
* It takes 3 arguments as well...
  1. The table for which you want to rename a column.  
  2. The current name of the column you'd like to change.  
  3. The new name of the column.  

Now if we run our migrations, we can see that the artists table in `db/schema.rb` has the proper `:photo_url` column. Although this process takes a bit more legwork, we don't run the risk of destroying databases or undoing migrations.

## Closing/Questions (10 minutes / 2:30)

## Homework

Complete the Models + Migrations portion of [Scribble](https://github.com/ga-wdi-exercises/scribble).

## Resources

* [List of Rails Commands](https://gist.github.com/jshawl/ce1de309ef993ec808d9).

## Sample Quiz Questions

* What are some common `rake` commons you will be using when developing a Rails application?
* How do we indicate a one-to-many relationship in a migration file?
* Why would we use a seed file to populate our database?
* What is the proper way of modifying the effects of an existing migration?
* What does `rake db:rollback` do?
