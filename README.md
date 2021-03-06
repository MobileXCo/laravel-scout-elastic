# Laravel Scout Elasticsearch Driver

[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

This package makes is the [Elasticsearch](https://www.elastic.co/products/elasticsearch) driver for Laravel Scout that works with AWS' Elasticsearch and does not require you to put credentials into .env files.

## Contents

- [Installation](#installation)
- [Usage](#usage)
- [Credits](#credits)
- [License](#license)

## Installation

You can install the package via composer:

``` bash
composer require tamayo/laravel-scout-elastic
```

You must add the Scout service provider and the package service provider in your app.php config:

```php
// config/app.php
'providers' => [
    ...
    Laravel\Scout\ScoutServiceProvider::class,
    ...
    ScoutEngines\Elasticsearch\ElasticsearchProvider::class,
],
```

## Configuration

This package assumes you are following good AWS security and /not/ putting password in your Laravel .env files an instead follows the practices outlined in [Credentials for the AWS SDK for PHP Version 3](https://docs.aws.amazon.com/sdk-for-php/v3/developer-guide/guide_credentials.html). To enable this, set the following variable in your .env.

```php
ELASTICSEARCH_PROVIDER=aws
```

For local development, inside vagrant for instance, you can use the normal Elasticsearch client by either omitting this variable or setting it as follows.

```php
ELASTICSEARCH_PROVIDER=elastic
```

One thing the AWS client needs is the Region. If you don't already have it in your .env, add it as such.

```php
AWS_REGION=us-west-2
```

## Testing with Scout

If your CI environment does not have access to a working Elasticsearch instance, any indexed Models will cause it to error. To solve this, add the following to your phpunit.xml. The single quotes wrapping the double quotes are the tricky part there.

```xml
<env name="SCOUT_DRIVER" value='"null"' />
```

## Laravel Configuration

After you've published the Laravel Scout package configuration

```bash
php artisan vendor:publish  --provider="ScoutEngines\Elasticsearch\ElasticsearchProvider"
```

you'll need to update the main scout configuration

```php
// config/scout.php
// Set your driver to elasticsearch
    'driver' => env('SCOUT_DRIVER', 'elasticsearch'),

...
    'elasticsearch' => [
        'index' => [
            // if using more than one index, you likely want to change this to each of
            // your indexes. quite possibly using searchableAs on your models.
            env('ELASTICSEARCH_INDEX', 'scout')
        ],
        'hosts' => [
            env('ELASTICSEARCH_HOST', 'http://localhost'),
        ],
        'perModelIndex' => true,
    ],
...
```

and this package's configuration

```php
// config/laravel-scout-elastic
// set this if you don't want to include it in your .env
    'provider' => env('ELASTICSEARCH_PROVIDER', 'elasticsearch'),
...
    'region' => env('AWS_REGION', 'us-west-2'),
...
```

and enable the artisan job:
```php
// App/Console/Kernel.php
protected $commands = [
    ...
    \App\Console\Commands\CreateIndex::class
    ...
],
```

### Elasticsearch Configuration
Scout will happily throw an error if it cannot create contact your Elasticsearch server or the index doesn't exist.

Creating an index can be kinda arcane, so if the index doesn't exist, you can include the following artisan command in your deployment stack to check if the index exists, and if it doesnt then it will create it.

```bash
php artisan scout:create-index
```

## Usage

Now you can use Laravel Scout as described in the [official documentation](https://laravel.com/docs/5.3/scout)
## Credits

- [Erick Tamayo](https://github.com/ericktamayo)
- [All Contributors](../../contributors)

## License

The MIT License (MIT).
