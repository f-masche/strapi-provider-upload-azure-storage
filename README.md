# Strapi Provider Upload Azure Storage

Plugin enabling image uploading to azure storage from strapi.

[![NpmVersion](https://img.shields.io/npm/v/strapi-provider-upload-azure-storage.svg)](https://www.npmjs.com/package/strapi-provider-upload-azure-storage) [![NpmDownloads](https://img.shields.io/npm/dt/strapi-provider-upload-azure-storage.svg)](https://www.npmjs.com/package/strapi-provider-upload-azure-storage)

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

* Node 10+
* npm 6+
* strapi@3.0.0-beta.16+

### Installing

Inside your strapi project run the following

```sh
yarn add strapi-provider-upload-azure-storage

# or

npm install strapi-provider-upload-azure-storage
```

## Usage


### Strapi version >= 4.0.0

With a stable release of Strapi 4.0.0, the configuration was moved to a JavaScript file. Official documentation [here](https://strapi.io/documentation/developer-docs/latest/development/plugins/upload.html#using-a-provider).

To enable the provider, create or edit the file at ```./config/plugins.js```.

This is an example plugins.js file for Azure storage:
```js
module.exports = ({ env }) => ({
  upload: {
    config: {
      provider: 'strapi-provider-upload-azure-storage',
      providerOptions: {
        account: env('STORAGE_ACCOUNT'),
        accountKey: env('STORAGE_ACCOUNT_KEY'),
        serviceBaseURL: env('STORAGE_URL'),
        containerName: env('STORAGE_CONTAINER_NAME'),
        cdnBaseURL: env('STORAGE_CDN_URL'),
        defaultPath: 'assets',
        maxConcurrent: 10
      }
    }
  }
});
```

#### Security Middleware Configuration

The default settings in the Strapi Security Middleware block all but local content (you will see `(blocked:csp)` on media requests in the network tab), you will need to modify the contentSecurityPolicy settings to allow loading of thumbnail previews in the admin panel.

To allow the azure storage content to be displayed, edit the file at ```./config/middlewares.js```.
You should replace the `strapi::security` string with the object below instead, see the [Middlewares configuration](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/required/middlewares.html) documentation for more details.

```js
module.exports = [
  // ...
  {
    name: 'strapi::security',
    config: {
      contentSecurityPolicy: {
        useDefaults: true,
        directives: {
          'connect-src': ["'self'", 'https:'],
          'img-src': ["'self'", 'data:', 'blob:', env('STORAGE_URL'), env('STORAGE_CDN_URL')],
          'media-src': ["'self'", 'data:', 'blob:', env('STORAGE_URL'), env('STORAGE_CDN_URL')],
          upgradeInsecureRequests: null,
        },
      },
    },
  },
  // ...
];
```

#### Additional image formats

This plugin can create additional image formats on upload. Additional images are uploaded side-by-side to the original format.

If enabled, additional urls are included in the API response.
The new urls are only added to `image.attributes.formats` and in `image.attributes.provider_metadata` for the original image.

```json
{
  "id": 15,
  "attributes": {
    "name": "image.jpg",
    "provider_metadata": {
      "webpUrl": "https://cdn.example.com/image_a62eb06604.webp",
      "avifUrl": "https://cdn.example.com/image_a62eb06604.avif"
    },
    ...
    "formats": {
      "thumbnail": {
        ...
        "url": "https://cdn.example.com/image_04cee326c9.jpg",
        "webpUrl": "https://cdn.example.com/image_04cee326c9.webp",
        "avifUrl": "https://cdn.example.com/image_04cee326c9.avif"
      },
      "small": {
        ...
        "url": "https://cdn.example.com/image_04cee326c9.jpg",
        "webpUrl": "https://cdn.example.com/image_04cee326c9.webp",
        "avifUrl": "https://cdn.example.com/image_04cee326c9.avif"
      }
    }
  }
}

```

##### WebP
The WebP image format can be enabled via the `config.webp` setting. We are using sharp for converting the image. Additional options can be passed to sharp through `config.webpOptions`. For all options see the [sharp documentation](https://sharp.pixelplumbing.com/api-output#webp)
```js
module.exports = ({ env }) => ({
  upload: {
    config: {
      provider: 'strapi-provider-upload-azure-storage',
      providerOptions: {
        ...
        webp: true
        webpOptions: { ... } 
      }
    }
  }
});
```

##### AVIF
The AVIF image format can be enabled via the `config.avif` setting. We are using sharp for converting the image. Additional options can be passed to sharp through `config.avifOptions`. For all options see the [sharp documentation](https://sharp.pixelplumbing.com/api-output#avif)
```js
module.exports = ({ env }) => ({
  upload: {
    config: {
      provider: 'strapi-provider-upload-azure-storage',
      providerOptions: {
        ...
        avif: true
        avifOptions: { ... } 
      }
    }
  }
});
```


### Strapi version >= 3.0.0 & < 4.0.0

With a stable release of Strapi 3.0.0, the configuration was moved to a JavaScript file. Official documentation.

To enable the provider, create or edit the file at ```./config/plugins.js```.

This is an example plugins.js file for Azure storage:
```js
module.exports = ({ env }) => ({
  upload: {
    provider: 'azure-storage',
    providerOptions: {
      account: env('STORAGE_ACCOUNT'),
      accountKey: env('STORAGE_ACCOUNT_KEY'),
      serviceBaseURL: env('STORAGE_URL'),
      containerName: env('STORAGE_CONTAINER_NAME'),
      cdnBaseURL: env('STORAGE_CDN_URL'),
      defaultPath: 'assets',
      maxConcurrent: 10
    }
  }
});
```

`serviceBaseURL` is optional, it is useful when connecting to Azure Storage API compatible services, like the official emulator [Azurite](https://github.com/Azure/Azurite/). `serviceBaseURL` would then look like `http://localhost:10000/your-storage-account-key`.  
When `serviceBaseURL` is not provided, default `https://${account}.blob.core.windows.net` will be used.

`cdnBaseURL` is optional, it is useful when using CDN in front of your storage account. Images will be returned with the CDN URL instead of the storage account URL.


## Contributing

Contributions are welcome

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/jakeFeldman/strapi-provider-upload-azure-storage/releases).

## Authors

* **Jake Feldman** - *Initial work* - [jakeFeldman](https://github.com/jakeFeldman)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* strapi.io
* Azure
