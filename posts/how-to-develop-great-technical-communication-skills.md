# How to develop great technical communication skills 
As you progress in your engineering career, you discover that today's hardest problems rarely require great technical abilities - they need great communication skills. You work within a team of peers, alongside product managers, QA, marketing, and so on. With the advancements in AI coding tools, you will find that the ability to clearly convey your intent to the tool is critical.  
Most people have attended a meeting that was derailed because a good technical solution was misunderstood, or one that failed after alignment slipped even when the facts were clear.  
The further you advance, you'll find that your impact depends less on your technical know-how and more on how well you articulate context, trade-offs, and help others maintain the mental model of a complex problem.

Great communication skills aren't just about making pretty slides. They are about making your thinking accessible so people can act on them.

## Clarity vs. Compliance
An architect presented his design for a system. By the end, we were left silent. If there were only a few unclear points, people might ask questions. His presentation muddied ideas, confused concepts, and contradicted a previously stated assumption. People were silent because they didn't even know where to start and somehow we just moved forward with an unclear path.

I've written asynchronous decision documents that nurtured great questions and challenges, and ended with a decision that everyone understood completely and actively supported, avoiding agreement by fatigue.

Good, well-structured, and clear communication is the difference maker. This is effective communication.

## The Communication Framework
After years of experience, I've come to understand that effective communication is not an innate talent; it is a process that requires planning, organization, and using the right tools. As you do this more and more, you'll find that this process becomes second nature.

This process starts with two parallel tracks: Understanding the audience and the goal you are aiming for. A good presentation or document should merge the audience's needs with the author's goal. It is not always easy to do this, but it is always worth it. Don't be afraid to ask others like your peers or mentors for help to figure this out. Once the two are clear, you will employ the right tools to create a path that easily leads the audience to the goal.

### Know Your Audience
Understanding who your audience is, what they know, and what they need from your document or presentation is the first step in creating effective communication.
Ask yourself:
- How well do they know the topic? Are they new to the topic or are they experts.
- Do they need in depth understanding or just a high level overview.
- Do they need the trade-offs and the context or are they just interested in the final decision.

With a better understanding of your audience, you will know how to structure the information and what pitfalls to avoid.

### Planting the Flag
Before you open a document or create a presentation, you need to "plant your flag": define what you are trying to achieve. Without a clear destination, your communication drifts - you'll bury critical information under unnecessary details, or rush past an essential point because you are not sure what matters.  
When you know your goal, you can cut a focused path through the complexity. It gives your content structure and determines what you say, when you say it, what you emphasize, and what you skip.  

Different goals demand different communication strategies, and recognizing which one you are pursuing is the next step in making your message land.

## The Different Audiences

### Developers
Developers are pragmatic and solution-oriented. They want accuracy and completeness, and will be skeptical of claims that are not backed up by facts. This doesn't necessarily mean that you have to have all the answers. It's ok to say "I don't know" or "I'm not sure", but make sure to later check up on the question and provide an answer.

Developers often don't want to waste time on fluff. They want to know what to do and how to do it. Code examples and low level diagrams usually work better than text.

Don't assume that they know all the lingo, understand protocols, or are familiar with tools and frameworks. It's better to provide some elaboration on it that people can skip if they want, than have none and assume they do.  
Developers often feel measured against other developers (because they usually are). This causes them to take their lack of knowledge as something THEY must fix, instead of asking for a clarification. That's not a bad thing, until their understanding does not align with others.

Best practices for developers:  
- Use accurate language instead of buzzwords.
- Include a glossary or references to documentation for terms and concepts that are not common knowledge.
- Use code examples and low level diagrams to explain the problem and the solution.
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
- Don't be afraid to say "I'm not sure. Let me check and get back to you." when challenged. It shows that you are open to learning, you don't try to fast talk your way out, and actually take the time to understand the issue.

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
What I found that helps most to avoid the sensation of product and engineering being in a tug-of-war is to provide the product manager with clear "knobs" and the knowledge of what will happen if you turn them, then allowing them to turn the knobs to pick the balanced outcome.  
When done right, this builds trust that engineering are providing solutions and not just restrictions, and on the other hand, builds confidence that the product manager is taking the technical impact into account.

Best practices for product managers:
- Lead with impact on the product, not the technical details. "Users will see a 2-3 second delay here" matters more than "The database query for this query is O(n^2)".
- Create paths, not roadblocks. If you cannot do X, first provide what CAN be done along with their trade-offs. Keep the reasons X is not possible to the end.
- Quantify trade-offs where possible: time, cost, user impact, quality, and what limitations it puts on the future.

You know the audience, now let's understand what you are trying to say. 

## The Different Goals

### Explaining a Topic
When your goal is to explain a concept or system architecture, you are attempting to build the reader's mental model. This usually happens when onboarding new team members, introducing a new component, or documenting how a complex flow operates under the hood. The focus here should be heavily oriented toward the "why" and "how it works" rather than a step-by-step tutorial.

Constructing a mental model can be difficult, and different people have different abilities to build and maintain one. To achieve your goal, you need to make it as easy as possible for the reader. You build slowly, layer by layer and constantly make sure your assumptions are clear and assert that the audience is following you. 

Best practices for explaining a topic:
- Start with the big picture and gradually zoom in.
- Apply tools that help build mental models like diagrams, analogies, and examples.
- Use a clear, concise, and most importantly, consistent language.
- Avoid jargon and technical terms where possible.
- Use a consistent formatting.

You need to avoid any unnecessary noise. Reduce the cognitive load as much as possible. If you change up your structure, formatting, or terminology mid-document, you force the reader to constantly re-orient themselves. This cognitive load is exhausting and makes your message harder to absorb.

### Creating Alignment
Alignment is making sure that everyone is on the same page. It is both crucial as teams and organizations grow larger, and harder to achieve at the same time. It can also be difficult because the alignment is needed to be maintained even when you are not in the room. You cannot become the Alignment Cop. You cannot be in every meeting and discussion. The only way to achieve alignment is to create documentation that can be used to maintain alignment.

The two classic examples are:
1. Conflict of terminology
2. Terminology that appears self-evident but is not

Conflict of terminology occurs often when different domains have different definitions for the same concept. For example, a publishing house logistical department are very interested in a book "Weight" in actual physical terms. The editorial department might consider the "Weight" of the book as a measure of the target audience. "This is a LIGHT read for a lazy sunday afternoon". When representatives of these teams interact, they might be talking past each other without realizing it.

Self-evident terminology can be even more dangerous. This is when a term seems so obvious that no one thinks to define it. For example, when talking about "Engagement" in a software product, it seems obvious what it means. But how do you measure it? Do you care about length of time or frequency of use? Is it the leads that convert or existing users that browse your system?

Both cases can be solved by the same solution: You must be explicit. Sit down with the stakeholders and define the terms. Write them down and be diligent about it. Ambiguity is the enemy of alignment.

Best practices for creating alignment:
- Use a glossary at the top. This helps avoid the audience going through half of your content before realizing they misunderstood something.
- Avoid acronyms as much as possible. If you must, the first time the acronym is mentioned, it must be followed directly with the original form. The exception to this rule is when the acronym is ubiqoutus (e.g. AWS).
- After giving the explicit definition, provide two or three examples using realistic use cases. If you have examples that actually happened, all the better.
- The document must be widely available and stored in a familiar place. Do not store it on your personal cloud storage.

You will know you've succeeded when people start to use these definitions and refer to this document on their own.

### Providing Instructions
Instructions are usually needed when on-boarding someone to a project or system. You need them to understand how the system works, and you want to help them get started. Whether a git repository, a no-code platform running ad campaigns, or a CRM used by a sales representative - the rules stay the same:  
- The guide must be clear - a user should not have questions regarding the instructions themselves.
- The guide must be accurate - a user should not encounter problems or difficulties running through it.

These two sound simple enough, but in practice can be one of the more difficult documents to setup. The problem stems from the fact that when you write it, you are already on-boarded. You understand how the system works. You understand how the UI behaves. You already resolved all the issues of setting up. To create a quality instructional guide, you need to put yourself in the shoes of someone who has none of these.

Think about the last Lego set or IKEA furniture you've built. These companies are known for their instruction manuals. Both share a lot of common concepts. They familiarize you with the elements (either in advance or for each step). Each step is detailed enough to avoid overwhelming the reader, but includes all the information needed. If the step includes any ambiguity, it is resolved immediately (e.g. parts look alike, but have a specific directionality).  
Now think of any product with a manual that was poorly translated into English, where the buttons are mislabeled, and the steps might have been taken from a different product.

Unlike Lego or IKEA, we get the additional difficulty or maintanence. The Lego set will always remain with the same set of bricks and the same exact build process. But your CI/CD pipeline changes or the CRM gets a UI update. You will need to maintain your guide and update it as time goes on. Doing this on your own will not work. Eventually, you'll move on to another project or someone else will change something that makes your guide incorrect. Hence, the ownership of the guide belongs to everyone working on the same project.

Best practices for providing instructions:
- When UI is involved, capture screenshots. If it is terminal commands, capture the expected output.
- Put the document at the most obvious place. A new user will not know where to look - it should be right in front of them.
- Avoid excessive explanations. Whoever is using this guide needs to get from point A to point B, they don't need to understand WHY.
- Try to follow the guide on a new machine or from a fresh account. That's one of the better ways to double check yourself.

### Driving a Decision
As a technical leader, you will often be asked to assist a team/group/organization/company in making a decision. Doing so has some overlap with "Explaining a Topic" in that you need to make sure the audience understands the problem and the proposed solutions. But, this is just the first step, and you can usually be more relaxed here because the people involved in the decision making tend to be familiar with the context and the problem space to some degree.

Imaging a group of you and your audience are in the woods. There is no clear path forward. There are a lot of possible ways to try and move forward, but it's difficult to see where they will lead you. How would you decide where to go? One practice you can use is that you advance on your own, just a little in every direction. You study the possible path, see where it leads to, and learn the differences. One way has a lot of thorns, but seems to lead to a wide clearing. Another is much easier to traverse, but you can't see far ahead. You gather all of this information and present it to the group.  
"We are trying to get to the beach" - You start, because without a shared agreement on the goal there is no possible way to decide on a path.  
"We are all experienced hikers with little equipment" - You explicitly state to avoid people assuming things, but also invite questions. Maybe someone forgot that they have a lot of food in their pack and even you weren't aware.  
"What we will do once we get there is not relevant right now" - You mention to make sure the discussion stays on track.  
"We can go through here, which will get us to the beach faster but it has a difficult climb. We can go through there, which will take us longer but is downhill" - You provide the options with the pros and cons of each one to enable a discussion. Do you have enough food for the longer path? Is everyone able to make the climb?

This is the same process for technical decisions as well. You are presenting, because you are the one that studied the different paths. You start by aligning everyone on the goal, the current situation, and important points to consider. To prevent the discussion from steering off, you also note topics that are related, but are not relevant. Finally, you present the available options without judgement to enable an open discussion.

Best practices for driving a decision:
- You want to drive towards a discussion. Leave room for comments and questions.
- An obvious path with non-obvious obstacles MUST be described even if, and especially if it is clearly a bad option. If you won't mention it, someone else will and you'll waste time talking about it.
- If you have a preferred option, present it first. You want the group to compare other options to it so that it will be clear why it is better.
- Do NOT skip options that you object to. When someone else suggests it, you will seem less professional for it.

## Putting It All Together
A new D2C (Direct to Consumer) business initiative meant creating a funnel that will allow us to gain high resolution data on how the different funnel sections are doing as well as low resolution data to enable sales reps to best communicate with the customers.  
It took very little time before the business leadership, the marketing team, the data analysts, and operations team developed their own understanding of what each funnel stage means, how a customer travels down it, and how to differentiate between customers in different funnel stages. This quickly led to confusion and arguments.

To help resolve this I understood that the document I was going to create will be used by executives and developers and the goal is to create alignment between them. I knew that the executives will need concrete answers and not a debate. The developers will need technical details in order to build what the project needs. I started to interview each team to detect the gaps and eventually had a meeting with a representative from each team. In that meeting, we created the following document:

**Glossary**
| Terminology | Meaning |
| Contact Information | At least first name and email address |
| Successful Meeting | The sales rep completed the call and marked it as "successful" in the CRM |
...

**The Funnel**
*Stage A*: A user is considered to be in this stage when
- They provided us with contact information.

*Stage B*: A user is considered to be in this stage when
- They match the criteria for *stage A*.
- They scheduled a meeting with our sales team.

*Stage C*: A user is considered to be in this stage when
- They match the criteria for *stage B*.
- They have at least one successful meeting recorded.

*Stage D*: ....

Think of a Venn Diagram made of circles that get smaller and smaller, completely contained in the previous circle. Each stage had to tighten the constraints in order to uniquely define each one. The language used in the document was selected to avoid conflicts and ambiguity. Basic concepts got a data level definition to align the operations team to know what is expected of them and the data analysts that were building dashboards based on the data.

The glossary helps connect the business language to technical details. The funnel definition does the reverse: uses boolean criteria to describe the business language.

Even though the document was short, and seemed to be obvious, it was critically needed to both START to create alignment and maintain it when questions were surfaced.

---

Now all that remains is to start communicating more and more. Practice makes perfect, and hopefully these tools will assist you to make it less scary and more natural.