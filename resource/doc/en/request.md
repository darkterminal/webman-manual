# Illustrate
## get the request object

webman will automatically inject the request object into the first parameter of the action method, for example

**Example**
```php
<?php
namespace app\controller;

use support\Request;

class User
{
    public function hello(Request $request)
    {
        $default_name = 'webman';
        // Get the name parameter from the get request, and return $default_name if the name parameter is not passed
        $name = $request->get('name', $default_name);
        // Return a string to the browser
        return response('hello ' . $name);
    }
}
```

Through the $requestobject we can get any data related to the request.

**Sometimes we want to get the currently requested object in other classes, at this time we just need to `$request` use the helper function `request()`**

## The `GET` Request Parameters

**Get the entire `GET` request array**
```php
$request->get();
```
Returns an empty array if the request has no get parameters.

**Get a value of the get array**
```php
$request->get('name');
```
Returns null if the get array does not contain this value.

You can also pass a default value to the second parameter of the get method, which returns the default value if no corresponding value is found in the get array. E.g:

```php
$request->get('name', 'tom');
```

## The `POST` Request Parameters

**Get the entire post array**
```php
$request->post();
```
Returns an empty array if the request has no post parameters.

**Get a value of the post array**
```php
$request->post('name');
```
Returns null if the post array does not contain this value.

Like the get method, you can also pass a default value to the second parameter of the post method, which will return the default value if the corresponding value is not found in the post array. E.g:
```php
$request->post('name', 'tom');
```

## Get The Original `POST` Request Body
```php
$post = $request->rawBody();
```

This function is similar `php-fpm` to the `file_get_contents("php://input");` operation in. Used to get the `HTTP` original request body. `application/x-www-form-urlencoded` This is useful when getting unformatted post request data.

## Get header
**Get the entire header array**
```php
$request->header();
```
Returns an empty array if the request has no header parameter. Note that all keys are lowercase.

**Get a value of the header array**
```php
$request->header('host');
```
Returns null if the header array does not contain this value. Note that all keys are lowercase.

Like the get method, you can also pass a default value to the second parameter of the header method, which will return the default value if the corresponding value is not found in the header array. E.g:
```php
$request->header('host', 'localhost');
```

## Get Cookies
**Get the entire cookie array**
```php
$request->cookie();
```
Returns an empty array if the request has no cookie parameter.

**Get a value of the cookie array**
```php
$request->cookie('name');
```
Returns null if the cookie array does not contain this value.

As with the get method, you can also pass a default value to the second parameter of the cookie method, which will return the default value if no corresponding value is found in the cookie array. E.g:
```php
$request->cookie('name', 'tom');
```

## Get All Input
Contains a `post` `get` collection of.
```php
$request->all();
```
## Get The Specified Input value
Get a value from `post` `get` the collection of.
```php
$request->input('name', $default_value);
```

## Get Some Input Data
Get partial data from `post` `get` the collection.
```php
// Get an array of username and password, ignore if the corresponding key does not exist
$only = $request->only(['username', 'password']);
// get all inputs except avatar and age
$except = $request->except(['avatar', 'age']);
```

## Get Uploaded File
**Get the entire uploaded file array**
```php
$request->file();
```
The returned file format is similar to:
```php
array (
    'file1' => object(webman\Http\UploadFile),
    'file2' => object(webman\Http\UploadFile)
)
```
It is an `webman\Http\UploadFile` array of instances. `webman\Http\UploadFile` The class inherits PHP's `SplFileInfo` class and also provides various methods for interacting with files.

**Notice:**

- The upload file size is limited by [defaultMaxPackageSize](http://doc.workerman.net/tcp-connection/default-max-package-size.html) , the default is **10M**, and the default value can be `config/server.php`  modified in the file. `max_package_size`

- Temporary files will be automatically cleared after the request ends.

- Returns an empty array if the request did not upload a file.

## Get a Specific upload File
```php
$request->file('avatar');
```
Returns an instance of the corresponding file if the file exists `webman\Http\UploadFile`, otherwise returns null.

**Example**
```php
<?php
namespace app\controller;

use support\Request;

class User
{
    public function file(Request $request)
    {
        $file = $request->file('upload');
        if ($file && $file->isValid()) {
            $file->move(public_path().'/files/myfile.'.$file->getUploadExtension());
            return json(['code' => 0, 'msg' => 'upload success']);
        }
        return json(['code' => 1, 'msg' => 'file not found']);
    }
}
```
## Get Host
Get the requested host information.
```php
$request->host();
```
If the requested address is a non-standard port `80` or `443`, the host information may carry the port, eg `example.com:8080`. If the port is not required, the first parameter can be passed `true` in.
```php
$request->host(true);
```

## Get Request Method
```php
$request->method();
```
The return value may be one of `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`.

## Get Request Uri
```php
$request->uri();
```
Returns the requested uri, including the path and queryString parts.

## Get Request Path
```php
$request->path();
```
Returns the path part of the request.

## Get Request queryString
```php
$request->queryString();
```
Returns the queryString part of the request.

## Get Request Url
`url()` The method returns `Query` the URL without the parameter.
```php
$request->url();
```
return something like `//www.workerman.net/workerman-chat`

`fullUrl()` The method returns a `Query` URL with parameters.
```php
$request->fullUrl();
```
return something like `//www.workerman.net/workerman-chat?type=download`

> Note: `url()` and `fullUrl()` do not return the protocol part (no http or https)

## Get Request HTTP Version
```php
$request->protocolVersion();
```
Returns a string `1.1` or `1.0`.

## Get Request sessionId
```php
$request->sessionId();
```
Returns a string consisting of letters and numbers

## Get Request Client IP
```php
$request->getRemoteIp();
```
## Get request client port
```php
$request->getRemotePort();
```
## Get the real IP of the requesting client
```php
$request->getRealIp($safe_mode=true);
```
> This method requires webman-framework >= 1.0.2

When a project uses a proxy (such as nginx), it `$request->getRemoteIp()` often gets the proxy server IP (similar `127.0.0.1` `192.168.x.x`) instead of the client's real IP. At this time, you can try to use `$request->getRealIp()` to get the real IP of the client.

`$request->getRealIp();` The principle is: if the client IP is found to be an intranet IP, try to obtain the real IP from the `Client-Ip`, `X-Forwarded-For`, `X-Real-Ip`, `Client-Ip` and Via HTTP headers. If it `$safe_mode` is `false`, it does not judge whether the client IP is an intranet IP (unsafe), and directly tries to read the client IP data from the above HTTP header. If the HTTP header does not have the above fields, `$request->getRemoteIp()` the return value used is returned as the result.

> Since HTTP headers are easy to forge, the client IP obtained by this method is not 100% trusted, especially if it `$safe_mode` is `false`. The most reliable way to get the client's real IP through a proxy is to know the secure proxy server IP, and to know exactly which HTTP header carries the real IP. If `$request->getRemoteIp()` the returned IP is confirmed to be a known secure proxy server, then pass `$request->header('HTTP header with real IP')` Get real IP.

## Get Server IP
```php
$request->getLocalIp();
```
## Get Server Port
```php
$request->getLocalPort();
```
## Determine whether it is an ajax request
```php
$request->isAjax();
```
## Determine whether it is a pjax request
```php
$request->isPjax();
```
## Determine whether it is expecting json to return
```php
$request->expectsJson();
```
## Determine whether the client accepts json return
```php
$request->acceptJson();
```
## Get the requested application name
Always return an empty string when using a single application `''`, and return the application name when using multiple applications
```php
$request->app;
```
> Because the closure function does not belong to any application, the request from the closure routing $request->appalways returns an empty string. `''`
> Closure routing see **[Routing](https://www.workerman.net/doc/webman/route.html)**

## Get the requested controller class name
Get the class name corresponding to the controller
```php
$request->controller;
```
return something like `app\controller\Index`

> Because the closure function does not belong to any controller, the request from the closure routing `$request->controller` always returns an empty string. `''`
> Closure routing see **[Routing](https://www.workerman.net/doc/webman/route.html)**

## Get the method name of the request
Get the controller method name corresponding to the request
```php
$request->action;
```
return something like `Index`

> Because the closure function does not belong to any controller, the request from the closure routing `$request->action` always returns an empty string. `''`
> Closure routing see **[Routing](https://www.workerman.net/doc/webman/route.html)**
