# Transloadit PHP SDK

[![Build Status](https://travis-ci.org/transloadit/php-sdk.png?branch=master)](https://travis-ci.org/transloadit/php-sdk)
[![Coverage Status](https://coveralls.io/repos/transloadit/php-sdk/badge.png?branch=master)](https://coveralls.io/r/transloadit/php-sdk)

A **PHP** Integration for [Transloadit](https://transloadit.com)'s file uploading and encoding service

## Important

If you are on `v0.10.0` or below, just pull-updating to the latest `v1.x.x `will break the SDK for you.
`v1.0.0` makes PHP 5.3.0 a requirement. For development use `composer install --dev` to get phpunit version and run `vendor/bin/phpunit test` to run tests.

## Intro

[Transloadit](https://transloadit.com) is a service that helps you handle file uploads, resize, crop and watermark your images, make GIFs, transcode your videos, extract thumbnails, generate audio waveforms, and so much more. In short, [Transloadit](https://transloadit.com) is the Swiss Army Knife for your files.

This is a **PHP** SDK to make it easy to talk to the [Transloadit](https://transloadit.com) REST API.

## Install

Add the Transloadit PHP SDK as a dependency to your `composer.json` file:

```json
{
  "require": {
    "transloadit/php-sdk": "dev-master"
  }
}
```

Consider replacing `dev-master` with the latest version in order to pin your dependencies.

Install the [Composer](https://getcomposer.org/) dependency:

```bash
php composer.phar install
```

Keep your Transloadit account's key & secret key nearby. You can check
the [API credentials](https://transloadit.com/accounts/credentials) page for
these values.

## Usage

<!-- This section is generated by: make docs -->

### 1. Upload and resize an image from your server

This example demonstrates how you can use the SDK to create an <dfn>Assembly</dfn>
on your server.

It takes a sample image file, uploads it to Transloadit, and starts a
resizing job on it.

```php
<?php
require 'vendor/autoload.php';

use transloadit\Transloadit;

$transloadit = new Transloadit(array(
  'key'    => 'YOUR_TRANSLOADIT_KEY',
  'secret' => 'YOUR_TRANSLOADIT_SECRET',
));

$response = $transloadit->createAssembly(array(
  'files' => array('/PATH/TO/FILE.jpg'),
  'params' => array(
    'steps' => array(
      'resize' => array(
        'robot' => '/image/resize',
        'width' => 200,
        'height' => 100,
      ),
    ),
  ),
));

// Show the results of the assembly we spawned
echo '<pre>';
print_r($response);
echo '</pre>';

```

### 2. Create a simple end-user upload form

This example shows you how to create a simple Transloadit upload form
that redirects back to your site after the upload is done.

Once the script receives the redirect request, the current status for
this <dfn>Assembly</dfn> is shown using `Transloadit::response()`.

Note: There is no guarantee that the <dfn>Assembly</dfn> has already finished
executing by the time the `$response` is fetched. You should use
the `notify_url` parameter for this.

```php
<?php
require 'vendor/autoload.php';

use transloadit\Transloadit;

$transloadit = new Transloadit(array(
  'key'    => 'YOUR_TRANSLOADIT_KEY',
  'secret' => 'YOUR_TRANSLOADIT_SECRET',
));

// Check if this request is a Transloadit redirect_url notification.
// If so fetch the response and output the current assembly status:
$response = Transloadit::response();
if ($response) {
  echo '<h1>Assembly status:</h1>';
  echo '<pre>';
  print_r($response);
  echo '</pre>';
  exit;
}

// This should work on most environments, but you might have to modify
// this for your particular setup.
$redirectUrl = sprintf(
  'http://%s%s',
  $_SERVER['HTTP_HOST'],
  $_SERVER['REQUEST_URI']
);

// Setup a simple file upload form that resizes an image to 200x100px
echo $transloadit->createAssemblyForm(array(
  'params' => array(
    'steps' => array(
      'resize' => array(
        'robot' => '/image/resize',
        'width' => 200,
        'height' => 100,
      )
    ),
    'redirect_url' => $redirectUrl,
  ),
));
?>
<h1>Pick an image to resize</h1>
<input name="example_upload" type="file">
<input type="submit" value="Upload">
</form>

```

### 3. Integrate the jQuery plugin into the previous example

Integrating the jQuery plugin simply means adding a few lines of JavaScript
to the previous example. Check the HTML comments below to see what changed.

```php
<?php
require 'vendor/autoload.php';

use transloadit\Transloadit;

$transloadit = new Transloadit(array(
  'key'    => 'YOUR_TRANSLOADIT_KEY',
  'secret' => 'YOUR_TRANSLOADIT_SECRET',
));

$response = Transloadit::response();
if ($response) {
  echo '<h1>Assembly status:</h1>';
  echo '<pre>';
  print_r($response);
  echo '</pre>';
  exit;
}

$redirectUrl = sprintf(
  'http://%s%s',
  $_SERVER['HTTP_HOST'],
  $_SERVER['REQUEST_URI']
);

echo $transloadit->createAssemblyForm(array(
  'params' => array(
    'steps' => array(
      'resize' => array(
        'robot' => '/image/resize',
        'width' => 200,
        'height' => 100,
      )
    ),
    'redirect_url' => $redirectUrl
  )
));
?>
<!--
  Including the jQuery plugin is as simple as adding jQuery and including the
  JS snippet for the plugin. See https://transloadit.com/docs/#jquery-sdk
-->
<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script type="text/javascript">
  var tlProtocol = (('https:' === document.location.protocol) ? 'https://' : 'http://');
  document.write(unescape("%3Cscript src='" + tlProtocol + "assets.transloadit.com/js/jquery.transloadit2.js' type='text/javascript'%3E%3C/script%3E"));
</script>
<script type="text/javascript">
  $(document).ready(function() {
    // Tell the transloadit plugin to bind itself to our form
    $('form').transloadit();
  });
</script>
<!-- Nothing changed below here -->
<h1>Pick an image to resize</h1>
<form>
  <input name="example_upload" type="file">
  <input type="submit" value="Upload">
</form>
```

Alternatively, check our [Uppy](https://transloadit.com/docs/#uppy), our next-gen file uploader for the web.

### 4. Fetch the Assembly Status JSON

You can just use the `TransloaditRequest` class to get the job done easily.

```php
<?php
require 'vendor/autoload.php';

$assemblyId = 'YOUR_ASSEMBLY_ID';
$transloadit = new Transloadit(array(
  'key'    => 'YOUR_TRANSLOADIT_KEY',
  'secret' => 'YOUR_TRANSLOADIT_SECRET',
));
$response = $transloadit->getAssembly($assemblyId);

echo '<pre>';
print_r($response);
echo '</pre>';
```

### 5. Create an assembly with a template.

This example demonstrates how you can use the SDK to create an <dfn>Assembly</dfn>
with <dfn>Templates</dfn>.

You are expected to create a <dfn>Template</dfn> on your Transloadit account dashboard
and add the <dfn>Template</dfn> ID here.

```php
<?php
require 'vendor/autoload.php';

use transloadit\Transloadit;

$transloadit = new Transloadit(array(
  'key'    => 'YOUR_TRANSLOADIT_KEY',
  'secret' => 'YOUR_TRANSLOADIT_SECRET',
));

$response = $transloadit->createAssembly(array(
  'files' => array('/PATH/TO/FILE.jpg'),
  'params' => array(
    'template_id' => 'YOUR_TEMPLATE_ID'
  ),
));

// Show the results of the assembly we spawned
echo '<pre>';
print_r($response);
echo '</pre>';

```
<!-- End of generated doc section -->

### Signature Auth

<dfn>Signature Authentication</dfn> is done by the PHP SDK by default internally so you do not need to worry about this :)

## Example

For fully working examples take a look at [`examples/`](https://github.com/transloadit/php-sdk/tree/master/examples).

## API

### $Transloadit = new Transloadit($properties = array());

Creates a new Transloadit instance and applies the given $properties.

#### $Transloadit->key = null;

The auth key of your Transloadit account.

#### $Transloadit->secret = null;

The auth secret of your Transloadit account.

#### $Transloadit->request($options = array(), $execute = true);

Creates a new `TransloaditRequest` using the `$Transloadit->key` and
`$Transloadit->secret` properties.

If `$execute` is set to `true`, `$TransloaditRequest->execute()` will be
called and used as the return value.

Otherwise the new `TransloaditRequest` instance is being returned.

#### $Transloadit->createAssemblyForm($options = array());

Creates a new Transloadit assembly form including the hidden 'params' and
'signature' fields. A closing form tag is not included.

`$options` is an array of `TransloaditRequest` properties to be used.
For example: `"params"`, `"expires"`, `"endpoint"`, etc..

In addition to that, you can also pass an `"attributes"` key, which allows
you to set custom form attributes. For example:

```php
$Transloadit->createAssemblyForm(array(
  'attributes' => array(
    'id'    => 'my_great_upload_form',
    'class' => 'transloadit_form',
  ),
));
```

#### $Transloadit->createAssembly($options);

Sends a new assembly request to Transloadit. This is the preferred way of
uploading files from your server.

`$options` is an array of `TransloaditRequest` properties to be used with the exception that you can
also use the `waitForCompletion` option here:

`waitForCompletion` is a boolean (default is false) to indicate whether you want to wait for the
Assembly to finish with all encoding results present before the callback is called. If
waitForCompletion is true, this SDK will poll for status updates and return when all encoding work
is done.

Check example #1 above for more information.

#### $Transloadit->getAssembly($assemblyId);

Retrieves the Assembly status json for a given Assembly ID.

#### $Transloadit->cancelAssembly($assemblyId);

Cancels an assembly that is currently executing and prevents any further encodings costing money.

This will result in `ASSEMBLY_NOT_FOUND` errors if invoked on assemblies that are not currently
executing (anymore).

#### Transloadit::response()

This static method is used to parse the notifications Transloadit sends to
your server.

There are two kinds of notifications this method handles:

- When using the `redirect_url` parameter, and Transloadit redirects
  back to your site, a `$_GET['assembly_url']` query parameter gets added.
  This method detects the presence of this parameter and fetches the current
  assembly status from that url and returns it as a `TransloaditResponse`.
- When using the `notify_url` parameter, Transloadit sends a
  `$_POST['transloadit']` parameter. This method detects this, and parses
  the notification JSON into a `TransloaditResponse` object for you.

If the current request does not seem to be invoked by Transloadit, this
method returns `false`.


### $TransloaditRequest = new TransloaditRequest($properties = array());

Creates a new TransloaditRequest instance and applies the given $properties.

#### $TransloaditRequest->key = null;

The auth key of your Transloadit account.

#### $TransloaditRequest->secret = null;

The auth secret of your Transloadit account.

#### $TransloaditRequest->method = 'GET';

Inherited from `CurlRequest`. Can be used to set the type of request to be
made.

#### $TransloaditRequest->endpoint = 'https://api2.transloadit.com';

The endpoint to send this request to.

#### $TransloaditRequest->path = null;

The url path to request.

#### $TransloaditRequest->url = null;

Inherited from `CurlRequest`. Lets you overwrite the above endpoint / path
properties with a fully custom url alltogether.

#### $TransloaditRequest->fields = array();

A list of additional fields to send along with your request. Transloadit
will include those in all assembly related notifications.

#### $TransloaditRequest->files = array();

An array of paths to local files you would like to upload. For example:

```php
$TransloaditRequest->files = array('/my/file.jpg');
```

or

```php
$TransloaditRequest->files = array('my_upload' => '/my/file.jpg');
```

The first example would automatically give your file a field name of
`'file_1'` when executing the request.

#### $TransloaditRequest->params = array();

An array representing the JSON params to be send to Transloadit. You
do not have to include an `'auth'` key here, as this class handles that
for you as part of `$TransloaditRequest->prepare()`.

#### $TransloaditRequest->expires = '+2 hours';

If you have configured a '`$TransloaditRequest->secret`', this class will
automatically sign your request. The expires property lets you configure
the duration for which the signature is valid.

#### $TransloaditRequest->headers = array();

Lets you send additional headers along with your request. You should not
have to change this property.

#### $TransloaditRequest->execute()

Sends this request to Transloadit and returns a `TransloaditResponse`
instance.

### $TransloaditResponse = new TransloaditResponse($properties = array());

Creates a new TransloaditResponse instance and applies the given $properties.

#### $TransloaditResponse->data = null;

Inherited from `CurlResponse`. Contains an array of the parsed JSON
response from Transloadit.

You should generally only access this property after having checked for
errors using `$TransloaditResponse->error()`.

#### $TransloaditResponse->error();

Returns `false` or a string containing an explanation of what went wrong.

All of the following will cause an error string to be returned:

- Network issues of any kind
- The Transloadit response JSON contains an `{"error": "..."}` key
- A malformed response was received

## Contributing

Feel free to fork this project. We will happily merge bug fixes or other small
improvements. For bigger changes you should probably get in touch with us
before you start to avoid not seeing them merged.

## Versioning

This project implements the Semantic Versioning guidelines.

Releases will be numbered with the following format:

`<major>.<minor>.<patch>`

And constructed with the following guidelines:

- Breaking backward compatibility bumps the major (and resets the minor and patch)
- New additions without breaking backward compatibility bumps the minor (and resets the patch)
- Bug fixes and misc changes bumps the patch

For more information on SemVer, please visit http://semver.org/.

## Releasing a new version

```bash
# first update CHANGELOG.md and commit all your work
source env.sh && VERSION=v3.0.1 ./release.sh
```

## License

[MIT Licensed](LICENSE)
