---
title: Advice to the Next Technical Leadership
layout: post
excerpt: Notes on decision making as a technical leader
---

# Advice to the Next Technical Leadership

Brian asked for example, why I picked RDS for our chartio db. 

Here’s how I think about things:

Always maximize your time for your customer, always use the right tool for the right job. I know, that feels like a non-answer. Hear me out--  know that while you are technologists first and foremost, the higher you get in a company, the more important it is that you’re also a great judge. With your decisions you’ll either give the company time to attack customer issues (and therefore money), or you’ll take it away. Be mindful daily/weekly/monthly how much time those around you are attacking new customer issues and how often they’re not. If you fall into a slump where you’re not, get the fuck out of it immediately! Immediately.

Some Micah tips that’ll help you make the right decisions--

Spend as little time as possible:

 - Maintaining infrastructure. Just use things that you’ve already got stable, or things someone else maintains (like cloud-offerings)
 - Maintaining troublesome code. If you wrote something that needs constant fixes, you wrote it wrong. There is no debating this. Learn where your judgement was off, fix it immediately and move on. If someone writes code that constantly needs bux-fix style maintaining, fire them.
 - Adding new technologies. When you add new tech, you have to write new infra-level code to blend it in. Then you have to educate your team on it. Then you have to find out the parts of it that don’t work the way you want. Then developing code around the parts of it that don’t work they way you want. 
 - Rewriting systems, just to get it to language X, or platform Y. Rewrite when not rewriting is your biggest problem.
 - Doing engineering things for engineering sake. Not everything tech-wise at GT is as optimized as it could be. Don’t optimize everything. Do enough that GT won’t take a loss and move on to helping customers.
 - In processes/meetings that don’t add unanimously obvious significant value
 - Building for scale you’re not at. Let me position this one for you. Technically, GT is setup to for instance handle nearly infinite API scale, because the API was architected to scale horizontally. So you may be like, “micah, why did you architect that for super scale? How are you not contradicting yourself?” Well, the work for scaling that from 1 VM to 2 was _necessary work for scale we have_, and the work involved from scaling from 2 to 1000 came for free with Azure Infrastructure. Another example: there is a scale of order submission (that we are VERY FAR AWAY FROM) where maybe, maybe you would need a nosql/distributed database. The implications of dropping ACID and doing this in our codebase would be massive cost, labor, and ongoing maintenance. If you did that now, that’s an example of making a decision for scale we’re nowhere near that would be hugely detrimental to our business. And i would find out you made it, and come to your home, and murder you and your family.


Spend as much time as possible:

 - ATTACKING ANOTHER CUSTOMER ISSUE
 - MAKING CUSTOMER HAVE HAPPY FACE


While we’re here, **don’t write any code, system, or API that doesn’t scale horizontally** (exception: scripts don’t need to scale horizontally obvs). And conversely, avoid using infrastructure that forces you into distributed systems problems _unless you are forced into that scale_, which as we covered, GT really really isn’t.  While we’re here and talking distsys, never write your own distributed systems code when you don’t have to. Go with a platform/framework that already solves it for you. Please tell me that if you’re reading this, you fully understand and can explain CAP theorem. How can you architect around distsys or understand why it needs to be avoided at all costs without getting CAP? Avoid distsys. Avoid distsys. Avoid distsys. Avoid distsys!!!

How to add new tech-- prototype and test the fuck out of it first. There is a reason why i wrote the slack plugin for CS to find customers as an azure function before i moved business logic over to azure functions-- i wanted to test how it worked before i staked part of the business on it. And look what we learned-- 1) it sucks, 2) javascript to mssql libraries blow. Important lessons! Pain avoided!

We have infrastructure level code in C# and Clojure. By infrastructure level code, i mean database, rmq, logging, caching, azure interop-- all the stuff you need when you’re trying to achieve business goals-- already completed. Solve problems via that infrastructure. Do not reinvent it or move on from it without serious, serious, serious reasons. GT could easily do 10-100x the throughput its doing with our current infra and platform (after a few clicks in azure web to allow vms to scale past current scale boundaries). Easily. Attack business problems! Make revenue! Where do you think your raises come from? ;)

Your job is to place developers in a box where they work on customer problems. If you already have all the infrastructure level code and decisions done, you will get your devs working on customer issues with 99% of their time. So have all the infra-level stuff done and leave them with only the business logic. It’s like only leaving them with the necessary complexity to solve. **Never have weak devs do your infra-level code**. Do it yourself. Make it dead-simple. Again, never trust anyone to build your infra.

So why did i choose RDS? It did everything we needed it to do for years at a cost of less than $100 a month. The cost of hosting my own backed up database was negligible against amazon-managed. There isn’t a cheaper option. The platforms we used to integrate with it were friendly with AWS-based tech (same cloud). Would argue mammoth’s current architecture is a cautionary tale in over engineering for scale we don’t have…. Beware! It’s a trap that’s easy to fall into. 

GT has never had a breaking change in any of its APIs. After you get a client on your platform, don’t make them spend additional time with changes. Achieve this via versioning.

Btw, if you’re ever worried about a decision anywhere, hit me on whatsapp. My hourly consulting rate i’m pretty sure brian will be ok with. :beers:
