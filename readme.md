
![alt text](./laravel-cdn.svg "Laravel CDN")

# Laravel CDN Assets Manager

##### Content Delivery Network Package for Laravel

The package provides the developer the ability to upload their assets (or any public file) to a CDN with a single artisan command.
And then it allows them to switch between the local and the online version of the files.

##### History
This project has had multiple homes and unfortunately the previous maintainers have lost interest into this project. I am 
planning on maintaining this package as we have several projects that make use of it. This is the latest version of Laravel CDN.

###### Fork from [Publiux/laravelcdn](https://github.com/publiux/laravelcdn)
###### Fork From [Vinelab/cdn](https://github.com/Vinelab/cdn)

This project has been forked originally from https://github.com/Vinelab/cdn. All credit for the original work goes there.

#### Laravel Support
- Laravel 6,7,8 and 9 are supported, as is package auto-discovery. Minimum PHP version is now 7.3 and for Laravel 9 8.0

## Highlights

- Amazon Web Services (AWS) - S3
- DigitalOcean (DO) - Spaces
- Artisan command to upload content to CDN
- Simple Facade to access CDN assets

### Questions
1. Is this package an alternative to Laravel FileSystem and do they work together?

+ No, the package was introduced in Laravel 4 and it's main purpose is to manage your CDN assets by loading them from the CDN into your Views pages, and easily switch between your Local and CDN version of the files. As well it allows you to upload all your assets with single command after specifying the assets directory and rules. The FileSystem was introduced in Laravel 5 and it's designed to facilitate the loading/uploading of files form/to a CDN. It can be used the same way as this Package for loading assets from the CDN, but it's harder to upload your assets to the CDN since it expect you to upload your files one by one. As a result this package still not a replacement of the Laravel FileSystem and they can be used together.


## Installation

#### Via Composer

Require `juhasev/laravelcdn` in your project:

```bash
composer require "juhasev/laravelcdn:~2.0"
```

*If you are using Laravel 5.4 or below, you need to register the service provider:*

Laravel 5.4 and below: Add the service provider and facade to `config/app.php`:

```
'providers' => array(
     //...
     SampleNinja\LaravelCdn\CdnServiceProvider::class,
)
```

```
'aliases' => array(
     //...
     'CDN' => SampleNinja\LaravelCdn\Facades\CdnFacadeAccessor::class
)
```

*If you are using Laravel 5.5, there is no need to register the service provider as this package is automatically discovered.*

Publish the package config file:

```bash
php artisan vendor:publish --provider 'SampleNinja\LaravelCdn\CdnServiceProvider'
```

## Environment Configuration

This package can be configured by editing the config/app.php file.  Alternatively, you can set many of these options in as environment variables in your '.env' file.

##### AWS Credentials
Set your AWS Credentials and other settings in the `.env` file.

*Note: you should always have an `.env` file at the project root, to hold your sensitive information. This file should usually not be committed to your VCS.*

```
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

##### CDN URL

Set the CDN URL:

```
'url' => env('CDN_Url', 'https://s3.amazonaws.com'),
```

This can altered in your '.env' file as follows:

```bash
CDN_Url=
```

##### Bypass

To load your LOCAL assets for testing or during development, set the `bypass` option to `true`:

```
'bypass' => env('CDN_Bypass', false),
```

This can be altered in your '.env' file as follows:

```
CDN_Bypass=
```

##### Cloudfront Support

```
'cloudfront'    => [
    'use' => env('CDN_UseCloudFront', false),
    'cdn_url' => env('CDN_CloudFrontUrl', false)
],
```

This can be altered in your '.env' file as follows:

```
CDN_UseCloudFront=
CDN_CloudFrontUrl=
```

##### Default CDN Provider
For now, the only CDN provider available is AwsS3. Although, as DO natively support the AWS API, you can utilise it by also providing the endpoint, please see the cdn.php config for more info. This option cannot be set in '.env'.

```
'default' => 'AwsS3',
```

##### CDN Provider Configuration

```
'aws' => [

    's3' => [

        'version'   => 'latest',
        'region'    => '',
        'endpoint'  => '', // For DO Spaces

        'buckets' => [
            'my-backup-bucket' => '*',
        ]
    ]
],
```

###### Multiple Buckets

```
'buckets' => [

    'my-default-bucket' => '*',

    // 'js-bucket' => ['public/js'],
    // 'css-bucket' => ['public/css'],
    // ...
]

```

#### Files & Directories

###### Include:

Specify directories, extensions, files and patterns to be uploaded.

```
'include'    => [
    'directories'   => ['public/dist'],
    'extensions'    => ['.js', '.css', '.yxz'],
    'patterns'      => ['**/*.coffee'],
],
```

###### Exclude:

Specify what to be ignored.

```
'exclude'    => [
    'directories'   => ['public/uploads'],
    'files'         => [''],
    'extensions'    => ['.TODO', '.txt'],
    'patterns'      => ['src/*', '.idea/*'],
    'hidden'        => true, // ignore hidden files
],
```

##### Other Configurations

```
'acl'           => 'public-read',
'metadata'      => [ ],
'expires'       => gmdate("D, d M Y H:i:s T", strtotime("+5 years")),
'cache-control' => 'max-age=2628000',
```

You can always refer to the AWS S3 Documentation for more details: [aws-sdk-php](http://docs.aws.amazon.com/aws-sdk-php/v3/guide/)

## Usage

You can 'push' your assets to your CDN and you can 'empty' your assets as well using the commands below.

#### Push

Only changed assets are pushed to the CDN. (THanks, )

Upload assets to CDN
```bash
php artisan cdn:push
```

You can specify a folder upload prefix in the cdn.php config file. Your assets will be uploaded into that folder on S3.

#### Empty

Delete assets from CDN
```bash
php artisan cdn:empty
```
CAUTION: This will erase your entire bucket. This may not be what you want if you are specifying an upload folder when you push your assets.

#### Load Assets

Use the facade `CDN` to call the `CDN::asset()` function.

*Note: the `asset` works the same as the Laravel `asset` it start looking for assets in the `public/` directory:*

```blade
{{CDN::asset('assets/js/main.js')}}        // example result: https://js-bucket.s3.amazonaws.com/public/assets/js/main.js

{{CDN::asset('assets/css/style.css')}}        // example result: https://css-bucket.s3.amazonaws.com/public/assets/css/style.css
```
*Note: the `elixir` works the same as the Laravel `elixir` it loads the manifest.json file from build folder and choose the correct file revision generated by  gulp:*
```blade
{{CDN::elixir('assets/js/main.js')}}        // example result: https://js-bucket.s3.amazonaws.com/public/build/assets/js/main-85cafe36ff.js

{{CDN::elixir('assets/css/style.css')}}        // example result: https://css-bucket.s3.amazonaws.com/public/build/assets/css/style-2d558139f2.css
```
*Note: the `mix` works the same as the Laravel 5.4 `mix` it loads the mix-manifest.json file from public folder and choose the correct file revision generated by webpack:*
```blade
{{CDN::mix('/js/main.js')}}        // example result: https://js-bucket.s3.amazonaws.com/public/js/main-85cafe36ff.js

{{CDN::mix('/css/style.css')}}        // example result: https://css-bucket.s3.amazonaws.com/public/css/style-2d558139f2.css
```

To use a file from outside the `public/` directory, anywhere in `app/` use the `CDN::path()` function:

```blade
{{CDN::path('private/something/file.txt')}}        // example result: https://css-bucket.s3.amazonaws.com/private/something/file.txt
```


## Test

To run the tests, run the following command from the project folder.

```bash
$ ./vendor/bin/phpunit
```

## Support

Please request support or submit issues [via Github](https://github.com/juhasev/laravelcdn/issues)


## Contributing

Please see [CONTRIBUTING](https://github.com/juhasev/LaravelCdn/blob/master/CONTRIBUTING.md) for details.

## Credits
- [Juha Vehnia](https://github.com/juhasev) (forker)
- [Raul Ruiz](https://github.com/publiux) (original forker)
- [Mahmoud Zalt](https://github.com/Mahmoudz) (original developer)
- [Filipe Garcia](https://github.com/filipegar) (contributred pre-fork, uncredited pull request for duplicate uploading verification)
- [Contributors from original project](https://github.com/Vinelab/cdn/graphs/contributors)
- [All Contributors for this Fork](../../contributors)


## License

The MIT License (MIT). Please see [License File](https://github.com/juhasev/LaravelCdn/blob/master/LICENSE) for more information.
