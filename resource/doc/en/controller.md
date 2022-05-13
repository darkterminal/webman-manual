## Controller
Create a new controller file `app\controller\Foo.php`.
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }

    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```
When `http://127.0.0.1:8787/foo` accessing, the page returns `hello index`.

When `http://127.0.0.1:8787/foo/hello` accessing, the page returns `hello webman`.

Of course, you can change routing rules through routing configuration, see [routing](https://www.workerman.net/doc/webman/route.html).

## Illustrate
- The framework will automatically pass the `support\Request` object to the controller, through which the user input data (get post header cookie and other data) can be obtained, see [Request](https://www.workerman.net/doc/webman/request.html)
- Controllers can return numbers, strings, or `support\Response` objects, but not other types of data.
- `support\Response` Objects can be `response()` `json()` `xml()` `jsonp()` `redirect()` created with helper functions such as .
## The life Cycle
- Controllers are only instantiated when they are needed.
- Once the controller is instantiated, it will remain in memory until the process is destroyed.
- Since controller instances are resident in memory, the controller is not initialized on every request.
## Controller Hook `beforeAction()` and `afterAction()`
In traditional frameworks, controllers are instantiated once per request, so many developer `__construct()` methods do some pre-request preparation.

Since the controller resides in memory, webman cannot `__construct()` do this work in it, but webman provides a better solution `beforeAction()` `afterAction()`, which not only allows developers to intervene in the process before the request, but also in the processing process after the request middle.

In order to intervene in the request process, we need to use middleware

1. Create a file `app/middleware/ActionHook.php` (the middleware directory does not exist, please create it yourself)
```php
<?php
namespace app\middleware;

use support\Container;
use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;
class ActionHook implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        if ($request->controller) {
            // Direct access to beforeAction afterAction is prohibited
            if ($request->action === 'beforeAction' || $request->action === 'afterAction') {
                return response('<h1>404 Not Found</h1>', 404);
            }
            $controller = Container::get($request->controller);
            if (method_exists($controller, 'beforeAction')) {
                $before_response = call_user_func([$controller, 'beforeAction'], $request);
                if ($before_response instanceof Response) {
                    return $before_response;
                }
            }
            $response = $next($request);
            if (method_exists($controller, 'afterAction')) {
                $after_response = call_user_func([$controller, 'afterAction'], $request, $response);
                if ($after_response instanceof Response) {
                    return $after_response;
                }
            }
            return $response;
        }
        return $next($request);
    }
}
```
2. Add the following configuration in `config/middleware.php`
```php
return [
    '' => [
        // .... Other configurations are omitted here ....
        app\middleware\ActionHook::class,
    ]
];
```
3. In this way, if the controller contains the `beforeAction` or `afterAction` method, it will be called automatically when the request occurs.
E.g:
```php
<?php
namespace app\controller;
use support\Request;
class Index
{
    /**
     * This method will be called before the request
     */
    public function beforeAction(Request $request)
    {
        echo 'beforeAction';
        // If you want to terminate the execution of the Action, you can directly return the Response object. If you do not want to terminate, you do not need to return
        // return response('Terminate execution of Action');
    }

    /**
     * This method will be called after the request
     */
    public function afterAction(Request $request, $response)
    {
        echo 'afterAction';
        // If you want to change the request result, you can directly return a new Response object
        // return response('afterAction');
    }

    public function index(Request $request)
    {
        return response('index');
    }
}
```
**`beforeAction` illustrate:**

- Called before the current controller is executed
- The framework will pass an `Request` object to `beforeAction`, from which the developer can get user input
- If you want to terminate the execution of the current controller, you only need `beforeAction` to return an `Response` object in it, such asreturn `redirect('/user/login')`;
- Do not return any data without terminating execution of the current controller

**`afterAction` illustrate:**

- Called after the current controller has been executed
- The framework will pass the `Request` object and the `Response` object to `afterAction`, from which the developer can obtain the user input and the response result returned after the controller is executed
- Developers can `$response->rawBody()` get the response content by
- Developers can `$response->getHeader()` get the response headers by
- Developers can `$response->getStatusCode()` get the http status code of the response by
- The developer can use the `$response->withBody()` `$response->header()` `$response->withStatus()` string to modify the response, or create and return a new `Response` object to replace the original response

> **Hint**
> You can create a base controller class that implements the `beforeAction()` and `afterAction()` methods. Other controllers inherit this base class, so that each controller does not have to implement the `beforeAction()` `afterAction()` method again.
