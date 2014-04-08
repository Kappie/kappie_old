---
layout: post
title: Jekyll, Sass and Coffee (without plugins) 
---

I improved my workflow by experimenting with Jekyll setups. Type this in:

    gem install foreman

[Foreman](http://theforeman.org/) manages processes from a single command. All you need to do is make a `Procfile` in your directory root. It's a plain text file:
    
    jekyll:  jekyll serve --watch
    compass: compass watch
    coffee: coffee --watch --compile --output javascripts/ _coffeescripts/

Type `foreman start` in directory root and you see:

    23:17:52 jekyll.1  | started with pid 29759
    23:17:52 compass.1 | started with pid 29761
    23:17:52 coffee.1  | started with pid 29762

Not a worry about recompiling `.coffee` or `.sass` or rebuilding the site.

### Setting up
Are you familiar with [Compass](http://compass-style.org/) and [CoffeeScript](http://coffeescript.org/)?

If not, check them out. Compass is a css framework that uses [Sass](http://sass-lang.com/), a delightful language that compiles into css. CoffeeScript is equally delightful and compiles into JavaScript. The good parts.

Let's examine the Procfile. 

    jekyll:  jekyll serve --watch

This rebuilds the site. Done that before. Note that `jekyll` is just a name. `jekyll serve --watch` is the command.

    compass: compass watch

This watches all your `.sass` files and compiles them into `.css` files in the appropriate folder. You need a `config.rb` file in the directory root to specify those folders. It looks like this:

{% highlight ruby %}
http_path = "/"
css_dir = "css"
sass_dir = "_sass"
images_dir = "images"
javascripts_dir = "javascripts"

# leave this out if you prefer the .scss syntax
preferred_syntax = :sass
{% endhighlight %}

In my directory root, I have folders `_sass` and `css`. In `_sass`, I have a file called `main.sass`. Here I import all my other `.sass` files, which start with an underscore, so that `compass` doesn't compile them. `main.sass` might look like this:

    @import normalize
    @import variables

It requires the files `_normalize.sass` and `_variables.sass`. You don't need to type the underscores. `main.sass` compiles to `main.css` in the css folder.

    coffee: coffee --watch --compile --output javascripts/ _coffeescripts/

This command watches all `.coffee` files in the folder `_coffeescripts`. Its output goes into `javascripts`.

If only everything in life was so simple.

**View code [on GitHub](https://github.com/Kappie/jekyll_scaffold)**.