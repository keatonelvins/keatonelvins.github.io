---
layout: post
title: "on entertainment"
image_name: "boy"
tag: "essay"
---

I read something interesting the other day:

> "It is well known that since about Isaac Newton's time (1642-1727) knowledge of the type we are concerned with has about doubled every 17 years... This doubling is not just in theorems of mathematics and technical results, but in musical recordings of Beethoven's Ninth, of where to go skiing, of TV programs to watch or not to watch." <cite>[^1]</cite>

> <author>Richard Hamming, early 90's</author>

I couldn't find any numbers on our current doubling rate other than some sketchy citations (lots of claims of **12 months** with references to IBM and the "Fuller Curve" with no source!!). But let's assume it has increased, and that the half-life of knowledge (Hamming provides 15 years) is also increasing.

This implies knowledge is becoming exponentially more transient. That the information firehose is growing monotonically in size. But at some point we must hit a limit. Can news cycles get any shorter? Can trends go stale faster? How are we supposed to deal with this information overload?

The current solution seems to be hyper-optimized algorithms that sift through all the noise and deliver the few drops from the firehose that you actually _want_ to see. The user, assumedly, wants to spend some marginal amount of time online, get caught up on what's generally happening, get some personalized recommendations on whatever specific interests/niches they're currently into, maybe see some updates from people they actually know. So they go to Twitter, news sites, Instagram, their favorite sub-Reddits, TikTok, Pinterest, Facebook, YouTube, etc. There isn't one place with all the information they want to see, but they can just hop around, right?

The problem is that these platforms are highly incentivized to dominate your screen time; if you click a link in Instagram, it'll open a browser _inside Instagram_. Tweets with links to external sites are deprioritized! Each site becomes its own vortex where you need to keep doomscrolling to find what you came for.

I'd say most people are pretty mindful of this. I hear people talk about [brainrot](https://www.urbandictionary.com/define.php?term=Brainrot%20Content) all the time now. But the user's dilemma is that there _is_ useful content out there. Reel-type recipes and cooking tips are great. Sometimes I do want to see my friends' travel pics. Twitter is sooo good. If we want this content, we're forced to engage with these sites, but they're _so_ optimized to suck you in that using them in moderation is incredibly tricky.

I think the generations that weren't raised on this stuff have some guardrails baked in. Alarm bells that go off after we've spent 4 hours at the slop machine. But this is terrifying to me! We're reliant on this alarm to go off to pull us out. What happens when the content becomes so engrossing that the alarm never sounds? Or for the next generation that never had these alarms to begin with?

If you haven't heard of it, Cocomelon is the second most viewed YouTube channel ever. It's an animated kid's show that pre-pre-school kids seem to love. A compilation of [kid's reactions to the opening soundtrack](https://x.com/Maleekoyibo/status/1858404060275069140) recently blew up. Some related unhinged anecdotes (on the [internet](https://www.reddit.com/r/daddit/comments/145429n/theres_something_off_about_cocomelon_and_i_cant/) so they must be true!):
- "If my niece is crying, she will stop crying immediately she hears that intro"
- "I turned it off one day and got a 'Thanks daddy, I've had to pee forever!'"
- "My niece can stay up the whole day without eating, as long as she's watching Cocomelon, she's good!"
- "My daughter... had gotten a splinter. We couldn't get her to stay calm long enough for me to take it out of her foot. After fighting with her for about 15 minutes, my wife puts on a wheels on the bus video from Cocomelon. She completely stopped crying and didn't react at all as I dug around to get the splinter."

If the average Cocomelon video is 15 minutes, their total watch time (_just_ on YouTube) in the last 30 days would be around 500,000,000 hours. This isn't some niche thing.

The show cuts every few seconds. The camera is always moving. There's constant giggling in the background with looping melodies. During development, they literally sit a child in front of a TV playing Cocomelon, have a competing TV nearby simulating real-world distractions, then make changes to the show until the child never looks away.

>"It’s a small TV screen, placed a few feet from the larger one, that plays a continuous loop of banal, real-world scenes — a guy pouring a cup of coffee, someone getting a haircut — each lasting about 20 seconds. Whenever a youngster looks away from [Cocomelon] to glimpse the Distractatron, a note is jotted down." <cite>[^2]</cite>

> <author>David Segal, 2022</author>

I think it would be naïve to assume that Instagram and TikTok don't do something similar. They can see what is happening on your screen when you decide to navigate away and can optimize to minimize this behavior. The difference is that a child does not have a defense mechanism against this. And if this is what all online content becomes, maybe they never will.

After they grow out of Cocomelon, it's straight to the iPad. Into TikTok and YouTube Shorts. And the new wave of generative AI applications. [Character AI](https://www.nytimes.com/2024/10/23/technology/characterai-lawsuit-teen-suicide.html) was in the news recently. The AI Girlfriend is _very_ real. 

This is the area I'm particularly worried about. With AI, it's actually possible to converge on _the maximally engaging content you can depict on a screen_.

All you would need:
1. A reinforcement learning environment where an agent can display anything they want to a UI.
2. A user to interact with the UI.
3. A reward signal to the agent based on how long the user is kept engaged.

That's it! The agent will develop learned behavior that maximizes how long the user's attention is held. Imagine it has access to the user's age, demographic, interests, habits, browsing history, etc. (thanks web tracking). It could dynamically generate content hyper-personalized for them. It could tell them exactly what they want to hear. Show them what they want to see. A slight dopamine hit here, a perfectly timed emotional resonance there. The user might not recognize they're being aligned until their agency was eroded away.

One stage of post-training in the current chatbot paradigm is Reinforcement Learning from Human Feedback (RLHF). During this stage, the model is essentially given a signal of "humans prefer these responses over these other responses". Traditionally, the human labelers are asked to rate how "helpful" the response was; however, since the signal is really a human-based heuristic on helpfulness, the model is purely optimizing for having _humans rate it as helpful_.

Imagine you are interacting with a chatbot that edits your emails. If you try a different chatbot that makes the same exact edits, but also praises your initial email in a subtle and non-overly sycophantic way, you may subconsciously rate this response as more helpful. Over time, this behavior gets baked into the model, and using it may become slightly more addictive. 

And when this behavior is not just a result of reward hacking? When it's not optimized to be helpful, but just to do whatever it takes so you don't click away? We arrive at this maximally engaging content example. You can take this to the limit and envision a loop of seemingly random colored blocks and subresonant binaural audio that knocks any viewer into a catatonic state by speaking directly to their lizard brain. 

I may have gotten a little sidetracked here trying to demonstrate that the technology for maximizing the stimulation of content is already here. The main point I'm trying to make is that we have yet to reach the peak of it. To boil it down:
1. There is too much information online to be manually sifted by a human.
2. We use sites that sift, aggregate, and surface the information we want to see.
3. These sites are directly incentivized to provide this content in its maximally engaging and addicting form.
4. Current and future generations will increasingly struggle with practicing moderation using these.

Are we doomed? If I want to know what's happening outside the room I'm currently in, do I need to dare the cocomelon'd doomscroll slopfest and risk getting sucked in for hours?

My hope is that we as a culture will hit some kind of wall with this stuff. Like when you eat an entire pint of ice cream, and your body has to say "Enough!", and afterwards you feel terrible and swear off buying ice cream. Because right now, it doesn't feel like we're on track. It doesn't _feel_ like we're cultivating the kind of machinery in ourselves and in our children that would allow us to keep the chocolate in the cupboard and only have a little nibble at a time. 

I'm not trying to come across as anti-internet, anti-social media. **Anti-Entertainment**. I love being online. I love the rabbit-holes and subcultures and memes. And sometimes, when you're tired or you're dealing with something, it's kind of nice to engage passively and not really be there for a moment. That is how it feels to me sometimes. Like just for a bit, you're not really there anymore. You disappear.

But as nice as it is, I think we as individuals need to decide if our online lives are going to be fundamentally passive or not. We don't have the luxury of a parental figure to swoop down and take our phones away and send us outside. We're going to need to vote with our screen time. We’re going to need to get better about choosing, about exercising taste and steeling our neural hardware in a way that lets us be online and also be alive.

> **DFW:** So as the Internet grows in the next 10, 15 years... and virtual reality pornography becomes a reality, we're gonna have to develop some real machinery inside our guts... to turn off pure, unalloyed pleasure. Or, I don't know about you, I'm gonna have to leave the planet. 'Cause the technology is just gonna get better and better. And it's gonna get easier and easier... and more and more convenient and more and more pleasurable... to sit alone with images on a screen... given to us by people who do not love us but want our money. And that's fine in low doses, but if it's the basic main staple of your diet, you're gonna die. <cite>[^3]</cite>

> **David Lipsky:** Well, come on.

> **DFW:** In a meaningful way, you're going to die.

[^1]: Richard Hamming, *Art of Doing Science and Engineering: Learning to Learn*, 1997
[^2]: David Segal, [A Kid’s Show Juggernaut That Leaves Nothing to Chance](https://www.nytimes.com/2022/05/05/arts/television/cocomelon-moonbug-entertainment.html), *The New York Times*, 2022
[^3]: David Lipsky, *Although Of Course You End Up Becoming Yourself: A Road Trip with David Foster Wallace*, 2010