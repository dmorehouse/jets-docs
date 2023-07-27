---
title: Assets Serving
subcategory: assets
order: 1
---

Jets handles asset like stylesheets, javascripts, and images by serving them from s3. When Jets deploys, it compiles assets and uploads them to the Jets managed s3 bucket. When your app is running on AWS Lambda, the Jets helper methods like `stylesheet_link_tag`, `javascript_include_tag`, `javascript_importmap_tags`, `asset_path`, etc will point to the s3 bucket url. You can also configure settings so that the assets are served from a CDN like CloudFront that you manage yourself.

Jets compiles assets with either:

1. [Asset Pipeline (sprockets and importmap)]({% link _docs/assets/importmap.md %}): The sprockets tool is used by Rails 7 to compile assets. Importmap is use to help map and serve the assets. This is the default for Jets v5 and beyond.
2. [Webpack (webpacker/jetpacker)]({% link _docs/assets/webpack.md %}): This is a javascript node native tool that was used in Rails 4,5,6 to compile assets. The webpack toolchain is deprecated and may be remove at any time. You should use the sprockets toolchain.

If you're interested in why the new default, see: [Why: Importmap vs Webpack]({% link _docs/assets/why.md %}).

## Pros and Cons

There several benefits to precompling assets and serving them directly.

* A sha checksums is added to the assets as a part of building them to the `public/assets` folder.  The checksums optimize performance since images can be cached for very long periods of time. IE: 10y.
* You'll be able to configure high values for the `max-age` response header. This article [Increasing Application Performance with HTTP Cache Headers](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers) covers how cache headers work.  The default max-age for assets serving out of s3 is 3600s or 1h.
* If you replace an image and keep the same name, you don't have to update the code as the `image_tag` uses the compiled image with the sha checksum. It's nice to offload menial tasks like file renames to automation. Instead, we can spend the precious mental energy on coding business logic.
* Binary images are particularly beneficial to be served by s3. Though APIGW can serve binary assets, found it to be a pain and sometimes the client requesting the asset needs to send a specific accept header as part of the request, which you don't always have control over.

Usually the downside with asset precompling is:

* Complexity. You usually have to precompile and upload the assets as part of your deploy process. Jets fortunately handles this for you.

## Configure Settings

You can override the setting and configure the folders with the [Application Configuration]({% link _docs/config.md %}).

```ruby
Jets.application.configure do
  # ...
  # Default assets settings
  config.assets.folders = %w[assets images packs]
  config.assets.max_age = 3600 # max_age is a short way to set cache_control and expands to cache_control="public, max-age=3600"
  # config.assets.cache_control = nil # IE: "public, max-age=3600" # override max_age for more fine-grain control.
  # config.assets.base_url = nil # IE: https://cloudfront.com/my/base/path, defaults to the s3 bucket url
                                 # IE: https://s3-us-west-2.amazonaws.com/demo-dev-s3bucket-1inlzkvujq8zb
end
```

{% include config/reference/header.md %}
{% include config/reference/assets.md %}
{% include config/reference/footer.md %}

## Public Folder Uploads Caching Notes

A few notes about the the public folders, `public/assets`, `public/images`, etc. These public folders are uploaded to s3 by default as a part of the Jets deploy process.

Even though you and add files directly to these folders and Jets will uploaded and served them.  The files in these folders do not get sha checksums added to their paths automatically. This makes it more difficult to cache the assets with longer TTLs. Even with short TTLs, some browsers and devices like iPhone seem to cache images indefinitely. Asset precompiling will take your assets and add a sha checksum as part of the path.

You should add the files to `app/assets` and `app/javascript` instead and allow Jets to precompile them as part of the deploy process.

Folder | Description | Checksum
--- | --- | ---
public/assets | Where sprockets-jets assets are compiled to. You defined js and css files in `app/assets` and sprockets will compile, add checksums to the file names and save them to `public/assets`. You can also create images have sprockets add the checksum to the images files. | Yes
public/packs | Where `bin/webpack` assets are compiled to. You defined js and css files in `app/javascript/packs` and webpacker will compile, add checksums to the file names and save them to `public/packs`. You can also create images in `app/javascript/images` and have webpacker add the checksum to the images files. | Yes
public/images | Public images. | No
