
Goal: Make `user` information available on all views.

Done with a View Composer: http://laravel.com/docs/5.1/views#view-composers

First, make the composer:
```bash
$ php artisan make:provider ComposerServiceProvider
```

Then update the composer:
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

Ref: http://stackoverflow.com/a/29817941/59479
