# Middleware

Middleware is generally used to intercept requests or responses. For example, before executing the controller, uniformly verify the user's identity, such as jumping to the login page when the user is not logged in. For example, add a header to the response. For example, count the proportion of a certain uri request and so on.

## Middleware Onion Model
```

            ┌──────────────────────────────────────────────────────┐
            │                     middleware1                      │
            │     ┌──────────────────────────────────────────┐     │
            │     │               middleware2                │     │
            │     │     ┌──────────────────────────────┐     │     │
            │     │     │         middleware3          │     │     │
            │     │     │     ┌──────────────────┐     │     │     │
            │     │     │     │                  │     │     │     │
 　── Request ──────────────> Controller - Response ────────────────> Client
            │     │     │     │                  │     │     │     │
            │     │     │     └──────────────────┘     │     │     │
            │     │     │                              │     │     │
            │     │     └──────────────────────────────┘     │     │
            │     │                                          │     │
            │     └──────────────────────────────────────────┘     │
            │                                                      │
            └──────────────────────────────────────────────────────┘
```

The middleware and the controller form a classic onion model. The middleware is like an onion skin layer by layer, and the controller is the onion core. If the shown request traverses middleware 1, 2, 3 like an arrow to the controller, the controller returns a response, which in turn traverses the middleware in the order 3, 2, 1 and finally returns to the client. That is to say, in each middleware, we can get both the request and the response, so that we can do many things in the middleware, such as intercepting the request or the response.

## Request Interception

Sometimes we don't want a request to reach the controller layer. For example, if we find that the current user is not logged in in an authentication middleware, we can directly intercept the request and return a login response. Then the process looks like this
```

            ┌───────────────────────────────────────────────────────┐
            │                     middleware1                       │
            │     ┌───────────────────────────────────────────┐     │
            │     │         Authentication Middleware         │     │
            │     │      ┌──────────────────────────────┐     │     │
            │     │      │         middleware3          │     │     │
            │     │      │     ┌──────────────────┐     │     │     │
            │     │      │     │                  │     │     │     │
 　── Request ───────┐   │     │    Controller    │     │     │     │
            │  response　│     │                  │     │     │     │
   <─────────────────┘   │     └──────────────────┘     │     │     │
            │     │      │                              │     │     │
            │     │      └──────────────────────────────┘     │     │
            │     │                                           │     │
            │     └───────────────────────────────────────────┘     │
            │                                                       │
            └───────────────────────────────────────────────────────┘
```
As shown in the figure, after the request reaches the authentication middleware, a login response is generated, and the response traverses from the authentication middleware back to middleware 1 and then returns to the browser.

# Middleware Interface
Middleware must implement `Webman\MiddlewareInterface` interface.
```php
interface MiddlewareInterface
{
    /**
     * Process an incoming server request.
     *
     * Processes an incoming server request in order to produce a response.
     * If unable to produce the response itself, it may delegate to the provided
     * request handler to do so.
     */
    public function process(Request $request, callable $handler): Response;
}
```
That is, the `process` method must be implemented. The `process` method must return a `support\Response` object. By default, this object is generated by `$handler($request)` (the request will continue to traverse the onion core), or it can be `response()` `json()` `xml( )` Responses generated by helper functions such as `redirect()` (requests to stop continuing to traverse the onion core).

## Example: Authentication Middleware
Create the file `app/middleware/AuthCheckTest.php` (if the directory does not exist, please create it yourself) as follows:
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class AuthCheckTest implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $session = $request->session();
        // User not logged in
        if (!$session->get('userinfo')) {
            // Intercept the request, return a redirect response, and the request stops traversing the onion core
            return redirect('/user/login');
        }
        // request to continue
        return $handler($request);
    }
}
```
Add the global middleware in `config/middleware.php` as follows:
```php
return [
    // global middleware
    '' => [
        // ... Other middleware is omitted here
        app\middleware\AuthCheckTest::class,
    ]
];
```
With the authentication middleware, we can concentrate on writing business code at the controller layer without worrying about whether the user is logged in.

## Example: Cross-Origin Request Middleware
Create the file `app/middleware/AccessControlTest.php` (if the directory does not exist, please create it yourself) as follows:
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class AccessControlTest implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $response = $request->method() == 'OPTIONS' ? response('') : $handler($request);
        $response->withHeaders([
            'Access-Control-Allow-Credentials' => 'true',
            'Access-Control-Allow-Origin' => $request->header('Origin', '*'),
            'Access-Control-Allow-Methods' => '*',
            'Access-Control-Allow-Headers' => '*',
        ]);

        return $response;
    }
}
```
According to the onion model, the response object can be obtained in the middleware. We directly call the `withHeaders` method of the response object to add a cross-domain http header to the response to achieve cross-domain.

> **Hint**
Cross-domain OPTIONS requests may be generated. We do not want OPTIONS requests to enter the controller, so we directly return an empty response (`response('')`) for the OPTIONS request to implement request interception.

Add the global middleware in `config/middleware.php` as follows:
```php
return [
    // global middleware
    '' => [
        // ... Other middleware is omitted here
        app\middleware\AccessControlTest::class,
    ]
];
```
> **Notice**
Cross-origin requests may include OPTIONS requests. If your cross-origin interface needs to set a route, please use Route::any(..) or Route::add(['POST', 'OPTIONS'], ..) to set it.

> **Notice**
If the ajax request has a custom header, you need to add this custom header to the Access-Control-Allow-Headers field in the middleware, otherwise it will report Request header field XXXX is not allowed by Access-Control-Allow-Headers in preflight response .

> **Notice**
If the business occurs 500, a cross-domain error will occur.

## Illustrate
- Middleware is divided into global middleware, application middleware (application middleware is only valid in multi-application mode, see [multi-application](https://www.workerman.net/doc/webman/multiapp.html)), routing middleware
- Currently, the middleware of a single controller is not supported (but a controller-like middleware function can be implemented in the middleware by judging `$request->controller`)
- The middleware configuration file location is `config/middleware.php`
- Global middleware is configured under key `''`
- The application middleware is configured under the specific application name, for example
```php
return [
    // global middleware
    '' => [
        app\middleware\AuthCheckTest::class,
        app\middleware\AccessControlTest::class,
    ],
    // api application middleware (application middleware is only valid in multi-application mode)
    'api' => [
        app\middleware\ApiOnly::class,
    ]
];
```
## Routing Middleware
> Note: requires workerman/webman-framework version >= 1.0.12
We can set middleware for a route or a group of routes.
For example, add the following configuration in `config/route.php`:
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
## Middleware Execution Order
- The middleware execution order is global `middleware` -> `application middleware` -> `routing middleware`.
- When there are multiple global middleware, they are executed according to the actual configuration order of the middleware (the same is true for application middleware and routing middleware).
- 404 requests will not trigger any middleware, including global middleware

## The middleware passes parameters to the controller
Sometimes the controller needs to use the data generated in the middleware, then we can pass parameters to the controller by adding properties to the `$request` object. E.g:

**Middleware:**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $request->data = 'some value';
        return $handler($request);
    }
}
```
**Controller**
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response($request->data);
    }
}
```
