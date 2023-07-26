assets.base_url | nil | Base url to use to serve assets. IE: https://cloudfront.com/my/base/path. By default this is the s3 website url that jets manages.
assets.cache_control | nil | The cache control expiry. IE: `public, max-age=3600`. Note, `assets.max_age` is a shorter way to set cache_control.
assets.folders | %w[assets images packs] | Folders to assets package and upload to s3
assets.max_age | 3600 | Default max age on assets
assets.webpacker_asset_host | s3_endpoint | Default uses the s3 endpoint url. You can set an explicit value if you need to override. Also if assets.base_url is use, that will be used. Precedence: 1. assets.webpacker_asset_host 2. assets.base_host 3. s3_endpoint