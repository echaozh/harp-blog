Starting a blog using [Harp](http://harpjs.com) is
[very easy](http://harpjs.com/docs/quick-start). Here's how to add teasers to the
front page, and how to make tag pages.

<!-- more -->

### Adding Teasers

Let's add support for WordPress style teasers: teaser and the rest of the content
separated by a `<!-- more -->` comment.

Well, first you add the comment to your posts, like this:

<script src="https://gist.github.com/echaozh/6916985.js"></script>
<noscript>
  <a href="https://gist.github.com/echaozh/6916985#file-post-example"
     target="_blank">
    Open the Gist
  </a>
</noscript>

Then, in `index.jade`, you can render the post and strip away the comment and
everything after it. Then you got your teaser. This is possible because partials
in the Harp flavored [Jade](jade-lang.com) is implemented by passing in a
`partial()` function which is callable by JavaScript code in the Jade template!
Calling `partial(postFilePath)` actually returns the rendered post as a
JavaScript string. Then it's easy to call `String.prototype.replace()` to strip
away everything you don't need.

The JavaScript code is really simple (assuming you put your posts under
public/posts):

<script src="https://gist.github.com/echaozh/6917018.js"></script>
<noscript>
  <a href="https://gist.github.com/echaozh/6917018#file-teaser-extraction-js"
     target="_blank">
    Open the Gist
  </a>
</noscript>

Prefix each line with `-` to make it unbuffered code in Jade.

The teaser will look like this:

<script src="https://gist.github.com/echaozh/6917031.js"></script>
<noscript>
  <a href="https://gist.github.com/echaozh/6917031#file-teaser-html"
     target="_blank">
    Open the Gist
  </a>
</noscript>

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

<script src="https://gist.github.com/echaozh/6917115.js"></script>
<noscript>
  <a href="https://gist.github.com/echaozh/6917115#file-tag-page-jade"
     target="_blank">
    Open the Gist
  </a>
</noscript>

### Jade Snippet to Find Page Data

Harp only sends in a `current` object with path info of the template. To find
data registered for the specific template, use the JavaScript snippet below (and
don't forget to prefix the lines):

<script src="https://gist.github.com/echaozh/6917128.js"></script>
<noscript>
  <a href="https://gist.github.com/echaozh/6917128#file-page-data-js"
     target="_blank">
    Open the Gist
  </a>
</noscript>

I put it in `_page.jade` and include it in almost every Jade template.
