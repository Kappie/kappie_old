What I'm doing now:

- writing my master's thesis in theoretical physics at the University of Amsterdam
- teaching undergrad problem classes at the University of Amsterdam
- occasionally teaching my co-written programming courses to students and high-schoolers (more information: [Rubyprog](http://rubyprog.nl/))

In the past:

- got a master's degree in software engineering at the University of Amsterdam

![Geert Kapteijns](/geert2.jpg)

## Contact

- Email: [ghkapteijns@gmail.com](mailto:ghkapteijns@gmail.com)

## Posts
<ul id="blog-posts" class="posts">
{% for post in site.posts %}
  <li><span>{{ post.date | date_to_string }} &raquo;</span>
  <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
