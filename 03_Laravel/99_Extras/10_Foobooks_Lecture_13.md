The following is a very rough outline of the modifications I'll make to foobooks during Lecture 13.

This should not be considered a stand-alone document; for full details please refer to the lecture video and the foobooks code source.


## Custom Model methods

Last week in the `getEdit()`` method in `BookController.php`, we used this code to construct an array of authors to be used for the authors drop down:

```php
$authors = \App\Author::orderby('last_name','ASC')->get();

$authors_for_dropdown = [];
foreach($authors as $author) {
    $authors_for_dropdown[$author->id] = $author->last_name.', '.$author->first_name;
}
```

This allowed us to create a dropdown of authors on the *Edit Book* page.

We need the same kind of dropdown on the *Add a Book* page.

Rather than repeat the above code in the `getAdd()` method, let's extract the $authors_for_dropdown code into its own method in the Authors model:

```php
# Author.php
public function getAuthorsForDropdown() {

    $authors = $this->orderby('last_name','ASC')->get();

    $authors_for_dropdown = [];
    foreach($authors as $author) {
        $authors_for_dropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    return $authors_for_dropdown;

}
```

Then update `getEdit()`:

```php
$authorModel = new \App\Author();
$authors_for_dropdown = $authorModel->getAuthorsForDropdown();
```

Now let's use this method in `getCreate()``:

```php
public function getCreate() {

    $authorModel = new \App\Author();
    $authors_for_dropdown = $authorModel->getAuthorsForDropdown();

    return view('books.create')->with('authors_for_dropdown',$authors_for_dropdown);
}
```

Then, `edit.blade.php` can use this data to create an authors dropdown:

```html
<div class='form-group'>
    <label for='author'>* Author:</label>
    <select name='author' id='author'>
        @foreach($authors_for_dropdown as $author_id => $author_name)
            <option value='{{ $author_id }}'> {{ $author_name }} </option>
        @endforeach
    </select>
</div>
```

Then update `postCreate()` because it will now get the author from a dropdown instead of a text input.

Remove the validation:
```php
'author' => 'required|min:5',
```

And set the `author_id` using the data from the dropdown in `$request`:

```php
$book->author_id = $request->author;
```

Test and make sure that adding a new book with the author dropdown works.




## Improving the flow for guests that try to view auth-only pages
Authenticate.php
Line 41: \Session::flash('flash_message','You have to be logged in to access /'.$request->path());




## Making data available to all views
Goal: Make `user` information available on all views.

Done with a View Composers: <http://laravel.com/docs/5.1/views#view-composers>

First, make the composer:
```bash
$ php artisan make:provider ComposerServiceProvider
```

Then update the resulting composer:
```php
# /app/Providers/ComposerServiceProvider.php
public function boot()
{
    // Make the variable "user" available to all views
    \View::composer('*', function($view) {
        $view->with('user', \Auth::user());
    });
}
```

Then in any view you can use the `$user`, for example:
```html
<li><a href='/logout'>Log out {{ $user->name }}</a></li>
```




## Integrating your existing data and users/auth

Couple different approaches:

1. One to Many: Every book is connected to a single user
2. Many to Many: Individual books aren't associated with any one user. Instead users can "Add book to your library", powered by a many to many relationship between books and users.

Lets go with #1.


### Connect books and users
To do this, we'll need a migration to update our `books` table to have the `user_id` fk. This is the same thing we did when connecting authors and books.

Create the migration:
```bash
php artisan make:migration connect_books_and_users
```

Fill in the migration:
```php
public function up()
{
    Schema::table('books', function (Blueprint $table) {

        # Add a new INT field called `user_id` that has to be unsigned (i.e. positive)
        $table->integer('user_id')->unsigned();

        # This field `user_id` is a foreign key that connects to the `id` field in the `authors` table
        $table->foreign('user_id')->references('id')->on('users');

    });
}

public function down()
{
    Schema::table('books', function (Blueprint $table) {

        # ref: http://laravel.com/docs/5.1/migrations#dropping-indexes
        $table->dropForeign('books_user_id_foreign');

        $table->dropColumn('user_id');
    });
}
```

### Books and authors Seeders
Because we've created a FK between `books` and `authors` our Seeding order needs to be updated; currently it looks like this:

```php
$this->call(AuthorsTableSeeder::class);
$this->call(BooksTableSeeder::class);
$this->call(TagsTableSeeder::class);
$this->call(BookTagTableSeeder::class);
$this->call(UsersTableSeeder::class);
```

We need to update it so that the `UsersTableSeeder` is invoked *before* the `BooksTableSeeder` (because the BooksTableSeeder will have a FK connecting to the `users` table).

New order:

```php
$this->call(AuthorsTableSeeder::class);
$this->call(UsersTableSeeder::class);
$this->call(BooksTableSeeder::class);
$this->call(TagsTableSeeder::class);
$this->call(BookTagTableSeeder::class);
```

Finally, update the BooksTableSeeder so that each book is associated with an author.

```php
$author_id = \App\Author::where('last_name','=','Fitzgerald')->pluck('id');
DB::table('books')->insert([
'created_at' => Carbon\Carbon::now()->toDateTimeString(),
'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
'title' => 'The Great Gatsby',
'author_id' => $author_id,
'user_id' => 1, # <--- NEW LINE
'published' => 1925,
'cover' => 'http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG',
'purchase_link' => 'http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565',
]);
```

Make the above change for all three books.

### Update models
Next, update the `Book` and `User` model so they're aware of this new relationship.

Add this to `Book.php`:
```php
public function user() {
    return $this->belongsTo('\App\User');
}
```

Add this to `User.php`:
```php
public function book() {
    return $this->hasMany('\App\Book');
}
```

Setup complete! Now lets make it so that when you're logged in you only see *your* books.


### Only see *your* books
Update `BookController.php` `getIndex()` so that the query includes a user_id filter:

```php
public function getIndex(Request $request) {

    // Get all the books "owned" by the current logged in users
    // Sort in descending order by id
    $books = \App\Book::where('user_id','=',\Auth::id())->orderBy('id','DESC')->get();

    return view('books.index')->with('books',$books);
}
```

Test it out.
How many books does Jill see?
Update the `books` table to reduce the number of books that Jill sees.

How many books does Jamal see? Should be none by default.

Update `books/index.blade.php` to account for this &ldquo;blank slate&rdquo; case:

```php
@if(sizeof($books) == 0)
    You have not added any books.
@else
    @foreach($books as $book)
        <div>
            <h2>{{ $book->title }}</h2>
            <a href='/books/edit/{{$book->id}}'>Edit</a><br>
            <img src='{{ $book->cover }}'>
        </div>
    @endforeach
@endif
```

### Associating new books with the logged in user
Update `BookController.php` `postCreate()`:

```php
// Code here to enter book into the database
$book = new \App\Book();
$book->title = $request->title;
$book->author_id = $request->author;
$book->user_id = \Auth::id(); # <--- NEW LINE
$book->cover = $request->cover;
$book->published = $request->published;
$book->purchase_link = $request->purchase_link;
```

### Route adjustments for guests vs. users
Now that books are associated with specific users, it doesn't make sense anymore that guest would be able to see an index of all the books. Instead, guests should see some sort of Welcome page.

So, let's create a `WelcomeController` with a `getIndex()` method:

```
<?php
# /app/Http/Controllers/WelcomeController.php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class WelcomeController extends Controller {

    public function getIndex() {
        return view('welcome.index');
    }

}
```

That `getIndex()` method should load this view:

```php
# /resources/views/welcome/index.blade.php
@extends('layouts.master')

@section('title')
    Welcome to Foobooks
@stop

@section('content')
    <p>
        Welcome to Foobooks, a personal book organizer.
        To get started <a href='/'>log in</a> or <a href='/register'>register</a>.
    </p>
@stop
```

And then we'll update our routes to look like this:
```
// Guests who visit "/" get a welcome page
Route::get('/', 'WelcomeController@getIndex');

Route::group(['middleware' => 'auth'], function() {

    // Logged in users who visit "/" get a listing of books
    Route::get('/', 'BookController@getIndex');

    Route::get('/books/create', 'BookController@getCreate');
    Route::post('/books/create', 'BookController@postCreate');

    Route::get('/books/edit/{id?}', 'BookController@getEdit');
    Route::post('/books/edit', 'BookController@postEdit');

    Route::get('/books', 'BookController@getIndex');
    Route::get('/books/show/{title?}', 'BookController@getShow');

});
```

Test it out:
+ While logged out, visit `http://localhost/`; you should see a welcome page.
+ While logged in, visit `http://localhost/`; you should see the book listing page.
+ While logged out, try and visit `http://localhost/books`; you should be directed to the login page.




## Using Tags/Many to Many
Last lecture, we set everything up (migrations, seeders, models) for a tags feature, now lets look at how we'd implement tags.

We need a way to associate tags with books (either from the *Edit Book* or *Create Book* page)

For authors, this was done with a dropdown which worked because each book can have only *one* author.

Each book can have *many* tags, though, so a dropdown won't do. Instead, lets show all possible tags with a checkbox.

To accomplish this, we'll need to gather the following data:

1. All the possible tags
2. All the tags associated with the book we're looking at.

```php
# BookController.php
public function getEdit($id = null) {

    # Get this book and eager load its tags
    $book = \App\Book::with('tags')->find($id);

    [...]

    # Get all the possible tags so we can include them with checkboxes in the view
    $tagModel = new \App\Tag();
    $tags_for_checkbox = $tagModel->getTagsForCheckboxes();

    /*
    Create a simple array of just the tag names for tags associated with this book;
    will be used in the view to decide which tags should be checked off
    */
    $tags_for_this_book = [];
    foreach($book->tags as $tag) {
        $tags_for_this_book[] = $tag->name;
    }

    return view('books.edit')
        ->with([
            'book' => $book,
            'authors_for_dropdown' => $authors_for_dropdown,
            'tags_for_checkbox' => $tags_for_checkbox,
            'tags_for_this_book' => $tags_for_this_book,
        ]);

}
```

```php
# edit.blade.php

[...]

<div class='form-group'>
    <label for='tags'>Tags</label>
    @foreach($tags_for_checkbox as $tag_id => $tag)
        <? $checked = (in_array($tag['name'],$tags_for_this_book)) ? 'CHECKED' : '' ?>
        <input {{ $checked }} type='checkbox' name='tags[]' value='{{$tag_id}}'> {{ $tag['name'] }}<br>
    @endforeach
</div>

[...]
```

```php
# Tag.php
public function getTagsForCheckboxes() {

    $tags = $this->orderBy('name','ASC')->get();

    $tagsForCheckboxes = [];

    foreach($tags as $tag) {
        $tagsForCheckboxes[$tag['id']] = $tag;
    }

    return $tagsForCheckboxes;

}
```

```php
# BookController.php
public function postEdit(Request $request) {

    [...]

    $book->save();

    if($request->tags) {
        $tags = $request->tags;
    }
    else {
        $tags = [];
    }
    $book->tags()->sync($tags);

    [...]

}
```

## Deleting Books

When deleting a book, you have to first delete any relationships to tags (that is, rows in the `book_tag` table).

This can be done via an [Event/Listener](http://laravel.com/docs/5.1/events) but we'll take a shortcut in the interest of time and just embed it in our deletion methods.


Two new routes:
```
Route::get('/books/confirm-delete/{id?}', 'BookController@getConfirmDelete');
Route::get('/books/delete/{id?}', 'BookController@getDoDelete');
```

Two new methods in BookController.php:

```php
public function getConfirmDelete($book_id) {

    $book = \App\Book::find($book_id);

    return view('books.delete')->with('book', $book);
}

public function getDoDelete($book_id) {

    $book = \App\Book::find($book_id);

    if(is_null($book)) {
        \Session::flash('flash_message','Book not found.');
        return redirect('\books');
    }

    if($book->tags()) {
        $book->tags()->detach();
    }

    $book->delete();

    \Session::flash('flash_message',$book->title.' was deleted.');

    return redirect('/books');

}
```

One new view:

```php
# resources/books/delete.blade.php
@extends('layouts.master')

@section('title')
    Delete Book
@stop

@section('content')
    <h1>Delete Book</h1>
    <p>Are you sure you want to delete <em>{{$book->title}}</em>?</p>
    <p><a href='/books/delete/{{$book->id}}'>Yes...</a></p>
@stop
```


Add a delete link in `books/index.blade.php`:
```html
<a href='/books/confirm-delete/{{$book->id}}'>Delete</a><br>
```
