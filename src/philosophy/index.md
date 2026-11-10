title: Basix Philosophy

# What is this page?

This page exists to elaborate on why Basix is the way it is, and to clarify on a few questions I'm sure people have.

Given I'm largely not a fan of blanket statementing rules, this page may also be treated as a form of code of conduct in lieu of one.

# Why does Basix use XYZ software instead of ZYX by default?

The default set of software is largely based on the lead Basix developers' (thats me hi) opinions on how a system should be, but what that largely means is a system that is sensibly usable by default but can be easily modified.

An example of this could be Busybox. Busybox may not be the desired set of coreutils but it's largely a good common base for software to have. Using this as an example, this allows things like `bpm` to target a more standard implementation of coreutils which would enable one to swap it out for something like GNU coreutils or maybe something like [chimerautils](https://github.com/chimera-linux/chimerautils) without fighting to do so.

This idea of software that is easy to replace is why the system is the way it is.

There are also some "quality" requirements, see the replacement of OpenSSL with LibreSSL.

In addition, absolutely none of my opinions should matter to you- if you don't want a particular component, simply don't install it (or uninstall it), and replace it with something else. If needed, you can shadow packages in the repository from your own as well.

# Does Basix have an LLM policy?

Kinda. This is a tough subject, so I'm going to provide a ballpark explanation for what I consider acceptable and what I consider unacceptable.

To explain this, I feel a definition of "vibecoding" is required.

What I consider "vibecoding" to be would be "<insert LLM here>, write me an init system. Make no mistakes." What I do *not* consider to be vibecoding, however, would be questions such as "<insert LLM here>, how does XYZ work?" and using it to gather a proper, genuine understanding of what you are interacting with. These are two distinct usages of slopbots, be responsible.

What I would like users of LLMs to consider, however, is the impact these corporations are having on the climate, the impact they have on those who unfortunately live near a datacenter, and the impact they may have on the user (watch Idiocracy). I would also like to ask if it is *really* necessary to waste water and fry your brain to understand software that is conceptually simple and small. I also can't afford storage at the moment due to this, so I'm pretty irritated personally.

Any contributions that are just copy and pastes from an LLM will result in that "contributor" being permanently banned from further contribution. Do not be a meat proxy.

I'd also like to throw it out there that the (maybe unintentional?) side effect of LLMs resulting in full decompilations of proprietary software left and right these days and perhaps the utter death of copyright is pretty cool.

To sum this up, I expect all contributions to be well understood by the contributor and hand-written by said contributor- I may not know if you've done otherwise admittedly, but please do not be a dick. If you use an LLM to gain understanding and information, that's fine. In my opinion, this is largely indifferent from the way things were prior to the age of clankers- that being gathering information off of half-related stackoverflow threads and some random dudes blog found via a search engine (though, do not discredit "random dudes blog," these are more often far more valid and correct than most LLM output I see.) Sticking ones head in the sand and pretending these don't exist and aren't to some degree useful for auditing, link aggregation, and for gathering an understanding of things is also rather ignorant, in my opinion.

# When you say Basix is non-corporate, what do you mean?

I am not a fan of things that are controlled by corporations. Corporate contributions to a project (the Linux kernel for example) may be fine and rather beneficial actually, but corporate control over a project is not, open source or not. Rarely (probably never) do corporate-owned projects actually care about the end user or the community around their software, and may rugpull users at any moment to appease shareholders. Not to mention, corporate-owned projects have a knack for collecting data/telemetry on their users, which is *fundamentally unacceptable*.

# Is Basix a political project then?

Everything is political.

# Who is allowed to be part of this community?

Everyone. In any Basix social groups, there are certain types of behavior that are completely impermissible. Things such as homophobia, transphobia, cisphobia, racism, etc. are not allowed. Do not be the type of person that would make me want to apply rules to a community that is largely loosely directed.

TL;DR, **don't be an asshole**. Everyone is different, otherwise life would be incredibly boring.

# Is this a democracy?

No. This project is run with the "benevolent dictator" mindset.

Decisions related to the distribution may largely be made based on consensus amongst whatever core developers may come to exist (for now it's unfortunately just me). However, commit access to `core` is very unlikely to be given to anyone else. If desired, you may fork off or mix your own repositories with core (this is part of the point of the distro), but that isn't meant to suggest that I am willing to shrug off opinions, input, and contributions other people may have. Let's have a discussion.

Basix pleasing the masses and newer unix-like users is a non-goal. It exists as a reasonably small meta-distribution for those who have opinions^tm.

# This system is very difficult to use.

I disagree. The base system is kept pretty small and conceptually simple specifically so there is *less stuff* on the system that needs to be understood, and to thus allow users to reap what they sow. For more advanced users, this is quite the blessing- less random stuff on the system they didn't ask for by default to deal with.

This is directly antithetical to most other distributions that conflate "literally everything the user could possibly even think of wanting is already here" with ease of use. If a distribution does not easily fit in the admins mind in its entirety, there's a problem.

# Basix doesn't have a specific bit of software I want!

Yeah, in its current state, it probably doesn't. Make a pull request.

In general, there are some requirements as to what gets to be in the repo, though:

1. Things that have their purpose already covered by another item in the repo are unlikely to be accepted, though this isn't a super strict rule.

2. Repo fluff like themes, 300 fetch programs, fonts that don't exist for the sake of readability, icon themes, etc. belong in your own repo.

3. There's a bit of an arbitrary quality requirement, I can't provide a hard set definition for this here, so this will be applied in the moment if needed.

4. Projects that are vibecoded (see above) are not to be packaged in core.

# Does Basix include any proprietary software?

Ideally it wouldn't at all, but it almost doesn't. The `linux-firmware` package unfortunately needs to exist for most hardware to work.

External to drivers, microcode, etc. that are required for hardware to work, proprietary software is strictly forbidden.

# Does Basix collect any information about its users?

No.

# Where did the name "Basix" come from?

Originally, this stood for "Bash Linux." This doesn't really hold anymore given all of this is now POSIX shell, but what it does mean is that it **isn't** pronounced "basics"
