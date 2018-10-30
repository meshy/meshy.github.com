---
layout: post
title: "Ordering Django applications"
description: "Why the order of <code>INSTALLED_APPS</code> matters in Django"
---

When you create a Django project, the `INSTALLED_APPS` setting will be
generated for you like this:

```
# Somewhere in settings.py...
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

Then, when you create an app, or install a library, you're faced with a
choice: do you add the new item _above_ or _below_ Django's apps?

Well, it's good practice to **add new apps _above_ Django's**, unless you
have a specific reason not to. I've seen this done the other way around a lot,
so it might not be what you expect.

This is because when Django is looking for something, it will go through the
`INSTALLED_APPS` list, from the top, one app at a time, until it has found what
it's looking for. **Putting your code first means it takes priority.**

For example, if you want to override a template in Django's admin, you can do
so by placing a modified version of the original into the `templates/`
directory of one of your apps.

This applies to:

- management commands,
- templates,
- template-tags,
- translations,
- probably a lot more.

It's also handy to know that this is the order in which apps will be
registered, and admin integration discovered.

Finally: don't add the core app's `templates/` directory into
`TEMPLATES[...]['DIRS']` (or `TEMPLATE_DIRS`)! Instead, simply add
each app containing `templates/` to `INSTALLED_APPS`.


## TLDR

In a Django projects, expect to see `INSTALLED_APPS` structured like this:

```
INSTALLED_APPS = [
    # The "core" (AKA "project" or "main") app.
    'core',

    # Other apps in the project.
    'foo',
    'bar',

    # Libraries.
    'orderable',
    'pgcrypto',
    'webpack_loader',

    # Django.
    'django.contrib.admin',
    'django.contrib.auth',
    ...  # Etc.
]
```

This ensures:

- The 'core' app can override everything.
- Your secondary apps can extend the libraries and Django.
- Libraries can be used to extend Django.
- As usual, Django provides a solid default.
