http://laravel.com/docs/5.1/views#view-composers

http://stackoverflow.com/a/29817941/59479

```bash
$ php artisan make:provider ComposerServiceProvider
```

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

```html
<li><a href='/logout'>Log out {{ $user->name }}</a></li>
```
