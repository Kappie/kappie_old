Ik ben natuurkundestudent aan de UvA. Met [ruby{prog}](http://rubyprog.nl) probeer ik middelbare scholieren aan het programmeren te krijgen.

![Geert Kapteijns](/geert.jpg)

## Contact

- Email: [ghkapteijns@gmail.com](mailto:ghkapteijns@gmail.com)
- Programmeerprojecten: [GitHub](https://github.com/Kappie/) 

## Berichten

<ul id="blog-posts" class="posts">
{% for post in site.posts %}
  <li><span>{{ post.date | date_to_string }} &raquo;</span>
  <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>

<small>Mijn pagina is gebaseerd op [Solo](https://github.com/chibicode/solo).</small>