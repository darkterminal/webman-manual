# Routing
## Default routing rules
webman default routing rules are `http://127.0.0.1:8787/{controller}/{action}`.

The default controller is `app\controller\Index`, and the default action is index.

For example visit:

- `http://127.0.0.1:8787` will default access to the methods `app\controller\Index` of the classindex
- `http://127.0.0.1:8787/foo` will default access to the methods `app\controller\Foo` of the class `index`
- `http://127.0.0.1:8787/foo/test` will default access to the methods `app\controller\Foo` of the class `test`
- `http://127.0.0.1:8787/admin/foo/test` `app\admin\controller\Foo` The methods of the class will be accessed by default `test` (refer to multiple applications)

Change the configuration file when you want to change the routing of a request `config/route.php`.

If you want to disable the default route, add the following configuration `config/route.php` to the :
```php
Route::disableDefaultRoute();
```
> `Route::disableDefaultRoute()` requires workerman/webman-framework version >= 1.0.13

## Closure routing
`config/route.php` Add the following routing code to
```php
Route::any('/test', function ($request) {
    return response('test');
});
```
> Since the closure function does not belong to any controller, `$request->app` `$request->controller` `$request->action` all are empty strings.

When the access address `http://127.0.0.1:8787/test` is, a `test` string will be returned.

## Class Routing
`config/route.php` Add the following routing code to
```php
Route::any('/testclass', [app\controller\Index::class, 'test']);
```
When the access address `http://127.0.0.1:8787/testclass` is, the return value of the method of the `app\controller\Index` class will be returned. `test`

## Resource Based Routing
When the app directory structure is very complex and webman cannot automatically resolve it, you can use reflection to automatically configure the route, for example `config/route.php`, add the following code in
```php
Route::resource('/test', app\controller\Index::class);

// Specify resource routing
Route::resource('/test', app\controller\Index::class,['index','create']);

// Non-Defined Resource Routing
// Such as notify access address is any type of route /text/notify or /text/notify/{id} routeName is test.notify
Route::resource('/test', app\controller\Index::class,['index','create','notify']);
```

| Verb | URI              | Action | Route Name |
|------|------------------|--------|------------|
| GET  | /test            | index  | `test.index` |
| GET  | /test/create     | create  | `test.create` |
| POST  | /test    | store  | `test.store` |
| GET  | /test/{id}    | show  | `test.show` |
| GET  | /test/{id}/edit | edit | `test.edit` |
| PUT  | /test/{id} | update | `test.update` |
| DELETE  | /test/{id} | delete | `test.delete` |
| PUT  | /test/{id}/recovery | recovery | `test.recovery` |

## Automatic route resolution
When the app directory structure is very complex and webman cannot be automatically resolved, you can install the automatic routing plug-in of webman, which will automatically retrieve all controllers and automatically configure the corresponding routes for them, so that they can be accessed through url.

Install the auto-routing plugin
```bash
composer require webman/auto-route
```

> **Hint**
> You can still manually set some routes in config/route.php, the auto-route plugin will take precedence over the configuration in config/route.php.

## Route Parameter
If there are parameters in the route, pass `{key}` to match, and the matching result will be passed to the corresponding controller method parameters (passed sequentially from the second parameter), for example:
```php
// matches /user/123 /user/abc
Route::any('/user/{id}', [app\controller\User:class, 'get']);
```
```php
namespace app\controller;
class User
{
    public function get($request, $id)
    {
        return response('received parameters'.$id);
    }
}
```
More examples:
```php
// matches /user/123, does not match /user/abc
Route::any('/user/{id:\d+}', function ($request, $id) {
    return response($id);
});

// matches /user/foobar, does not match /user/foo/bar
Route::any('/user/{name}', function ($request, $name) {
   return response($name);
});

// matches /user /user/123 and /user/abc
Route::any('/user[/{name}]', function ($request, $name = null) {
   return response($name ?? 'tom');
});
```
## Group Routing
> **Notice**
> Group routing requires workerman/webman-frameworkversion >= 1.0.9

Sometimes a route contains a large number of the same prefix, in this case we can use route grouping to simplify the definition. E.g:
```php
Route::group('/blog', function () {
   Route::any('/create', function ($rquest) {return response('create');});
   Route::any('/edit', function ($rquest) {return response('edit');});
   Route::any('/view/{id}', function ($rquest, $id) {return response("view $id");});
});
```
Equivalent to
```php
Route::any('/blog/create', function ($rquest) {return response('create');});
Route::any('/blog/edit', function ($rquest) {return response('edit');});
Route::any('/blog/view/{id}', function ($rquest, $id) {return response("view $id");});
```
group nested use

> **Notice**
> Requires workerman/webman-frameworkversion >= 1.0.12
```php
Route::group('/blog', function () {
   Route::group('/v1', function () {
      Route::any('/create', function ($rquest) {return response('create');});
      Route::any('/edit', function ($rquest) {return response('edit');});
      Route::any('/view/{id}', function ($rquest, $id) {return response("view $id");});
   });
});
```
## Routing Middleware
> **Notice**
> Requires `workerman/webman-framework` version >= 1.0.12

We can set middleware for a route or a group of routes.
E.g:
```php
Route::any('/admin', [app\admin\controller\Index::class, 'index'])->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);

Route::group('/blog', function () {
   Route::any('/create', function () {return response('create');});
   Route::any('/edit', function () {return response('edit');});
   Route::any('/view/{id}', function ($r, $id) {response("view $id");});
})->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);
```
> **Notice:**
> `->middleware()` When the routing middleware acts on the group group, the current route must be under the current group
```php
# Example of wrong use
Route::group('/blog', function () {
   Route::group('/v1', function () {
      Route::any('/create', function ($rquest) {return response('create');});
      Route::any('/edit', function ($rquest) {return response('edit');});
      Route::any('/view/{id}', function ($rquest, $id) {return response("view $id");});
   });
})->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);
```
```php
# Examples of correct use
Route::group('/blog', function () {
    Route::group('/v1', function () {
        Route::any('/create', function ($rquest) {return response('create');});
        Route::any('/edit', function ($rquest) {return response('edit');});
        Route::any('/view/{id}', function ($rquest, $id) {return response("view $id");});
    })->middleware([
        app\middleware\MiddlewareA::class,
        app\middleware\MiddlewareB::class,
    ]);
});
```
## URL Generation
> **Notice**
> `workerman/webman-framework` Version >= 1.0.10 is required .
> Group nested routing is temporarily not supported to generate url

For example route:
```php
Route::any('/blog/{id}', [app\controller\Blog::class, 'view'])->name('blog.view');
```
We can use the following method to generate the url for this route.
```php
route('blog.view', ['id' => 100]); // The result is /blog/100
```
This method can be used when using the url of the route in the view, so that no matter how the routing rules change, the url will be automatically generated, avoiding the situation that the view file is changed a lot due to the adjustment of the route address.

## Handling 404
When the route is not found, it returns the 404 status code by default and outputs the `public/404.html` file content.

If developers want to intervene in the business process when the route is not found, they can use the fallback routing `Route::fallback($callback)` method provided by webman. For example, the following code logic is to redirect to the home page when the route is not found.
```php
Route::fallback(function(){
    return redirect('/');
});
```
Another example is to return a json data when the route does not exist, which is very useful when webman is used as an api interface.
```php
Route::fallback(function(){
    return json(['code' => 404, 'msg' => '404 not found']);
});
```

## Routing Interface
```php
// Set the route requested by any method of $uri
Route::any($uri, $callback);
// Set the route for the get request of $uri
Route::get($uri, $callback);
// Set the route of the request for $uri
Route::post($uri, $callback);
// Set the route for the put request of $uri
Route::put($uri, $callback);
// Set the route of the patch request for $uri
Route::patch($uri, $callback);
// Set the route for the delete request of $uri
Route::delete($uri, $callback);
// Set the route for the head request of $uri
Route::head($uri, $callback);
// Set routes for multiple request types at the same time
Route::add(['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', 'OPTIONS'], $uri, $callback);
// group routing
Route::group($path, $callback);
// Fallback routing, set the default routing bottom line
Route::fallback($callback);
```
If the uri has no corresponding route (including the default route), and the fallback route is not set, it will return 404.
