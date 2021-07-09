# Request Object & POST Data

Time to hook up the vote functionality. Here's the plan: these up and down vote
icons are actually *buttons*. I'll show you: in `show.html.twig`... it's
a `<button>` with `name="direction"` and `value="up"`.

[[[ code('bbc4d2e6c1') ]]]

Thanks to the `name` and `value` attributes, if we wrapped this in a `<form>` and
then click one of these buttons, the form would submit and send a POST parameter
called `direction` that's equal to either `up` or `down`, based on which button
was clicked. It's like having an extra input in your form.

So that's *exactly* what we're going to do: wrap this in a form, make it submit
to a new endpoint, read the `direction` value and increase or decrease the vote
count. We *could* do this with an AJAX call instead of a form submit. From
a Doctrine and Symfony perspective, it really makes no difference. So I'll
keep it simple and leave JavaScript out of this.

## Creating a POST-Only Endpoint

Let's start by creating that endpoint. In `src/Controller/QuestionController` -
because this is still related to questions - at the bottom, create a new
method called `questionVote()`. Above, add the normal `@Route()`. For the URL,
how about `/questions/{slug}` - that's equal to the show page above - then
`/vote`. And because I *know* we'll need to generate a URL to this route for the
form, give it a name: `name=""`, how about, `app_question_vote`. Finally, add
`methods="POST"`.

[[[ code('4a61265541') ]]]

This means that I can *only* make a `POST` request to this endpoint. If we try
to make a `GET` request, the route won't match. That's nice for 2 reasons. First,
it's a best-practice: if an endpoint *changes* data on the server, it should
*not* allow `GET` requests. The second reason is... really an example of *why* this
best practice exists. If we allowed `GET` requests, then it would make it *too*
easy to vote: someone could post the voting URL somewhere and unknowing users
would vote *just* by clicking it. Worse, bots might *follow* that link and start
voting themselves.

*Anyways*, like before with the show page, we have a `{slug}` route wildcard that
we need to use to query for the `Question` object. Let's do that the same way:
add an argument with a `Question` type-hint. And, for now, just `dd($question)`.

[[[ code('0845ea7974') ]]]

## Adding the Form

Time for the form. In `show.html.twig`, add a `<form>` element above the vote
buttons... and a closing `</form>` after them. Inside the `form` tag, we need a few
things, like `action=""` set to `{{ path() }}` to generate a URL to the
`app_question_vote` route. Set the `slug` wildcard to `question.slug`. The form
tag also needs `method="POST"`.

[[[ code('dd5c87bcd8') ]]]

Cool! With any luck, when we refresh the page, we should be able to click either
button to submit to the endpoint. And... yes! Symfony queried
for the `Question` object and we dumped it.

## Getting the Request Object

This form doesn't *look* much like a traditional HTML form: it doesn't have any
inputs or other real fields. But because it *does* have these two buttons and each
has a `name="direction"` attribute, when we click a vote button, it will send a
`direction` POST field... exactly like if we had typed the word "up" in a text box
and submitted.

So the question now is: how can we read POST data from inside of Symfony? Well,
whenever you need to read POST data or query parameters or headers, what you're
*really* doing is reading information from the `Request`. And, in Symfony, there
is a `Request` *object* that holds *all* of this data. To read POST data, we
need to get the `Request` object!

And because needing the request is so common, you can get it in a controller
by using its type-hint. Check this out: add `Request` - make sure you get the one
from `HttpFoundation` - and then `$request`.

[[[ code('d60c162ddd') ]]]

This *looks* like service autowiring. It looks *just* like how we can type-hint
`EntityManagerInterface` to get that service. But... the truth is that the
`Request` is *not* a service in the container.

## All the Arguments Allowed to a Controller Method

What we're seeing here is one of the *final* cases of "things that you can have
as controller arguments". Let's review by listing *all* of the things that
we're allowed to have as arguments to a controller.

First, we can have an argument whose *name* matches one of the *wildcards* in
the route. Second, we can autowire services with their type-hint. Third, we can
type-hint an *entity* class to tell Symfony to automatically query for
it. And *finally*, we can type-hint the `Request` class to get the request. Yep,
this *specific* class has its own special case.

There *are* a few other possible types of arguments that you can have in your
controllers, but these are the main ones.

## Fetching POST Data

Now that we have the `Request` object, we're in luck! This is a simple
class: it has a bunch of methods & properties to help us read *anything* from
the request, like POST parameters, headers, cookies or the IP address. If you need
to read some info from the request, it's usually a matter of just looking at the
class or Googling:

> Symfony request ip address

to find the right method. Let's dump one part of the request:
`$request->request->all()`.

[[[ code('cd03f1b5a3') ]]]

Yeah, I know: it looks a little funny: `$request->request`?. Technically, POST
parameters are known as "request" parameters. So this `$request->request` is
a small object that holds *all* of the POST parameters. The `->all()` method
returns them as an array.

So when we go over now and refresh... yes! We see `'direction' => 'up'`!

## The UPDATE Query

Now, we're dangerous. In the controller, add `$direction = $request->`. Oh, and
here you can see some other ways to get data - like `$request->query` is how
you get query parameters and `$request->headers->get()` can be used to read a
header. In this case, use `$request->request->get('direction')`.

[[[ code('5a49b00ee5') ]]]

Now, if `$direction === 'up'`, then
`$question->setVotes($question->getVotes() + 1)`. Else if `$direction === 'down'`,
do the same thing, but ` - 1`.

[[[ code('27b4e158ec') ]]]

If the direction is some other value, let's just ignore it. That probably means
that someone is messing with our form and ignoring it is safe. At the bottom,
`dd($question)` to see what it looks like.

Ok, right now this question has 10 votes. When we refresh... yes! 11! Go back to
the show page and hit down. 9!

But... this did *not* save to the database yet: it's just updating the value on our
PHP object. And also, I think we can accomplish this `+ 1` and `- 1` logic in
a cleaner way.

Next, let's talk about anemic versus rich models. Then we'll learn how to make
an `UPDATE` query to update the vote count. Hint: we already know how to do this.
