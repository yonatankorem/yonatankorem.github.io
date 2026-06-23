# Inheritance Is The Easy Path Into The Wrong Place
## I've seen problems bent to fit a tree. The tree was rarely the problem's actual shape.

---

Inheritance can feel like the most powerful tool. The "is-a" relationship makes it all feel principled and deliberate. A `Manager` is an `Employee`. A `Circle` is a `Shape`. Once you can say "is-a" out loud, extending a base class feels less like a choice and more like the correct answer.

When applied properly, inheritance is exactly the right move. The problem starts when "is-a" becomes a "one-size-fits-all" expression. An API endpoint "is-an" event handler. A `Stack` "is-a" `Vector`. It sounds right, but the fit is in name alone. When you start defaulting to inheritance, it feels like the right call, but you start to treat two elements that share a behavior or a dependency as an obvious fit.

## The example everyone learns from

Almost every OOP course teaches a version of this:

```typescript
class Employee {
  pay() { ... }
  generateHoursReport() { ... }
  assignVehicle() { ... }
}

class Manager extends Employee {
  assignEmployee() { ... }
}

class CEO extends Manager { }
```

Read it as taxonomy and it is airtight. A CEO is a manager. A manager is an employee. A CEO is an employee. Everyone shares the base behavior, each level adds a bit more.

The structure is technically correct, but is it the only one?

## The first crack: what is this class actually responsible for?

The Single Responsibility Principle gets quoted as "a class should do one thing." The version Robert Martin later settled on is sharper: a module should have only one reason to change.

So look at `Employee` again and ask who can force a change to it:

1. **Payroll** changes how salary is calculated. `pay` changes.
2. **Fleet** decides only managers get a vehicle this year. `assignVehicle` changes.
3. **HR** swaps reporting tools and wants a new template. `generateHoursReport` changes.

That is three departments holding three independent reasons to edit the same class. As a single class, this problem has nothing to do with inheritance. But add it into the mix and you hide the problem in the inheritance tree as these dependencies become less visible.

## What that actually costs you

Outside courses and examples, two things go wrong in practice. The first you can see in the class list. The second is easily missed, which enables the cost to slowly grow larger and larger.

**The tree starts modeling your class names instead of your domain.**

To continue with the `Employee` example, add a `Contractor`. They file an hours report, but you do not pay them through this system, you pay their company. So `Contractor extends Employee` drags in a `pay` method that should not exist. Flip it the other way and an `Employee` is not a `Contractor` either. So you invent a base class that represents nothing real, purely to have something to extend from.

Do that a few more times and you know exactly where it ends. A year later the hierarchy contains `RootEmployee`, `ContractorWithVehicle`, and a `BaseManager` that is a manager that manages no one. None of those are concepts in your business. They are scars left by forcing every relationship through "is-a".

That rot is ugly, but it is at least visible. You can read the class list and tell something went wrong. The second failure shows you nothing.

**A subclass inherits a side effect it cannot see and never asked for.**

Remember `generateHoursReport`, one of the three jobs we stapled to `Employee`. The root was written long ago, and it does two things: it builds the report, and as a final step it persists it. Back then every report was a CSV, so persistence writes a `.csv` file:

```typescript
class Employee {
  generateHoursReport(data: Row[]): string {
    const content = this.buildReport(data);
    this.persistReport(content);              // defined once, inherited by every report below
    return content;
  }

  protected buildReport(data: Row[]): string {
    return toCsv(data);
  }

  protected persistReport(content: string): void {
    storage.write(`reports/${this.id}.csv`, content);
  }
}
```

By now the tree underneath it is large: dozens of slightly different employees, four and five levels deep, each adding a small feature or changing a behavior. Then, a new requirement lands: executives want their hours report as HTML. You find the leaf you need to change and do the obvious, correct thing. You override the part that builds the content:

```typescript
class ExecutiveLevelManager extends PeopleManager {
  protected buildReport(data: Row[]): string {
    return toHtml(data);
  }
}
```

You were not careless. You overrode exactly the method meant for extension, and your unit test confirms it returns valid HTML. Every other class in the tree still passes its own tests, because none of them changed.

The system is broken anyway. `generateHoursReport` builds your HTML, then runs the inherited `persistReport`, which saves it to a file named `report.csv`. Downstream, the pipeline that ingests `reports/*.csv` parses your HTML as comma-separated values. Nothing fails in your code. It fails somewhere else, later, in a system you have never opened, with an error that points nowhere near the class you wrote.

No test caught it because there was nothing you knew to test. You did not know the side effect existed. It lived in the root, four classes above you, in a method you had no reason to open.

You could teach `persistReport` to pick the extension from the format, of course. But `persistReport` is shared by every class in the tree. First you have to suspect that a method you never touched is the culprit, then change it without breaking the dozens of classes that depend on it.

None of this is carelessness on anyone's part. It is the design making the cheap path the wrong one.

## The depth was never a discipline problem

There is a fair objection to all of this: the real mistake was letting the tree grow that deep, and a disciplined team would have capped it long ago. The instinct is right. But in real life situations, by the time you actually noticed, it is too far gone.

Nobody sat down and designed a five-level hierarchy. Each `extends` was added the day it was needed, and on that day it was the smallest, most reasonable change on the table. A new kind of employee, a new kind of report, one more subclass. No single one of those decisions ever looks like the mistake, which is exactly why the tree keeps deepening and why no review ever stops it.

And the growth is not linear, because `Employee` has more than one reason to change. Payment varies, reporting varies, vehicle policy varies. Put all of that in one tree and the levels multiply. To cover four payment types, four report types, and four vehicle policies, a single hierarchy has to spell out the combinations, and you are heading toward as many as sixty-four leaf classes, stacked one axis on top of the next. That is where a name like `ContractorWithVehicle` comes from. The combination had nowhere to live except the class name.

Finding and fixing it early only ever offers two roads. Refactor to a different shape, which is the argument of this whole post. Or keep going and add the missing combinations as still more classes, which makes the multiplication worse. There was never a disciplined version of the deep tree. The depth is simply what it costs to model several independent concerns with one inheritance hierarchy, and the way out is to stop multiplying and start adding.

## Compose the behaviors instead

Here is the same system built around what each part *does*, not what the employee *is*:

```typescript
interface PaymentStrategy {
  pay(): void;
}

interface HoursReporter {
  generateHoursReport(): Report;
}

interface VehiclePolicy {
  assignVehicle(): void;
}

class Employee {
  constructor(
    private payment: PaymentStrategy,
    private reporter: HoursReporter,
    private vehicle: VehiclePolicy,
  ) {}

  pay()                 { this.payment.pay(); }
  generateHoursReport() { return this.reporter.generateHoursReport(); }
  assignVehicle()       { this.vehicle.assignVehicle(); }
}

const employee = new Employee(
  new SalariedPayment(),
  new HourlyReporter(),
  new LeasedVehicle(),
);

const ceo = new Employee(
  new ExecutivePayment(),
  new HourlyReporter(),
  new CompensatedVehicle(),
);
```

One `Employee` class. The differences between an employee, a manager, and a CEO are no longer levels in a tree; they are the behaviors you hand the constructor. The `Contractor` that warped the tree earlier is now just a different combination: a real `HoursReporter`, and a payment behavior that does nothing because you do not pay them here. No fake base class required.

And the silent coupling is gone by construction. The reporter is its own object that you build and hand to the employee. An HTML reporter carries its own persistence, chosen on purpose, sitting right there in the code that constructs it. There is no parent quietly contributing a `.csv` write from a few lines up, because there is no parent. To couple two behaviors now, you have to wire them together yourself, which is the point.

## Inheritance still has a job

This is not a story about deleting `extends` from the language. Notice where inheritance survived in that refactor: *inside* a single behavior. `SalariedPayment`, `ExecutivePayment`, and `HourlyPayment` can absolutely share a base, because there the "is-a" is real and total. Every one of them genuinely is a `PaymentStrategy`, and any of them can stand in for the others wherever a `PaymentStrategy` is expected.

No wonder that this pattern works well. It is the Liskov Substitution Principle. A subtype has to be usable anywhere its supertype is, without breaking what the caller was promised, in every context the parent is used and not only the convenient ones. The HTML report failed exactly this test. `ExecutiveLevelManager` could not stand in for the class above it without breaking what `persistReport` quietly guaranteed. It looked like a subtype. It did not behave like one. When substitution genuinely holds, inheritance is the right call and the tree stays shallow and honest. When it only sort of holds, you are about to grow a `ContractorWithVehicle`.

And composition is not free. You trade one tree for a scatter of small objects and the wiring that assembles them, and that wiring brings failure modes of its own. There is no silver bullet here, only the removal of a default.

The ways composition goes wrong are real enough to deserve their own post.

Even so, the composition version is easier to test, since each behavior stands alone with nothing to mock around it, and it lands on the right side of Liskov and Interface Segregation. Those are consequences, not the reason. The reason is that it models the problem you actually have.

## It is about picking the right tool for the job: no defaults
Inheritance is easy to default into because the "is-a" requirement is too vague.

When you find yourself writing `class X extends Y`, run one check before you commit to it. Ask whether `X` *is* a `Y` everywhere a `Y` gets used, or whether `X` just *has* some of what `Y` does. Most of the time, if you are honest, the answer is "has". Build for that answer, and the weird base classes never get born.