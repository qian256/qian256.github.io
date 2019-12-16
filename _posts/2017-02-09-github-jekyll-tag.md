---
layout: post
title: Jekyll Tags on Github Pages
description: Github page does not allow customized plugins, and jekyll-tagging is not one of the supported GEMs of Github pages. It needs some effort to add tag support your Jekyll blog hosted by Github page. This blog shows you how to do this step by step.
tags: jekyll blog github-page
---


<style>
.highlight-left {margin-left: 0}
</style>


If you want to add tags to your Jekyll blog but found out that it is simply not supported by Github pages. You are at the right place here.

It is **true** that Github pages do not support customized Ruby plugins for Jekyll site. It is also **true** that the great [jekyll-tagging](https://github.com/pattex/jekyll-tagging) is not one of Github default jekyll plugins.

As a result, we need to implement Jekyll tagging without plugins, it sounds hard, but here is a **step-by-step guide** to achieve so.

If you want to see live demo, you are already in a sample site that is hosted by Github, and has tags! Check out the source code: [Github](https://github.com/qian256/qian256.github.io/).

### 1. Add tags to your posts

In each of your post file (Markdown, presumably), you need to add the tags to the specific post in the front matter section. For example, for this post, the front matter looks like this:

```
---
layout: post
title: Jekyll Tags on Github Pages
description: blablabla
tags: jekyll blog github-page
---
```

In the `tags` entry, each tag is separated by white space. It is recommended to use lowercases for tag names.

### 2. Collect tags of all posts

What we need is a list named `site.tags` for our site. In order to do so, we can add a script to collect these tags using [Liquid Templating](https://jekyllrb.com/docs/templates/) of Jekyll.

In your `_includes` folder, add an html file called `collecttags.html`:

{% raw %}
```liquid
{% assign rawtags = "" %}
{% for post in site.posts %}
  {% assign ttags = post.tags | join:'|' | append:'|' %}
  {% assign rawtags = rawtags | append:ttags %}
{% endfor %}
{% assign rawtags = rawtags | split:'|' | sort %}

{% assign site.tags = "" %}
{% for tag in rawtags %}
  {% if tag != "" %}
    {% if tags == "" %}
      {% assign tags = tag | split:'|' %}
    {% endif %}
    {% unless tags contains tag %}
      {% assign tags = tags | join:'|' | append:'|' | append:tag | split:'|' %}
    {% endunless %}
  {% endif %}
{% endfor %}
```
{% endraw %}

This liquid scripts creates the list `site.tags`.

### 3. Execute the collection

You need to include the `collecttags.html` somewhere so that it is executed before `site.tags` are used.

My solution is to include it in `head.html` where Jekyll blogs defines the header. Google analytics script is also included here.

So, insert the following lines to `_includes/head.html`, inside `head` html section:

{% raw %}
```liquid
{% if site.tags != "" %}
  {% include collecttags.html %}
{% endif %}
```
{% endraw %}

This script runs the `collecttags.html`, and runs only once.

### 4. Display the tags of a post

We would like to display the tags of a post inside the post page, like this page. Then the modifications go to the format of posts: `_layouts/post`.

For example, the following code is inserted into the post layout:

{% highlight html %}{% raw %}
<span>[
  {% for tag in page.tags %}
    {% capture tag_name %}{{ tag }}{% endcapture %}
    <a href="/tag/{{ tag_name }}"><code class="highligher-rouge"><nobr>{{ tag_name }}</nobr></code>&nbsp;</a>
  {% endfor %}
]</span>
{% endraw %}{% endhighlight %}{: .highlight-left }

I removed part of the style for this code snippets. You should adapt it to your own website design.

Note that a link is inserted to each of the tags of the post, at `/tag/tag_name`. We will come to that in the next step.

### 5. Generate the tag page

If you click on the tag displayed in this page, it will navigate you to a tag page, like this: <a class="no-underline" href="http://longqian.me/tag/hololens/"><code class="highligher-rouge"><nobr>hololens</nobr></code></a> **<- click it**.

#### 5.1. Define the layout of tag page

The tag page needs to be generated for all tags, because no plugin can be used to generate it when Github is building the page. Therefore, we treat the tag page as a kind of normal Jekyll page, with the only difference that it uses another layout.

OK. Now let's create the layout of tag pages. Add another html file at `_layouts/tagpage.html`:

{% highlight html %}{% raw %}
---
layout: default
---
<div class="post">
<h1>Tag: {{ page.tag }}</h1>
<ul>
{% for post in site.tags[page.tag] %}
  <li><a href="{{ post.url }}">{{ post.title }}</a> ({{ post.date | date_to_string }})<br>
    {{ post.description }}
  </li>
{% endfor %}
</ul>
</div>
<hr>
{% endraw %}{% endhighlight %}{: .highlight-left }

The above is the format of all tag pages. It displays all the posts that has the current tag, including post title, date and description.

#### 5.2. Trigger the building of tag pages

In order to tell Github Jekyll engine to create a tag page, we need a markdown file specifying the specific tag to display and the above tagpage format.

Create a folder called `tag` in the root folder, and create a markdown file `hololens.md` like this:

```
---
layout: tagpage
title: "Tag: hololens"
tag: hololens
---
```

It triggers the building of a page at `site_dir/tag/hololens/`, with layout `_layouts/tagpage.html`, and parameter `page.tag = hololens`.

#### 5.3. Automatic tag page creation (Optional)

> If you would like to repeat step 5.2. for all the tags of your site, then you can skip this step.

Otherwise, I created a python script that crawls all the tags in the folder `_posts/` and generate the desired tag page at `tag/`. It can be accessed here: [tag generater](https://github.com/qian256/qian256.github.io/blob/master/tag_generator.py).

You need to run this script **before** you push everything to Github repository and wait for your Github page.

### 6. Tag cloud display (Optional)

> If you don't want the tag cloud, you can skip this step.

Tag cloud can be like this: <a class="no-underline" href="http://longqian.me/tag/hololens/"><code class="highligher-rouge"><nobr>hololens</nobr></code></a> **<- click it** (the bottom part). If you want such effect, you can also create the tag cloud without any plugin. You can change the size of the tag text depending on the number of posts related to it. Or you can display tags in the order of reference counts like my page.

What you need to do is to add another html page at `_includes/archive.html`:

{% highlight html %}{% raw %}
<h2>Archive</h2>
{% capture temptags %}
  {% for tag in site.tags %}
    {{ tag[1].size | plus: 1000 }}#{{ tag[0] }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign sortedtemptags = temptags | split:' ' | sort | reverse %}
{% for temptag in sortedtemptags %}
  {% assign tagitems = temptag | split: '#' %}
  {% capture tagname %}{{ tagitems[1] }}{% endcapture %}
  <a href="/tag/{{ tagname }}"><code class="highligher-rouge"><nobr>{{ tagname }}</nobr></code></a>
{% endfor %}
{% endraw %}{% endhighlight %}{: .highlight-left }

This script fetches `site.tags` and sort them by the size of its referenced posts. The sorted list is in `sortedtemptags`. And it is iterated to be visualized. I am inspired by this [stackoverflow answer](http://stackoverflow.com/questions/13025281/how-to-get-a-sorted-tags-list-in-jekyll).

You can add the following line to anywhere you want to display the tag cloud:
{% highlight liquid %}{% raw %}
{% include archive.html %}
{% endraw %}{% endhighlight %}{: .highlight-left }


### 7. Push to Github

Done! Happy blogging!

If you like this post, please **STAR** [my repository](https://github.com/qian256/qian256.github.io) to motivate me!

Thanks for reading!  <i class="em em-lq"></i>

