
<!--
Creator: Team
Last Edited by: Brianna
Location: SF
-->
![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# Rails Asset Pipeline

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->
*This workshop is important because:*

The asset pipeline reduces load time for users. Also, it's important to use it correctly if you want your site to work with many other gems that modify the front end. 

> The asset pipeline provides a framework to concatenate and minify or compress JavaScript and CSS assets. It also adds the ability to write these assets in other languages and pre-processors such as CoffeeScript, Sass and ERB. It allows assets in your application to be automatically combined with assets from other gems.

- [The Asset Pipeline Rails Guide](http://guides.rubyonrails.org/asset_pipeline.html)


### What are the objectives?
<!-- specific/measurable goal for students to achieve -->
*After this workshop, developers will be able to:*

- Describe how the Rails Asset Pipeline works and why we use it.
- Require custom and third-party assets in Rails. 
- Precompile assets in development to debug asset issues in production.


### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Before this workshop, developers should already be able to:*

- Build HTML pages with a templating language like ERB, Handlebars, EJS, Jade, etc. 
- Explain how JavaScript and CSS contribute to the front end of a website. 
- Explain why the order in which JavaScript or CSS files are loaded matters. 

## Asset Pipeline Overview

**The goal of the asset pipeline is to:**

* Compress assets (CSS and JavaScript files) so they are as small as possible, since smaller files load faster.
* Reduce the number of files transferred to the client, by:
  1. Combining files before sending them to the browser.
    * All CSS files are combined into a single `application.css` file.
    * All JavaScript files are combined into a single `application.js` file.
  2. Caching files in the browser.  
* Facilitate use of languages like SASS and CoffeeScript, which compile into CSS or JavaScript.

## How the Rails Asset Pipeline Works

**Have you ever done somethint like this?** 

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <script type="text/javascript" src="vendor/scripts/jquery-3.1.1.min.js"></script>
  <script type="text/javascript" src="vendor/scripts/handlebars-v4.0.5.js"></script>
  <script type="text/javascript" src="app.js"></script>
  <title>Load things!</title>
</head>
<body>
</body>
</html>
```
In the chrome network tab you see something like: 

![chrome network tab](assets/images/chrome_network_tab.png "chrome network tab")

That's three different requests to the server, for three different files. And that's just for JavaScript.  See how that time adds up? 

In Rails, the asset pipeline is a better way. Here's how Rails requires files, using the asset pipeline:

```html
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>
  <head>
    <!-- sign the page for security -->
    <%= csrf_meta_tags %>
    <!-- application stylesheet -->
    <%= stylesheet_link_tag :application, media: :all %>
    <!-- application javascript file -->
    <%= javascript_include_tag :application %>
    <title>Load things with Rails Asset Pipeline!</title>
  </head>
  <body>
  </body>
</html>
```

That's just two files, **one** for JavaScript and **one** for CSS! 

In Rails, the asset pipeline:

* concatentates files together
* minifies them
* compresses them 

> Note: There are similar tools for JS backends as well, but they usually require more setup than Rails does.


####Check for Understanding

If your site has one JavaScript and one CSS file that are linked in the `<head>` of `app/views/layouts/application.html.erb`, which pages will those scripts and styles apply to?

  <details><summary>click for answer</summary>
  **All** of your JavaScript and CSS is active on **EVERY PAGE**.  So is your CSS.  When writing code for a specific page, you need to think about whether it will affect other pages on the site.
  </details>


### Concatenation and The Manifest

With the Asset Pipeline, instead of manually adding `<script>` tags, you're going to use `<%= javascript_include_tag :application %>` in your application's `<head>`. Then, you'll specify what exactly that "includes" in a file called `app/assets/javascripts/application.js`. 

Inside this file, there's a weird looking series of comments called a "manifest":

``` javascript
// app/assets/javascripts/application.js
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require_tree .
```

Actually, **these aren't comments!** This manifest file lists a series of instructions saying *which files* need to be loaded in your HTML, and *in what order*.  With these instructions, a gem called `sprockets-rails` loads the files specified, processes them if necessary, **concatenates** them into one single file, and then compresses them (if `Rails.application.config.assets.compress` is `true`).

The Asset Pipeline will look for the name of the file (e.g. `jquery`) in the following directories:

1. `app/assets/` - application-specific code
2. `lib/assets/` - custom libraries not specific to this app
3. `vendor/assets/` - third-party libraries

After Rails processes all the files, it replaces the `<%= javascript_include_tag :application %>` tag in `app/views/layouts/application.html.erb` with a link to the new file it generates.  It does the same with the `stylesheet_link_tag`.  

> Note that by default, Rails links both CSS and JavaScript in the `<head>`.  You can JavaScript to the bottom of the `<body>` to make sure it doesn't slow down the loading of your HTML.

####Check for Understanding

1. Why is it important to specify the order of files in a JavaScript or CSS manifest?

1. Which file do you think is the CSS manifest?

  <details><summary>click for answer</summary>
  It's `app/assets/stylesheets/application.css`, and by default it looks like this:
  
  ```css
   /* inside app/assets/stylesheets/application.css
   *= require_tree .
   *= require_self
   */
  ```
  
  It determines the load order of CSS files just like the JavaScript manifest determines the order of JavaScript files. 
  </details>
  
  
#### Images

In views, you can access images in the `public/assets/images` directory like this: `<%= image_tag "rails.png" %>`.

Images can also be organized into subdirectories, and then can be accessed by specifying the directory's name in the tag: `<%= image_tag "icons/rails.png" %>`.

#### ERB and Asset Path Helpers

The asset pipeline automatically evaluates ERB. This means if you add a `.erb` extension to a CSS asset (for example, `application.css.erb`), then helpers like [`asset_path`](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetUrlHelper.html) are available in your CSS rules:

```css
.header {
  background-image: url(<%= asset_path 'image.png' %>)
}
```

This writes the path to the particular asset being referenced. In this example, it would make sense to have an image in one of the asset load paths, such as `app/assets/images/image.png`, which would be referenced here. If this image is already available in `public/assets` as a fingerprinted file, then that path is referenced. You don't have to guess the fingerprint.

Similarly, if you add a `.erb` extension to a JavaScript asset (like `application.js.erb`), you can then use the `asset_path` helper in your JavaScript code:

```js
$('#logo').attr({ src: "<%= asset_path('logo.png') %>" });
```




## Caching

Manifests help Rails know how to order all of the different files and combine them into one. But Rails does something with the file name tha tmight seem odd. It adds a large string of random-looking characters in production. For example, you might see a CSS file called `application-908e25f4bf641868d8683022a5b62f54.css`.   Why not just call it `application.css`?

As you know, web applications can be configured to "cache" files in the client's browser, including JavaScript and CSS.

![cached file](assets/images/headers_cache.png "cached file")

Cached files don't need to be requested *on every page load*. They're already there, ready to go. This means **less wait time** and **faster page loads**.


#### Cache-Busting with Fingerprints

By default, Rails enables caching. 

But what if you changed the file, and the browser was still using an older version? This is where "fingerprinting" comes in - it gives us a way to bust the cache! 

In Rails, assets are given a "fingerprint" that changes every time the file is updated. The asset pipeline  inserts a hash of the file contents into the file name itself. That's why we'll see file names like `application-908e25f4bf641868d8683022a5b62f54.js` instead of `application.js`.

**Cache-busting works like this:**

* If the fingerprint is the same, the browser simply uses its cached copy.
* If the fingerprint has changed, the browser requests the new version of the file (and then caches it!).


### Turbolinks

You may have noticed a line in a new Rails app's `app/assets/javascripts/application.js` that says to `//= require turbolinks`.

Turbolinks is a gem that ships with Rails and that eliminates page refreshes when navigating around your app in the browser. 

This sounds pretty cool, but the downside is that it only loads your assets once, on the first page-load, then it never loads them again.  This means  **any jQuery events you have set to run on page load won't work on subsequent pages**.  Turbolinks gives developers new events to listen to in order to get around this, like `"turbolinks:load"`.  However, most apps the size we're working with won't need this added complexity. Plus, it won't hurt to practice jQuery with regular events over the next few weeks. 

**We suggest removing turbolinks for the time being.**

To create an app without turbolinks:  

1. Run the `rails new` command with an additional `--skip-turbolinks` option.

To remove turbolinks from an existing app:

1. Remove `'data-turbolinks-track' => true` from `<%= stylesheet_link_tag 'application' %>` and `<%= javascript_include_tag 'application' %>` in `app/views/layouts.application.html.erb`.
2. Remove `//= require turbolinks` from `app/assets/javascripts/application.js`.
3. Delete `gem 'turbolinks'` from your `Gemfile`, and `bundle` again.




## Compression / Minification / Uglification

In addition to putting all the JavaScript in one file and naming it based on contents, the Asset Pipeline compresses this file so it's smaller. 

#### Check for Understanding 

1. What's the difference between <a href="https://code.jquery.com/jquery-1.11.3.js">jquery-1.11.3.js</a> and <a href="https://code.jquery.com/jquery-1.11.3.min.js">jquery-1.11.3.min.js</a>?

1. Use the `curl` command to do a quick comparison on the command line:

  ```zsh
  # uncompressed jQuery file
  ➜  curl https://code.jquery.com/jquery-1.11.3.js | wc
  #   lines   words   bytes
  #   10351   43235   284394

  # compressed (minified) jQuery file
  ➜  curl https://code.jquery.com/jquery-1.11.3.min.js | wc
  #   lines   words   bytes
  #   5       1413    95957
  ```

Rails also compresses CSS this way. 

## Precompiling Assets

All of these techniques make Rails faster, but in the Rails development environment many of the above techniques are **not enabled**.  We can try them out just to see what it's like. 

Take a look at the instructions for [Precompiling Assets](precompile_assets.md) for a way to try out the full power of the asset pipeline in your development environment.

> Note: You will probably not precompile assets by hand in WDI.


## What about CDNs?

A CDN is a "content delivery network" and a handy way to deliver common "vendor" or "third-party" libraries to your application. It's common to use a CDN for jQuery, Bootstrap, Handlebars, etc.

But is it fast?

If a common file like jQuery is delivered via CDN it is likely:

  1. Cached in your browser.
  2. Cached by your ISP (Internet Service Provider).
  3. Dispatched from a nearby server.

But is that faster than just sending one JavaScript file from your own server?

It could be! It's more likely for very widespread libraries that your user's browser might have cached already. You'll have to make a decision about whether you want to host third-party libraries (like jQuery) on your server, or if you want to use a public CDN.


## Challenges

* See [exercises](exercises.md)

## Resources

* <a href="http://guides.rubyonrails.org/asset_pipeline.html">Rails Guides: Asset Pipeline</a>
* <a href="https://github.com/rails/sprockets#sprockets-directives">Sprockets Directives</a>
* List of [view helpers for asset tags](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html).
* [Caching in Rails](http://guides.rubyonrails.org/caching_with_rails.html)
* See [Controller-Specific Assets](http://guides.rubyonrails.org/asset_pipeline.html#controller-specific-assets) and/or [Working with JS in Rails](working-with-js.md) for strategies on scoping your JS.
* See [Additional Reading](additional-reading.md) for a more in-depth discussion of view helpers, assets and **disabling turbolinks**.
