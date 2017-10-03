# Telegram OAuth2 Provider for Laravel

**Note:** This package utilizes an unofficial service [telepass.me](https://telepass.me). Official OAuth API for Telegram does not exist.

**Note:** This package requires php version 7.0 or higher.

## Installation

### 0. Credentials
Obtain your app ID and secret from [telepass.me](https://telepass.me). You must set up a valid name and callback URL.

### 1. Composer
This assumes that you have composer installed globally:
```bash
composer require telegram/passport-laravel
```

### 2. Service Provider
- Remove `Laravel\Socialite\SocialiteServiceProvider` from your `providers[]` array in `config\app.php` if you have added it already.
- Add `\SocialiteProviders\Manager\ServiceProvider::class` to your `providers[]` array in `config\app.php`.

For example:

```php
'providers' => [
    // a whole bunch of providers
    // remove 'Laravel\Socialite\SocialiteServiceProvider',
    \SocialiteProviders\Manager\ServiceProvider::class, // add
];
```
**Note:** If you would like to use the Socialite Facade, you need to [install it](https://github.com/laravel/socialite).

### 3. Add the Event and Listeners
- Add `SocialiteProviders\Manager\SocialiteWasCalled` event to your `listen[]` array in `<app_name>/Providers/EventServiceProvider`.
- Add your listeners (i.e. the ones from the providers) to the `SocialiteProviders\Manager\SocialiteWasCalled[]` that you just created.
- The listener that you add for this provider is `'SocialiteProviders\Telegram\TelegramExtendSocialite@handle',`.

**Note:** You do not need to add anything for the built-in socialite providers unless you override them with your own providers.

For example:

```php
/**
 * The event handler mappings for the application.
 *
 * @var array
 */
protected $listen = [
    \SocialiteProviders\Manager\SocialiteWasCalled::class => [
        // add your listeners (aka providers) here
        'SocialiteProviders\Telegram\TelegramExtendSocialite@handle',
    ],
];
```

### 4. Environment Variables
If you add environment values to your .env as exactly shown below, **you do not need to add an entry to the services array**.

#### Append provider values to your `.env` file
```env
// other values above
TELEGRAM_KEY=yourkeyfortheservice
TELEGRAM_SECRET=yoursecretfortheservice
TELEGRAM_REDIRECT_URI=https://example.com/login   
```

#### Add to `config/services.php`.
**You do not need to add this if you add the values to the .env exactly as shown above.**
The values below are provided as a convenience in the case that a developer is not able to use the .env method
```php
'telegram' => [
    'client_id' => env('TELEGRAM_KEY'),
    'client_secret' => env('TELEGRAM_SECRET'),
    'redirect' => env('TELEGRAM_REDIRECT_URI'),  
],
```

## Usage
You should now be able to use it like you would regularly use Socialite (assuming you have the facade installed):
```php
return Socialite::with('telegram')->redirect();
```

### Lumen Support
You can use Socialite providers with Lumen. Just make sure that you have facade support turned on and that you follow the setup directions properly.

**Note:** If you are using this with Lumen, all providers will automatically be stateless since **Lumen** does not keep track of state.

Also, configs cannot be parsed from the `services[]` in Lumen. You can only set the values in the `.env` file as shown exactly in this document. If needed, you can also override a config (shown below).

### Stateless
You can set whether or not you want to use the provider as stateless. Remember that the OAuth provider (Twitter, Tumblr, etc) must support whatever option you choose.

**Note:** If you are using this with Lumen, all providers will automatically be stateless since Lumen does not keep track of state.

```php
// to turn off stateless
return Socialite::with('telegram')->stateless(false)->redirect();

// to use stateless
return Socialite::with('telegram')->stateless()->redirect();
```

### Retrieving the Access Token Response Body

Laravel Socialite by default only allows access to the `access_token`. Which can be accessed via the `\Laravel\Socialite\User->token` public property. Sometimes you need access to the whole response body which may contain items such as a `refresh_token`.

You can get the access token response body, after you called the `user()` method in Socialite, by accessing the property `$user->accessTokenResponseBody`;

```php
$user = Socialite::driver('telegram')->user();
$accessTokenResponseBody = $user->accessTokenResponseBody;
```

## Credits
Thanks to the [Socialite Providers](socialiteproviders.github.io) project for documentation and provider template.
