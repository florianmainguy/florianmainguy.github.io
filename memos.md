---
layout: default
---

<div class="container menu">
    <ul>
      <li><a href="/about">About</a></li>
      <li><a href="/">Posts</a></li>
	  <li><a href="/memos">Memos</a></li>
    </ul>

    <hr />
</div>

<div class="container content">

  <ul class="post-list">
    {% for memo in site.memos %}
      <li>
        <span class="post-meta"></span>
          <a href="{{ post.url | prepend: site.baseurl }}">
              <div>
                  <span class="index-title">{{ memo.title }}</span>
                  <span class="post-date">{{ memo.date | date: "%b %-d, %Y" }}</span>
              </div>
          </a>
      </li>
    {% endfor %}
  </ul>

</div>
