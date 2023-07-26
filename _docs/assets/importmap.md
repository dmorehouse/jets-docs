---
title: "Assets Serving: Importmap"
nav_text: Importmap
category: assets
order: 1
---

In Jets v5, sprockets and importmap is the default way to handle assets like stylesheets, javascripts, and images.

## Usage: High Level

In the view, you use the `javascript_importmap_tags` helper:

app/views/layouts/application.html.erb

    <%= javascript_importmap_tags %>

For your source code, you define them in these folders:

    app
    ├── assets
    │   ├── config
    │   │   └── manifest.js
    │   ├── images
    │   ├── javascripts
    │   └── stylesheets
    │       └── application.css
    └── javascript
        └── application.js

The `manifest.js` contain special comment directives that tell sprockets how to build the the final `public/assets/manifest-7c370d953.js` that contain the javascript assets. Example:

app/assets/config/manifest.js

```javascript
//= link_tree ../images
//= link_directory ../stylesheets .css
//= link_tree ../javascripts .js
//= link_tree ../../javascript .js
```

Here's the docs for the directives: [rails/sprockets#directives](https://github.com/rails/sprockets#directives).

## Importmap Loading Trace

To help understand how importmap works, we'll cover in detail how it all works.

In the view, you use a helper:

app/views/layouts/application.html.erb

    <%= javascript_importmap_tags %>

The rendered HTML looks something like this:


    <script type="importmap" data-turbo-track="reload">
    {
      "imports": {
        "application": "/assets/application-561a5525.js",
        "jquery": "https://ga.jspm.io/npm:jquery@3.7.0/dist/jquery.js",
        "@rubyonjets/ujs-compat": "https://ga.jspm.io/npm:@rubyonjets/ujs-compat@1.1.0/index.js"
      }
    }
    </script>
    <link rel="modulepreload" href="/assets/application-561a5525.js">
    <script src="/assets/es-module-shims.min-c6977838.js" async="async" data-turbo-track="reload"></script>
    <script type="module">import "application"</script>

The last line above contains the **single point-of-entry**. Repeated here:

    <script type="module">import "application"</script>

The **importmap** definition is defined at the top. Repeated here as a snippet for clarity and conciseness:

    <script type="importmap" data-turbo-track="reload">
    {
      "imports": {
        "application": "/assets/application-561a5525.js",
        ...
      }
    }
    </script>

As the name suggests, it tells Javascript where to load files when you use the `import` keyword in your javascript source code.

The `import "application"` loads `/assets/application-561a5525.js`, which is the digested version that sprockets created from `app/javascript/application.js`.

**Essentially**: `import "application" => app/javascript/application.js`

## Sprockets Digest Files

The  [sprockets-jets](https://github.com/boltops-tools/sprockets-jets) gem integrates [sprockets](https://github.com/rails/sprockets) with jets. Locally in development mode, sprockets compiles the digest file on-the-fly as part of the request. Sprockets is a "Rack-based asset packaging system". On production, sprockets serves **precompiled** assets.

## Single Point-of-Entry to Javascript World

Once we pass the point-of-entry Javascript takes over. The single point-of-entry is repeated here for clarity:

    import "application" => app/javascript/application.js

Whatever you've defined in your `application.js` is loaded via **pure Javascript**. Example:

app/javascript/application.js

```javascript
import jquery from 'jquery'
window.$ = jquery
import Jets from "@rubyonjets/ujs-compat"
Jets.start()
```

So where does the `jquery` and `@rubyonjets/ujs-compat` come from? Again, back to the original `<script type="importmap">` above. Here's a relevant snippet:

```json
{
  "imports": {
    "application": "/assets/application-561a5525.js",
    "jquery": "https://ga.jspm.io/npm:jquery@3.7.0/dist/jquery.js",
    "@rubyonjets/ujs-compat": "https://ga.jspm.io/npm:@rubyonjets/ujs-compat@1.1.0/index.js"
  }
}
```

Once we pass the single point-of-entry, the cycle repeats itself.

1. Javascript import some file
2. Back to the importmap definition
3. Javascript import some file - possibly continue again until there no more import keywords

It's interesting to note that everything you've seen aside from the initial helper `<%= javascript_importmap_tags %>` done with **pure Javascript**. Everything that's necessary was calculated ahead of time to generate the HTML script importmap tag and kick off the single point-of-entry `import "application"` javascript call.

## Ruby DSL: config/importmap.rb

How did `javascript_importmap_tags` originally generate the map?

It does it from a Ruby DSL.

config/importmap.rb

```ruby
pin "application", preload: true
pin "jquery", to: "https://ga.jspm.io/npm:jquery@3.7.0/dist/jquery.js", preload: true
pin "@rubyonjets/ujs-compat", to: "https://ga.jspm.io/npm:@rubyonjets/ujs-compat@1.1.0/index.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

The DSL is the **source-of-truth** for the importmap. It is your responsibility to add pins to the `config/importmap.rb` when you introduce a new `import MODULE` in your javascript source code. The DSL is how you tell Javascript where `import MODULE` should load the javascript files from.

The `pin` method `to` option points to a URLs above. The `pin` method `to` option can also point to a file name with or without the extension. The files can be in any of the searched `assets.paths`, IE: `app/javascript` and `vendor/javascript`, as defined by the [importmap-jets engine](https://github.com/boltops-tools/importmap-jets/blob/main/lib/importmap_jets/engine.rb#L26-L27).

The `javascript_importmap_tags` helper evaluates the DSL and uses sprockets to generate the digest map ahead of time. This is provided by the [importmap-jets](https://github.com/boltops-tools/importmap-jets) gem. Some useful files to take a look at:

* [app/helpers/importmap/importmap_tags_helper.rb](https://github.com/boltops-tools/importmap-jets/blob/main/app/helpers/importmap/importmap_tags_helper.rb): The `javascript_importmap_tags` helper.
* [lib/importmap/map.rb](https://github.com/boltops-tools/importmap/blob/main/lib/importmap/map.rb): DSL Processing of your `config/importmap.rb`.

## Rails importmap Debugging: CLI and Console

You can see the JSON snippet with the `bin/importmap` CLI:

    ❯ bin/importmap json
    {
      "imports": {
        "application": "/assets/application-561a5525.js",
        "jquery": "https://ga.jspm.io/npm:jquery@3.7.0/dist/jquery.js",
        "@rubyonjets/ujs-compat": "https://ga.jspm.io/npm:@rubyonjets/ujs-compat@1.1.0/index.js"
      }
    }

You can also check it out with the `jets console`

    ❯ jets console
    Jets booting up in development mode!
    > Jets.application.importmap.class
    => Importmap::Map
    > Jets.application.importmap.packages
    =>
    {"application"=>#<struct Importmap::Map::MappedFile name="application", path="application.js", preload=true>,
    "jquery"=>#<struct Importmap::Map::MappedFile name="jquery", path="https://ga.jspm.io/npm:jquery@3.7.0/dist/jquery.js", preload=false>,
    "@rubyonjets/ujs-compat"=>
      #<struct Importmap::Map::MappedFile
      name="@rubyonjets/ujs-compat",
      path="https://ga.jspm.io/npm:@rubyonjets/ujs-compat@1.1.0/index.js",
      preload=false>}
    >

The `Jets.application.importmap` contains the instance of the "drawn" importmap from evaluating the DSL.

## DSL Again: config/importmap.rb

Let's repeat the `importmap.rb` to highlight the usage of `preload: true` and `pin_all_from`.

config/importmap.rb

```ruby
pin "application", preload: true
pin "jquery", to: "https://ga.jspm.io/npm:jquery@3.7.0/dist/jquery.js", preload: true
pin "@rubyonjets/ujs-compat", to: "https://ga.jspm.io/npm:@rubyonjets/ujs-compat@1.1.0/index.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

### modulepreload script tags

In the `importmap.rb` DSL, you can see usage of `preload: true` options. This tells `javascript_importmap_tags` to generate the `<link rel="modulepreload"` tags. IE:

    <link rel="modulepreload" href="/assets/application-561a5525.js">

It does what it sounds like. It preloads the javascript files right when the page loads in parallel. Otherwise, importmap won't load the javascript files until it encounters them serially in `application.js`.

Here's a snippet of the `application.js` repeated for clarity:

app/javascript/application.js

```javascript
import jquery from 'jquery'
// ...
import Jets from "@rubyonjets/ujs-compat"
```

Each `import` makes a network call to load the javascript file. Waiting for each network call and loading them serially would take longer.

### pin_all_from

The `pin_all_from` method in the DSL produces multiple importmap items. Here's an example if files are defined in `app/javascript/controllers`:

    "controllers/application": "/assets/controllers/application-561a5525.js",
    "controllers/hello_controller": "/assets/controllers/hello_controller-549135e8.js",
    "controllers": "/assets/controllers/index-2db729dd.js"

It's all the files within the `app/javascript/controllers` folder.

    app/javascript/controllers/
    ├── application.js
    ├── hello_controller.js
    └── index.js

## Summary

How Javascript importmap loading works:

1. Point of entry: `import "application"`.
2. The importmap from `<script type="importmap">` points `application` to `app/javascript/application.js`.
3. From that point on, it's pure Javascript import and loading.
4. If more `import` are keywords are found in `application.js`, they get loaded via the same importmap defined at the top.
5. The cycle repeats until no more `import` keywords are encountered.

The DSL in `config/importmap.rb` is the source-of-truth and defines where module files should come from. You should update it when you add and use `import` keywords in your javascript files like `app/javascript/application.js`. You can use `bin/importmap pin` and `bin/importmap unpin` to manage `config/importmap.rb`.
