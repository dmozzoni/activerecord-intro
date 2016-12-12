# Active Record

## Screen Casts for code
- [Part 1](https://youtu.be/wbAPxnuIXck)
- [Part 2](https://youtu.be/J9w1CDXrOXc)
- [Part 3](https://youtu.be/NQNBnmpG5tE)
- [Part 4](https://youtu.be/EkD5LEyALvM)
- [Part 5](https://youtu.be/VOH26fAzc4o)

## Learning Objectives

- Define the term `ORM` and why we use it over a database language.
- Explain what Active Record is and what problems it solves.
- Explain convention over configuration and how it relates to Active Record
- Define a class that inherits from AR
- Utilize AR to perform CRUD actions on a database
- Differentiate between class and instance methods in Active Record objects/classes
- Utilize `has_many`, `belongs_to` to establish associations/relationships with AR
- Seed a database using AR

## Framing (5 / 5)

Think about what we have learned so far in this unit. We now have a way to persist data in a database. We've also learned about how OOP allows us to programmatically represent real things as objects in ruby. Which is AWESOME! But really databases just seems like data in this kind of cryptic place on our local computer.  We have to make super long SQL statements to do CRUD. It'd be really nice if we had some kind of way to interface between the database and our servers/applications in order to streamline this process. Enter ORM's.

### Information Dive (5 / 10)

For the next 5 minutes, research what ORM's are.

[ORM - wikipedia](https://en.wikipedia.org/wiki/Object-relational_mapping)

[AR Read 1.1 - 1.3](http://guides.rubyonrails.org/active_record_basics.html)

Try to answer these questions:

1. At a high level, what are ORM's and how might they be useful?
2. What is the importance of interfacing the server with the database?

## ORM's & Active Record (10 / 25)

- *Official* wikipedia definition. A programming technique for converting data between incompatible type systems in object-oriented programming languages.

> Thats sounds like a lot of 5 dollar words, but what does it really mean?

We need a way to encapsulate our databases into objects so that we can talk to our server. ORM's serve that purpose. Remember those tables we created in SQL? Well, its an object represented on our server now. That's what ORM's do.

More concretely ORM's:
- *'Map'* (translate) objects to rows in our DB (and vice versa)
- **Conventions**:
  - 1 table per Model/Class/Entity
  - table name is model name pluralized
  - each column is an attribute for that model
- Table associations are handled using foreign keys

It just so happens you will be learning one of the best ORM's on the market. It has some of the best documentation and best syntax (because ruby is awesome) the industry has to offer. This ORM is Active Record.

> Active Record is the M in MVC - the model - which is the layer of the system responsible for representing business data and logic. Active Record facilitates the creation and use of business objects whose data requires persistent storage to a database. It is an implementation of the Active Record pattern which itself is a description of an Object Relational Mapping system. - Taken from AR docs

## Active Record

For you to use Active Record to write ruby which manipulates data, we need to be able to talk about the **models** of our data.

<details>

<summary>What is Domain modeling and why do we do it?</summary>

```
Domain modeling is the act of describing entities
and their relationships in an application's data.
This method is useful for deciding data what needs to be persisted.
```

</details>

<br>
When we we data model, we tend to be talking about the **Nouns** in our
application. These are the names of the *tables* in our database and the names
of our Ruby *classes*.

Likewise when we write queries, we use **Verbs** to describe the specific data
we want.

Essentially, in order to store and retrieve information, a lot of what we do
today will look like some form of the equation:

 > **Noun** + **Verb** = **Data**

With the help of Active Record, we can begin to write programs that follow this
simple pattern to manipulate data.

### Convention Over configuration (ST-WG - 10 / 35)

Before we get started with code, I want to highlight a reoccurring theme with Active Record and Rails in general. You'll often here us say Convention over Configuration.

<details>
<summary>**Question:**  Without getting into the specifics of AR, what do you think we mean by convention over configuration?</summary>
<br>

```
Basically Active Record and Rails, and other frameworks have a whole bunch of conventions that they follow so that you do not have to mess with different configuration details later. These conventions exist because developers agree on best practices, and therefore allows us to spend less time trying to configure, when there all ready is an accepted way to do things. Some of the common ones we will encounter are naming conventions such as: plural vs single, capital or lower, camel or snake.
```
</details>


**BOARD:** Throughout this lesson, I will write on the board Active Record's conventions so we can list them as we go.

In a nutshell, if you don't follow the conventions, you're going to have a bad time.

Alright! Let's get started with some code!

### Setup SQL - Tunr (You Do - 10 / 45)

> [Tunr Deployed link](https://wdi-dc-tunr.herokuapp.com/artists)

Make sure to review our domain model for tunr

![Tunr Erd](tunr-erd.png)

We want to be able to do CRUD for these models with Active Record. We'll be going into greater detail about how we are going to use Active Record as an interface between our server and our database, but to start, the first thing that we want to do is create/setup a database.

```bash
$ git clone git@github.com:ga-wdi-pvd/tunr-active-record.git
$ cd tunr-active-record
$ bundle install
$ createdb tunr_db
$ psql -d tunr_db < db/schema.sql
$ psql -d tunr_db < db/seeds.sql
$ atom .
```

You'll know you did this right if:

Run your program(`$ ruby app.rb`) and then when you enter `pry` run this line of code:

you should get something like this(it won't be the same letters and numbers)
```bash
pry(main)> Artist.all
=> #<Artist::ActiveRecord_Relation:0x3fc4911af384>
```

**STOP**

### Code Review / Walkthrough - Tunr (I Do 10 / 45)

Let's do a quick walkthrough of our code base so far...

> The `app.rb` file is our main application file. This is where a lot of the main program logic will live.

> The `Gemfile` contains all the dependencies for our program.

> The `models/artist.rb` file will contain the class definition for the Artist class that will represent the artists table in SQL

> **Note**: by convention we always name our model file names singular

#### The `Gemfile` - an aside

A `Gemfile` is a file we create which is used for describing gem dependencies for Ruby programs. Allows for easier collaboration of apps. TLDR: Take someone's code, use it in your program.

We stand on the shoulders of giants. Oh there's code for that? yoink.

In the `Gemfile`:

```ruby
source 'https://rubygems.org'

gem "pg"  # this gem allows ruby to talk to postgres
gem "activerecord"  # this gem provides a connection between your ruby classes to relational database tables
gem "pry"  # this gem allows access to REPL
```

> Without these gems, we would be unable to talk to our SQL database.

#### Defining our Models

In the `models/artist.rb` file, we need to define our `artist` model:

```ruby
class Artist < ActiveRecord::Base
  # AR classes are singular and capitalized by convention
end
```

> In this ruby file, we create a class of Artist that inherits from `ActiveRecord::Base`. Essentially, when we inherit from `ActiveRecord::Base`, it gives this class a whole bunch of functionality that maps the ruby `Artist` class to the `artists` table in postgres.

#### Connecting to Postgres

Now, in order to connect our application to our database we have `db/connection.rb` file:

```ruby
ActiveRecord::Base.establish_connection(
  :adapter => "postgresql",
  :database => "tunr_db"
)
```

#### Wiring the application together

Finally, we have our `app.rb` file(which in this case is just dropping us into pry):

```ruby
require "bundler/setup" # require all the gems we'll be using for this app from the Gemfile. Obviates the need for `bundle exec`
require "pg" # postgres db library
require "active_record" # the ORM
require "pry" # for debugging

require_relative "db/connection" # require the db connection file that connects us to PSQL, prior to loading models
require_relative "models/artist" # require the Artist class definition that we defined in the models/artist.rb file


# This will put us into a state of the pry REPL, in which we've established a connection
# with the tunr_db database and have defined the Artist Class as an
# ActiveRecord::Base class.
binding.pry

puts "end of application"
```

> note the difference between `require` and `require_relative`. With `require` we are getting gems and `require_relative` we are getting files relative to the location of the file we wrote `require_relative` in

**STOP**

## Break (10 / 60)

### Methods - Tunr (I Do 15 / 75)

Great! We've got everything done that we need to get setup with single model CRUD in our application. Let's run it in the terminal:

```bash
$ ruby app.rb
```

When we run this app, we can see that it drops us into pry. Let's write some code in pry to update our database... **IN REALTIME!!!**

**Board:** I want to come up with an ongoing list of *class* methods vs *instance* methods that we can add to as we use them

Let's create an instance of the `Artist` object on the ruby side:

> **Note** the syntax for creating a new instance.

```ruby
kanye = Artist.new(name: "Kanye West", nationality: "American")
```

To save our instance to the database we use `.save`:

```ruby
kanye.save
```

If we want to initialize an instance of an object AND save it to the database we use `.create`:

```ruby
elvis = Artist.create(name: "Elvis Presley", nationality: "American")
```

<!-- SQL for create  -->
<details>
<summary>This would roughly translate to `SQL` code</summary>
>
```sql
INSERT INTO artists (name, nationality) VALUES ('Elvis Presley', 'American');
```

</details>

<br>
**(ST-WG)** Why is there a distinction between when it's saved in one command versus two?

One really handy feature we get from an Active Record inherited class is that all of the attribute columns of our model are now `attr_accessor`'s as well. So we can do things like:

```ruby
kanye.name
# gets the name of kanye
kanye.name = "Yeezus"
# sets name to "Yeezus"
kanye.save
# saves the model to the database
```

> **Note:** Should be noted that when we set the attribute using a setter, it doesn't automatically save to the database, its not until we call `.save` on the object that it saves to the database

To get all of the songs we use `.all`:

```ruby
Artist.all
```

We can also find a song by its ID using `.find`:

```ruby
Artist.find(1)
```

<!-- SQL for find  -->
<details>
<summary>This would roughly translate to `SQL` code</summary>
>
```sql
SELECT * FROM artists WHERE id = 1 LIMIT 1;
```

</details>

<br>
Additionally we can also find a song by an attribute using `.find_by`:

```ruby
Artist.find_by(name: "Elvis Presley")
```

> Note that find_by only returns the first object that meets the requirements of the arguments

If you want all songs that meet a certain query then we use `.where`:

```ruby
Artist.where(nationality: "American")
```

We can also update attributes and save at the same time using the `.update` method:

```ruby
elvis = Artist.find_by(name: "Elvis Presley")
elvis.update(nationality: "Canadian")
```

> If we want to update the attribute and not save we would have to use something from this [post](http://www.davidverhasselt.com/set-attributes-in-activerecord/)

Finally we can also just destroy/delete rows in our database with the `.destroy` method:

```ruby
kanye = Artist.find_by(name: "Yeezus")
kanye.destroy
# goodbye kanye you're gone forever
```

> This is exciting stuff by the way, imagine, while we do these things, that our artists model is instead a post on Facebook, or a comment on Facebook. So the next time you comment on someone's Facebook page you have an idea now of whats happening on the database layer. Maybe not the whole picture, but you have an idea. We're going to build on that idea in the coming week and half, and thats really exciting.

### Methods - Tunr (You Do (In Pry!) - 15 / 90)

[Part 1.3 - Use Your Artist Model](https://github.com/ga-wdi-pvd/tunr-active-record#part-13---use-your-artist-model)

## Associations

### Reframing (10 / 100)

<details>
<summary>**Q**. We have a lot of choice when it comes to databases, why are we using SQL?</summary>

<br>

```
We use SQL because it is a relational database. But what does that really mean? Basically we want the ability to associate models in our domain. That can come in a variety of ways in a relational database, but at the heart of it is essentially this: One model has many other instances of another model. And that other model belongs to the original.
```

</details>

<br>
Let's take a look at an example in the wild.

With the application Facebook, there are many users. Each of these users have several models associated with them. Let's look at one more closely. We'll be looking at posts(status updates)

The first model we'll be looking at is Users. The second is Posts.

A User has many Posts
And every Post belongs to a certain user.

> Note the plurality of the nouns used in these two sentences

When we start organizing our objects in this manner and program these associations, it becomes much easier to query our database for what we need. IE. If we were on Adrian's Facebook page, it wouldn't make sense for us to query EVERY post in facebook and then do `.where(facebook_user: "Adrian")` That would get really expensive. Instead we can do something like, `adrian.posts` and then BAM, we got all of adrian's posts.

> Note that this is just speaking at a high level and not modeling the exact syntax, however it's incredibly close to how the code actually would look if it were modeled in AR.

Let's see what some of this stuff looks like in code. We're going to be adding an artist model to our program.

### Associations in Schema - Tunr (I Do - 5 / 105)

**NOTE:** In this section, we are reviewing our schema and how it reflects associations for our domain. We are NOT updating the schema file.

Looking back at our schema, in `schema.sql`:

```sql
DROP TABLE IF EXISTS songs;
DROP TABLE IF EXISTS artists;

CREATE TABLE artists(
  id SERIAL PRIMARY KEY,
  name TEXT,
  photo_url TEXT,
  nationality TEXT
);

CREATE TABLE songs(
  id SERIAL PRIMARY KEY,
  title TEXT,
  album TEXT,
  preview_url TEXT,
  artist_id INT
);
```

Make note of the foreign key in `songs`

### Updating Class Definitions - Tunr (I Do - 5 / 110)

Next we need to update the models to reflect the relationships in our application.

```ruby
class Artist < ActiveRecord::Base
  has_many :songs
end
```

Now In our `models/song.rb` we have to reflect the association:

```ruby
class Song < ActiveRecord::Base
  belongs_to :artist
end
```
> note the plurality of `songs` and singularity of `artist`.  

We also need to include the `models/song.rb` file into our `app.rb` so in `app.rb` we need to add

```ruby
require_relative "models/song"
```

### Updating Class Definitions - Tunr (You Do - 5 / 115)

[Part 1.2 - Create Your Song Model / Setup Associations](https://github.com/ga-wdi-pvd/tunr-active-record#part-12---create-your-song-model--setup-associations)

[solution code](https://github.com/ga-wdi-pvd/tunr-active-record/archive/v1.2.zip)

### Association Helper Methods - Tunr (I Do - 10 / 125)

So we added some code, but we can't yet see the functionality it gives us.

Basically when we added those two lines of code `has_many :songs` `belongs_to :artist` we created some helper methods that allow us to query the database more effectively.

Lets create some objects in our `app.rb` so we can see what were talking about:

```ruby
Song.destroy_all
Artist.destroy_all

ratatat = Artist.create(name: "Ratatat", nationality: "American")
beatles = Artist.create(name: "The Beatles", nationality: "British")

# NOTE: you can pass an array to create for bulk creation
Song.create([
    {title: "Wildcat", album: "Classics", artist: ratatat },
    {title: "Loud Pipes", album: "Classics", artist: ratatat },
    {title: "Neckbrace", album: "LP4", artist: ratatat },
    {title: "Twist and Shout", album: "Please Please Me", artist: beatles},
    {title: "Hello, Goodbye", album: "Magical Mystery Tour", artist: beatles},
    {title: "Revolution", album: "The Beatles", artist: beatles},
  ])
```

> ST-WG Why do we need to use `.destroy_all`?

Now that we have this association, we can now easily query the database for the relevant records.

If we want to get all of The Beatles' songs or set The Beatles' songs, we can now write this code:

```ruby
beatles = Artist.find_by(name: "The Beatles")
beatles.songs
# will return all songs that belong to The Beatles, this is .songs being used as a getter method

beatles.songs = [Song.first, Song.last]
# this is .songs being used as a setter method
```
> note that when songs is being used as a setter method above, it actually changes the artist_id column for those songs to match The Beatles' primary ID. Any song that previously was assigned to The Beatles and not reassigned in the setter will now have an artist_id of nil

Alternatively if I wanted to get a song's artist I could write this code:

```ruby
loud_pipes = Song.find_by(title: "Loud Pipes")
loud_pipes.artist
# will return loud_pipes' artist, this is .artist being used as a getter method

beatles = Artist.last
loud_pipes.artist = beatles
loud_pipes.save
# this .artist being used as a setter method, and now loud_pipes's artist is Adrian
```

We can also create new songs under a certain artist by doing the following:

```ruby
beatles.songs.create(title: "Hey Jude", album: "Beatles Chillout (Vol. 1)")
# this will create a new song that belongs to the Artist object named beatles.
```
> **Note** that we did not pass in an artist id above. Active Record is smart and does that for us.

### Association Helper Methods - Tunr (You Do - 10 / 135)

[Part 1.5 - Use Your Model Assocations](https://github.com/ga-wdi-pvd/tunr-active-record#part-15---use-your-model-associations)

### Seeding a Database - Tunr (I Do - 10 / 145)
Seeding a database is not all that different from the things we've been doing today. What's the purpose of seed data? **(ST-WG)**

We want some sort of data in our database so that we can test our applications. Let's create a seed file in the terminal: `$ touch db/seeds.rb`

In our `db/seeds.rb` file let's put the following:

```ruby
require "bundler/setup" # require all the gems we'll be using for this app from the Gemfile. Obviates the need for `bundle exec`

require "pg"
require "active_record"
require "pry"

require_relative "../models/song"
require_relative "../models/artist"

require_relative "../db/connection.rb"


Artist.destroy_all
Song.destroy_all
# destroys existing data in database

chili_peppers = Artist.create(name: "Red Hot Chili Peppers", nationality: "American")
led = Artist.create(name: "Led Zeppelin", nationality: "British")

chili_peppers.songs.create([
    {title: "Can't Stop", album: "By The Way" },
    {title: "Scar Tissue", album: "Californication" },
    {title: "Californication", album: "Californication" },
    {title: "Dani California", album: "Stadium Arcadium" },
    {title: "Dark Necessities", album: "The Getaway"}
  ])

led.songs.create([
    {title: "Whola Lotta Love", album: "Led Zeppelin II" },
    {title: "Stairway to Heaven", album: "Led Zeppelin IV" },
    {title: "Kashmir", album: "Physical Graffiti" },
    {title: "Black Dog", album: "Led Zeppelin IV" },
    {title: "All My Love", album: "In Through the Out Door"}
    ])
```

Once we get rid of this duplicate CRUD code in `app.rb` we can just run this seed file once and know our data is good.

Now when we run our application with `ruby app.rb`, we enter into Pry with all our data loaded.

## Closing (5 / 150)

Who misses writing SQL queries by hand? Exactly. Active Record is extremely powerful and helpful, and allows us to easily interface with the business models for our applications.

Review Learning Objectives


### Resources
- [Active Record Basics](http://guides.rubyonrails.org/active_record_basics.html)
- [Active Record Query Interface](http://guides.rubyonrails.org/active_record_querying.html)
- [GA WD ActiveRecord Lesson](https://github.com/ga-wdi-lessons/activerecord-intro)

### Appendix

#### Instance vs Class Methods
| method              | instance | class    |
|---------------------|:--------:|:--------:|
| .new                |          |     x    |
| .save               |     x    |          |
| .create             |          |     x    |
| attribute accessors |     x    |          |
| .all                |          |     x    |
| .find               |          |     x    |
| .find_by            |          |     x    |
| .where              |          |     x    |
| .update             |     x    |          |
| .destroy            |     x    |          |
| .destroy_all        |          |     x    |

#### Conventions in AR
|description      | Rule    |
|-----------------|---------|
|table names      |snake case and plural|
|model file names |snake case and singular|
|model definition |Camel case and singular|
|argument for has_many| snake case and plural|
|argument for belongs_to| snake case and singular|
|more to follow....|||
