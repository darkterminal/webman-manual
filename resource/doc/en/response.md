# Response
The response is actually an `support\Responseobject`, in order to facilitate the creation of this object, webman provides some helper functions.

## Return an Arbitrary Response
**Example**
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

The response function is implemented as follows:
```php
function response($body = '', $status = 200, $headers = array())
{
    return new Response($status, $headers, $body);
}
```
You can also create an empty object first and then return the content with the settings responsein place. `$response->cookie()` `$response->header()` `$response->withHeaders()` `$response->withBody()`
```php
public function hello(Request $request)
{
    // Create an Response Object
    $response = response();

    // .... Business logic omitted

    // set cookies
    $response->cookie('foo', 'value');

    // .... Business logic omitted

    // set http header
    $response->header('Content-Type', 'application/json');
    $response->withHeaders([
                'X-Header-One' => 'Header Value 1',
                'X-Header-Tow' => 'Header Value 2',
            ]);

    // .... Business logic omitted

    // Set the data to return
    $response->withBody('data returned');
    return $response;
}

```

## Return JSON
**example**
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return json(['code' => 0, 'msg' => 'ok']);
    }
}
```
The json function is implemented as follows
```php
function json($data, $options = JSON_UNESCAPED_UNICODE)
{
    return new Response(200, ['Content-Type' => 'application/json'], json_encode($data, $options));
}
```

## Return XML
**example**
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        $xml = <<<XML
               <?xml version='1.0' standalone='yes'?>
               <values>
                   <truevalue>1</truevalue>
                   <falsevalue>0</falsevalue>
               </values>
               XML;
        return xml($xml);
    }
}
```
The xml function is implemented as follows:
```php
function xml($xml)
{
    if ($xml instanceof SimpleXMLElement) {
        $xml = $xml->asXML();
    }
    return new Response(200, ['Content-Type' => 'text/xml'], $xml);
}
```
## Back to View
Create a new file `app/controller/Foo.php` as follows
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return view('foo/hello', ['name' => 'webman']);
    }
}
```
Create a new file `app/view/foo/hello.html` as follows
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>webman</title>
</head>
<body>
hello <?=htmlspecialchars($name)?>
</body>
</html>
```
## Redirect
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return redirect('/user');
    }
}
```
The redirect function is implemented as follows:
```php
function redirect($location, $status = 302, $headers = [])
{
    $response = new Response($status, ['Location' => $location]);
    if (!empty($headers)) {
        $response->withHeaders($headers);
    }
    return $response;
}
```
## Header Settings
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return response('hello webman', 200, [
            'Content-Type' => 'application/json',
            'X-Header-One' => 'Header Value'
        ]);
    }
}
```
You can also use `header` and `withHeaders` methods to set headers individually or in batches.
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return response('hello webman')
        ->header('Content-Type', 'application/json')
        ->withHeaders([
            'X-Header-One' => 'Header Value 1',
            'X-Header-Tow' => 'Header Value 2',
        ]);
    }
}
```
You can also set the header in advance, and finally set the data to be returned.
```php
public function hello(Request $request)
{
    // Create Response Object
    $response = response();

    // .... Business logic omitted

    // set http header
    $response->header('Content-Type', 'application/json');
    $response->withHeaders([
                'X-Header-One' => 'Header Value 1',
                'X-Header-Tow' => 'Header Value 2',
            ]);

    // .... Business logic omitted

    // Set the data to return
    $response->withBody('data returned');
    return $response;
}
```
## Set Cookies
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return response('hello webman')
        ->cookie('foo', 'value');
    }
}
```
You can also set cookies ahead of time and set the data to be returned at the end.
```php
public function hello(Request $request)
{
    // Create Response Object
    $response = response();

    // .... Business logic omitted

    // 设置cookie
    $response->cookie('foo', 'value');

    // .... Business logic omitted

    // Set the data to return
    $response->withBody('data returned');
    return $response;
}
```
The complete parameters of the cookie method are as follows:
```php
cookie($name, $value = '', $max_age = 0, $path = '', $domain = '', $secure = false, $http_only = false)
```
return file stream
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return response()->file(public_path() . '/favicon.ico');
    }
}
```
The file method will automatically add `if-modified-since` headers and detect the headers on the next request, `if-modified-since` and return 304 directly if the file is not modified to save bandwidth.

## Download File
```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function hello(Request $request)
    {
        return response()->download(public_path() . '/favicon.ico', 'your-optional-filename.ico');
    }
}
```
The difference between the download method and the file method is that the download method is generally used to download and save files, and can set the downloaded file name. The download method does not check `if-modified-since` the header.
