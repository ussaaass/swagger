# Hyperf Swagger

Swagger/OpenAPI integration for [Hyperf](https://hyperf.io). The package uses PHP attributes to scan Hyperf controllers and generates OpenAPI JSON documents, with an optional YAML output.

## Requirements

- PHP 8.2 or later
- Hyperf 3.2
- `zircote/swagger-php` 4.x (the compatibility test path also covers 6.x)

## Installation

```bash
composer require ussaaass/hyperf-swagger
```

If request validation is needed, install Hyperf Validation as well:

```bash
composer require hyperf/validation
```

Publish the package configuration:

```bash
php bin/hyperf.php vendor:publish hyperf/swagger
```

The default configuration is written to `config/autoload/swagger.php`.

## Annotating a controller

Each controller can be assigned to a Hyperf server with `HyperfServer`. The name must match a key in `swagger.server`.

```php
<?php

namespace App\Controller;

use Hyperf\Swagger\Annotation as OA;
use Hyperf\Swagger\Request\SwaggerRequest;

#[OA\Info(title: 'Example API', version: '1.0.0')]
#[OA\HyperfServer('http')]
class UserController
{
    #[OA\Get('/users', summary: 'List users', tags: ['users'])]
    public function index(SwaggerRequest $request): array
    {
        return [];
    }
}
```

The package provides `Get`, `Post`, `Put`, `Patch`, `Delete`, `Head`, and `Options` attributes, together with OpenAPI attributes such as `Info`, `Schema`, `Response`, `RequestBody`, `JsonContent`, and `Property`.

## Request validation

Use `SwaggerRequest` in a controller method and put validation rules on the corresponding Swagger attributes:

```php
#[OA\Post('/users', summary: 'Create user')]
#[OA\QueryParameter(
    name: 'name',
    required: true,
    rules: 'required|string|max:50',
    attribute: 'user name',
)]
public function create(SwaggerRequest $request): array
{
    return [];
}
```

Rules can also be placed on `OA\Property` inside a request body. `SwaggerRequest` reads them through `ValidationCollector` and exposes them to Hyperf Validation.

## Configuration

The important defaults are:

```php
return [
    'enable' => true,
    'port' => 9500,
    'url' => '/swagger',
    'auto_generate' => true,
    'json_dir' => BASE_PATH . '/storage/swagger',
    'yaml_dir' => null,
    'scan' => [
        'paths' => null,
    ],
    'server' => [
        'http' => [
            'servers' => [
                ['url' => 'http://127.0.0.1:9501'],
            ],
            'info' => [
                'title' => 'Example API',
                'version' => '1.0.0',
            ],
        ],
    ],
];
```

`server` is keyed by the value passed to `#[OA\HyperfServer(...)]`. The generated JSON is saved as `<server>.json`. Set `yaml_dir` to a directory to generate `<server>.yaml` as well.

## Generating the specification

When `auto_generate` is enabled, the specification is generated during application boot. It can also be generated explicitly:

```bash
php bin/hyperf.php gen:swagger
```

To generate a schema class from a Hyperf model:

```bash
php bin/hyperf.php gen:swagger-schema --model App\\Model\\User
```

With the default server configuration, Swagger UI is available at:

```text
http://127.0.0.1:9500/swagger?search=/http.json
```

The JSON document is served from the Swagger server; YAML files are intended for tooling and distribution.

## Compatibility tests

The repository contains a PHP 8.2/8.4/8.5 matrix for both swagger-php 4.x and 6.x:

```bash
composer update
vendor/bin/phpunit -c phpunit.xml.dist

COMPOSER=composer.swagger-php-6.json composer update
vendor/bin/phpunit -c phpunit.xml.dist
```

The 4.x path is retained for existing users. The 6.x path verifies the updated processor pipeline and attribute constructor compatibility.

## License

MIT
