I study physics in Amsterdam. I teach high schoolers how to program with [ruby{prog}](http://rubyprog.nl).

![Geert Kapteijns](/geert.jpg)

## Contact

- Email: [ghkapteijns@gmail.com](mailto:ghkapteijns@gmail.com)
- Programming projects: [GitHub](https://github.com/Kappie/) 

## Posts

<ul id="blog-posts" class="posts">
{% for post in site.posts %}
  <li><span>{{ post.date | date_to_string }} &raquo;</span>
  <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

<small>My page is based on [Solo](https://github.com/chibicode/solo).</small>