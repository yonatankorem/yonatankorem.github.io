# The Dangerous Illusion of the 10x AI Engineer  
## AI didn’t just speed you up—it made your mistakes scale

---
  
For years, the industry has tried to define what makes a “10x engineer.” The answers varied, but they rarely had much to do with raw output. The best engineers weren’t the ones who wrote the most code—they were the ones who made the best decisions.

AI changes the equation, but not in the way people think.

Yes, it dramatically increases output. It drives the cost of writing code toward zero. Entire categories of friction: syntax, boilerplate, even unfamiliar frameworks, start to disappear.

And that’s exactly where the illusion begins.

Because when the cost of producing code collapses, the cost of *bad decisions* becomes the dominant risk. AI doesn’t just help you move faster, it helps you move faster in the wrong direction at a much larger scale while making you feel confident.

This is the dangerous illusion of the 10x AI engineer.

---

## AI *Does* Change Something Real

To deny the actual impact AI has, and will continue to have, on software development would be naive at best, and dishonest at worst. These days, when I need to compare the APIs of multiple competing services, I don't bother searching their docs. I send an agent to do it and just verify the results. Anyone can do this. If you have an idea that you want to implement and you are not sure how, agents got your back.  
We used to chase our project manager around, trying to get latency limits. "What is the longest time it is valid for the app to load?" It was difficult because it is an amorphous "thing" to define without actually seeing it in action. Now, we built a tiny app that the project manager can use, play around with sliders that determine various load times and delays, and hit "play" to get a visualization of how the website will load. This sort of one off development would never happen without AI. The ROI for it would not be good enough. Now? We don't bother even asking if we should do it.

This new ability allows you to replace old development patterns with better ones because the investment on them is so small, that the ROI is now much greater.

We now have a new tool in our toolbox, and it's pretty amazing. But people might be signing a dark bargain by using it.

## It was never about output

I worked with Tim. Tim was one of the best engineers I've ever met.  
Tim did not write code or documents faster than the rest. He didn't complete more tickets than the rest.

His scope of knowledge was immense. He was able to tackle any problem, implement any feature, all while maintaining high quality. He constantly found the bottlenecks that slowed the team down and found ways to fix them. He pushed back on changes that the product manager asked for that created inconsistencies, and pushed back on the development team when they wanted to implement monitoring tools instead of using an off the shelf tool. 

Tim was a 10x engineer.

Tools like Codex, Cursor, or Claude Code do not make you into a 10x engineer. They just amplify YOU ten times.
When you ask them to change the color of the buttons to Magenta, they will never tell you that Magenta is a bad choice for your color scheme. Or that to do so, it'll need to modify 37 places because no one told it to create a proper theme. They will happily do it, one way or another.

These tools do not give you experience, knowledge, or judgment. They just give you speed.
And speed, it turns out, is corrosive to judgment in ways I didn't expect.

I noticed it in myself. AI's output is voluminous, and reviewing voluminous output is cognitively expensive. So I started rationalizing. The first few paragraphs look good, so the rest is probably fine. The overall structure seems right, so I won't go line by line. I told myself this was pragmatism. It was lowered standards, dressed up as efficiency.

## Uncle Bob is still right

> The only way to go FAST, is to go WELL  
> *Uncle Bob (Robert Martin)*
  
This quote is basically ancient by software development standards. It has never been more relevant. It seems like everyone except R&D want everything to go as fast as possible. Cut whatever corners you need because this feature needs to go live ASAP.  
Good R&D teams try to create the balancing act of slowing down in order to go fast.

AI says: "You want fast? I'll show you fast!".

It creates thousands of lines of code handling edge cases that will never happen. Files and files of documentation that will recursively eat up tokens and context in your next conversation. Or it'll just remove your production environment. [Just ask Amazon AWS](https://www.reddit.com/r/cybersecurity/comments/1rbnwlf/amazon_kiro_deleted_a_production_environment_and/).

That is not quality. That is not "well". You only feel fast. This is the illusion of the 10x AI engineer.

## Didn't you want a short feedback loop?

The fast feedback loop is real — but only for visible problems.
The bug that crashes the app. The feature users ignore. The query that times out under load. AI helps you find these faster, and fix them faster.

The problem is that AI's failure mode isn't the visible bug. It's the decision that looked fine, passed review because there was too much code to review carefully, and quietly accumulated interest for six months before anyone noticed. It's the architectural assumption baked into generated code that nobody wrote intentionally and nobody owns. It's the moment, eighteen months from now, when your team sits in a room and realizes that adding any new feature requires untangling something fundamental — and nobody remembers why it was built that way, because nobody built it. It just appeared.

That's not a fast feedback loop problem. That's a compounding debt problem.
And AI doesn't just create debt at a normal rate. It creates it at the rate it creates everything else — faster than any team has ever been able to sustain. You get ten times the output. You also get ten times the liability. The velocity you feel today is partly borrowed from the team that has to work in this codebase a year from now.

Uncle Bob's point was never about craftsmanship for its own sake. It was about this: the shortcuts you take today don't disappear. They become the floor of everything you build next.

AI didn't repeal that. It just made the borrowing feel free.