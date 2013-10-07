I have migrated my blog to [Harp](http://harpjs.com)! Now I've waved goodbye to
[Jekyll](http://jekyllrb.com) and [Blaze HTML](http://jaspervdj.be/blaze/),
and am using [Jade](http://jade-lang.com) instead. I am pretty happy with the
migration.

Though I am not using Harp as a server, I'm happy with its compilation process.
With native Jade support, I can finally get rid of my Makefiles! Jade also
supports embedded JavaScript logic, so tag pages can be generated directly in the
Jade template, rather than in a Jekyll plugin, which is definately more
convenient in my eyes.

Also, the meta data is put in a separate `_data.json` file
in each directory, rather than scattering in each page. This is arguably better
than in-page meta data, as it allows easy tag extraction. And it makes each page
clean, as the meta data is not always syntactically correct, which sometimes
makes me, a purist, itch somewhere unscratchable. And it eases meta data
processing logic in the static site generator. Though, sometimes it is more
annoying to open up another file to write meta data to, and you tend to forget.

On the whole, I am pretty happy with the transition. Oh, I had to patch the
Terraform library depended by Harp to make nested layout templates possible. I
will send the patch upstream some time later. Can't believe they left such useful
functionality out.
