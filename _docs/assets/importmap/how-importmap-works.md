---
title: "How Importmap Works"
nav_text: How It Works
category: assets-importmap
order: 1
---

We'll cover how importmap works.

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
    <script type="module">import "application"</script>

The last line above contains the **single point-of-entry**. Repeated here:

    <script type="module">import "application"</script>

The importmap definition at the top defines the **importmap**. Repeated here without the `data-turbo-track="reload"` for clarity and conciseness:

    <script type="importmap">...</script>

As the name suggests, it tells Javascript where to load javascript files from when using the `import` keyword.

The `import "application"` loads `/assets/application-37f365cb.js`, which is the digested `app/javascript/application.js`.

**Essentially**: `import "application" => app/javascript/application.js`

## Sprockets Digest Files

Note, the digested file is built from the original [sprockets-rails](https://github.com/rails/sprockets-rails) and [sprockets](https://github.com/rails/sprockets) libraries. What is old is now new again 🤣

Locally in development mode sprockets compiles the digest file on-the-fly as part of the request. Remember, sprockets is a "Rack-based asset packaging system".  On production, sprockets serves **pre-compiled** assets. Lastly, spockets-rails integrates sprockets with rails.

## After Point of Entry Javascript Takes Over

Once we pass the point-of-entry **pure Javascript** takes over. The point-of-entry is repeated here for clarity:

    import "application"` => app/javascript/application.js

Whatever you've defined in your `application.js` is loaded via pure Javascript. Example:

app/javascript/application.js

```javascript
import "@hotwired/turbo-rails"
import "controllers"
```

So where does the `@hotwired/turbo-rails` and `controllers` come from?  Again, back to the original `<script type="importmap">` above. Here's a relevant snippet:

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
2. Back to importmap
3. Javascript import some file - possibly again until no more import keywords

## Ruby DSL: config/importmap.rb

How did `javascript_importmap_tags` originally generate the map?

It does it with a Ruby DSL.

config/importmap.rb

```ruby
pin "application", preload: true
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "@hotwired/stimulus", to: "stimulus.min.js", preload: true
pin "@hotwired/stimulus-loading", to: "stimulus-loading.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

The DSL is the **source-of-truth** for the importmap. It is your responsible to add to pins to the `config/importmap.rb` when you introduce new Javascript `import MODULE`, so javascript will know which file to get the module from.

The `javascript_importmap_tags` helper evaluates the DSL and uses sprockets to generate the digest map ahead of time. This is provided by the [importmap-rails](https://github.com/rails/importmap-rails) gem.  Some useful files to take a look at:

* [app/helpers/importmap/importmap_tags_helper.rb](https://github.com/rails/importmap-rails/blob/main/app/helpers/importmap/importmap_tags_helper.rb): The `javascript_importmap_tags` helper.
* [lib/importmap/map.rb](https://github.com/rails/importmap-rails/blob/main/lib/importmap/map.rb): DSL Processing of your `config/importmap.rb`.

## Rails importmap Debugging: CLI and Console

You can see the json snippet with the importmap CLI:

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
    > Rails.application.importmap.preloaded_module_paths(resolver: helper)
    =>
    ["/assets/application-05f01ae8.js",
    "/assets/turbo.min-f309baaf.js",
    "/assets/stimulus.min-d03cf1df.js",
    "/assets/stimulus-loading-1fc59770.js"]
    >

The `Rails.application.importmap` contains the instance of the "drawn" importmap from evaluating the DSL.

## DSL Again: config/importmap.rb

Let's repeat the `importmap.rb` to note the usage of `preload: true` and `pin_all_from`.

config/importmap.rb

```ruby
pin "application", preload: true
pin "@hotwired/turbo-rails", to: "turbo.min.js", preload: true
pin "@hotwired/stimulus", to: "stimulus.min.js", preload: true
pin "@hotwired/stimulus-loading", to: "stimulus-loading.js", preload: true
pin_all_from "app/javascript/controllers", under: "controllers"
```

### modulepreload script tags

In the `importmap.rb` DSL, you can see usage of `preload: true` options. This is what tells `javascript_importmap_tags` to generate the `<link rel="modulepreload"` tags. IE:

    <link rel="modulepreload" href="/assets/application-37f365cb.js">

It does what it sounds like. It preloads the javascript files right when the page loads in parallel. Otherwise, importmap won't load the javascript files until it encounters them serially in `application.js`. Repeated for clarity:

app/javascript/application.js

```javascript
import "@hotwired/turbo-rails"
import "controllers"
```

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
2. The importmap from `<script type="importmap">` essentially points `application` to `app/javascript/application.js`.
3. From that point on, it's pure Javascript import and loading.
4. If more `import` are keywords are found in `application.js`, they get loaded via the same importmap defined at the top.
5. The cycle repeats until there are no more `import` keywords are encountered.

The DSL in `config/importmap.rb` is the source-of-truth and defines where module files should come from. You should update it when you add and use `import` keywords in your javascript files like `app/javascript/application.js`.
