The following is a very rough outline of the modifications I'll make to foobooks during Lecture 13.

This should not be considered a stand-alone document; for full details please refer to the lecture video and the foobooks code source.



## Edit Book + Authors
Previously when "author" was just a text field in the `books` table, we just needed a text input to update the author:

```html
<label for='title'>* Author:</label>
<input
    type='text'
    id='author'
    name="author"
    value='{{$book->author}}'
>
```

Now that we've extracted authors to their own table, we need to switch our approach from a text input to a dropdown filled with all the authors.

To make this happen, we first need an __array of authors__ where the key is the author id and the value is the author name.

```php
# BookController.php
public function getEdit($id = null) {

    $book = \App\Book::find($id);

    # Get all the authors
    $authors = \App\Author::orderBy('last_name','ASC')->get();

    # Organize the authors into an array where the key = author id and value = author name
    $authors_for_dropdown = [];
    foreach($authors as $author) {
        $authors_for_dropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    [...]

    # Make sure $authors_for_dropdown is passed to the view
    return view('books.edit')->with([
        'book' => $book,
        'authors_for_dropdown' => $authors_for_dropdown
    ]);
```

Then, we can construct the dropdown (`<select>`) using this data:

```html
<label for='title'>* Author:</label>
<select id='author_id' name='author_id'>
    @foreach($authors_for_dropdown as $author_id => $author_name)

        {{ $selected = ($book->author->id == $author_id) ? 'SELECTED' : '' }}

        <option value='{{ $author_id }}' {{ $selected }}>
            {{ $author_name }}
        </option>
    @endforeach
</select>
```




## Skinny controllers, fat models
The code above for getting all the authors could be useful in other places, so lets extract it out of the controller and add it as a method to the Author model:

```php
# app/Author.php
public static function authorsForDropdown() {

    $authors = \App\Author::orderBy('last_name','ASC')->get();
    $authors_for_dropdown = [];
    foreach($authors as $author) {
        $authors_for_dropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    return $authors_for_dropdown;
}
```

Then in `getEdit()`:
```
$authors_for_dropdown = \App\Author::authorsForDropdown();
```




## Adapting the Add functionality to work with the authors table
Try this one on your own.
Same procedure as above, except there should be no default selected.
