---
layout: post
title: "lunr search in octopress"
date: 2014-06-26 21:36:50 +0800
comments: true
categories:
  - octopress
---

After migrating my blog from `wordpress` to `octopress` on Github, I miss some features like `search` and easy-customized theme.

This article is inspired by <a href="https://github.com/slashdotdash/jekyll-lunr-js-search" target="_blank">jekyll-lunr-js-search</a>, <a href="https://github.com/yortz/octopress-lunr-js-search" target="_blank">octopress-lunr-js-search</a> and <a href="http://wangmuy.github.io/blog/2013/09-01-octopress-setup.html#jekylllunrjs-7">this</a>.

Here we go.

1. Get essential files
---

Download .rb <a href="https://raw.githubusercontent.com/slashdotdash/jekyll-lunr-js-search/master/build/jekyll_lunr_js_search.rb" target="_blank">jekyll_lunr_js_search.rb</a> to directory *~/octopress/plugins/* .

Download .js files below to directory *~/octopress/source/javascripts/* .

<a href="https://raw.github.com/olivernn/lunr.js/master/lunr.min.js" target="_blank">lunr.min.js</a>

<a href="https://raw.githubusercontent.com/janl/mustache.js/master/mustache.js" target="_blank">mustache.js</a>

<a href="http://stevenlevithan.com/assets/misc/date.format.js" target="_blank">date.format.js</a>

<a href="https://github.com/medialize/URI.js/blob/gh-pages/src/URI.min.js" target="_blank">URI.min.js</a>

2. Add search page
---

Add a page of search

```
$ rake new_page["search"]
```

with following contents:

```
---
layout: page
title: "Search"
comments: false
sharing: false
footer: false
#exclude_from_search: false  # with this parameter, this page won't be indexed
---

<div id="search">
	<form action="/search" method="get">
   	<input type="text" id="search-query" name="q" placeholder="Search" autocomplete="off">
 	</form>
</div>

<section id="search-results" style="display: none;">
	<p>Search results</p>
 	<div class="entries">
  	</div>
</section>
 
<script src="/javascripts/libs/jquery.min.js" type="text/javascript" charset="utf-8"></script>
<script src="/javascripts/lunr.min.js" type="text/javascript" charset="utf-8"></script>
<script src="/javascripts/mustache.js" type="text/javascript" charset="utf-8"></script>
<script src="/javascripts/date.format.js" type="text/javascript" charset="utf-8"></script>
<script src="/javascripts/URI.min.js" type="text/javascript" charset="utf-8"></script>
<script src="/javascripts/jquery.lunr.search.js" type="text/javascript" charset="utf-8"></script>
 
{% raw %}
<script id="search-results-template" type="text/mustache">
  	{{#entries}}
    	<article>
      		<h3>
      		{{#date}}<small><time datetime="{{pubdate}}" pubdate>{{displaydate}}</time></small>{{/date}}
        	<a href="{{url}}">{{title}}</a>
      		</h3>
    	</article>
  	{{/entries}}
</script>
{% endraw %}
 
<script type="text/javascript">
	$(function() {
    	$('#search-query').lunrSearch({
      		indexUrl: '/search.json',             // URL of the `search.json` index data for your site
      		results:  '#search-results',          // jQuery selector for the search results container
      		entries:  '.entries',                 // jQuery selector for the element to contain the results list, must be a child of the results element above.
      		template: '#search-results-template'  // jQuery selector for the Mustache.js template
    	});
  	});
</script>
```
	
Add `<li><a href="{{ root_url }}/search">Search</a></li>` to *~/octopress/source/_includes/custom/navigation.html* .

3. Modify octopress configuration .
---

Add following lines to *~/octopress/_config.yml* to block indexing.

```
lunr_search:
  excludes: [rss.xml, atom.xml]
```

Add `gem 'nokogiri'` to *~/octopress/Gemfile* to build the site then run

```
$ bundle
```
	
4. Test
---

```
$ rake clean
$ rake generate
$ rake preview
```
