{% assign postsByYear = site.posts | group_by_exp:"post", "post.date | date: '%Y'" %}
{% for year in postsByYear %}
  <h1>{{ year.name }}</h1>
  {% assign postsByMonth = year.items | group_by_exp:"post", "post.date | date: '%B'" %}

{% for month in postsByMonth %}
<h2>{{ month.name }}</h2>
<ul>
  {% for post in month.items %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <br>{{ post.excerpt }}<br><br>
    </li>
  {% endfor %}
</ul>

{% endfor %}
{% endfor %}

---
Check out my latest book: [Combine: Asynchronous Programming with Swift](https://store.raywenderlich.com/a/742/link/27)

[![](Assets/CombineBook.png)](https://store.raywenderlich.com/a/742/link/27" alt="Combine: Asynchronous Programming with Swift)

It's packed with coverage of Combine concepts, hands-on exercises in playgrounds, and complete iOS app projects. Everything you need to _transform yourself_ from novice to expert with Combine — and have fun while doing it!

<sub><a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.</sub>
<br><sub>© 2020 Scott Gardner</sub><br>
