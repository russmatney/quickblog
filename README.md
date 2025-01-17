# Quickblog fork roughly supporting org files

This is a fork of [borkdude/quickblog](https://github.com/borkdude/quickblog)
with some rough commits supporting my personal blog usage:

- Rendering org files rather than markdown, mostly via
  [russmatney/org-crud](https://github.com/russmatney/org-crud). (Though markdown is
  still supported!)
- supporting a passed list of files (`:post-paths`) rather than a fixed `posts/`
  directory, so that I can write and publish directly from my `org-dir`. My goal
  is to write directly into the org-dir and have org-items tagged with
  `posts`/`daily`/etc publishable without intermediary files.

If there is interest in including any of these features in the original repo,
feel free to let me know either here or on clojurians slack - for now I figure
blogs are fairly personal hacks, but perhaps there are others looking to publish
a blog via clojure rather than emacs from their org directory?

There are some missing pieces for properly supporting org->html in org-crud -
there are notes in that repo's readme, but be aware that not all org structures
are supported yet (e.g. tables, images) - I hope to extend support for those
things in the future.

--

Original readme below:

# Quickblog

The blog code powering my [blog](https://blog.michielborkent.nl/).

See [API.md](API.md) on how to use this.

Compatible with [babashka](#babashka) and [Clojure](#clojure).

Includes hot-reload. See it in action [here](https://twitter.com/borkdude/status/1547912740156583936).

## Quickstart

### Babashka

quickblog is meant to be used as a library from your Babashka project. The
easiest way to use it is to add a task to your project's `bb.edn`.

This example assumes a basic `bb.edn` like this:

``` clojure
{:deps {io.github.borkdude/quickblog
        #_"You use the newest SHA here:"
        {:git/sha "389833f393e04d4176ef3eaa5047fa307a5ff2e8"}}
 :tasks
 {:requires ([quickblog.cli :as cli])
  :init (def opts {:blog-title "REPL adventures"
                   :blog-description "A blog about blogging quickly"})
  quickblog {:doc "Start blogging quickly! Run `bb quickblog help` for details."
             :task (cli/dispatch opts)}}}
```

To create a new blog post:

``` clojure
$ bb quickblog new --file "test.md" --title "Test"
```

To start an HTTP server and re-render on changes to files:

```
$ bb quickblog watch
```

### Clojure

Quickblog can be used in Clojure with the exact same API as the bb tasks.
Default options can be configured in `:exec-args`.

``` clojure
:quickblog
{:deps {io.github.borkdude/quickblog
        #_"You use the newest SHA here:"
        {:git/sha "389833f393e04d4176ef3eaa5047fa307a5ff2e8"}
        org.babashka/cli {:mvn/version "0.3.35"}}
 :main-opts ["-m" "babashka.cli.exec" "quickblog.cli" "run"]
 :exec-args {:blog-title "REPL adventures"
             :blog-description "A blog about blogging quickly"}}
```

After configuring this, you can call:

```
$ clj -M:quickblog new --file "test.md" --title "Test"
```

To watch:

```
$ clj -M:quickblog watch
```

etc.

## Features

### favicon

**NOTE:** when enabling or disabling a favicon, you must do a full re-render of
your site by running `bb quickblog clean` and then your `bb quickblog render`
command.

To enable a [favicon](https://en.wikipedia.org/wiki/Favicon), add `:favicon
true` to your quickblog opts (or use `--favicon true` on the command line).
quickblog will render the contents of `templates/favicon.html` and insert them
in the head of your pages.

You will also need to create the favicon assets themselves. The easiest way is
to use a favicon generator such as
[RealFaviconGenerator](https://realfavicongenerator.net/), which will let you
upload an image and then gives you a ZIP file containing all of the assets,
which you should unzip into your `:assets-dir` (which defaults to `assets`).

You can read an example of how to prepare a favicon here:
https://jmglov.net/blog/2022-07-05-hacking-blog-favicon.html

quickblog's default template expects the favicon files to be named as follows:
- `android-chrome-192x192.png`
- `android-chrome-512x512.png`
- `apple-touch-icon.png`
- `browserconfig.xml`
- `favicon-16x16.png`
- `favicon-32x32.png`
- `favicon.ico`
- `mstile-150x150.png`
- `safari-pinned-tab.svg`
- `site.webmanifest`

If any of these files are not present in your `:assets-dir`, a quickblog default
will be copied there from `resources/quickblog/assets`.

### Social sharing

Social media sites such as Facebook, Twitter, LinkedIn, etc. display neat little
preview cards when you share a link. These cards are populated from certain
`<meta>` tags (as described in "[How to add a social media share card to any
website](https://dev.to/mishmanners/how-to-add-a-social-media-share-card-to-any-website-ha8)",
by Michelle Mannering) and typically contain a title, description / summary, and
preview image.

By default, quickblog adds tags for the page title for all pages and
descriptions for the following pages:
- Index: `{{blog-description}}`
- Archive: Archive - `{{blog-description}}`
- Tags: Tags - `{{blog-description}}`
- Tag pages: Posts tagged "`{{tag}}`" - `{{blog-description}}`

If you specify a `:blog-image URL` option, a preview image will be added to the
index, archive, tags, and tag pages. The URL should point to an image; for best
results, the image should be 1200x630 and maximum 5MB in size. It may either be
an absolute URL or a URL relative to `:blog-root`.

For post pages, meta tags will be populated from `Description`, `Image`,
`Image-Alt`, and `Twitter-Handle` metadata in the document.

If not specified, `Twitter-Handle` defaults to the `:twitter-handle` option to
quickblog. The idea is that the `:twitter-handle` option is the Twitter handle
of the person owning the blog, who is likely also the author of most posts on
the blog. If there's a guest post, however, the guest blogger can add their
Twitter handle instead.

For example, a post could look like this:

``` text
Title: Sharing is caring
Date: 2022-08-16
Tags: demo
Image: assets/2022-08-16-sharing-preview.png
Image-Alt: A leather-bound notebook lies open on a writing desk
Twitter-Handle: quickblog
Description: quickblog now creates nifty social media sharing cards / previews. Read all about how this works and how you can maximise engagement with your posts!

You may have already heard the good news: quickblog is more social than ever!
...
```

The value of the `Image` field is either an absolute URL or a URL relative to
`:blog-root`. As noted above, images should be 1200x630 and maximum 5MB in size
for best results.

`Image-Alt` provides alt text for the preview image, which is extremely
important for making pages accessible to people using screen readers. I highly
recommend reading resources like "[Write good Alt Text to describe
images](https://accessibility.huit.harvard.edu/describe-content-images)" to
learn more.

Resources for understanding and testing social sharing:
- [Meta Tags debugger](https://metatags.io/)
- [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/)
- [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/)
- [Twitter Card Validator](https://cards-dev.twitter.com/validator)

## Templates

quickblog uses the following templates in site generation:
- `base.html` - All pages. Page body is provided by the `{{body}}` variable.
- `post.html` - Post bodies.
- `style.css` - Styles for all pages.
- `favicon.html` - If `:favicon true`, used to include favicon in the `<head>`
  of all pages.

quickblog looks for these templates in your `:templates-dir`, and if it doesn't
find them, will copy a default template into that directory. It is recommended
to keep `:templates-dir` under revision control so that you can modify the
templates to suit your needs and preferences.

The default templates are occasionally modified to support new features. When
this happens, you won't be able to use the new feature without making the same
modifications to your local templates. The easiest way to do this is to run `bb
quickblog refresh-templates`.

## Breaking changes

### posts.edn removed

quickblog now keeps metadata for each blog post in the post file itself. It used
to use a `posts.edn` file for this purpose. If you are upgrading from a version
that used `posts.edn`, you should run `bb quickblog migrate` and then remove the
`posts.edn` file.

## Improvements

Feel free to send PRs for improvements.

My wishlist:

- There might be a few things hardcoded that still need to be made configurable.
- Upstream improvements to [markdown-clj](https://github.com/yogthos/markdown-clj)
