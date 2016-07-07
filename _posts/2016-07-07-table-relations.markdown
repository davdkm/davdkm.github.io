---
layout: post
title:  "SQL Library Lab"
categories:
---
In this post, we'll be going over how to solve the [SQL Library Lab](https://learn.co/lessons/sql-library-lab) found on [Learn.co](http://learn.co). This lab covers writing SQL statements with complex relationships. We're going to be following the structure in the lab, so fork and clone the files. Also keep running the rspec tests included to gauge your progress.

####SECTION 1: SCHEMA.SQL
Build out the schema for our Fantasy Library database. Setting up the tables is pretty straightforward. We're just creating basic tables with columns and data types outlined by the lab and tests.
{% highlight sql %}
CREATE TABLE series (
  id INTEGER PRIMARY KEY,
    title TEXT,
    author_id INTEGER,
    subgenre_id INTEGER
);

... etc

CREATE TABLE books (
  id INTEGER PRIMARY KEY,
    title TEXT,
    year INTEGER,
    series_id INTEGER
);

CREATE TABLE characters (
  id INTEGER PRIMARY KEY,
    name TEXT,
    species TEXT,
    motto TEXT,
    series_id INTEGER,
    author_id INTEGER
);
{% endhighlight %}
The columns that have `_id` allow us to create associations with other tables. The `series_id` in the `characters` table, allows us to set the fantasy series that a character belongs to.

Books have many characters and characters can appear in many books. We'll need a join table in order to create that complex relationship. That means we need a table that stores the key of a character with the key of the books that character belongs to.
{% highlight sql %}
CREATE TABLE character_books (
  id INTEGER PRIMARY KEY,
    book_id INTEGER,
    character_id INTEGER
);
{% endhighlight %}

####SECTION 2: INSERT.SQL
Populate the database with data. You can make up your own, but the SQL statements are going to look something like the code below. The main thing to remember is the syntax for inserting into the table and the order of the columns and values.
{% highlight sql %}
INSERT INTO books (id, title, year, series_id) VALUES (1, "Game of Thrones", 1996, 1), (2, "A Clash of Kings", 1998, 1), (3, "A Storm of Swords", 2000, 1), (4, "First Book", 2002, 2), (5, "Second Book", 2003, 2), (6, "Third Book", 2005, 2);

INSERT INTO character_books (id, book_id, character_id) VALUES (1, 1, 1), (2, 1, 2), (3, 2, 2), (4, 3, 2), (5, 1, 3), (6, 2, 3), (7, 3, 3), (8, 1, 4);
{% endhighlight %}
The statement is telling our database to work on the `books` table and match the columns `(id, title, year, series_id)` with the values to be inserted in the same order `(1, "Game of Thrones", 1996, 1)`, and multiple sets of values are comma separated.

####SECTION 3: UPDATE.SQL
Update the species of the last character in the database to "Martian" by writing an update statement in `update.sql` file. Update statements are pretty straightforward as well, just remember the syntax!
{% highlight sql %}
UPDATE characters SET species = 'Martian' WHERE characters.id = 8;
{% endhighlight %}
What the above statement is selecting the row in the `characters` table, where the `characters.id` is equal to 8 (our last character), and then sets the `species` column value to `'Martian'`

####SECTION 4: QUERYING YOUR DATABASE
Check `spec/querying_spec.rb`, complete the tests by writing the appropriate queries to satisfy the queries in the `querying.rb` file. Note that for this section, the database will be seeded with external data so don't expect it to reflect the data you added above. The methods to be called are already named in the ruby file, we just need to write the sql statements that would return the desired values.

Let's break down one of the more complex statements. The method `select_series_title_with_most_human_characters` should be interesting to write. What we'll need is the series title name that has the most human characters across it's books.
{% highlight sql %}
  SELECT series.title FROM series JOIN books ON series.id = books.series_id JOIN character_books ON books.id = character_books.book_id JOIN characters ON character_books.character_id = characters.id WHERE characters.species = 'human' GROUP BY series.title ORDER BY COUNT(\*) DESC LIMIT 1;
{% endhighlight %}
Let's break down what's happening here. We don't have a direct relationship for us to get the information we need, so we're going to create some joins to bridge the gaps. We're going to have bridge our `series` -> `books` -> `character_books` -> `characters` where the `species` are of the character is human.

`SELECT series.title FROM series` is saying we want the `title` of the series from the `series` table.

`JOIN books ON series.id = books.series_id` here we're joining the `books` table with the `series` table where id of the series matches the `series_id` of entries in the `books` table.

`JOIN character_books ON books.id = character_books.book_id` is using the join table `character_books` to get the characters associated with books from our first join.

`JOIN characters ON character_books.character_id = characters.id` gets the characters in the books, where books are part of a series retrieved from former joins.

`WHERE characters.species = 'human'` is limiting our characters to only those who are human.

`GROUP BY series.title ORDER BY COUNT(*) DESC` is sorting all of the series returned by the number of human characters, with the highest number of characters at the top and descending to the lowest at the bottom. We use `GROUP BY` keyword because we are using an aggregate function `COUNT`.

`LIMIT 1` is saying we want to only have the first result, which is the `series.title` that has the most human characters due to our `ORDER BY` and `DESC` sorts.

Fill out the rest of the methods and make sure to pass all the tests! Happy coding!
