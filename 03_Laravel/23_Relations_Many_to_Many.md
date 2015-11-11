## Relationships: Many to Many
In defining the *One to Many* relationship for books and authors, we operated under the constraint that each book only had a single author.

Certain features, though, won't fit into this design. For example, a tagging feature.

We could add a `tag_id` field to the books table, to associate a tag (novel, fiction, history, etc.) with a book, but that would limit us to a single tag per book, which is not ideal.

To accomplish what we want, we need to implement a *Many to Many* relationship so that *many* books can have *many* tags (and vice versa)

<img src='http://making-the-internet.s3.amazonaws.com/laravel-many-to-many@2x.png' style='max-width:657px; width:100%' alt='Many to Many'>

## Pivot tables
*Many to Many* relationships require an additional table, called a pivot table to keep track of the many connections.

+ A pivot table links two tables together using foreign keys.
+ Also known as: &ldquo;lookup table&rdquo;, &ldquo;join table&rdquo;.
+ A primary key is not required on a pivot table.
+ Naming convention:
    + Use the singular version of the name of the two tables you're joining, separated with an underscore, in alphabetical order.
    + Ex: If you're joining the `books` table with the `tags` table, the resulting pivot table name would be `book_tag`.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-many-to-many-diagram@2x.png' style='max-width:931px; width:100%' alt='Many to Many'>

## Implementation
Notes coming in Lecture 12.




## Reference
+ [Docs: Relationships: Many to Many](http://laravel.com/docs/eloquent-relationships#many-to-many)
