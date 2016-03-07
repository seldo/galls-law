# [fit] Cheating Gall's Law
# [fit] How we split a
# [fit] monolith
# [fit] and lived to tell the tale

^ npm last year shipped a big rewrite of its registry service, a successful rewrite. this is such an unusual event in our careers that we thought we'd talk a bit about why it's hard & how we did it. Let's start with Gall's Law.

---

![left](images/alpaca.jpg)
# [fit] Laurie Voss
## [fit] CTO, ![](images/npm.png)

## [fit] @seldo

^ How many of you have ever used npm to install something? How many of you use it daily? The story I'm about to tell will be very relevant to you both because the work we did affected you & because the technique I'm about to describe will be helpful.

---

# [fit] Gall's Law

# [fit] A complex system that works is
# [fit] invariably found to have evolved
# [fit] from a simple system that worked.

^ Gall's law says: a complex system that works is invariably found to have evolved from a simple system that worked. A complex system designed from scratch never works and cannot be patched up to make it work. You have to start over with a working simple system.

---

# [fit] Systemantics:
# [fit] How Systems Really Work
# [fit] and How They Fail

^ John Gall: a pediatrician. Interested in systems theory. This is the title of his big book, which is a darkly funny set of observations on systems of all kind. Insight: Systems operate in a constant state of failure. Gall is just a bundle of fun here. Let's apply this to software systems.

---

# [fit] starting simple
# [fit] often means starting with a
# [fit] monolith

^ Start with a simple system that works: you write a small thing & then build on it. You've done this! It's how every system builds organically if you don't think about it much but just make things.

---

![](images/Monolith-Sun-Moon.png)

^ What you end up with is a monolith! What's a monolith besides a big black thing with Richard Strauss musical accompaniment?

---

![](images/Monolith-Sun-Moon.png)
# [fit] monolith
# [fit] everything in one process

^ A monolith is a disparaging term for everything in one process. Really easy to write, often easy to deploy. Easy to think about. Easy to squish around as you're discovering what your app needs to be. npm registry started with one of these inside couchdb.

---

![](images/Monolith-Sun-Moon.png)
# [fit] monoliths
# [fit] work just fine

^ The registry was just fine as a couchdb app, using couchdb's auth. When there were 10K modules, practically anything was going to work. This illustrates a general true thing: when you're small, just about everything works.

---

# [fit] whatever it takes
# [fit] to build a system
# [fit] that satisfies your users

^ When you start, your job is to make a thing people want to use. It's hard to make something that delights your users and is a viable product. You don't have a scaling problem. Then, if you're lucky, something terrible happens.

---

# [fit] success!
# [fit] now scale it.

^ Success happens. Nightmare! Your next job is to SURVIVE the fact that people want to use it. Recall the story of the early days of Twitter, when they built your timeline by querying mysql. That fell over hard once Twitter started succeeding. Your first reaction is probably to make your monolith bigger, maybe the size of Jupiter. (Geddit? Jupiter?)

---

![](images/easterisland021.jpg)

^ The next thing you do is you start making lots of copies of your now-huge monolith. This was also how Arthur C. Clarke scaled his monolith, as it happens.

---

![](images/easterisland021.jpg)

# [fit] scaling monoliths
# [fit] many copies of the full thing

^ Scaling horizontally is very effective! You've got a single widget. You just make more copies of it & load balance among them. If your database can cope with that, you're in good shape for a while. But sometimes you *really* succeed.

---

![fit](images/exponential-growth.png)

^ This is all of YOU using node to write web apps. This was node growing & the registry being good enough at making its users happy. We are now approaching a billion downloads a *week*.

---

![](images/easterisland021.jpg)
# [fit] exponential monoliths
# [fit] were going to be expensive

^ Exponentially larger numbers of monoliths get expensive. We have to scale some way other than making a lot of copies of our couchdb.

---

# [fit] splitting the
# [fit] monolith

^ We need to scale differently: we break up the monolith.

---

# [fit] yay microservices?

^ Just rewrite it all in a million tiny pieces! no problem!

---

# [fit] Your monolith is complex.
# [fit] A split system is more complex.

^ But this adds complexity. And complexity is the enemy of everything.

---

# [fit] what did that Gall guy say about
# [fit] complex working systems?

^ And didn't we just hear something about complex systems? Your target for a rewrite is the full complicated thing that's running in production today, not the simple thing you started with. Famous problem: second system effect.

---

# [fit] how do you split
# [fit] a monolith
# [fit] successfully?

^ So how to rewrite without hitting Gall's law, and creating a broken second system? npm is not on fire today, so obviously there's a way.

---

# [fit] Let's cheat.

^ I promised you we'd be cheating. You can't break the law, no matter how tempted you are to argue with it. You have to write simple working systems first.

---

# [fit] Q: How do you cheat?
# [fit] A: By not rewriting the whole thing.

^ So how do you cheat? By rewriting small pieces. There are a lot of ways to do this. I'm going to tell you the one npm used, because it's a networked service. If you're writing web services, this might work for you.

---

# [fit] slice off a part of the system
# [fit] into a module
# [fit] with a clearly-defined interface

^ Find some part of your system, some feature, can be thought of as a module. Maybe it even is a module inside the monolith. This is the heart of modularization. It's how you slice apart code.

---

# [fit] then write a
# [fit] second implementation
# [fit] of that interface

^ Then write that module again. This time standalone, outside the monolith. Ideally, make it a tiny slice that answers a whole type of request. Maybe it's login. Maybe it's the home page.

---

# [fit] send requests to that
# [fit] second implementation
# [fit] with a proxy

^ Now put a proxy in front of your monolith, and send just those requests to your new service.

---

![fit](images/use_a_proxy.png)

^ For you visual thinkers, here's this incredibly simple approach. Let's look at an example.

---

![](images/registry_monolith.png)
# [fit] npm's monolith:
# [fit] embedded in couchdb

^ npm registry 1.0: a monolith npm like a lot of systems was originally very simple. The registry was a few thousand lines of javascript embedded inside CouchDB. Auth was couch's auth. Package tarballs stored inside couch.

---

![fit](images/registry_monolith.png)

^ Here's npm's monolith pre-split. Now watch what we do.

---

![fit](images/proxying_1.png)

^ We used Varnish for our proxy. We sent package downloads to a box full of tarball files with nginx on it. Meanwhile, everything else was still going to couchDB. This simple technique was our first step to breaking up the monolith. This one change relieved about 90% of our couchdb performance problems.

---

# [fit] now let's make our proxy smart
# [fit] Replace Varnish
# [fit] with a microservice

^ You can put some logic into Varnish, but get a lot more mileage out of proxying at the application level. Way more control & awareness. We used node. You can use whatever you want.

---

![fit](images/proxying_2.png)

^ We called it the registry frontdoor, because all traffic goes through it. You'll notice that it's doing nothing right now.

---

# [fit] now do it again

^ (go fast through these)

---

![fit](images/proxying_3.png)

^ Authorization & package validation are now pulled out of couch into services. Woah, we just rewrote two things.

---

# [fit] and again

---

![fit](images/registry_john_madden.png)

^ Here's a simplified diagram of our services. We changed our platform AND our database. We went from js inside couchdb to js in node services. Also changed how we stored our data. Want to show you something cool.

---

![fit](images/registry_monolith_still.png)

^ Notice something here? It's our original monolith, still there, chugging along. What it does is reduced to the things couchdb is fantastic at: storing json blobs & replicating the data to the rest of our system.

---

# [fit] beat Gall's Law with
# [fit] modularity
# [fit] aka information hiding

^ We beat Gall's Law with modularity. We hid the complexity. You think about this at the code level, but here we're doing it at the service level. The implementation of each module is hidden behind the API of the service.

---

# [fit] each service is a module
# [fit] a simple system
# [fit] when viewed in isolation

^ Gall's Law is still in force. We still must write simple things & build from there. In this case, the new simple system is the new service. The existing working system evolved slightly to be more complex.

---

# [fit] you have a working system
# [fit] every step of the way

^ This approach is much safer than a total rewrite. You can test each piece separately. This system is getting steadily more complex, but that's okay. You're evolving a working system. This approach won't always be what you need, but it is fantastic when you do. Put it into your back pocket and pull it out whenever you're looking at a rewrite.

---

# [fit] simple working service first
# [fit] scale it later

^ Build first, and scale later. Don't be ridiculous about scaling, but don't worry about it. You'll be able to afford it later if you've built something people want. Be ruthless about this.

---

# [fit] respect Gall:
# [fit] rewrite in pieces

^ Respect Gall. Avoid complexity of a full rewrite by rewriting small parts. Modularity!

---

# [fit] use a proxy
# [fit] to divide & conquer

^ Use a proxy to divide and conquer. It lets you chop up your problem into smaller ones. You can test your assumptions. Send a portion of traffic through & load test. Flip back if you've got bugs.

---

# [fit] Be bold
# [fit] you can change your system

^ Be bold. I've seen needed rewrites never even start because of fear. I cannot help you with the politics, but I can tell you that you don't have to be afraid. You can change one small piece of a system at a time. This approach works.

---

# [fit] Nobody noticed
# [fit] that we slowly replaced
# [fit] the entire npm registry

^ How well did it work? We rewrote the entire registry, and you didn't notice. Over a year we had 120 seconds of downtime due to exactly 1 bad deploy. If we can do it, you can do it.

---

# [fit] ![](images/npm.png) loves you
# [fit] `npm install -g npm@latest`

^ So thank you! And remember to upgrade your copy of npm.
