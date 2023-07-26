assets.base_url | nil | Base url to use to serve assets. IE: https://cloudfront.com/my/base/path. Default uses the s3 endpoint url that jets manages. You can set an explicit value if you need to override and want to use a CDN. Precedence: 1. assets.base_url 32 s3_endpoint (jets managed s3 bucekt)
assets.cache_control | nil | The cache control expiry. IE: `public, max-age=3600`. Note, `assets.max_age` is a shorter way to set cache_control.
assets.folders | `["assets", "images", "packs"]` | Folders to assets package and upload to s3
assets.max_age | 3600 | Default max age on assets
assets.paths | `["app/assets/config","app/assets/images","app/assets/javascripts","app/assets/stylesheets"]` | Sprockets-related. Search path for sprockets to lookup asset files.
assets.precompile | [] | Sprockets-related.
assets.prefix | "/assets" | Sprockets-related.
assets.use_precompiled | false | Sprockets-related. For development false, for production true. Assets are precompiled and uploaded to s3 for Jets to serve from.