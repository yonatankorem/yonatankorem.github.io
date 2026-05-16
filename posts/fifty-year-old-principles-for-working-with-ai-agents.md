# Fifty-Year-Old Principles for Working With AI Agents
Since Copilot, Claude Code, Codex, and the rest became a primary tool we developers
use, we've learned a lot about improving results. Employing `CLAUDE.md` files that
load into context and provide core behavioral guidelines. Skills that give agents
step-by-step instructions on specific tasks, or hand off work to deterministic scripts
instead of relying on the probabilistic nature of LLMs. Subagents that tighten the
scope of instructions and give a fresh context to handle specific topics. Compaction
strategies and hand-off summaries that prevent long conversations from degrading in
quality.

These are all deliberate, active choices you make when configuring or interacting with
an agent.

Over the past year, working with Claude Code across codebases of different sizes and
ages, I noticed something quieter: when the codebase itself was structured in certain
ways, results improved without any agent instruction at all. More accurate edits, fewer
hallucinated method calls, better behavior from smaller models. I started calling these
"passive context management": practices that improve agent results without any
specific interface with the agent.

I'll walk you through seven of them. I'll tell you upfront: every single one is a
classic software engineering principle, most of them decades old. If you're an
experienced engineer, you'll recognize all of them before I name them. The interesting
part isn't the list. It's what it says about code quality that these principles map so
cleanly onto AI agent performance.

---

## 1. Files With a Highly Focused Topic Over "God Files"
Keep your files **focused** as much as possible. A file is "focused" when it contains *everything* it needs to function and *nothing else*. It is responsible for a single topic and is only accessed when that specific topic is needed (for understanding or modification). A React component file that contains only the HTML and UI logic for it has all it needs to work. It does not *need* to have the code that sends the form to the backend, or the color scheme to style it.

How does an agent decide which files to load into its context? If it has been tasked with modifying business logic regarding how the system handles new messages, it might:
- Look for files containing keywords like `incoming` or `message`
- Look for files containing message queue subscription code
- Filter files based on their path or name to abandon files that are probably irrelevant for this query

A class that handles incoming messages, business logic, DB access, and error handling might feel like it will improve results as the agent will find everything it needs in it, but it will also include a lot of things it does not need which we know has a negative impact of context rot. Anthropic's own guidance reflects this. They recommend capping CLAUDE.md at 200 lines specifically because long context degrades model attention and increases the risk of the model losing track of earlier content.  
If the business logic is spread around over a number of files, we get the worst of both worlds: it is more likely that the LLM misses a bit of related code, or needs to load multiple files into the context in order to be able to reason about it all.

A focused UI component only enters the context when the agent needs to understand or change the UI. Nothing else drags it in.

## 2. Program To Interfaces, Not Implementations
A function/class/module should not be aware of the concrete implementation, but use only the defined interface. When the agent goes to change something in how this function works, it will be limited to the interface alone, and hopefully not even load the implementation into context.  
The agent is less likely to invent new functionality, use a method not intended for this flow, or expose private methods because it is "convenient".  

Compound this with smaller, dedicated interfaces for even better results:  

**Bad**
```typescript
class DBAccessImpl {
    getUser(userId: string): User {
        const details = this._getUserDetails(userId);
        ...
    }

    _getUserDetails(userId: string): UserDetails {
        ...
    }

    getUserPurchases(userId: string): Purchase[] {
        ...
    }
}

class Logic {
    db: DBAccessImpl;

    areUserDetailsMatching(userId, details) {
        const user = this.db.getUser(userId); // An agent could try to use _getUserDetails
        if (...)
    }
}
```

**Better**
```typescript
interface DBAccess {
    getUser(userId: string): User;
    getUserPurchases(userId: string): Purchase[];
}

class DBAccessImpl implements DBAccess {
    getUser(userId: string): User {
        const details = this._getUserDetails(userId);
        ...
    }

    _getUserDetails(userId: string): UserDetails {
        ...
    }

    getUserPurchases(userId: string): Purchase[] {
        ...
    }
}

class Logic {
    db: DBAccess;

    areUserDetailsMatching(userId, details) {
        const user = this.db.getUser(userId); // An agent won't be able to use _getUserDetails, but could possibly use getUserPurchases
        if (...)
    }
}
```

**Best**
```typescript
interface UserAccess {
    getUser(userId: string): User;
}

interface PurchasesAccess {
    getUserPurchases(userId: string): Purchase[];
}

class DBAccessImpl implements UserAccess, PurchasesAccess {
    getUser(userId: string): User {
        const details = this._getUserDetails(userId);
        ...
    }

    _getUserDetails(userId: string): UserDetails {
        ...
    }

    getUserPurchases(userId: string): Purchase[] {
        ...
    }
}

class Logic {
    db: UserAccess;

    areUserDetailsMatching(userId, details) {
        const user = this.db.getUser(userId); // An agent is limited to the getUser only
        if (...)
    }
}
```

Combining this with tip 1 "keeping the interface and implementation in separate files" means the agent never loads the implementation into the context at all.

## 3. Deterministic Validity Over Probabilistic Reasoning
Some passive improvements are about reducing the need for reasoning instead of just context window management.

When a function takes a variable `get_item(idx)` it is easy to reason that `idx` is a number.  
In `createUser(firstName, lastName, birthDate)` it is safe to assume that all parameters are strings.  
What about `fetch(url, options)`? Yes, `url` is probably a string and even though `fetch` is an incredibly common function, it is not immediately obvious what `options` contains.  
But worse still is the uncommon function. The event handler `handleMessage(message, context)` is a real problem. `context` is a real mystery without reasoning what it means by inspecting the function implementation.

This version will require no reasoning whatsoever:
```typescript
function handleMessage(message: string, context: { source: string, timezone: string, isUrgent: boolean}) {
    ...
}
```
Even if the type is imported from a different file, the agent will still easily add the correct file into the context.

Statically typed variables and functions remove the need for reasoning by explicitly stating everything.  
Additionally, they also provide a safety net for the agent. A failing type system tells the agent that it did something wrong without needing to execute code or even any test.

## 4. Established Patterns Over Re-inventing The Wheel
This one helps by leveraging how LLMs work internally to improve results. LLMs are trained on massive corpora, so code that follows well-documented, widely-used patterns appears frequently in training data, making it far more reliably generated.

These snippets do the exact same thing:
```typescript
    /// Reinventing the wheel:
    class NotificationService {
        constructor() {
        this.emailCallbacks = [];
        this.smsCallbacks = [];
        this.pushCallbacks = [];
        }

        onEmail(cb) { this.emailCallbacks.push(cb); }
        onSms(cb) { this.smsCallbacks.push(cb); }
        onPush(cb) { this.pushCallbacks.push(cb); }

        send(type, data) {
            if (type === 'email') {
                this.emailCallbacks.forEach(cb => cb(data));
            } else if (type === 'sms') {
                this.smsCallbacks.forEach(cb => cb(data));
            } else if (type === 'push') {
                this.pushCallbacks.forEach(cb => cb(data));
            }
        }
    }

  /// Using established patterns and libraries:
  const EventEmitter = require('events');

  class NotificationService extends EventEmitter {}

  const service = new NotificationService();
  service.on('email', (data) => console.log('Email sent:', data));
  service.on('sms', (data) => console.log('SMS sent:', data));
  service.emit('email', { to: 'user@example.com' });
```

Established pattern code appears far more frequently in training data. This is part of the reason it is "well established". Hence, it is both easier to reason about what it does and it is easier to extend for the LLM, as it has more "examples" on how to use it.

## 5. Named Constants Over Magic Numbers
This again touches on the need to reason about pieces of code. Magic numbers (hard coded numeric values) are just more difficult to reason about, on a number of different "axes".

**WHAT is the meaning of a value?**
`publishMessage(msg, 3)`
Is 3 the number of attempts? Is it related to some backoff? Maybe it's an ID? Or even worse: is it an enum value index? The only way to know is to review the function implementation.

**WHY is this value chosen?**
`for (i = 0; i++; i < 50) { ... }`
Why are we iterating 50 times? why not 100 or 5? To know that, we have to review the code within the loop.

**WHERE is this value reused?**
`run_with_max_depth(..., 4)` and `run_with_min_depth(..., 4)`. Is the value the same for a reason? should we change both at the same time?

By providing a name for each, we give these numbers context that can be crucial.
`publishMessage(msg, MAX_RETRY)`  
`for (i = 0; i++; i < CHUNK_LIMIT) { ... }`  
`run_with_max_depth(..., AVOIDING_OVERFLOW_LIMIT)` and `run_with_min_depth(..., MINIMAL_DEPTH_FOR_HIGH_P_VALUE)`
This context removed the need for reasoning, which allows smaller scale models to generate better results.

One small caveat: There are magic number patterns that are so familiar that naming them is not critical at all: time spans. For the overwhelming majority of cases `1000` is one second. `60000` or `60*1000` is a minute, etc.

## 6. Comment Only When The Why Or How Are Not Obvious
Comments are important but they have the ability to negatively impact the LLM.  

**At best they take up context for no good reason**
```typescript
// Iterate on the entire array to search for an element that matches the criteria
for (i = 0; i++; i < arr.length) { if (isCriteriaMatch(arr[i])) return i; }
```
**At worst they become stale and outdated, forcing additional reasoning**
```typescript
// Check if user is admin to prevent unauthorized access
if (user.role === 'moderator') {
    allowDelete();
}
```
The comment says "admin" but the code checks for "moderator". Was the code changed and the comment forgotten? Or is the comment wrong about the intent? A reader can't trust either one now.

When used sparingly and when actually needed, they become a reliable source of important context, which can *improve* the LLM results.

**To explain the HOW (for a non trivial implementation)**
```typescript
// We iterate backwards through the array and swap with the current position.
// Forward iteration would miss newly-eligible items pushed to the end during filtering.
function filterAndCompact(items) {
    for (let i = items.length - 1; i >= 0; i--) {
        if (!isValid(items[i])) {
            items[i] = items[items.length - 1];
            items.pop();
        } 
    }
    return items;
}
```
The algorithm is subtle—without this comment, a reader would see backwards iteration and wonder why, potentially "fixing" it to forward iteration and breaking the logic.

**To explain the WHY (of non-obvious reasons)**
```typescript
// This endpoint does NOT use our standard pagination. We cap results at 500
// because the payment processor's webhook has a 30-second timeout and our
// largest customer has ~400 transactions/second. Going higher causes us to
// exceed their SLA and triggers rate-limiting on their side.
async function getTransactionHistory(limit = 500) {
    if (limit > 500) limit = 500;
    return fetchTransactions({ limit });
}
```
The constraint isn't technical laziness—it's a business/integration reality. Without this comment, the next person thinks it's arbitrary and changes it, breaking production.

## 7. Fewer Branching Points Require Less Reasoning
Code with a lot of branching points requires more reasoning and is more error prone. As the probabilistic LLM "chooses" where to make a change within the branching points, the chances of making mistakes increases. This can be mitigated with additional reasoning, which incurs higher costs.

**Lots of compounding branching points**
```typescript
function processOrder(order) {
    if (order.items && order.items.length > 0) {
      if (order.customer) {
        if (order.customer.isActive) {
          if (order.total > 0) {
            if (order.payment.isValid()) {
              if (!order.isProcessed) {
                chargeCustomer(order);
                updateInventory(order);
                sendConfirmation(order);
                return true;
              } else {
                logError("Order already processed");
                return false;
              }
            } else {
              logError("Invalid payment");
              return false;
            }
          } else {
            logError("Order total is zero");
            return false;
          }
        } else {
          logError("Customer inactive");
          return false;
        }
      } else {
        logError("No customer attached");
        return false;
      }
    } else {
      logError("No items in order");
      return false;
    }
  }
```

**Few compounding branching points**
```typescript
function processOrder(order) {
    if (!isValidOrder(order)) return false;
    if (!isValidCustomer(order.customer)) return false;
    if (!isValidPayment(order.payment)) return false;
    if (order.isProcessed) {
      logError("Order already processed");
      return false;
    }

    chargeCustomer(order);
    updateInventory(order);
    sendConfirmation(order);
    return true;
  }

  function isValidOrder(order) {
    if (!order.items?.length) {
      logError("No items in order");
      return false;
    }
    if (order.total <= 0) {
      logError("Order total is zero");
      return false;
    }
    return true;
  }

  function isValidCustomer(customer) {
    if (!customer) {
      logError("No customer attached");
      return false;
    }
    if (!customer.isActive) {
      logError("Customer inactive");
      return false;
    }
    return true;
  }

  function isValidPayment(payment) {
    if (!payment.isValid()) {
      logError("Invalid payment");
      return false;
    }
    return true;
  }
```
Ironically enough, Haiku generated the fewer branching points example just fine, but created the first with errors which Sonnet was able to resolve...

---

## The Payoff

If you haven't encountered these principles before, here is what each tip maps to:

| Tip | Classic principle |
|---|---|
| Files with a highly focused topic | Single Responsibility Principle |
| Program to interfaces, not implementations | Interface Segregation + Dependency Inversion |
| Deterministic validity over probabilistic reasoning | Static type safety |
| Established patterns over reinventing the wheel | Design patterns |
| Named constants over magic numbers | Code readability and naming |
| Comment only when the why or how are not obvious | Self-documenting code |
| Fewer branching points require less reasoning | Low cyclomatic complexity |

These have been documented and argued about since the 1970s and 1980s.

For those who already knew the list: the surprising part is not the list itself. It is that the list works at all.

My starting assumption was that these principles were optimized for human cognition. We get tired. We lose context. We cannot hold 600 lines in working memory. LLMs can. So I expected them to be immune to the same quality problems we have. They are not. I have observed that a model working through a god-file makes mistakes that a model working through focused files does not, even when the full content fits within the context window. Why the optima align so closely is not something I can prove definitively. But the alignment is real, and it is the more interesting observation.

Code that requires more reasoning to understand will demand it, from humans and machines equally. We have had the right instincts about this for fifty years. It just took a machine reader to finally make the cost undeniable.