# PHP Slack API Client
[![Version](https://img.shields.io/packagist/v/spatialmedia/slack-client.svg)](https://packagist.org/packages/spatialmedia/slack-client)
[![License](https://img.shields.io/packagist/l/spatialmedia/slack-client.svg)](https://packagist.org/packages/spatialmedia/slack-client)
[![Downloads](https://img.shields.io/packagist/dt/spatialmedia/slack-client.svg)](https://packagist.org/packages/spatialmedia/slack-client)

## History
- 2015: Originally built by [sagebind](https://github.com/sagebind), for use in [Slackyboy](https://github.com/sagebind/slackyboy).
- 2016: The plugin is re-distributed under the vendor name [mpociot](https://github.com/mpociot) as `mpociot/slack-client`.
- 2022: The plugin is forked by the Spatial Media team and made available as `spatialmedia/slack-client`.

The `spatialmedia/slack-client` is currently maintained by the Spatial Media team and You (the open source community).

## Requirements

- PHP >= 8.0

## Installation
Install with [Composer](http://getcomposer.org), obviously:

```sh
$ composer require spatialmedia/slack-client
```

Please note that the current version has unstable dependencies.

In order to install those dependencies, you can set `"minimum-stability"` in your `composer.json`, and recommend that you set `"prefer-stable"`:

```json
{
    "minimum-stability": "dev",
    "prefer-stable": true
}
```

## Usage
First, you need to create a client object to connect to the Slack servers. You will need to acquire an API token for your app first from Slack, then pass the token to the client object for logging in. Since this library uses React, you must also pass in an event loop object:

```php
$loop = \React\EventLoop\Factory::create();

$client = new \Slack\ApiClient($loop);
$client->setToken('YOUR-TOKEN-HERE');
// ...
$loop->run();
```

Assuming your token is valid, you are good to go! You can now use wrapper methods for accessing most of the [Slack API](http://api.slack.com). Below is an example of posting a message to a channel as the logged in user:

```php
$client->getChannelById('C025YTX9D')->then(function (\Slack\Channel $channel) use ($client) {
    $client->send('Hello from PHP!', $channel);
});
```

### Advanced messages
Slack supports messages much more rich than plain text through attachments. The easiest way to create a custom message is with a `MessageBuilder`:

```php
use Slack\Message\{Attachment, AttachmentBuilder, AttachmentField};

$message = $client->getMessageBuilder()
    ->setText('Hello, all!')
    ->setChannel($someChannelObject)
    ->addAttachment(new Attachment('My Attachment', 'attachment text'))
    ->addAttachment(new Attachment('Build Status', 'Build failed! :/', 'build failed', 'danger'))
    ->addAttachment(new AttachmentBuilder()
        ->setTitle('Some Fields')
        ->setText('fields')
        ->setColor('#BADA55')
        ->addField(new AttachmentField('Title1', 'Text', false))
        ->addField(new AttachmentField('Title2', 'Some other text', true))
        ->create()
    ]))
    ->create();

$client->postMessage($message);
```

Check the [API documentation](http://sagebind.github.io/slack-client/api) for a list of all methods and properties that messages, attachments, and fields support.


### Asynchronous requests and promises
All client requests are made asynchronous using [React promises](https://github.com/reactphp/promise). As a result, most of the client methods return promises. This lets you easily compose request orders and handle them as you need them. Since it uses React, be sure to call `$loop->run()` or none of the requests will be sent.

React allows the client to perform well and prevent blocking the entire thread while making requests. This is especially useful when writing real-time apps, like Slack chat bots.

### Real Time Messaging API
You can also connect to Slack using the [Real Time Messaging API](http://api.slack.com/rtm). This is often useful for creating Slack bots or message clients. The real-time client is like the regular client, but it enables real-time incoming events. First, you need to create the client:

```php
$client = new \Slack\RealTimeClient();
$client->setToken('YOUR-TOKEN-HERE');
$client->connect();
```

Then you can use the client as normal; `RealTimeClient` extends `ApiClient`, and has the same API for sending requests. You can attach a callback to handle incoming Slack events using `RealTimeClient::on()`:

```php
$client->on('file_created', function($data) {
    echo 'A file was created called ' . $data['file']['name'] . '!\n';
});
```

Below is a very simple, complete example:

```php
$loop = React\EventLoop\Factory::create();

$client = new Slack\RealTimeClient($loop);
$client->setToken('YOUR-TOKEN-HERE');

// disconnect after first message
$client->on('message', function ($data) use ($client) {
    echo "Someone typed a message: ".$data['text']."\n";
    $client->disconnect();
});

$client->connect()->then(function () {
    echo "Connected!\n";
});

$loop->run();
```

See the [Slack API documentation](http://api.slack.com/events) for a list of possible events.

## Running tests
You can run automated unit tests using [PHPUnit](http://phpunit.de) after installing dependencies:

```sh
$ vendor/bin/phpunit
```

## Where to get help
Need help? Checkout the [issues page](https://github.com/spatialmedia/slack-client/issues) for the repo.

## License
This library is licensed under the MIT license. See the [LICENSE](LICENSE) file for details.
