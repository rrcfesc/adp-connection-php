# ADP Client Connection Library for PHP

The ADP Client Connection Library is intended to simplify and aid the process of authenticating, authorizing and connecting to the ADP Marketplace API Gateway. The Library includes a sample application that can be run out-of-the-box to connect to the ADP Marketplace API **test** gateway.

### Version
1.0.3

### Installation

**Composer**

Use Composer from the location you wish to use as the root folder for your project.

```sh
$ composer require adpmarketplace/api-connection
```

This will install the Connection module to your project.  If you wish to build the sample client, do the following:

```sh
$ cd adplib/connection/tools
$ php makeclient.php
```

If you want to test immediately, copy and paste the proper commands that are generated by the makeclient.php script.  In case you missed them, you can do it this way as well:

```sh
(from the project root)
$ cd client
$ php -S 127.0.0.1:8889
```

This starts an HTTP server on port 8889 (this port must be unused to run the sample application).

**Run the sample app**

Note, to test the sample app you must first run the makeclient.php script, and then start the PHP server as outlined above.

Once this is done, open your web browser and go to:

```sh
http://localhost:8889
```

 The sample app allows you to connect to the ADP test API Gateway using the **client_credentials** and **authorization_code** grant types. For the **authorization_code** connection, you will be asked to provide an ADP username (MKPLDEMO) and password (marketplace1).

## Examples
### Create Client Credentials ADP Connection

```php

require_once ("config.php"); // Contains settings
require_once ($libroot . "connection/adpapiConnection.class.php");


//----------------------
// Create Config Array
//----------------------

$configuration = array (
        'grantType'             => 'client_credentials',
        'clientID'              => $ADP_CC_CLIENTID,
        'clientSecret'          => $ADP_CC_CLSECRET,
        'sslCertPath'           => $ADP_CERTFILE,
        'sslKeyPath'            => $ADP_KEYFILE,
        'tokenServerURL'        => $ADP_APIROOT
    );

//----------------------
// Create the class
//----------------------

try {

    $adpConn = adpapiConnectionFactory::create($configuration);

}
catch (adpException $e) {

    showADPException($e);
    exit();

}

//-------------------------------------------
// Request a token for API access
//-------------------------------------------

try {

    $result = $adpConn->Connect();
}
catch (adpException $e) {

    showADPException($e);
    exit();

}

```

### Create Authorization Code ADP Connection

The Authorization Code requires 2 pieces.  The first piece handles the creation of the request URI to be sent to the gateway.  The second piece handles the response from the gateway and uses it to retrieve an access token.

Generating the request URI:

```php

require_once ("config.php"); // Contains settings
require_once ($libroot . "connection/adpapiConnection.class.php");

//----------------------
// Create Config Array
//----------------------

$configuration = array (
        'grantType'             => 'authorization_code',
        'clientID'              => $ADP_AC_CLIENTID,
        'clientSecret'          => $ADP_AC_CLSECRET,
        'sslCertPath'           => $ADP_CERTFILE,
        'sslKeyPath'            => $ADP_KEYFILE,
        'tokenServerURL'        => $ADP_APIROOT,
        'scope'                 => 'openid',
        'responseType'          => 'code',
        'redirectURL'           => $ADP_REDIRECTURL
    );


//----------------------
// Create the class
//----------------------

try {
    $adpConn = adpapiConnectionFactory::create($configuration);
}
catch (adpException $e) {

    showADPException($e);
    exit();

}

//-----------------------------------------------
// Request an authentication URL from Connection
//-----------------------------------------------

try {

    $result = $adpConn->getAuthRequest();
}
catch (adpException $e) {

    showADPException($e);
    exit();

}

//-----------------------------------------------
// Send the browser to the generated URI
//-----------------------------------------------

header("Location: " . $result);

```

And the second part, when the gateway returns to us:

```php

require ($webroot . "config.php");
require ($libroot . "connection/adpapiConnection.class.php");

// Check to see if there is an error

if (isset($_GET['error'])) {

    echo "<h1>Gateway Error</h1>";
    echo "<div class='alert alert-danger'>\n";
    echo "The error returned is: " . $_GET['error'];
    echo "</div>\n";
    exit();

}


// Get the authorization code, if available.

$retcode = $_GET['code'];

// restore session object from session.  Handle sessions the way you see fit.
$adpConn = unserialize($_SESSION['adpConn']);

//--------------------------------------------------
// Add the returned code to the connection object
//--------------------------------------------------


$adpConn->auth_code         = $retcode;

//-------------------------------------------
// Request a token for API access
//-------------------------------------------

try {
    $result = $adpConn->Connect();
}
catch (adpException $e) {

    showADPException($e);
    exit();

}


```


## API Documentation ##

Documentation on the individual API calls provided by the library are documented in the 'doc' folder.  Open the index.html file in your browser to view the documentation.


## Dependencies ##

This library has the following dependencies.

* php >= v5.3 with CURL support.  If you are using OSX, this means you will need to rebuild PHP with CURL support.
* composer

## Tests ##

Our testing and code coverage is handled through PHPUNIT, which you must install to run the tests.  For code coverage to work, you must also have Xdebug installed.  The test units are located in the "test" folder, and the configurations are already loaded into the phpunit.xml.  In order to run the tests and view code coverage:

```
phpunit
```
## Linter ##

You must use PHP's built-in linter to validate the structure of your code:

```php
php -l <sourcefile>
```

## Contributing ##

To contribute to the library, please generate a pull request. Before generating the pull request, please insure the following:

1. Appropriate unit tests have been updated or created.
2. Code coverage on the unit tests must be no less than 95%.
3. Your code updates have been fully tested and linted with no errors.
4. Update README.md and API documentation as appropriate.
 
## License ##

This library is available under the Apache 2 license (http://www.apache.org/licenses/LICENSE-2.0).
