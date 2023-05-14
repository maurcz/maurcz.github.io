---
title: "People Over Systems"
date: 2022-09-03T23:13:58-04:00
categories: ["Python", "Opinion", "Interview"]
tags: ["interview", "hiring", "job", "mental health", "people", "systems", ]
summary: My new approach when looking for a software engineering job.
---

Last time I was looking for a job, while going through many tiring interview processes, I've deliberately decided not to ask about things like tech stack, CI/CD or any other topics that would make a "top things you should ask while interviewing" list. Might sound a bit crazy for a software engineer that has his fair share of situations that led some gray threads to grow out of his head, but that decision boils down to a simple motto: people over systems.

## Journey

I've worked for many different companies over my career, sometimes as a consultant, others as a full time employee. And even though conventions, youtube, stackoverflow, github, books and many experts are saying that they _know for sure_ how to get things right, literally all of my past employers had tons of issues. Broken processes, garbage code, legacy systems that crashed every Friday at around 5p - you name it. It was often hard not to feel like I'd just made a wrong decision and the solution was simply to move on to the next gig. And sometimes, I did. But again and again, after a sweet "honeymoon phase", the veil would collide and reveal the horrendous skeletons in the closet. Convoluted architectures due to over engineering. Duplicating logic in another language and calling them "tests". Shiny new tools all over that served zero to no purpose. Virtually no security.

When getting to that point in my journey at those companies, after my initial reaction of wanting to puke and hide in the closet, I'd try to follow the teachings of Uncle Bob and be as Pragmatic[^1] as possible to always strive to make things a bit better. But it's really not just about me, is it?

Companies don't get to terrible systems on a whim: it's a long process, full of individuals and micro decisions, that end up conceiving a bad design or terrible code committed to `main`. And if you dig deeper, oftentimes you'll find that every single software engineer involved with the system was just trying to do the best they could with the knowledge they had in their hands. Of course, there are some people that are, ahm, massive jerks, but I tend to believe that people are not coming to work in the morning and trying to find ways to deliberately make everyone's lives miserable (well, despite a couple of terrible professionals I came to know and hope you never will). No single developer can solve all the problems in the world - we **must** work together.

Management-savvy people might be thinking that those environments simply lacked proper leadership. It's true that good leaders can implement processes to anticipate and eliminate many issues, but honestly I can count in the fingers of my right hand the amount of _really_ good leaders I've interacted with along my career. And good leadership is something that's extremely hard to scale - empirically I'd argue that scaling good management practices is way harder than scaling technical pieces of a company.

The truth is that I really can't remember most of the systems I've worked with. Ask me about what piece of code I wrote 10 years ago and I'll tell you I have absolutely no idea what I was coding (and if I had to bet, it was terrible code). But I do remember my coworkers.

## The Man-Machine

"People over systems”. I say that sometimes and people _seem_ to understand right away. As software engineers, we're constantly bombarded by all sorts of mental attacks: burnout, impostor syndrome, analysis paralysis, anxiety, stress. Code has no feeling, but the humans that have violated pep8 completely, sure do. This profession trains us to forget that there's people behind `if`s and `for`s - all that matters is to find all the possible flaws in whatever someone else has created. Literally while writing this, my brain can't help but think of all the ways in which people can counter argument everything I've just written. This constant state of being defensive is really hard to deal with, and not everyone is able to cope with the fact that humans _will_ break eventually. And when humans break, so do the systems they write.

I'm not advocating to drop PR review processes or to stop challenging people when designing (or refactoring) systems. But we do have to force ourselves to remember that we're all humans with feelings and they _will_ affect our ability to write good code.

Ask yourself how many times you've seen people go for the suboptimal solution just because they can't stand the "other team". Or how a developer loses their will to improve and experiment due to a leader that micromanages. What about that one person that believes they are better than everyone else, disseminating a "us vs. them" mentality? And that time someone settled for a solution just because they were tired of dealing with a know-it-all. There are tons of other instances of humans interfering with our ability to create good code - and I feel like many software engineers just forget about it in their search for perfection.

So last time I had to look for a job, I decided to try to find a team that I could connect with. Took me a while and I passed on a couple of companies that claimed they had a "state of the art" codebase. It wasn't easy due to the limited time nature of interviews, but I did my best to assess wether I'd enjoy working with them. I've focused on trying to learn more about what they liked about the company (while forcing them to skip the classic "I like the people" answer) and how different engineers interacted with each other. The team mattered more than the code.

A year later, I honestly do not regret my decision and would do it again in a heartbeat. There are _many_ technical stuff that needs to be improved at my current company, but being backed up by people that respect you and will put your personal life and struggles in front of anything else is really powerful. Even though we're remote, I was able to connect to a few people on a personal level and know I'll be in touch regardless of the future.

Now, I acknowledge that this strategy might not be suitable for everyone. After all, I was always a bit more about "feeling" than "logic" when it comes to human interaction. If you're like me and is struggling to decide on your next move, consider thinking more about the hands than the words that come out of the keyboard. You'll still need to deal with bugs, memory leaks, broken pipelines and all the beautiful things that happen when you write software at scale - but at least you won't have to deal with people that "like offending people because they think people who get offended should be offended"[^2].

Thanks and see you next time!

[^1]: A miserable attempt to make a joke with this [book](https://en.wikipedia.org/wiki/The_Pragmatic_Programmer).

[^2]: This is a famous quote by Linus Torvalds. Last time I heard of him, [he was trying to improve](https://arstechnica.com/gadgets/2018/09/linus-torvalds-apologizes-for-years-of-being-a-jerk-takes-time-off-to-learn-empathy/), but as you can get by the statements at the end of that article, there will always be people that view code as the only thing that matters.


