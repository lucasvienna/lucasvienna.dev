---
layout: post
title: How I built this website - Pt. 1
subtitle: A tutorial on integrating Jekyll, GitHub Pages, and Travis CI
---

Specifically, how I used Jekyll and GitHub Pages to generate my website and how to automate builds and deployments with Travis CI. On this part, I'll cover requirements and the setup.

# Motivation & Requirements

I wanted a platform where I could showcase my development skills and write posts about how I acquired/use said skills. My idea was to have a portfolio website with an integrated, simple blog where I could write my musings. This right here is the result of a few hours of writing code, testing everything out and eventually deciding to use Travis CI to automate the whole process of deploying the site.

I had a few requirements in mind before starting out:

1. Hosting should be affordable. Free is best
2. The website should be lightweight and fast
3. Very little maintenance required
4. Custom domains with SSL support

Regarding point #2, I wanted a small, fast, and performant website that "just worked" every time, and that even hamster-powered networks could load. I decided a templating solution would work well for me.

Enter point #1. Hosting is rather cheap, specially for my use-case: hosting a static, template-based website. Enter [GitHub Pages](https://pages.github.com/) and their free hosting. Oh, what is that? They user Jekyll under the hood too? _And_ they support custom URLs with SSL enabled? Bingo.

Why do I need SSL, you might ask? Simple. My chosen domain has a `.dev` TLD, meaning I need HTTPS for all requests.

You can see the source code for this website in my GitHub profile: [https://github.com/Avyiel/blog-demo](https://github.com/Avyiel/blog-demo)

# Setting up
## Git

First order of business is setting up an orphan branch on your repository that will be used to deploy your website:

Inside your website's root, create a new orphan branch and push it:

{% highlight shell %}
$ git checkout --orphan gh-pages
$ git rm -rf .
$ echo "Placeholder page" > index.html
$ git add index.html
$ git commit -a -m "Placeholder page"
$ git push origin gh-pages
{% endhighlight %}

## GitHub

Then, we configure our GitHub repository to use GitHub Pages, and use our newly pushed branch as the source:

Go to "Settings" and scroll down to the "Pages" section. If everything worked well, you should see the following:

![Options - Pages](/assets/images/portfolio/options-gh-pages.png)

If not, first enable Pages, and then choose our recently-created `gh-pages` branch as the source.

## Custom Domain

If you have a custom domain and wish to use it, this is the time to type it in the designated field. It will generate a `CNAME` file in the root of your repository, but you'll still need to edit your DNS records. GitHub has excellent documentation on how to do this, so simply follow the instructions [on their own help page](https://help.github.com/en/articles/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain) to set your DNS records up. It should look somewhat like this after you're done:

![DNS Records](/assets/images/portfolio/dns-records.png)

## Travis CI

Next up is Travis CI. First, we need two personal access tokens:

Head over to [https://github.com/settings/tokens/new](https://github.com/settings/tokens/new) and create the first token: our TravisCI token, with `repo` permissions. **Do not close, refresh the page or navigate away. This token will only be visible once**.

![Travis CI Personal Access Token](/assets/images/portfolio/travis-token.png)

Then, head to [Travis CI](https://travis-ci.com/) (login or create an account) and find your repository. Click on "More Options" and select "Settings". Scroll down to _Environment Variables_ and create a new one named _"GITHUB_TOKEN"_, paste the token you created on the previous step and make sure "_Display value in build log"_ is **un**checked.

![Environment Variable GITHUB_TOKEN](/assets/images/portfolio/travis-token-env.png)

Go back to GitHub and create a second token: our Jekyll token, with `public_repo` permissions. Again, **do not close, refresh the page or navigate away. This token will only be visible once**.

![Jekyll Personal Access Token](/assets/images/portfolio/jekyll-token.png)

Head over to Travis CI again, and on the same page create a second environment variable, named _"JEKYLL_GITHUB_TOKEN"_, paste the token you created on the previous step and make sure "_Display value in build log"_ is **un**checked. You should save this token for later as well, so paste it somewhere safe.

![Environment Variable JEKYLL_GITHUB_TOKEN](/assets/images/portfolio/jekyll-token-env.png)

Lastly, create a third environment variable, named _"JEKYLL_ENV"_, enter `production` in the value field and make sure _"Display value in build log"_ is **checked**.

![Environment Variable JEKYLL_ENV](/assets/images/portfolio/jekyll-env-env.png)

It should look like this in the end:

![Environment Variables](/assets/images/portfolio/env-vars.png)

# Jekyll

Now that we have our tooling all set up, it's time to actually write our website. I am working under the assumption that your repository only has 3 files: `LICENSE`, `README.md`, and `CNAME`.

> If you already have more things (say, some posts, assets and theme), you can port them later on.

Also, I am working on macOS, so the instructions here apply to this OS. If you have a different one, make sure to change commands to their equivalent ones.

From the [Jekyll website](https://jekyllrb.com/docs/):

- Install a [full Ruby development environment](https://jekyllrb.com/docs/installation/macos/#install-ruby)
- Install Jekyll and [bundler](https://jekyllrb.com/docs/ruby-101/#bundler) [gems](https://jekyllrb.com/docs/ruby-101/#gems):

{% highlight shell %}
$ gem install --user-install bundler jekyll
{% endhighlight %}

- Backup your current repository folder:

{% highlight shell %}
$ mv ./myblog ./myblog_old
{% endhighlight %}

- Create a new Jekyll site:

{% highlight shell %}
$ jekyll new myblog
$ cd myblog
{% endhighlight %}

- Copy over your `.git` folder and existing files:

{% highlight shell %}
$ cp -r ../myblog_old/.git .
$ cp ../myblog_old/LICENSE .
$ cp ../myblog_old/README.md .
$ cp ../myblog_old/CNAME .
{% endhighlight %}

- Build the site and make it available on a local server

{% highlight shell %}
$ bundle exec jekyll serve
{% endhighlight %}

Obviously, swap out `myblog` for your chosen repository name. If everything went according to plan, you sould see something like this in your browser:

![Default Jekyll Website](/assets/images/portfolio/default-site.png)

## Customizing your installation

Now for the interesting part: making that site yours. Go ahead and create a few folders:

{% highlight shell %}
$ mkdir _layouts _includes _sass assets
{% endhighlight %}

Then, clean up your Gemfile. It comes heavily commented and has some unnecessary gems, namely the Windows-relate ones. If you're on Windows, do keep those gems in there.

{% highlight ruby %}
source "https://rubygems.org"

# GitHub Pages. To upgrade, run `bundle update github-pages`
gem "github-pages", group: :jekyll_plugins

# Jekyll theme
gem "minima", "~> 2.0"

gem "dotenv"
{% endhighlight %}

You will have noticed that I added `dotenv` to our list of dependencies. This comes into play later on, in order to let us fetch GitHub metadata using that token we created earlier.

Now, install/update these gems:

{% highlight shell %}
$ bundle install
{% endhighlight %}

Next up, configure your Jekyll environment. Open `_config.yml` in your favourite editor and read through the comments, it should give you a good idea of what is useful for you or not. A few entries you should have in there are shown below. Remember to replace the actual values with your own information.

> Since I'll be using my own theme and no plugins, I've removed those entries. Also remove the theme gem from your Gemfile if you're not using one.

{% highlight yml %}
# Site settings
title: My title
subtitle: A subtitle
description: Describe your site here

# Personal Info
owner: Lucas Vienna
email: dev@lucasvienna.dev
url: https://avyiel.github.io # the custom domain you configure ealier
baseurl: /blog-demo # what comes after the '.io'
repository: Avyiel/lucasvienna.dev # only the path part of the repository URL

# Blogging Defaults
author: Lucas Vienna

# Build settings
markdown: kramdown # this is the default, you can omit this line if desired
highlighter: rouge # this is the default, you can omit this line if desired
include: ['_pages']
compress_html:
  blanklines: true
  comments: ["<!-- ", " -->"]
  ignore:
    envs: [development] # don't compress during development
{% endhighlight %}

The most important part here are the _Build settings_. They define the markdown processor, the code highlighter, and configure HTML compression on production build (this helps make the website smaller). The `include` entry tells Jekyll to process these directories, so that we can have our static pages grouped up.

You can also define your own variables, that can be accessed throught your templates. I've done that for my social handles and some display options. Once again, replace these values with your own.

{% highlight yml %}
# Social References
twitter:
  handle: Avyiel7
  url: https://twitter.com/avyiel7
github:
  username: Avyiel
  url: https://github.com/Avyiel
linkedin:
  username: lucasvienna
  url: https://linkedin.com/in/lucasvienna

# Display settings
show_github: false
show_downloads: false
{% endhighlight %}

## Adding theming support

For this part, those folders you created earlier come in handy. Jekyll uses specially-named folders to generate your website.

> `_layouts` is what the name suggests, and describes how different pages can be formatted, AKA your templates.

> `_includes` holds snippets of a sort, smaller components you can insert into any page (read: in your templates).

> `_sass` contains our SASS files, the normaizer, and the `rouge` highlighter.

> `assets` contains our images and the entry point for the whole SASS business. Let's start there.

Create a new folder inside `assets`, and add the CSS entrypoint:

{% highlight shell %}
$ mkdir -p assets/css
$ touch assets/css/styles.scss
{% endhighlight %}

Now paste this into `styles.scss`:

{% highlight scss %}
---
---

@import 'theme';
{% endhighlight %}

> This snippet is responsible for loading our main SASS file, `theme.scss`. Do not remove those hyphens at the start; they are the front matter, and are required by Jekyll.

Next up, let's install `rouge`. Since Jekyll is stuck with Rouge `2.2.1` ([see this GitHub issue](https://github.com/github/pages-gem/pull/652)), we'll poach an updated SCSS version of it from the Cayman theme. Head over to [https://github.com/pages-themes/cayman/blob/master/_sass/rouge-github.scss](https://github.com/pages-themes/cayman/blob/master/_sass/rouge-github.scss), download this file and save it in your own `_scss` folder. Once Jekyll upgrades, I will update this article accordingly.

To install `compress_html`, head over to [http://jch.penibelst.de/](http://jch.penibelst.de/) and follow their instructions. You'll end up with two new files inside your `_layouts` folder: `compress.html` and `default.html`. We'll go over the default layout later on.

Then, install `normalize.css`. All you need to do is copy the contents from [the source page](https://github.com/necolas/normalize.css/blob/master/normalize.css) and save it as `normalize.scss`.

Lastly, create the `theme.scss` file and paste the following inside:

{% highlight scss %}
// sensible normalized styles
@import "normalize";
// code highliting
@import "rouge-github";
{% endhighlight %}

That's it for now. You should have all the basic themeing setup covered.

---

## Closing

On Part 2, I'll cover layouts, includes and more specific theming options.

#### Sources

A few websites that I used as reference while building my own solution.

- [https://jekyllrb.com/docs/](https://jekyllrb.com/docs/)
- [https://developpaper.com/automated-deployment-of-github-pages-with-travis-ci/](https://developpaper.com/automated-deployment-of-github-pages-with-travis-ci/)
- [https://voorhoede.github.io/front-end-tooling-recipes/travis-deploy-to-gh-pages/](https://voorhoede.github.io/front-end-tooling-recipes/travis-deploy-to-gh-pages/)
- [https://github.com/jekyll/jekyll/issues/920#issuecomment-63093764](https://github.com/jekyll/jekyll/issues/920#issuecomment-63093764)
