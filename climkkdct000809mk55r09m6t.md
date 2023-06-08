---
title: "Working on a Project Interview Assessment at Dwarves"
seoTitle: "Working on a Project Interview Assessment at Dwarves"
seoDescription: "Paid project interview on collecting Discord Engagement metrics for analytics, with teamwork, and a technical journey with Go."
datePublished: Thu Jun 08 2023 03:18:46 GMT+0000 (Coordinated Universal Time)
cuid: climkkdct000809mk55r09m6t
slug: working-on-a-project-interview-assessment-at-dwarves
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/Fi-GJaLRGKc/upload/0e869a4e820f90ee695659aee848fcdd.jpeg
tags: interview, analytics, go, projects, discord

---

I had my fair share of interviews. Most of them are "traditional" QnA sessions, and unpaid take-home projects, and coding assessments. Dwarves's paid project is different and left a good impression on me.

## The Beginning

At the time, I had been looking for a new job for a while, and somehow found Dwarves, which is a rare functional programming (Elixir) shop in Vietnam. Skimming the site and what people wrote, I found the organization fascinating: they value learning and craftsmanship. Getting intrigued by what I had discovered, I joined Dwarves's Discord, and sent a mail to apply, only to realize that I can ask people about open roles directly on Discord. I opened a ticket as instructed and got invited to a private channel to get to know the team. An online call (or kind of a screening round) was set up on the day after. A bounty (ticket of a task/paid project) was assigned to me a week later.

## Working on The Paid Project

If I were to pick a fancy name for what I had done, it would be: Discord Engagement Analytics. The project's end goal is to know Discord members' engagement with the server's channels or categories. Having screen time data (how much time are people spending on a particular channel, etc.) would be the perfect answer. However, all we have are messages and reactions from Discord, so we only can count the numbers (how many messages did a person sent to a channel, etc.), and use them to answer our questions imperfectly.

My leader/main helper for the project was @[Tom X Nguyen](@monotykamary) , who gave me a lot of useful suggestions. I occasionally get technical help from Nam and Huy, too.

In the first few days, I spent a bit of time to design the architecture and to get myself familiar with the codebases. Tom and I agreed that there would be two versions: AOT (stands for "ahead of time"; simpler; only allows current state queries) and JIT (stands for "just in time"; more complex; allow history state queries).

The simplest way to explain AOT and JIT's differences is to compare the data. Let us look at AOT's simplified table design:

* `discord_user_id`
    
* `channel_id`
    
* `message_count`
    
* `reaction_count`
    

The logic is that whenever we "catch" a new message, we increase `message_count`. This approach's advantages are its simplicity and low memory requirement. The disadvantage is that we only have the current state (how many messages have been since the beginning of time), and are unable to know the history state (how many messages were sent yesterday).

JIT's simplified table design looks like this:

* `message_id`
    
* `discord_user_id`
    
* `channel_id`
    
* `date_sent`
    

The logic is that whenever we "catch" a new message, we create a new record like that. The advantage of this approach is that we will be able to the query history state, and the disadvantage is that the data can potentially be huge, and we might need complex data processing and storage solutions.

From the design stage, Tom and I agreed that I would try to complete the AOT version in 2 weeks. I finished the AOT version in around 10 day-ish. My demonstration is to send a message in the private Discord server, and to see `message_count` increases by one. In the few days left, I worked a bit on the JIT version while waiting for the final assessment to come.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686145235568/a7873cda-d822-49e2-bb2e-4d4c47746cce.gif align="center")

(you can see here that `message_count` increased after I sent a new message; the same went for `reaction_count` when I sent reactions)

Overall, the project went well without any major issues. However, I got a bit unlucky at my company-wide demonstration: sharing my screen on Discord did not work, and the host had to skip my section.

## Conclusion

I learned a lot from this project: working with Discord API and understanding its limitation is one; new data engineering technical jargon to explain backend problems and solutions is another.

I heard about paid projects as an interviewing method before, but doing it with Dwarves is my first real experience with the method, and I feel fairly positive after all. The benefits are clear: the interviewer is going to have a clear understanding and a full evaluation of the interviewee, and the interviewee can also experience first-hand how is it working at the company. The drawback of time consumption for both sides can also be easily seen. Unable to be used at scale for manpower problems is another drawback that I find.

In the end, I enjoy my interviewing experience with Dwarves, and feel that they live up to their value of craftsmanship.