django-seo-js
=============

![Build status](https://circleci.com/gh/greenkahuna/django-seo-js.png?circle-token=90ca5d5cbeb2af20bc378faf1b196e6c03e69f26)

django-seo-js is a drop-in app that provides full SEO support for angular, backbone, ember, famo.us, and other SPA apps built with django.

It's simple to set up, configurable to use multiple services, and easy to customize.

Quick-links:
- [Installation](README.md#installation)
- [Options](README.md#options)
    - [General Settings](README.md#General-Settings)
    - [Backend settings](README.md#Backend-settings)
        - [Prerender.io](README.md#Prerender-io)
        - [Custom-hosted prerender](README.md#custom-hosted-prerender)
- [How it all works](README.md#how-it-all-works)
- [Contributing](README.md#contributing)
- [Releases](README.md#releases)


# Installation

Pip install:

```bash
pip install django-seo-js
```


Add to your `settings.py`:

```python
# If in doubt, just include both.  Details below.
MIDDLEWARE_CLASSES = (
    'django_seo_js.middleware.HashBangMiddleware',  # If you're using #!
    'django_seo_js.middleware.UserAgentMiddleware',  # If you want to detect by user agent
) + MIDDLEWARE_CLASSES

INSTALLED_APPS += ('django_ses_js',)

# If you're using prerender.io (the default backend):
SEO_JS_PRERENDER_TOKEN = "123456789abcdefghijkl"
```

Add this to your `base.html`

```twig
{% load django_seo_js %}
<head>
    {% seo_js_head %}
    ...
</head>
```

That's it!  Your js-heavy pages are now rendered properly to the search engines. Have a lovely day.

# Updating the render cache

If you know a page's contents have changed, some backends allow you to manually update the page cache.  `django-seo-js` provides helpers to make that easy.

```python
from django_seo_js.helpers import update_cache_for_url

update_cache_for_url("/my-url")
```

So, for instance, you might want something like:

```python
def listing_changed(sender, instance, created, **kwargs):
    update_cache_for_url("%s%s" % ("http://example.com/", reverse("listing_detail", instance.pk))

post_save.connect(listing_changed, sender=Listing)
```


# Options

## General settings

For the most part, you shouldn't need to override these - we've aimed for sensible defaults.

```python
# Backend to use
SEO_JS_BACKEND = "django_seo_js.backends.PrerenderIO"   # Default


# User-agents to render for, if you're using the UserAgentMiddleware
# Defaults to the most popular.  If you have custom needs, pull from the full list:
# http://www.useragentstring.com/pages/Crawlerlist/
SEO_JS_USER_AGENTS = [
    "Googlebot",
    "Yahoo",
    "bingbot",
    "Badiu",
    "Ask Jeeves",
]
```



## Backend settings

### Prerender.io
django-seo-js defaults to using prerender.io because it's both [open-source](https://github.com/prerender/prerender) if you want to run it yourself, *and* really reasonably priced if you don't.


To use [prerender.io](http://prerender.io),

```python
# Prerender.io token
SEO_JS_PRERENDER_TOKEN = "123456789abcdefghijkl"
```

You don't need to set `SEO_JS_BACKEND`, since it defaults to `django_seo_js.backends.PrerenderIO`.


### Custom-hosted prerender

If you're hosting your own instance of [prerender](https://github.com/prerender/prerender), configuration is similar

```python
SEO_JS_BACKEND = "django_seo_js.backends.PrerenderHosted"
SEO_JS_PRERENDER_URL = "http://my-prerenderapp.com"
SEO_JS_PRERENDER_RECACHE_URL = "http://my-prerenderapp.com/recache"
```


## How it all works

If you're looking for a big-picture explanation of how SEO for JS-heavy apps is handled, the clearest explanation I've seen is [this StackOverflow answer](http://stackoverflow.com/a/20766253).

If even that's TL;DR for you, here's a bullet-point summary:

- If requests come in with an `_escaped_fragment_` querystring or a particular user agent, a pre-rendered HTML response is served, instead of your app.
- That pre-rendered HTML is generated by a service with a headless browser that runs your js then caches the rendered page.
- Said service is generally a third party (there are many: [prerender.io](https://prerender.io/), [Brombone](http://www.brombone.com/), [seo.js](http://getseojs.com/), [seo4ajax](http://www.seo4ajax.com/).) You can also run such a service yourself, using [prerender](https://github.com/prerender/prerender), or re-invent your own wheel for fun.


# Contributing

PRs with additional backends, bug-fixes, documentation and more are definitely welcome! 

Please add tests to any new functionality - you can run tests with `python manage.py test`


# Releases

### 0.1 - May 8, 2014

* Includes `PrerenderIO` and `PrerenderHosted` backends.
* First release - we're using this in production at [GreenKahuna ScrapBin](http://scrapbin.com).

