# Generate URLs

Go back to the "show" page for a question. The logo on top is a link... that doesn't
go anywhere yet. This *should* take us back to the homepage.

Because this is part of the layout, the link lives in `base.html.twig`. Here it is:
`navbar-brand` with `href="#"`. 

[[[ code('9faea27c3a') ]]]

To make this link back to the homepage, we can just change this to `/`, right? 
You *could* do this, but in Symfony, a better way is to ask Symfony to *generate* 
the URL to this route. That way, if we decide to change this URL later, all our 
links will update automatically.

## Each Route Has a Name!

To see how to do that, find your terminal and run:

```terminal
php bin/console debug:router
```

This lists *every* route in the system... and hey! Since the last time we ran this,
there are a *bunch* of new routes. These power the web debug toolbar and the
profiler and are added automatically by the WebProfilerBundle when we're in `dev`
mode.

Anyways, what I *really* want to look at is the "Name" column. *Every* route has
an internal name, *including* the two routes that *we* made. Apparently their
names are `app_question_homepage` and `app_question_show`. But... uh... where
did those come from? I don't remember typing either of these!

So... every route *must* be given an internal name. But when you use annotation
routes... it lets you cheat: it chooses a name *for* you based on the controller
class and method... which is awesome!

But... as soon as you need to generate the URL to a route, I recommend giving it
an *explicit* name, instead of relying on this auto-generated name, which could
change suddenly if you rename your method. To give the route a name, add `name=""`
and... how about: `app_homepage`.

[[[ code('41da313f69') ]]]

I like to keep my route names short, but `app_` makes it long enough that I
could search my project for this string if I ever needed to.

*Now* if we run `debug:router` again:

```terminal-silent
php bin/console debug:router
```

Nice! We are in control of the route's name. Copy the `app_homepage` name and
then go back to `base.html.twig`. The goal is simple, we want to say:

> Hey Symfony! Can you please give me the URL to the `app_homepage` route?

To do that in Twig, use `{{ path() }}` and pass it the route name.

[[[ code('c164d6ba40') ]]]

That's it! When we move over and refresh... *now* this links to the homepage.

## Linking to a Route with {Wildcards}

On the homepage, we have two hard-coded questions... and each has two links that
currently go nowhere. Let's fix these!

Step one: now that we want to generate a URL to this route, find the route and add
`name="app_question_show"`.

[[[ code('ed0b336b2a') ]]]

Copy this and open the template: `templates/question/homepage.html.twig`.
Let's see... right below the voting stuff, here's the first link to a
"Reversing a spell" question. Remove the pound sign, add `{{ path() }}` and paste
in `app_question_show`.

But... we can't stop here. If we try the page now, a glorious error!

> Some mandatory parameters are missing - "slug"

That makes sense! We can't just say "generate the URL to `app_question_show`"
because that route has a wildcard! Symfony needs to know what value it should use
for `{slug}`. How do we tell it? Add a *second* argument to `path()` with `{}`.
The `{}` is a Twig associative array... again, just like JavaScript. Pass `slug` set
to... let's see... this is a hardcoded question right now, so hardcode
`reversing-a-spell`.

[[[ code('fad5a0b0dc') ]]]

Copy that *entire* thing, because there's one other link down here for the same
question. For the second question... paste again, but change it to
`pausing-a-spell` to match the name. I'll copy that... find the last spot... and
paste.

[[[ code('dc81dc99c3') ]]]

Later, when we introduce a database, we'll make this fancier and avoid repeating
ourselves so many times. But! If we move over, refresh... and click a link, it
works! Both pages go to the same route, but with a different slug value.

Next, let's take our site to the next level by creating a JSON API endpoint that
we will consume with JavaScript.
