---
layout: post
title: Let's do I18n!
---

## I18n for people
***

 * without conflict
 * editing directly on the site
 * don't need developers for edit
 * always in the context of the page.
  
In our company, we often write web solutions with support for many languages, and for the correct translation it is very important to see the context of the content, so after a long search for the right and handy solution, we’ve come to TRS:

1. Editing on the page of real content.
2. Easy control of locals and the creation of new ones.
3. The components for diffefent frameworks.

TRS structure consists of:
 * Administrative part
 * Server API
 * Plugins

The administrative part allows you to create users and projects. For each project, you could create a variety of localization files. Frontend part created using ember.js, backend uses [phoenix](http://www.phoenixframework.org/) framework and [postgresql](http://www.postgresql.org/) database, localization files are stored in [couchdb](http://couchdb.apache.org/).

API part provides an opportunity to interact with the localization files through the api and the token authentication:

`http://api.translations.api-hogs.io/api/v1/languages?project=${ID}`

```json
  {"en": {"index": {"title": "Index Page"}}, "de": {"index": {"title": "Index page de"}}}
```

For Ember.js applications there is a plugin that makes it easy to integrate TRS into your project - [ember](https://github.com/api-hogs/trs-translation).

TRS can be used as self hosted application. To do this, you need to deploy TRS on your server.

Repo: [TRS](https://github.com/api-hogs/trs)

Love example? [Try it](http://translations.api-hogs.io)

