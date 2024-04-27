If you’ve been in the software world for any amount of time you know that tech‐ nology changes. It is inevitable. Early in my career I became despondent because I thought I’d “missed the wave” of some technology or another. What I didn’t appreciate（欣赏、喜欢） at the time is that new technologies are a lot like buses—one comes every 15 minutes or so.

As architects, we have to remember that we cannot stop change, but we can manage it. In the rest of this chapter, we will cover the lessons（教训） the past can teach, a reminder that you can’t predict what the future holds and why you shouldn’t be so quick to jettison（放弃） legacy（传统） skillsets.

## Haven’t We Seen This Before?

> Those who cannot remember the past are condemned to（注定） repeat it. —George Santayana

For most developers, time began with the first language they learned. That initial experience becomes our lodestar （指导原则）and we often compare new languages to that inaugural（首次的，初始的） experience, which becomes our programming origin story. While our first language is indeed（的确；实际上） foundational, it can leave us short-sighted.

Many developers fail to appreciate（理解，明白） that the “new” feature just added to their favorite language has, in fact, been part of programming for decades（数十年）. When Java 8 added lambda expressions, some developers did not understand why they needed this “new fangled（新奇的）” feature. I’m sure more than a few Lisp programmers found that assertion amusing.

With experience, you will come to recognize that technologies often repeat themselves. Reflect（深思，反省） for a moment on distributed applications. Today it is fashionable to talk about microservices and serverless technologies. But step back for a minute and look at the evolution（进化，演化） from bespoke（定制化的） servers to virtual machines to containers. If you squint a bit（仔细点看一下）, you’ll recognize the echo（重复，回声） of serviceoriented architectures. In fact, some people like to say microservices are just SOA done right. Remember all the blog posts, articles, books, and conference talks devoted to SOA?

But step back even further. Isn’t SOA an awful（非常，极其） lot like Enterprise Java Beans? They were the “it technology” not that long ago, filling more than a few conference tracks. Even then though（即便如此）, our industry continued to build on the shoulders of what came before（以前的基础上）. Another step back and you might start to see the faint outlines of CORBA...

But that’s not to say things haven’t improved. Cloud computing removes many of the constraints that drove（推动） us toward shared infrastructure and all its consequences（后果）. Still, it is important to recognize how today’s technology is informed（影响） by its past. The past shapes the future and with that historical grounding, you can make educated（有根据的） guesses about where a given technology is headed（朝某个方向前进）.

When you know where our industry has come from, you can start to recognize patterns. Noticing a recycled technology enables you to “skip” something you’ve seen fail before. You can also start to see why something that didn’t work five years ago can work today.

But to see these patterns you have to study our past.

## Learn From the Past

> Study the past, if you would divine the future. —Confucius

Unlike most other industries(行业), ours doesn’t do a great job of learning from its past. Every discipline（知识领域） studies its history except（除了） ours. A typical（典型的） computer science curriculum（课程） might include a “history of programming languages” requirement but in general we give short shrift（忏悔，承认） to the past. The focus is always the cuttingedge（前沿的） language, the latest framework, the new hotness（热点）. But without that exposure（结露） to the past, we can’t appreciate when it is, indeed（的确）, repeating itself.

Software is an immature（未发育完全的） industry. Truth is, we don’t like to talk about our mistakes. But we should. I will never forget August 1, 2007. That evening, the 35W bridge over the Mississippi river collapsed（坍塌） in rush hour（在高峰时间） traffic killing 13 and injuring another 145. I had been on that very same bridge earlier that week.

That fall（那年秋天）, practicing engineers (as well as engineering students) around the world studied that tragedy（悲惨的事，灾难）. When the National Transportation Safety Board released their accident（事故） report I guarantee practicing engineers (as well as engineering students) around the world read it. And (I hope) the design flaw（缺陷） that ultimately（最终） led to that disaster will not be repeated in future bridge designs.

Mistakes Were Made...But Not By Me

Contrast（对比） that with software. When was the last time *you* were involved（参与） in a, shall we say, less than successful project? Did your company have a culture that allowed for an honest（坦诚的） conversation（对话，交谈）, were you allowed to discuss the lessons learned from your experiences, or were you forced to say the project went swimmingly（顺利）?

Years ago, I was on a project that, to say the least（至少可以说）, did not meet all our expectaions（期望）. After we wrapped up, my technical lead put together a postmortem presentation（验尸报告） so that other teams could learn from our mistakes. He showed it to our manager and she...changed some things. Our manager shared the presentation with her boss who also made some modifications. By the time we actually presented（展示） our findings, the message was so watered down（如此淡化） it basically professed（公开表明） the project was a rousing（振奋人心的） success!

I understand why we aren’t keen on sharing our failures with the rest of the industry, but when we can’t even have an internal discussion with our colleagues, we are more likely to make the same mistakes over and over again. We are also more apt（更有...倾向） to reinvent（重新发明） the wheel—unnecessarily.

## The Technology Merry-Go-Round

Early in my career, I sat down with my then manager and asked her a very simple question: Why did you decide to become a manager? She said she got tired of constantly having to learn new things, that she was weary（厌倦） of the technology merry-go-round. I will admit（承认）, as a fresh out-of-school developer, I didn’t understand her lament （哀叹）but having now lived through（经历） more front-end frameworks than I can count, I understand why she felt that way.

Technology *will* change and frankly（坦白的说） that is what attracts（吸引） many people to this industry. Learning new things keeps things fresh! But more than a few of us are guilty（内疚的） of practicing resume-driven design, of choosing a technology not for fitness to purpose, but to be able to add it to our CV.

## The Futility（徒劳无功的） of Predictions（预测）

As Yogi Berra once said, “It’s tough（艰难地） to make predictions, especially about the future.” This prescient quote should be brought to bear（就应该用这句有先见之明的话来证明这一点） anytime someone on your team thinks they have *finally* found *the* answer in the technology space. To be honest（说实在的）, the number of “can’t miss” technologies you’ve seen crater is proportional to the number of years you’ve been in the profession.

I distinctly remember attending a presentation where a prominent technology company introduced a fascinating new approach to front-end development. There was a palpable buzz in the jam-packed room. The introduction was a high‐ light of the conference and people started adopting it almost immediately. At the time, it seemed like *the* answer. A good friend of mine dove in head first in an effort to ride the wave. Conferences were littered with talks touting the new framework. Thousands of words were spilt in books, blogs, and articles.

A few years later, the script flipped. Teams were scrambling to remove a technol‐ ogy that had turned into an anchor on velocity. User interfaces had to be rewrit‐ ten, kicking off another cycle of technology evaluation. Even the original company behind it quietly moved on and today you are highly unlikely to run into anyone developing net new code on it.

Based purely on developer excitement, that technology seemed like a can’t miss proposition. But it did. At the same time it was announced, few would have believed we’d soon build applications with tens of thousands of lines of Java‐ Script. Or that Objective-C would soon be one of the most in demand languages. Or that Clojure would bring Lisp to the JVM.

We can’t predict the future, but there is a good chance it will be different than today. Technologies rarely become extinct—see COBOL. But I can guarantee in five years we’ll be all excited about something that hasn’t even been invented yet.

While we have no idea what *specific* technology will dominate in the years to come, broad trends can guide your decision making. Consider the shift from handcrafted servers to virtual machines to cloud hosting. Based on cost struc‐ tures alone, it is highly unlikely we will ever return to the era of treating servers like pets. And while we can’t predict which technologies will dominate the indus‐ try it seems likely innovation will continue to cause disruption.

It is a fool’s errand to make specific predictions in the technology space, but developers will no doubt continue to chase the latest hot technology.

The Value of Legacy Skillsets

Playing with the latest language or framework can be fun...but it is also frustrat‐ ing. Anyone that has tried to install a 0.x framework is bound to have experi‐

4 | Chapter 1: Technology Changes

enced the pain of a missing dependency or some other small difference between the Hello World example and their laptop.

The software industry fetishizes the new and many developers fear a legacy skill‐ set. No one wants to be the last of the X developers,3 instead preferring to work with the freshest tech. But we need to appreciate the central tenet of “bleeding edge”: the bleeding edge implies you will bleed. That might be perfectly fine when you are scratching your own itch at home, but you need to be more measured when suggesting technologies to your customers.

And just as it is far more exciting to announce the opening of a new bridge than it is to tout the maintenance of an existing one, many technologists prefer to be pioneers, preferring to be on the bleeding edge. While it can be more exciting to work on something new, it comes with its share of tradeoffs.

“Our application has *four* UI technologies...” —Anonymous Technology Professional

I bet you can appreciate the preceding quote and I suspect more than a few of you can relate. How did my student find herself in this situation? Admittedly, no one felt empowered to say “no” and everyone wanted to enable innovation. While I certainly applaud teams willing to try new things, we cannot simply chase a fad; we have to understand what a new technique, technology, approach, framework, language brings to the unique situation we find ourselves in.

Dunning-Kruger Effect

The Dunning-Kruger effect is a well-known cognitive bias. In essence, it means that more competent practitioners underestimate their abilities while those with less expertise believe they are better than they really are.

Some of the inaccurate evaluation of skills relates to an ignorance of what compe‐ tency actually looks like. As Donald Rumsfeld infamously said, “There are known knowns. These are things we know that we know. There are known unknowns. That is to say, there are things that we know we don’t know. But there are also unknown unknowns. There are things we don’t know we don’t know.” In other words, less competent individuals don’t know what they don’t know.

Keep the Dunning-Kruger effect in mind when someone on your team becomes enamored with yet another technology fad.

3 Though it can pay rather handsomely.

The Value of Legacy Skillsets | 5

Legacy Skillsets Can be Valuable Too!

I understand why most developers prefer new technologies—employability. From the beginning of our career we are taught a healthy fear of a legacy skillset. We don’t want to be the proverbial dinosaur of our organization.

While I generally advise people to stay current in our field, it is important to remember you *can* make a very good living working with “old,” established tech‐ nologies. Take COBOL for example. How many billions of lines of COBOL are there in the world? With fewer and fewer experienced COBOL experts in the workforce, the hourly rate of a COBOL programer is only going to go up.

Early on in my career I had a colleague who had a “retirement countdown screensaver.” Shortly after the counter hit zero, we had a retirement party for him. Imagine my surprise when I ran into him a few months later. I asked him if he was visiting for lunch and he said “No, it’s the darnedest thing, I’m working three days a week and they’re paying me *more* than when I was a full-time guy!” I thought to myself...that’s great, how do I get that gig?

My colleague had a legacy skillset—he knew COBOL, JCL, TELON, IMS, and a host of other mainframe technologies. He also knew the old systems intimately and he knew where all the skeletons were buried because he put most of them there. He was able to turn these “dead” skills into a solid payday, even after retire‐ ment.

But COBOL is just one example. Years ago, I was involved in a corporate merger and one of the systems we inherited was written in a technology we had zero experience with. We weren’t alone. In fact, the install base was so small there were just a handful of contractors in the country. The time pressure of the merger didn’t afford us the luxury of doing a rewrite immediately. We had to close the merger; only *then* we could address the technical debt. We had to make a tactical choice, leaving us with one option: pay inflated rates for a consultant that had the knowledge we lacked.

That said, once we completed the merger, one of our first priorities was removing the legacy dependency—and the very expensive contractor—from our portfolio. Again, it is in your best interest to have up-to-date skills. Sometimes we’re too quick to remove an old technology from our resumes. Other times we stay in the past too long. The challenge is knowing when to jump from the old to the new.

Eventually that legacy language will disappear. If you are investing in an older technology, you need to be ready to move on. Keep a weather eye on the horizon —is it getting harder to find work with that technology? Check job boards and industry news sites to see where the trends are heading. Ping your network, are your peers noticing decreasing demand? It will likely happen a year or two earlier than you expect. Be prepared and continue to invest in your skillset.

6 | Chapter 1: Technology Changes

If we are constantly experimenting with the new, we never develop any expertise in the now. Changing frameworks every few months might keep us at the fore‐ front of buzzword bingo, but it is hard to be good at something you’ve just picked up. And as tempting as it is to just throw away the old and rebuild on the ashes of the past,4 it is hard to deliver any business value if we are in a constant state of flux.

Over the course of the rest of this book, we’ll discuss various techniques and approaches you can utilize to help you make the right decision for your projects.

The rest of this book follows the arc from discovering a new technology through to keeping things running smoothly. Chapter 2 will help you keep up with the software industry followed by how to pick technologies in Chapter 3. Then you will learn about evaluating technologies in Chapter 4, and Chapter 5 will help you introduce those new technologies into your company. Finally, Chapter 6 will discuss how to maintain those technologies in your organization.

Katas

As mentioned in the preface, each chapter will present opportunities for effortful study. Take some time to consider the current trends. What makes for a success‐ ful technology? What characteristics might indicate a technology won’t work out in the end?

- Think back to a technology that, despite the fanfare, didn’t take over the world. Why didn’t it? What was lacking? Is there anything that could have changed the outcome?
- Consider a technology that *did* come to dominate. Why did it? What could have caused it to fail?
- Based on today’s trends, what technologies do you think are worth investing in over the next year? Why?
- Based on today’s trends, what technologies do you think are *not* worth investing in over the next year? Why not?