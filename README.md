# Keycloak Provider for OAuth 2.0 Client
[![Latest Version](https://img.shields.io/github/release/thephpleague/oauth2-keycloak.svg?style=flat-square)](https://github.com/thephpleague/oauth2-keycloak/releases)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)
[![Build Status](https://img.shields.io/travis/thephpleague/oauth2-keycloak/master.svg?style=flat-square)](https://travis-ci.org/thephpleague/oauth2-keycloak)
[![Coverage Status](https://img.shields.io/scrutinizer/coverage/g/thephpleague/oauth2-keycloak.svg?style=flat-square)](https://scrutinizer-ci.com/g/thephpleague/oauth2-keycloak/code-structure)
[![Quality Score](https://img.shields.io/scrutinizer/g/thephpleague/oauth2-keycloak.svg?style=flat-square)](https://scrutinizer-ci.com/g/thephpleague/oauth2-keycloak)
[![Total Downloads](https://img.shields.io/packagist/dt/league/oauth2-keycloak.svg?style=flat-square)](https://packagist.org/packages/league/oauth2-keycloak)

This package provides Keycloak OAuth 2.0 support for the PHP League's [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client).

## Installation

To install, use composer:

```
composer require league/oauth2-keycloak
```

## Usage

Usage is the same as The League's OAuth client, using `\League\OAuth2\Client\Provider\Keycloak` as the provider.

Use `authServerUrl` to specify the Keycloak server URL. You can lookup the correct value from the Keycloak client installer JSON under `auth-server-url`, eg. `http://localhost:8080/auth`.

Use `realm` to specify the Keycloak realm name. You can lookup the correct value from the Keycloak client installer JSON under `realm`, eg. `master`.

### Authorization Code Flow

```php
$provider = new League\OAuth2\Client\Provider\Keycloak([
    'authServerUrl'     => '{keycloak-server-url}',
    'realm'             => '{keycloak-realm}',
    'clientId'          => '{keycloak-client-id}',
    'clientSecret'      => '{keycloak-client-secret}',
    'redirectUri'       => 'https://example.com/callback-url',
]);

if (!isset($_GET['code'])) {

    // If we don't have an authorization code then get one
    $authUrl = $provider->getAuthorizationUrl();
    $_SESSION['oauth2state'] = $provider->getState();
    header('Location: '.$authUrl);
    exit;

// Check given state against previously stored one to mitigate CSRF attack
} elseif (empty($_GET['state']) || ($_GET['state'] !== $_SESSION['oauth2state'])) {

    unset($_SESSION['oauth2state']);
    exit('Invalid state, make sure HTTP sessions are enabled.');

} else {

    // Try to get an access token (using the authorization coe grant)
    try {
        $token = $provider->getAccessToken('authorization_code', [
            'code' => $_GET['code']
        ]);
    } catch (Exception $e) {
        exit('Failed to get access token: '.$e->getMessage());
    }     

    // Optional: Now you have a token you can look up a users profile data
    try {

        // We got an access token, let's now get the user's details
        $user = $provider->getResourceOwner($token);

        // Use these details to create a new profile
        printf('Hello %s!', $user->getName());

    } catch (Exception $e) {
        exit('Failed to get resource owner: '.$e->getMessage());
    }

    // Use this to interact with an API on the users behalf
    echo $token->getToken();
}
```

### Refreshing a Token

```php
$provider = new League\OAuth2\Client\Provider\Keycloak([
    'authServerUrl'     => '{keycloak-server-url}',
    'realm'             => '{keycloak-realm}',
    'clientId'          => '{keycloak-client-id}',
    'clientSecret'      => '{keycloak-client-secret}',
    'redirectUri'       => 'https://example.com/callback-url',
]);

$token = $provider->getAccessToken('refresh_token', ['refresh_token' => $token->getRefreshToken()]);
```

## Testing

``` bash
$ ./vendor/bin/phpunit
```

## Contributing

Please see [CONTRIBUTING](https://github.com/thephpleague/oauth2-keycloak/blob/master/CONTRIBUTING.md) for details.


## Credits

- [Thomas Raehalme](https://github.com/raehalme)
- [All Contributors](https://github.com/thephpleague/oauth2-keycloak/contributors)


## License

The MIT License (MIT). Please see [License File](https://github.com/thephpleague/oauth2-keycloak/blob/master/LICENSE) for more information.