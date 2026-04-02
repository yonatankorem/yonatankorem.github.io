# How to develop great technical communication skills 
As you progress in your engineering career, you discover that today's hardest problems rarely require great technical abilities - they need great communication skills. You work within a team of peers, alongside product managers, QA, marketing, and so on. This requires communication skills. With the advancements in AI coding tools, you will find that the ability to clearly convey your intent to the tool is critical.  
Most likely you have attended a meeting that was derailed because a good technical solution was misunderstood, or one that failed after alignment slipped even when the facts were clear.  
The further you advance, you'll find that your impact depends less on your technical know-how and more on how well you articulate context, trade-offs, and help others maintain the mental model of a complex problem.

Great communication skills aren't just about making pretty slides. They are about making your thinking accessible so people can act on them.

## Clarity vs. Compliance
An architect presented his design for a system. By the end, we were left silent. If there were only a few unclear points, people might ask questions. His presentation muddied ideas, confused concepts, and contradicted a previously stated assumption. People were silent because they didn't even know where to start and somehow we just moved forward with an unclear path.

I've written asynchronous decision documents that nurtured great questions and challenges, and ended with a decision that everyone understood completely and actively supported, avoiding agreement by fatigue.

Good, well-structured, and clear communication is the difference maker. This is effective communication.

## Creating Effective Communication
After years of experience, I've come to understand that effective communication is not an innate talent, it is a process that requires planning, organization, and using the right tools. As you do this more and more, you'll find that this process becomes second nature and you'll need less time and effort to do it - eventually allowing you to even do it on the fly.

This process starts with two parallel tracks: Understanding the audience and the goal you are aiming for. A good presentation or document should merge the audience's needs with the author's goal. It is not always easy to do this, but it is always worth it. Don't be afraid to ask others like your peers or mentors for help to figure this out. Once the two are clear, you will employ the right tools to create a path that easily leads the audience to the goal.

## Know Your Audience
Understanding who your audience is, what they know, and what they need from your document or presentation is the first step in creating effective communication.
Ask yourself:
- How well do they know the topic? Are they new to the topic or are they experts?
- Do they need in depth understanding or just a high level overview?
- Do they need the trade-offs and the context or are they just interested in the final decision?

### Developers
Developers are pragmatic and solution-oriented. They want accuracy and completeness, and will be skeptical of claims that are not backed up by facts. This doesn't necessarily mean that you have to have all the answers. It's ok to say "I don't know" or "I'm not sure", but make sure to later check up on the question and provide an answer.

Developers often don't want to waste time on fluff. They want to know what to do and how to do it. Code examples and low level diagrams usually work better than text.

Don't assume that they know all the lingo, understand protocols, or are familiar with tools and frameworks. It's better to provide some elaboration on it that people can skip if they want, than have none and assume they do.  
Developers often feel measured against other developers (because they usually are). This causes them to take their lack of knowledge as something THEY must fix, instead of asking for a clarification. That's not a bad thing, until their understanding does not align with others.

Best practices for developers:  
- Use accurate language instead of buzzwords
- Include a glossary or references to documentation for terms and concepts that are not common knowledge
- Use code examples and low level diagrams to explain the problem and the solution
- Avoid hiding the trade-offs and negatives of a solution, even if the decision was already made. It will make you look bad when someone questions it.

### Architects
Architects are the people that look at the big picture stuff. They bridge the different domains of understanding the business needs, the product as a whole, the restrictions of the tech stack, and so on. As such, they care about consistency, system boundaries, and decisions that are hard to reverse.  
Unlike developers, they aren't usually looking for "the right answer" - they are looking for **consistency** even if it means a suboptimal solution that "fits" better.

Architects can be detached from the codebase itself as they work on higher-level decisions. Combined with their years of experience, this means they often evaluate systems through pattern matching rather than parsing out the code.

Similarly to developers, do not assume anything about their knowledge, especially when it comes to specifics: the latest framework, the trending AI models, and even the current codebase. But unlike developers, they might be ok with skimming through the details as long as they understand the big picture.  
The motivation of an architect is often impact. The good ones care about actual impact. The not so good care about maintaining their **aura of impact**.

Best practices for architects:
- Use standard terminology and patterns. Avoid attempting to invent your own.
- Explain how this suggestion fits into the big picture.
- Use diagrams to explain the system and suggested changes without delving into code.
- Don't be afraid to say "I'm not sure. Let me check and get back to you." when challenged. It shows that you are open to learning, you don't try to bullshit your way out, and actually take the time to understand the issue.

### Management
Managers are the ones that make the decisions that impact the team/group/organization/company. Unlike architects, their decision-making process is not always based on the best technical solution, but on the best business solution.

They juggle constraints that are often invisible to others: budgets, headcount changes, long-term roadmaps, cross-team politics, and confidential HR matters. A manager might be the only person in the room who knows that Jim is about to leave and the team's capacity is about to drop—but they can't say it out loud.

Managers spend their day in meetings, usually overlapping ones. They switch contexts constantly and make decisions under incomplete information. Because of this, they value brevity over exhaustive detail, but they require clarity.  
You cannot properly recommend a specific 3rd party service without knowing how much it will cost, how much it will save (either in terms of time or money), but they will trust your judgement that the service fits the need.  
This trust is important, but do not expect blind trust. They might challenge or question your process - making sure you have considered all the angles.

Best practices for managers:
- You have to know the questions you are expected to answer. If the information they need is not included in your presentation, you might have wasted their time and be seen as ill prepared.
- Start with some context to make sure the manager is on the same page. They might even remember the context from the last meeting.
- Lead with the answers to the questions the manager is looking for. 
- Provide a concise summary of the trade-offs and the context. Use quantifiable metrics where possible.

### Product
Product managers optimize for user experience, for business value, for product-market fit. They think in terms of user flows, features, and business metrics. Technical details matter to them only when they become constraints or trade-offs that affect what they can ship.  
From their perspective, engineering is a black box that can turn requirements into features. They rarely need to know the details of the implementation. They need to know what it can and cannot do, and what are the trade-offs of a solution.

When communicating technical information to product managers, it usually falls into one of two categories:  
- Boundaries: "We can't do X because the system cannot support it without changing this infrastructure."
- Trade-offs: "We can do X, but it won't scale past 10 users" or "it will delay Y by two sprints."

Product managers are often under pressure to deliver quickly and may push back on your opinions about the best technical solution. It is important to remember that this is not adversarial. Product managers and engineers are on the same team, and the goal is to deliver the best possible product.  
What I found that helps most to avoid the sensation of product and engineering being in a tug-of-war is to provide the product manager with clear "knobs" and the knowledge of what will happen if turn them, then allowing them to turn the knobs to pick the balanced outcome.  
When done right, this builds trust that engineering are providing solutions and not just restrictions, and on the other hand, builds confidence that the product manager is taking the technical impact into account.

Best practices for product managers:
- Lead with impact on the product, not the technical details. "Users will see a 2-3 second delay here" matters more than "The database query for this query is O(n^2)".
- Create paths, not roadblocks. If you cannot do X, first provide what CAN be done along with their trade-offs. Keep the reasons X is not possible to the end.
- Quantify trade-offs where possible: time, cost, user impact, quality, and what limitations it puts on the future.

## Planting the Flag
Before you open a document or create a presentation, you need to "plant your flag": define what you are trying to achieve. Without a clear destination, your communication drifts - you'll bury critical information under unnecessary details, or rush past an essential point because you are not sure what matters.  
When you know your goal, you can cut a focused path through the complexity. It gives your content structure and determines what you say, when you say it, what you emphasize, and what you skip.  
Different goals demand different communication strategies, and recognizing which one you are pursuing is the next step in making your message land.

### Explaining a Topic
When your goal is to explain a concept or system architecture, you are attempting to build the reader's mental model. This usually happens when onboarding new team members, introducing a new component, or documenting how a complex flow operates under the hood. The focus here should be heavily oriented toward the "why" and "how it works" rather than a step-by-step tutorial.

Constructing a mental model can be difficult, and different people have different abilities to build and maintain one. To achieve your goal, you need to make it as easy as possible for the reader. You build slowly, layer by layer and constantly make sure your assumptions are clear and assert that the audiance is following you. 

Best practices for explaining a topic:
- Start with the big picture and gradually zoom in.
- Apply tools that help build mental models like diagrams, analogies, and examples
- Use a clear, concise, and most importantly, consistent language.
- Avoid jargon and technical terms where possible.
- Use a consistent formatting.

You need to avoid any unnecessary noise. Reduce the cognitive load as much as possible. If you change up your structure, formatting, or terminology mid-document, you force the reader to constantly re-orient themselves. This cognitive load is exhausting and makes your message harder to absorb.

### Creating Alignment
Alignment is making sure that everyone is on the same page. It is both crucial as teams and organizations grow larger, and harder to achieve at the same time. It can also be difficult because the alignment is needed to be maintained even when you are not in the room. You cannot become the Alignment Cop. You cannot be in every meeting and discussion. The only way to achieve alignment is to create documentation that can be used to maintain alignment.

The two classic examples are:
1. Conflict of terminology
2. Terminology that appears self-evident but is not

Conflict of terminology occurs often when different domains have different definitions for the same concept. For example, a publishing house logistical department are very interested in a book "Weight" in actual physical terms. The editorial department might consider the "Weight" of the book as a measure of the target audiance. "This is a LIGHT read for a lazy sunday afternoon". When representatives of these teams interact, they might be talking past each other without realizing it.

Self-evident terminology can be even more dangerous. This is when a term seems so obvious that no one thinks to define it. For example, when talking about "Engagement" in a software product, it seems obvious what it means. But how do you measure it? Do you care about length of time or frequency of use? Is it the leads that convert or existing users that browse your system?

Both cases can be solved by the same solution: You must be explicit. Sit down with the stakeholders and define the terms. Write them down and be diligent about it. Ambiguity is the enemy of alignment.

Best practices for creating alignment:
- 

### Providing Instructions

### Source of Truth

### Driving a Decision
As a technical leader, you will often be asked assist a team/group/organization/company in making a decision. Doing so has some overlap with "Explaining a Topic" in that you need to make sure the audience understands the problem and the proposed solutions. But, this is just the first step, and you can usually be more relaxed here because the people involved in the decision making are usually familiar with the context and the problem space to some degree.