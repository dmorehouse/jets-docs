---
title: "How Importmap Works: Rails"
nav_text: How It Works
category: assets-importmap
order: 1
---

These are some of my notes on how importmap works with Rails, not Jets. They're useful for reference.

In Rails 7, importmap is the new way to serve assets. One of the key benefits of this is that you don't need to use node to compile assets. We'll cover how importmap works.

## Importmap Loading Trace

In the view, you use a helper:

app/views/layouts/application.html.erb

    <%= javascript_importmap_tags %>

The rendered HTML looks something like this:

    <script type="importmap" data-turbo-track="reload">
    {
      "imports": {
        "application": "/assets/application-37f365cb.js",
        "@hotwired/turbo-rails": "/assets/turbo.min-f309baaf.js",
        "@hotwired/stimulus": "/assets/stimulus.min-d03cf1df.js",
        "@hotwired/stimulus-loading": "/assets/stimulus-loading-1fc59770.js",
        "controllers/application": "/assets/controllers/application-368d9863.js",
        "controllers/hello_controller": "/assets/controllers/hello_controller-549135e8.js",
        "controllers": "/assets/controllers/index-2db729dd.js"
      }
    }
    </script>
    <link rel="modulepreload" href="/assets/application-37f365cb.js">
    <link rel="modulepreload" href="/assets/turbo.min-f309baaf.js">
    <link rel="modulepreload" href="/assets/stimulus.min-d03cf1df.js">
    <link rel="modulepreload" href="/assets/stimulus-loading-1fc59770.js">
    <script src="/assets/es-module-shims.min-4ca9b3dd.js" async="async" data-turbo-track="reload"></script>
    <script type="module">import "application"</script> <!-- SINGLE POINT OF ENTRY -->

The last line above contains the **single point-of-entry**. Repeated here:

    <script type="module">import "application"</script>

The **importmap** definition is defined at the top. Repeated here as a snippet for clarity and conciseness:

    <script type="importmap" data-turbo-track="reload">
    {
      "imports": {
        "application": "/assets/application-37f365cb.js",
        ...
      }
    }
    </script>

As the name suggests, it tells Javascript where to load files when you use the `import` keyword in your javascript source code.

The `import "application"` loads `/assets/application-37f365cb.js`, which is the digested version that sprockets created from `app/javascript/application.js`.

**Essentially**: `import "application" => app/javascript/application.js`

## Sprockets Digest Files

Note, the digested file is built from the original [sprockets-rails](https://github.com/rails/sprockets-rails) and [sprockets](https://github.com/rails/sprockets) libraries.

Locally in development mode, sprockets compiles the digest file on-the-fly as part of the request. Sprockets is a "Rack-based asset packaging system". On production, sprockets serves **pre-compiled** assets. Lastly, spockets-rails integrates sprockets with rails.

Some light history: Sprockets is what handles asset packaging for Rails 3. Then the nodejs world evolved and created their own tools like webpack. So Rails introduced webpacker which uses webpack in Rails 5. In Rails 7, Rails is going back to the original sprockets asset packaging. What is old is now new again 🤣

## Single Point-of-Entry to Javascript World

Once we pass the point-of-entry Javascript takes over. The single point-of-entry is repeated here for clarity:

    import "application" => app/javascript/application.js

Whatever you've defined in your `application.js` is loaded via **pure Javascript**. Example:

app/javascript/application.js

```javascript
import "@hotwired/turbo-rails"
import "controllers"
```

So where does the `@hotwired/turbo-rails` and `controllers` come from? Again, back to the original `<script type="importmap">` above. Here's a relevant snippet:

```javascript
{
  "imports": {
    @hotwired/turbo-rails => /assets/turbo.min-f309baaf.js
    controllers => /assets/controllers/index-2db729dd.js
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
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "@hotwired/stimulus", to: "stimulus.min.js", preload: true
pin "@hotwired/stimulus-loading", to: "stimulus-loading.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

The DSL is the **source-of-truth** for the importmap. It is your responsibility to add pins to the `config/importmap.rb` when you introduce a new `import MODULE` in your javascript source code. The DSL is how you tell Javascript where `import MODULE` should load the javascript files from.

The `pin` method `to` option points to a file name with or without the extension. The files can be in any of the searched `assets.paths`, IE: `app/javascript` and `vendor/javascript`, as defined by the [importmap-rails engine](https://github.com/rails/importmap-rails/blob/9eec49a9ea3feaab224871437cf1bc2479801796/lib/importmap/engine.rb#L47-L48).

The `javascript_importmap_tags` helper evaluates the DSL and uses sprockets to generate the digest map ahead of time. This is provided by the [importmap-rails](https://github.com/rails/importmap-rails) gem. Some useful files to take a look at:

* [app/helpers/importmap/importmap_tags_helper.rb](https://github.com/rails/importmap-rails/blob/main/app/helpers/importmap/importmap_tags_helper.rb): The `javascript_importmap_tags` helper.
* [lib/importmap/map.rb](https://github.com/rails/importmap-rails/blob/main/lib/importmap/map.rb): DSL Processing of your `config/importmap.rb`.

## Rails importmap Debugging: CLI and Console

You can see the JSON snippet with the `bin/importmap` CLI:

    ❯ bin/importmap json
    {
      "imports": {
        "application": "/assets/application-05f01ae8.js",
        "@hotwired/turbo-rails": "/assets/turbo.min-f309baaf.js",
        "@hotwired/stimulus": "/assets/stimulus.min-d03cf1df.js",
        "@hotwired/stimulus-loading": "/assets/stimulus-loading-1fc59770.js",
        "controllers/application": "/assets/controllers/application-368d9863.js",
        "controllers/hello_controller": "/assets/controllers/hello_controller-549135e8.js",
        "controllers": "/assets/controllers/index-2db729dd.js"
      }
    }

You can also check it out with the `rails console`

    ❯ rails console
    Loading development environment (Rails 7.0.5)
    > Rails.application.importmap.class
    => Importmap::Map
    > Rails.application.importmap.packages
    =>
    {"application"=>#<struct Importmap::Map::MappedFile name="application", path="application.js", preload=true>,
    "@hotwired/turbo-rails"=>#<struct Importmap::Map::MappedFile name="@hotwired/turbo-rails", path="turbo.min.js", preload=true>,
    "@hotwired/stimulus"=>#<struct Importmap::Map::MappedFile name="@hotwired/stimulus", path="stimulus.min.js", preload=true>,
    "@hotwired/stimulus-loading"=>
      #<struct Importmap::Map::MappedFile name="@hotwired/stimulus-loading", path="stimulus-loading.js", preload=true>,
    "jquery"=>#<struct Importmap::Map::MappedFile name="jquery", path="jquery.js", preload=false>}
    > Rails.application.importmap.preloaded_module_paths(resolver: helper)
    =>
    ["/assets/application-05f01ae8.js",
    "/assets/turbo.min-f309baaf.js",
    "/assets/stimulus.min-d03cf1df.js",
    "/assets/stimulus-loading-1fc59770.js"]
    > puts Rails.application.importmap.to_json(resolver: helper)
    {
      "imports": {
        "application": "/assets/application-05f01ae8ad58cce103547a14e3295d1a03f1dda93058015ba3afac46c1526dd9.js",
        "@hotwired/turbo-rails": "/assets/turbo.min-f309baafa3ae5ad6ccee3e7362118b87678d792db8e8ab466c4fa284dd3a4700.js",
        "@hotwired/stimulus": "/assets/stimulus.min-d03cf1dff41d6c5698ec2c5d6a501615a7a33754dbeef8d1edd31c928d17c652.js",
        "@hotwired/stimulus-loading": "/assets/stimulus-loading-1fc59770fb1654500044afd3f5f6d7d00800e5be36746d55b94a2963a7a228aa.js",
        "jquery": "/assets/jquery-6059b4a1dbd223b4e85b70257b824c686de103cd2db58636e18cdec46f1aab6e.js",
        "controllers/application": "/assets/controllers/application-368d98631bccbf2349e0d4f8269afb3fe9625118341966de054759d96ea86c7e.js",
        "controllers/hello_controller": "/assets/controllers/hello_controller-549135e8e7c683a538c3d6d517339ba470fcfb79d62f738a0a089ba41851a554.js",
        "controllers": "/assets/controllers/index-2db729dddcc5b979110e98de4b6720f83f91a123172e87281d5a58410fc43806.js"
      }
    }
    >
The `Rails.application.importmap` contains the instance of the "drawn" importmap from evaluating the DSL.

## DSL Again: config/importmap.rb

Let's repeat the `importmap.rb` to highlight the usage of `preload: true` and `pin_all_from`.

config/importmap.rb

```ruby
pin "application", preload: true
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "@hotwired/stimulus", to: "stimulus.min.js", preload: true
pin "@hotwired/stimulus-loading", to: "stimulus-loading.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

### modulepreload script tags

In the `importmap.rb` DSL, you can see usage of `preload: true` options. This tells `javascript_importmap_tags` to generate the `<link rel="modulepreload"` tags. IE:

    <link rel="modulepreload" href="/assets/application-37f365cb.js">

It does what it sounds like. It preloads the javascript files right when the page loads in parallel. Otherwise, importmap won't load the javascript files until it encounters them serially in `application.js`.

Here's a snippet of the `application.js` repeated for clarity:

app/javascript/application.js

```javascript
import "@hotwired/turbo-rails"
import "controllers"
```

Each `import` makes a network call to load the javascript file. Waiting for each network call and loading them serially would take longer.

### pin_all_from

The `pin_all_from` method in the DSL produces multiple importmap items:

    "controllers/application": "/assets/controllers/application-368d9863.js",
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
