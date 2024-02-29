+++
title = 'The Return of Woodpecker'
date = 2024-02-27T21:05:27-05:00
+++

## The Past

In my free time, I like to design small electronic systems. Nothing is
more satisfying than seeing a circuit work for the first time, even if
it just blinks an LED. Electronics is also a nice hobby because it
doesn't require a lot of space.

However, a few years ago, I realized that my storage situation was
getting out of hand. A few times, I purchased components for a new
prototype, only to realize that I had already some in a cardboard box
somewhere in my storage closet. I had already gone through a few
rounds of sorting and tidying up my collection at this point, but it
felt like I just couldn't compete with the number of individual
components growing steadily in my collection.

That's when I decided that I needed to build an inventory app. I
called it Woodpecker and it was pretty nice! It was made using
Flutter, used little QR-codes printed on small stickers to identify
items and stored all its data in an app service called Nhost.

This project worked really well and helped me keep my inventory in
check for about a year, but it had some issues. First, the little
[Avery sticky labels][1] it was using were a pain to print and didn't
stick to items very well. But most importantly, the database service
was giving me a lot of issues. [Nhost's pricing][2] is pretty steep,
jumping from a free tier where the project pauses after some
inactivity directly to a 25$/month solution. Since I couldn't really
justify spending this amount of money on this app, I constantly had to
log into their control dashboard and reactivate the app when it was in
sleep. But also, after a year of this, the project changed version in
a non-backward compatible way and ate all my data!

## The Present

I am now back to square one, with an electronics inventory in
shambles. I want to bring Woodpecker back, but in a form that will
last for longer this time.

To solve the issue of labels, I've settled on using a small wireless
thermal laser printer ([Phomemo M110][3]). I'll certainly have some
fiddling to do to get it working with my app, but there is [a Dart
library](https://pub.dev/documentation/phomemo/latest/) that looks
promising so it should be within reach.

The harder issue is the one of the database backend. So let's explore
our solutions.

## The Future

In database terms, my inventory is really small potatoes. It would
easily fit on the device in a small MySQL database. Heck, storing it
as JSON in a text file would certainly do the trick.

But no, for some reason, I don't want to do that. First because I
would like it to scale reasonnably well just in case this app turns
into something a small business may want to use. But mostly because I
would want to be able to access the data from other phones and also
ideally from the web in the future.

Yes, I have thought of using Dropbox (or another file sharing service)
to replicate the file accross devices. No, I don't like it either.

Using an object storage service like Amazon S3 would be the logical
next step. If I don't want to use a file-sharing service, just make
one directly using S3. I'll consider this a solution only if
everything else fails. I would much prefer using a "real" database.

DynamoDB is a real database. It is pay-as-you-go, which scales nicely
with use. For my needs, it would only cost cents per month! Also, it
is reliable and stable. It will still be here in 5 years and it won't
eat my data. One big issue though; My application requires full-text
search (ideally fuzzy and all), and DynamoDB does not support
it natively. I would need to use AWS OpenSearch or another equivalent
service but they are all priced per hour ($$$).

On Google's side, Cloud Firestore [does not
support](https://firebase.google.com/docs/firestore/solutions/search)
native full-text search either. All their other database offerings
look to be priced per hour, although Cloud Spanner is quite misleading
on this topic.

To complete the "big three" cloud service providers, I looked at
Azure. On their side, everything looks to be priced per hour as well
instead of pay-as-you-go, which means event the cheapest solution is
still more than 5$/month which sounds ridiculous for such a trivial
application. One caveat, their Azure SQL Database product includes a
[serverless compute
tier](https://learn.microsoft.com/en-us/azure/azure-sql/database/serverless-tier-overview?view=azuresql&tabs=general-purpose). It
still bills per hour but apparently automatically pauses the server
when it detects inactivity, bringing the hourly cost to zero. This
*could* be good, but I'm not convinced by their documentation.

I looked at the "cool kids" databases next, starting with MongoDB. It
includes a serverless price model that looks very cheap on the surface
and it also supports full text search.

The issue is that I don't know how many "read capacity units" doing
the text search uses, and doing a google search turns up a lot of
troubling [forum
posts](https://www.mongodb.com/community/forums/t/warning-mongo-db-serverless-pricing/187607)
about prices going through the roof. Overall, this still sounds like a
viable option (the best so far), but I would really need to put alarms
in place to monitor usage and cost.

I gave a glance at Couchbase, but as far as I can see, it only offers
per-hour pricing so I just discarded it.

There is also [Supabase](https://supabase.com/), which looks to be a
direct competitor to Nhost. They have a free 500MB tier, but the
database shuts down after a week of inactivity. I could setup a lambda
function to run once a day and make a dummy call to it to keep it
active, but I'd rather find a different solution.

Cockroach DB is another one that offers full Postgres functionality
and has a completely free tier with 50M request units and 10 GB. Very nice!

Lastly, Oracle also offers an "always free tier" with up to 20 GB
storage. Cool stuff!

CockroachDB and Oracle both look like I should be able to run my
application with plenty of capacity to spare for free.



[1]: https://www.avery.com/products/labels/5418
[2]: https://nhost.io/pricing
[3]: https://phomemo.com/en-ca/products/m110-label-maker
