Starting a blog using [Harp](http://harpjs.com) is
[very easy](http://harpjs.com/docs/quick-start). Here's how to add teasers to the
front page, and how to make tag pages.

<!-- more -->

### Adding Teasers

Let's add support for WordPress style teasers: teaser and the rest of the content
separated by a `<!-- more -->` comment.

Well, first you add the comment to your posts, like this:

```markdown
The [python-dbtxn](https://github.com/echaozh/python-dbtxn) is a library I wrote to ease db accessing from Python programs. Directly calling Python DBI leaves a lot of boilerplate code all over the place, and boilerplate code is bad. I googled, and there are no dbtxn like libraries, so I wrote my own.

<!-- more -->

There are 2 sources of boilerplate code:

...
```

Then, in `index.jade`, you can render the post and strip away the comment and
everything after it. Then you got your teaser. This is possible because partials
in the Harp flavored [Jade](jade-lang.com) is implemented by passing in a
`partial()` function which is callable by JavaScript code in the Jade template!
Calling `partial(postFilePath)` actually returns the rendered post as a
JavaScript string. Then it's easy to call `String.prototype.replace()` to strip
away everything you don't need.

The JavaScript code is really simple (assuming you put your posts under
public/posts):

```javascript
var teaserForPost = function(name) {
  var full = partial ("posts/" + name);
  return full.replace (/<!-- more -->(.|\n)*/, '', 'm');
}
```

Prefix each line with `-` to make it unbuffered code in Jade.

The teaser will look like this:

```html
<p>The <a href="https://github.com/echaozh/python-dbtxn">python-dbtxn</a> is a library I wrote to ease db accessing from Python programs. Directly calling Python DBI leaves a lot of boilerplate code all over the place, and boilerplate code is bad. I googled, and there are no dbtxn like libraries, so I wrote my own.</p>
```

Put it in the front page.

### Making Tag Pages

Or, more precisely, making a tag page to dynamically list posts for different
tags. Harp, AFAIK, doesn't allow you to create pages without a template. And it's
for developing WebApps. So the solution is obvious: you need client side
JavaScript to make it possible (unless you create a page template for each tag
by hand).

My solution is to populate the tag page with all tags, and hide those tags you
don't want. The wanted tag name is passed in via the hash part in the URL.

Tags, as I mentioned in the [Migrated Blog to Harp](/posts/to-harp.html) post,
should be specified in the `_data.json` file. Extracting tags from the file is
really straightforward.

Source of my tag template:

```jade
- var tagsForPosts = function() {
-   var data = public.posts.data;
-   var tags = {};
-   for (var name in data) {
-     if (name[0] != '_') {
-       var ts = data[name].tags;
-       for (var i in ts) {
-         tag = ts[i];
-         tags[tag] = tags[tag] || []
-         tags[tag].push (name);
-       }
-     }
-   }
-   return tags;
- }

h1 Posts tagged with&nbsp;
  span#tagName

each posts, tag in tagsForPosts()
  ul.posts(id=tag, style="display: none")
    each name in posts
      li
        - var teaser = teaserForPost(name);
        +postTeaser(teaser)

script.
  var tag = window.location.hash.substr (1);
  document.getElementById ("tagName").innerText = tag;
  document.getElementById (tag).style.display = "block";
```

### Jade Snippet to Find Page Data

Harp only sends in a `current` object with path info of the template. To find
data registered for the specific template, use the JavaScript snippet below (and
don't forget to prefix the lines):

```javascript
var page = (function (cur) {
  var data = public;
  for (var i = 0; i < cur.path.length - 1; ++i) {
    data = data[cur.path[i]];
  }
  data = data.data[cur.path[cur.path.length - 1]];
  return data;
}) (current);
```

I put it in `_page.jade` and include it in almost every Jade template.

<script>hljs.initHighlightingOnLoad();</script>
