---
layout: post
title: How I built this website - Pt. 2
subtitle: A tutorial on integrating Jekyll, GitHub Pages, and Travis CI
---

Specifically, how I used Jekyll and GitHub Pages to generate my website and how to automate builds and deployments with Travis CI. On this part, I'll go over the layouts and included page snippets necessary to populate every page.

## Default layout

How about we create a baseplate for your pages? Let's start with the default layout. If you've done everything corretly so far, your `default.html` should look like this:

{% highlight html %}
{% raw %}
---
layout: compress
---

<html>
{{ content }}
</html>
{% endraw %}
{% endhighlight %}

Not very exciting. Let's scaffold a proper HTML website. Start by replacing the contents of this file with the following:

{% highlight html %}
{% raw %}
---
layout: compress
---

<!DOCTYPE html>
<html lang="{{ page.lang | default: site.lang | default: "en" }}">
  <head>

    <!-- Non-nocial metatags -->
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <meta name="theme-color" content="#bb1f39">

    <!-- Social metatags -->
    {% include metatags.html %}

    {% if page.title %}
    {% assign page-title = page.title | escape %}
    {% else %}
    {% assign page-title = site.title | escape %}
    {% endif %}
    <title>{{ page-title }}</title>

    <link rel="canonical" href="{{ page.url | replace:'index.html','' | absolute_url }}">

    <link rel="shortcut icon" href="{{ site.url }}/favicon.ico">
    <meta name="robots" content="noarchive">
  </head>
  <body>

    {% include navbar.html %}
    {% include header.html %}

    <div class="container">
      <section class="main-content">
        {{ content }}
      </section>
    </div>

    {% include footer.html %}
  </body>
</html>
{% endraw %}
{% endhighlight %}

You will get a few build errors complaining about missing files. That's expected, since I use a few import tags in this snippet, but we have no files corresponding to these tags. Let me first do a quick overview of what you just pasted, skipping the `include` tags.

---

{% highlight html %}{% raw %}<html lang="{{ page.lang | default: site.lang | default: "en" }}">{% endraw %}{% endhighlight %}
This line tell the browser the language of your website. It defaults to `en` if neither `page.lang` nor `site.lang` are set. You can specify a custom, non-English language in your `_config.yml` by setting the `lang` tag or in your post's front matter.

---

{% highlight html %}{% raw %}<meta name="theme-color" content="#bb1f39">{% endraw %}{% endhighlight %}
This line tells your phone's browser what color to use when filling the status bar. You can remove it if you want, it's not really necessary.

---

{% highlight html %}{% raw %}
{% if page.title %}
{% assign page-title = page.title | escape %}
{% else %}
{% assign page-title = site.title | escape %}
{% endif %}
<title>{{ page-title }}</title>
{% endraw %}{% endhighlight %}
This whole construct dynamically assigns each page's title, if it has one set, or just defaults to the site-wide title. Pretty neat.

---

{% highlight html %}{% raw %}<link rel="canonical" href="{{ page.url | replace:'index.html','' | absolute_url }}">{% endraw %}{% endhighlight %}
This is some light SEO optimization, it tells the browser what the "default" or "source" URL should be. It prevents duplicate pages from harming your SEO score.

---

{% highlight html %}{% raw %}<link rel="shortcut icon" href="{{ site.url }}/favicon.ico">{% endraw %}{% endhighlight %}
Tells your browser/phone what icon to use when adding a shortcut to the home screen.

---

{% highlight html %}{% raw %}<meta name="robots" content="noarchive">{% endraw %}{% endhighlight %}
Asks (politely) that search engines crawlers do not cache your website. Useful to prevent stale caches and outdated articles from sticking around.

---

{% highlight html %}{% raw %}
<div class="container">
  <section class="main-content">
    {{ content }}
  </section>
</div>
{% endraw %}{% endhighlight %}
The meat and potatoes of the whole template. It tells Jekyll where to insert yor content. These CSS classes will be useful later on, when we add Semantic UI as our CSS framework.

---

## Includes

Now that we've covered what everything in our baseplate does, let's take a look at the `include` tags. There are 4 of them in that layout:

{% highlight liquid %}{% raw %}
{% include metatags.html %}
{% include navbar.html %}
{% include header.html %}
{% include footer.html %}
{% endraw %}{% endhighlight %}

> This means that the content of these 4 referenced files should be inserted where the tag is, verbatim. These files should be saved inside (surprise, surprise) `_includes`.

I'll start off with the `metatags.html` file. Head over to [this demo's repository](https://github.com/Avyiel/blog-demo/blob/master/_includes/metatags.html), and copy its contents to your own `metatags.html`. I won't go into detail, but this include helps other websites display information about your blog, such as title, description, author, etc. You can skip this include if you don't feel like integrating it, but it doesn't hurt to have these tags.

### Navbar

Let's take a look at the navbar now. I opted for an auto-generated list of pages, so that I don't have to manually input them as I add top-level pages. Start by copying the following code to your `navbar.html`.

{% highlight html %}
{% raw %}
{% assign default_paths = site.pages | map: "path" %}
{% assign page_paths = site.header_pages | default: default_paths %}

<div class="container">
  <nav class="navbar navbar-expand-md navbar-light bg-white">
    <a class="navbar-brand" href="{{ "/" | relative_url }}">{{ site.title | escape }}</a>
    <button
      class="navbar-toggler"
      type="button"
      data-toggle="collapse"
      data-target="#navbarSupportedContent"
      aria-controls="navbarSupportedContent"
      aria-expanded="false"
      aria-label="Toggle navigation"
    >
      <span class="navbar-toggler-icon"></span>
    </button>

  {% if page_paths %}
  <div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav mr-auto">
      {% for path in page_paths %}
        {% assign my_page = site.pages | where: "path", path | first %}
        {% if my_page.title %}{% unless my_page.title contains "404" %}
          {% if my_page.url == page.url %}
          <li class="nav-item active">
            <a class="nav-link" href="{{ my_page.url | relative_url }}">{{ my_page.title | escape }}
              <span class="sr-only">(current)</span>
            </a>
          </li>
          {% else %}
            <li class="nav-item">
              <a class="nav-link" href="{{ my_page.url | relative_url }}">{{ my_page.title | escape }}</a>
            </li>
          {% endif %}
        {% endunless %}{% endif %}
      {% endfor %}
      </ul>
    </div>
    {% endif %}
  </nav>
</div>
{% endraw %}
{% endhighlight %}

> Simply add a title and permalink, and that page is now added to the navbar.

This automatically skips paths such as images, stylesheets and blog posts without a permalink. Useful for adding new static pages like an "About" page, or a "Contact", or a "Projects" page. On mobile, these links fold into a hamburger menu. Hurray responsiveness!

### Header

Next on this list is the `header.html` include. This is an important one, since it's usually the most eye-catching thing people see when they first load in. I opted for a large banner-like section in my college's colors, and it's where I put my title, subtitle and other informational tidbits like publication date.

On the home page, it's where my profile picture sits, alongside my name, college degree and (predicted) graduation year, as well as some important links (GitHub, LinkedIn and contact info). I plan to have my resumè in this section at some point in the future.

So, let's see some code:

{% highlight html %}
{% raw %}
<header class="site-header" role="banner">
  {% if page.layout == 'home' %}

  <div class="jumbotron jumbotron-fluid header header-home">
    <div class="container">

      <div class="row">
        <div class="col-12 col-md-4 col-lg-3 mb-3 mb-md-0">
          <img id="profile" src="{{ site.baseurl }}/assets/images/profile.jpg">
        </div>
        <div class="col-12 col-md-8 col-lg-9">
          <h1>{{ site.title }}</h1>
          <h2>{{ site.subtitle }}</h2>

          <!-- Action Buttons -->
          <ul class="action list-inline banner-social-buttons">
            <li class="list-inline-item">
              <a class="btn btn-dark" href="mailto:me@lucasvienna.dev">
                <i class="fas fa-envelope"></i> Email
              </a>
            </li>
            <li class="list-inline-item">
              <a class="btn btn-dark" href="{{ site.linkedin.url }}">
                <i class="fab fa-linkedin"></i> LinkedIn
              </a>
            </li>
            <li class="list-inline-item">
              <a class="btn btn-dark" href="{{ site.github.url }}">
                <i class="fab fa-github"></i> GitHub
              </a>
            </li>
          </ul>

          {% if page.layout == 'home' and site.github.is_project_page and site.show_github %}
          <a href="{{ site.github.repository_url }}" class="btn">View on GitHub</a>
          {% if site.show_downloads %}
          <a href="{{ site.github.zip_url }}" class="btn">Download .zip</a>
          <a href="{{ site.github.tar_url }}" class="btn">Download .tar.gz</a>
          {% endif %}
          {% endif %}
        </div>
      </div>
    </div>
  </div>

  {% else %}

  <div class="jumbotron jumbotron-fluid header header-page">
    <div class="container">
      <h1 class="project-name">{{ page.title }}</h1>
      <h5 class="project-tagline">{{ page.subtitle }}</h5>

      <!-- Post tagline -->
      {% if page.layout == 'post' %}
      <h2 class="project-date">
        <time datetime="{{ page.date | date_to_xmlschema }}" itemprop="datePublished">
          {% assign date_format = site.date_format | default: "%b %-d, %Y" %}
          {{ page.date | date: date_format }}
        </time>
      </h2>
      {% endif %}
      <!-- End: Post tagline -->
    </div>
  </div>

  {% endif %}
</header>
{% endraw %}
{% endhighlight %}

At this point, you should be able to recognize what's going on here. The include has a conditional that determines whether we're in the home page or not, and adjusts accordingly. It also checks for a `post` layout to determine whether or not to add the post tagline. Both pages have a title and subtitle, although I use the global `site.title` in the home, whereas regular pages get `page.title`.

The action buttons pull their URLs from your `_config.yml`, so make sure the information there is correct. It is done like this because these URLs are used in other parts of the site as well (spoiler: the footer).

Like before, some CSS classes here are not part of Bootstrap, and instead come from my custom SCSS files (which I'll cover in Part 3). You can ignore them for now, but do not remove them, since we'll be using these classes later on.

### Footer

The last of our includes is the footer. It is a simple affair, with links to my GitHub page, my twitter (although I do not use it regularly) and my "About" page. Check out the code below:

{% highlight html %}
{% raw %}
<footer class="site-footer text-center">
  <div>
    <!-- SVG icons from https://iconmonstr.com -->
    <!-- Github icon -->
    <span class="footer-icon">
      <a
        href="{{ site.github.url }}"
        title="{{ site.github.username }}'s GitHub"
        aria-label="{{ site.github.username }}'s GitHub"
      >
        <svg class="footer-svg-icon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
          <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z" />
        </svg>
      </a>
    </span>

    <!-- Twitter icon -->
    <span class="footer-icon">
      <a
        href="{{ site.twitter.url }}"
        title="{{ site.twitter.handle }}'s Twitter"
        aria-label="{{ site.twitter.handle }}'s Twitter"
      >
        <svg class="footer-svg-icon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
          <path d="M12 0c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm6.066 9.645c.183 4.04-2.83 8.544-8.164 8.544-1.622 0-3.131-.476-4.402-1.291 1.524.18 3.045-.244 4.252-1.189-1.256-.023-2.317-.854-2.684-1.995.451.086.895.061 1.298-.049-1.381-.278-2.335-1.522-2.304-2.853.388.215.83.344 1.301.359-1.279-.855-1.641-2.544-.889-3.835 1.416 1.738 3.533 2.881 5.92 3.001-.419-1.796.944-3.527 2.799-3.527.825 0 1.572.349 2.096.907.654-.128 1.27-.368 1.824-.697-.215.671-.67 1.233-1.263 1.589.581-.07 1.135-.224 1.649-.453-.384.578-.87 1.084-1.433 1.489z" />
        </svg>
      </a>
    </span>

    <!-- Contact icon -->
    {% assign about_page = site.pages | where: "name", "about.md" | first %}
    {% if about_page.title %}
    <span class="footer-icon">
      <a
        href="{{ about_page.url | relative_url }}"
        title="Contact {{ site.owner }}"
        aria-label="Contact"
      >
        <svg class="footer-svg-icon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
          <path d="M12 .02c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm6.99 6.98l-6.99 5.666-6.991-5.666h13.981zm.01 10h-14v-8.505l7 5.673 7-5.672v8.504z" />
        </svg>
      </a>
    </span>
    {% endif %}
  </div>

  <span class="footer-credits">This page was generated by <a href="https://pages.github.com">GitHub Pages</a>.</span>
  <div class="footer-owner">&copy; {{ site.owner }}, 2019</div>
</footer>
{% endraw %}
{% endhighlight %}

As you can see, a very simple snippet. We have three links with icons and two lines of text, one regarding GH Pages and the other saying who this website belongs to.

## Page Layout

With the includes done, we're left with a few layouts before Jekyll will stop complaining about missing things. Let's start with the `page` layout, as it is the simplest one.

{% highlight html %}
{% raw %}
---
layout: default
---
<article>

  <div>
    {{ content }}
  </div>

</article>
{% endraw %}
{% endhighlight %}

Yeah, that's all. Just an article tag with content inside. Our regular pages are just that, regular.

## Post Layout

A more interesting layout, but still very simple. It lightly implements a `BlogPosting` schema by using the `articleBody` prop, and lays the groundwork for using Disqus, if you need comments in your posts.

{% highlight html %}
{% raw %}
---
layout: default
---
<article itemscope itemtype="http://schema.org/BlogPosting">

  <div itemprop="articleBody">
    {{ content }}
  </div>

  {% if site.disqus.shortname %}
    {% include disqus_comments.html %}
  {% endif %}
</article>
{% endraw %}
{% endhighlight %}

## Home Layout

And the last layout, the home page. It really is more of the same, with a neat little post list.

{% highlight html %}
{% raw %}
---
layout: default
---

<div>

  {{ content }}

  <h2>Latest Posts</h2>

  <div>&nbsp;</div>

  <ul class="post-list">
    {% for post in site.posts %}
    <li>

      {% assign date_format = site.date_format | default: "%b %-d, %Y" %}
      <span class="post-meta">{{ post.date | date: date_format }}</span>

      <h2>
        <a class="post-link" href="{{ post.url | relative_url }}" title="{{ post.title }}">{{ post.title | escape }}</a>
      </h2>

      <span>{{ post.excerpt | markdownify | truncatewords: 50 }}</span>

    </li>
    {% endfor %}
  </ul>

</div>
{% endraw %}
{% endhighlight %}

To take note here is the fact that this little list does not do pagination of content, meaning that if you have something like 300 posts, your home page will end up being veeeery long. Otherwise, it works very well for its purpose.

At this point, your site should be looking something like this:

![Barebones Demo Blog](/assets/images/portfolio/demo-blog-bare.png)