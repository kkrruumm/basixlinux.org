title: basix philosophy

# What is this page?

This page exists to elaborate on why Basix is the way it is, and to clarify on a few questions I'm sure people have.

Given I'm largely not a fan of blanket statementing rules, this page may also be treated as a form of code of conduct in lieu of one.

# Why does Basix use XYZ software instead of ZYX by default?

The default set of software is largely based on the lead Basix developers' (thats me hi) opinions on how a system should be, but what that largely means is a system that is sensibly usable by default but can be easily modified.

An example of this could be Busybox. Busybox is admittedly not a great set of coreutils for interactive use but it's largely a good common base for software to have. Using this as an example, this allows things like `bpm` to target a more standard implementation of coreutils which would enable one to swap it out for something like GNU coreutils or maybe something like [chimerautils](https://github.com/chimera-linux/chimerautils) (what i use) without fighting to do so.

This idea of software that is easy to replace is why the system is the way it is.

On the note of s6, I find that s6 provides the "sanest" implementation of most everything related to init/service management, where "sane" may be somewhat arbitrary/anecdotal. Note that this doesn't mean things are chosen solely on vibes alone.

There are also some "quality" requirements, see the replacement of OpenSSL with LibreSSL.

In addition, absolutely none of my opinions should matter to you- if you don't want s6, simply don't install it and toggle off the global `services` use flag via bpm and install whatever the hell you want.

# Does Basix have an LLM policy?

Kinda. This is a bit of a tough subject, so I'm going to provide a bit of a ballpark explanation for what I think is acceptable and what isn't.

Those who are caught vibecoding contributions will be permanently banned from further contribution, full stop. With that said, I appreciate that LLMs are rather useful for gathering information and may aid in familiarity with something when used as "a search engine on steroids."

What I would like users of LLMs to consider, however, is the impact these disgusting corporations are having on the climate, the impact they have on those who unfortunately live near a datacenter, and the impact they may have on their users (watch Idiocracy).

To TL;DR, I expect all contributions to be well understood by the contributor and hand-written by said contributor. Not that I'll reasonably know if you've done otherwise, but don't be a dick. If you use an LLM to gain understanding and information, whatever, fuck it, given in my mind it's largely indifferent from the way things were prior to this shitfest- that being gathering information off of half-related stackoverflow threads and some random dudes blog found via a search engine (though, "random dudes blog" is very likely to be much more valid).

In addition, if you send in some vibecoded bullshit, why should anyone show you the respect of reading through it and interacting at all if you can't show the respect of contributing something of quality that you put effort into? Don't be an asshole.

# When you say Basix is non-corporate, what do you mean?

I am largely not a fan of things that are controlled by corporations. Corporate contributions to a project (the Linux kernel) for example may be fine, but corporate control over a project is not. Rarely (probably never) do corporate-owned projects actually care about the end user or the community around their software, and may rugpull users at any moment to appease shareholders. This may be blamed on the existence of capital in general, or may be blamed on the concept of shareholders. Either way, Basix will have none of it.

# Is Basix a political project then?

Everything is political.

# Who is allowed to be part of this community?

Everyone. In any Basix social groups, there are certain types of behavior that are completely impermissible. Things such as homophobia, transphobia, cisphobia, racism, etc. are not allowed. Do not be the type of person that would make me want to apply rules to a community that is largely loosely directed.

TL;DR, **don't be an asshole**. Everyone is different, otherwise reality would be incredibly boring.

# Is this a democracy?

No. I am an evil dictator.

On a serious note, decisions related to the distro may largely be made based on consensus amongst whatever core contributors may come to exist. Bluntly, commit access to stuff will not be given to others- you can fork off or mix your own repositories with core, but that isn't to suggest that I am willing to shrug off the opinions and input others may have. Let's have a discussion.

Basix isn't meant to please the masses, but exists as a reasonably small meta-distribution those who have opinions^tm can turn into something that they like and that is actually theirs, while also being usable on its own (note: I started this distro for myself).

# This system is very difficult to use.

I would disagree. The base system is largely kept rather small and conceptually simple specifically for ease of understanding and to allow users to reap what they sow. 

I'd also like to suggest that "difficulty" is largely an arbitrary concept here, I would suggest that more "fleshed out" distributions these days are more difficult to get an actual understanding of due to the sheer amount of moving parts and otherwise.

# Basix doesn't have a specific bit of software I want!

Yeah, in its current state, it probably doesn't. Make a pull request.

In general, there are some requirements as to what gets to be in the repo, though:

1. Things that have their purpose already covered by another item in the repo are unlikely to be accepted, though this isn't a super strict rule.

2. Repo fluff like themes, 300 fetch programs, fonts that don't exist for the sake of readability, icon themes, etc. belong in your own repo.

3. There's a bit of an arbitrary quality requirement, I can't provide a hard set definition for this here, so this will be applied in the moment if needed.

# Where did the name "Basix" come from?

Originally, this stood for "Bash Linux." This doesn't really hold anymore given all of this is now POSIX shell, but what it does mean is that it **isn't** pronounced "basics"
