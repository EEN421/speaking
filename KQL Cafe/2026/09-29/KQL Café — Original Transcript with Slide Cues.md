# THE QUERY RAN SUCCESSFULLY. THAT’S THE PROBLEM.

**KQL Café — Ian Hanley**  
**Target spoken runtime: ~42–45 minutes**  
**Theme: How security engineers have to think differently about KQL, telemetry, detection logic, and AI-generated analysis**

---

## [SLIDE 1 | TITLE: THE QUERY RAN SUCCESSFULLY. THAT’S THE PROBLEM.]

# 0:00 — OPENING

Hi! I'm Ian, a security researcher, engineer, and author who likes figuring out how security systems behave once they leave the whiteboard and meet the real world.

I work at Kai, the company rebuilding cybersecurity to run at machine speed; work that used to take security teams weeks now happens in minutes, and it's driving risk down through auto remediation. Human defenders don't just keep up, (and this is my favourite part)... they become superhuman.

Lastly, Being a husband and dad means i know a think or two about incident response, risk management, and chaos engineering, than any certification ever could.

So last time I came on KQL Café, I showed you KQL I was proud of.

So naturally, this time I brought KQL that's wrong.

And the annoying part is that most of it runs perfectly.

No syntax errors.

No red squiggles.

No complaints from Kusto.

It returns exactly what I asked for.

And that's the problem.

Because over the last few months I've been doing something slightly ridiculous.

## [CHANGE TO SLIDE 2 | I BUILT A MACHINE THAT WRITES DETECTIONS]

I built a machine that writes detections.

My DevSecOpsDadAttack pipeline pulls in threat intelligence, extracts behaviors, scores and clusters them, generates detection hypotheses, turns those into KQL candidates, critiques them, and eventually puts something in front of me for review.

On a normal week, that can mean roughly thirty detection candidates.

Which is fantastic.

AI allows me to cover more research, compare more material, and move much faster than I could manually.

But thirty detections a week also means thirty opportunities to be confidently wrong.

And that's really what led me into the subject I want to talk about today.

Because automation increases throughput.

It does **not** eliminate assumptions.

If anything, automation makes assumption management more important.

When I manually write one detection, I probably remember why I made every decision in it.

If a machine produces thirty?

Now I have the ability to operationalize a questionable assumption thirty times before lunch.

And that is really where my **KQL Detection of the Week** series came from.

The machine was producing detections.

A lot of them looked good.

Some were genuinely good.

But I realized I needed a recurring human review process.

Not just:

> Does the KQL compile?

But:

> Does this detection actually mean what we think it means?

And that question turns out to be much harder.

Because a query can be syntactically perfect.

Technically clever.

Beautifully written.

It can return exactly the rows you expected.

And it can still be wrong.

Not because Kusto failed.

Kusto is extremely good at answering the question you asked.

The problem is that sometimes the question you asked...

isn't the question you thought you asked.

And that's really the theme today.

Not just how to write KQL.

But how you have to **think** when you're writing KQL for security.

---

## [CHANGE TO SLIDE 3 | ZERO ROWS / NOTHING HAPPENED?]

# 3:00 — ZERO ROWS

One of the oldest examples of this for me actually predates the automated pipeline.

A few years ago I wrote a series called **KQL Detective**.

And the problem I was investigating was wonderfully boring.

Something was missing.

That was it.

Expected data wasn't there.

And absence is one of the most dangerous things in detection engineering because of what our brains automatically do with it.

We run a query.

We get zero rows.

And somewhere in our heads this translation happens:

> Nothing happened.

But that's not what Kusto told you.

Kusto told you:

> I found zero rows matching these predicates in the data you gave me.

Those are **very** different statements.

And honestly, that distinction might be half of detection engineering.

Because what does zero rows actually mean?

No attacker activity?

Maybe.

The query is wrong?

Maybe.

The field isn't populated?

Maybe.

The endpoint isn't reporting?

Maybe.

The parser changed?

Maybe.

The data exists somewhere else?

Maybe.

Your `where` clause eliminated it?

Maybe.

You don't know yet.

So:

## Zero rows is not an answer.

It's a clue.

And that's where the security mindset has to be different.

---

## [CHANGE TO SLIDE 4 | THE DATABASE IS NOT REALITY]

# 5:00 — SCENIC ROUTE: THE DATABASE IS NOT REALITY

This is something I wish we taught more explicitly.

When you're working in security, you're almost never really asking:

> What does the data say?

You're asking:

> What happened in the real world, and how much of that reality survived long enough to become data?

Those are completely different questions.

Because the database is not reality.

The SIEM is not reality.

Microsoft Defender is not reality.

Splunk is not reality.

Sentinel is not reality.

They're representations of reality.

Something happened on a device.

A sensor had to observe it.

The sensor had to decide it was worth recording.

The event had to be generated.

It had to survive collection.

Transport.

Ingestion.

Parsing.

Normalization.

Retention.

And then finally your query.

By the time you're looking at a row in Kusto, reality has been transformed several times.

So I like this phrase:

## We don't investigate reality.

## We investigate what survived ingestion.

And that means a security engineer cannot simply take the data at face value.

You have to understand the boundary around the data.

What could this sensor see?

What could it not see?

Under what conditions would this event never exist?

What fields are optional?

What gets normalized?

What gets thrown away?

And then there's another thing that makes security different from ordinary analytics.

The subject of your analysis may be actively trying to make your data wrong.

Your sales figures generally aren't trying to evade your dashboard.

The attacker gets a vote.

That's a fundamentally different analytical environment.

An adversary can change representations.

Use a different execution path.

Add another process generation.

Use another protocol.

Move to another sensor boundary.

Exploit the thing your detection quietly assumes will always be true.

So when you're writing security KQL, you need this slightly uncomfortable mindset where you're constantly asking:

> What else could explain this?

And:

> What could happen that this query would never show me?

That isn't pessimism.

That's the job.

---

## [CHANGE TO SLIDES 5-7 | KQL MOMENT: LEFTANTI]

# 8:00 — KQL TEACHABLE MOMENT #1: `leftanti`

And this brings us to one of my favorite KQL operators for thinking about absence:

`leftanti`.

The left-anti join is basically:

> Give me everything on the left that does **not** have a matching row on the right.

Conceptually:

```kusto
ExpectedDevices
| join kind=leftanti (
    ObservedDevices
) on DeviceId
```

That is incredibly useful in security.

Show me endpoints that should be reporting but aren't.

Show me identities in this population that have no corresponding event.

Show me something I expected to observe...

but didn't.

This is the **dog that didn't bark** pattern.

And it's powerful.

But here's the important distinction.

`leftanti` does **not** prove that something does not exist.

It proves that no matching row exists **in the right-hand dataset you supplied, using the join condition you specified**.

That's it.

Kusto isn't telling you:

> This endpoint did not perform the action.

It's telling you:

> I couldn't find the matching evidence over there.

Those are different claims.

And that's the theme again.

The operator is doing exactly what you asked.

You are responsible for understanding what that actually allows you to conclude.

So whenever I use something like `leftanti`, I immediately want to ask:

Is the right-hand dataset complete?

Is the join key stable?

Is the time window correct?

Could ingestion latency explain the absence?

Could the source simply not be reporting?

Did we choose the right table?

Because otherwise this beautiful absence detection becomes a machine for manufacturing false confidence.

---

## [CHANGE TO SLIDES 8-9 | THE FIELD THAT WASN’T THERE]

# 11:00 — THE FIELD THAT WASN'T THERE

That leads naturally into one of my Detection of the Week examples.

I called it:

## The Field That Wasn't There.

Imagine I have a perfectly reasonable detection.

It depends on a particular `ActionType`.

Or maybe it depends on `RemoteUrl`.

The field exists.

Microsoft documents it.

The table exists.

The query compiles.

Everything looks fine.

But:

## Schema availability is not telemetry availability.

A column existing in documentation does not mean that column is meaningfully populated in your environment.

An event type existing in a platform does not mean your estate produces it.

And this is why I like the rule:

## Validate first. Detect second.

Before you write the brilliant detection, ask some embarrassingly boring questions.

Does this telemetry exist?

How frequently?

On which devices?

For which operating systems?

How often is the field null or empty?

Are there obvious gaps?

Is this actually representative of the environment?

Because suppose your detection looks for malicious activity using `RemoteUrl`.

You run it.

Zero rows.

Excellent.

No malicious activity.

Except what if `RemoteUrl` is populated in three percent of the relevant telemetry?

Now zero rows doesn't tell you very much about attacker activity.

It tells you something about your visibility.

Sometimes the most honest output from a security query is:

> I don't have enough evidence to answer this safely.

And that's okay.

I'd rather have an honest unknown than fictional coverage.

---

## [CHANGE TO SLIDE 10 | FALSE POSITIVES / FALSE CONFIDENCE]

# 14:00 — FALSE POSITIVES VERSUS FALSE CONFIDENCE

We spend a lot of time in security complaining about false positives.

And reasonably so.

False positives are expensive.

They annoy analysts.

They create alert fatigue.

They waste time.

But there is something I worry about more.

False confidence.

Because false positives are visible.

A noisy detection gets investigated.

Everybody knows it has a problem.

A detection that quietly doesn't work?

That one can sit there for two years with a green checkmark next to it.

Nobody complains.

Nobody opens a ticket.

Nobody tunes it.

It just doesn't fire.

And eventually somebody asks:

> Do we have coverage for this technique?

And someone points at the detection rule and says:

> Yep.

That scares me much more.

So:

## A noisy detection gets investigated.

## A quiet broken detection gets trusted.

And that's one reason I like reviewing detections that haven't fired in something like sixty days.

Not because sixty days is a magic number.

Some detections **should** be rare.

But inactivity is a reason to ask:

Can this still fire?

Does the prerequisite telemetry still exist?

Can I create a benign condition that should trigger it?

Did a parser change?

Did the product change?

Did the schema change?

Is the detection still asking a meaningful question?

Trust decays.

Detection logic has maintenance requirements just like software does.

Possibly worse, because your input contract is basically:

> Whatever telemetry this product decides to give me today.

---

## [CHANGE TO SLIDES 11-12 | A MEETING IN 2050]

# 16:00 — A MEETING IN 2050

Sometimes the telemetry isn't missing.

It's simply somewhere you never thought to look.

One of my favorite recent examples was calendar command-and-control.

Imagine I tell you:

There's a meeting on the calendar.

It's scheduled for the year 2050.

Nobody is attending.

And it's command-and-control.

Which sounds ridiculous.

That's why I love it.

The behavior abused calendar objects as synchronized storage.

Create something.

Attach something.

Manipulate it.

Retrieve it somewhere else.

Suddenly the calendar becomes:

> the most synchronized, least-read database in your organization.

But the lesson isn't:

> Everyone needs a calendar-C2 analytic tomorrow.

The lesson is about how your mental model constrains your query.

If I begin with:

> Command-and-control is network activity...

then I've already limited the investigation.

The attacker does not care which table I think C2 belongs in.

The behavior exists independently of my schema.

So sometimes when your network query returns zero rows, the problem isn't the syntax.

It's not even the telemetry.

You may simply be asking the wrong sensor.

And you can't query your way out of visibility you never had.

That's another place where I think ATT&CK can accidentally get us into trouble.

I love MITRE ATT&CK.

It's incredibly useful.

But:

## ATT&CK is a map.

## It is not the terrain.

The attacker does not know they are currently in Credential Access.

They're stealing credentials.

They don't know they've transitioned from Execution to Persistence.

They're doing whatever gets them the outcome they want.

The framework is how **we** organize our understanding.

It is not how the attack experiences itself.

And if we're not careful, we end up detecting the framework instead of detecting the behavior.

---

## [CHANGE TO SLIDE 13-14 | THE STRING IS NOT THE THING]

# 19:00 — THE STRING IS NOT THE THING

Now let's assume the telemetry exists.

We picked the right sensor.

Wonderful.

We're still not safe.

Because representation can lie to us too.

One Detection of the Week example involved metadata-service SSRF.

Everybody recognizes:

`169.254.169.254`

So the obvious detection is something like:

```kusto
| where RemoteIP == "169.254.169.254"
```

Looks reasonable.

Except an IP address is basically an integer wearing a costume.

There are different ways to represent the same underlying value.

Decimal.

Hexadecimal.

Historical octal-style representations.

Integer forms.

And depending on the parser and application, multiple representations can resolve to the same underlying address.

So if I'm detecting the spelling...

I may not actually be detecting the thing.

I've detected one representation of it.

And that distinction sounds pedantic right up until an attacker changes the representation without changing the destination.

So:

## Detect meaning, not spelling.

Normalize first.

Then compare.

And this applies everywhere.

Domains.

URLs.

Paths.

Hashes.

Encoding.

Escaping.

Case.

Unicode.

Canonicalization.

The string is not necessarily the thing you're trying to detect.

It's one representation of the thing.

And this is where security queries often become dangerous because they're plausible.

An obviously broken query gets fixed.

A plausible query gets shipped.

Six months later someone points to it and says:

> We have coverage for metadata-service SSRF.

Do we?

Or do we have coverage for one convenient way of spelling the address?

Coverage is a claim.

Not a property.

You don't automatically own coverage because a rule exists in Git.

---

## [CHANGE TO SLIDES 15-16 | THE CHARACTER IS NOT THE PAYLOAD]

# 22:00 — THE CHARACTER IS NOT THE PAYLOAD

Unicode gave me another version of exactly the same lesson.

One of the behaviors I reviewed involved Unicode tag characters.

Characters in the U+E0000 through U+E007F range.

They can be visually invisible.

So the obvious first attempt is:

> Let's detect suspicious invisible Unicode.

Okay.

But:

## The character is not the payload.

The meaningful thing isn't simply:

> Strange invisible character present.

Those characters can represent encoded information.

So now the questions are:

Which code points are present?

What do they map to?

Can we decode them?

What does the decoded content mean?

What behavior does that enable?

Again:

Representation versus semantics.

Completely different technology.

Same analytical mistake.

And that's been one of the most interesting things about doing Detection of the Week.

The technologies change every week.

The mistakes don't.

---

## [CHANGE TO SLIDE 17 — KQL MOMENT: PREV()]

# 24:00 — KQL TEACHABLE MOMENT #2: `prev()` DOESN'T KNOW WHAT AN ENTITY IS

Now let's assume:

We have the right sensor.

The telemetry exists.

We've normalized representation.

We're still not safe.

Because now we need to talk about relationships.

One of my favorite KQL traps is `prev()`.

`prev()` is fantastic.

I use it all the time.

But `prev()` is wonderfully literal.

Give me the previous row.

That's what it does.

It does not know what a computer is.

It does not know what a user is.

It does not know what a process is.

It does not know what a security boundary is.

It knows:

> previous row.

So imagine something like this:

```kusto
Events
| sort by TimeGenerated asc
| serialize
| extend PreviousAction = prev(ActionType)
```

Perfectly valid.

Except what if the previous row belongs to a different machine?

Machine A produces event one.

Machine B produces event two.

Machine C produces event three.

And now I've accidentally created a beautifully convincing sequence...

that never happened anywhere.

The query ran successfully.

The story is fictional.

So the pattern I like is:

## SORT → SERIALIZE → GUARD.

You establish ordering deliberately.

You serialize where appropriate.

And then you guard the entity boundary.

Something conceptually like:

```kusto
Events
| sort by DeviceId asc, TimeGenerated asc
| serialize
| extend
    PreviousDevice = prev(DeviceId),
    PreviousAction = prev(ActionType)
| where DeviceId == PreviousDevice
```

Now at least I've explicitly told the query:

> The previous row only matters to me when it belongs to the same analytical entity.

Because:

## `prev()` knows rows.

## You know entities.

Don't confuse the two.

---

## [CHANGE TO SLIDE 18 — A JOIN IS AN ACCUSATION]

# 27:00 — SCENIC ROUTE: A JOIN IS AN ACCUSATION

And this applies more broadly to joins.

I like thinking of every security join as an accusation.

Because when I write:

```kusto
A
| join B on DeviceId
```

I'm making a claim.

I'm saying:

> These two pieces of evidence belong together because they share this identifier.

Kusto will happily join them.

It's not Kusto making the accusation.

It's me.

And my join key is the evidence.

So the right question is not merely:

> Did the join work?

It's:

> What relationship does this join actually prove?

Same username?

Maybe.

But usernames can be reused.

Same IP?

Maybe.

But NAT exists.

DHCP exists.

Proxies exist.

Same process name?

Definitely not enough.

Same five-minute window?

Temporal proximity is not causality.

Two things happening near each other does not automatically mean one caused the other.

I drank coffee this morning and Azure probably had an incident somewhere.

I'm willing to accept correlation.

Microsoft Legal may want slightly more before we establish causality.

[PAUSE]

The point is:

Kusto works in rows.

Security works in relationships.

And the relationship is something **you** have to justify.

---

## [CHANGE TO SLIDES 19-20 | SINS OF THE GRANDFATHER / PARENT ≠ LINEAGE]

# 29:00 — SINS OF THE GRANDFATHER

A related Detection of the Week example involved an npm lifecycle attack.

The query was looking for something conceptually like:

Node launches shell.

Shell launches curl.

Very reasonable.

Parent-child relationship.

Except attackers are under no obligation to fit within one generation of your process tree.

Maybe npm launches node.

Node launches shell.

Shell launches another interpreter.

And **that** launches curl.

The malicious relationship still exists.

Your query simply expected it one edge earlier.

I called this:

## Sins of the Grandfather.

Because:

## Parent does not equal lineage.

If the behavior you're trying to understand is ancestry, then model ancestry.

Don't quietly substitute parenthood because it's easier to query.

Again:

The KQL can be valid.

The process names can all be correct.

The timestamps can line up.

The join can return rows.

And the analytic can still misunderstand the behavior.

This is why I increasingly think the hardest part of detection engineering isn't syntax.

It's ontology.

What is the thing?

What is the entity?

What relationship actually matters?

And what claim am I entitled to make from what I've observed?

---

## [CHANGE TO SLIDE 21 | MAKE_SET() IS NOT A TIMELINE]

# 31:00 — KQL TEACHABLE MOMENT #3: `make_set()` IS NOT A TIMELINE

This is another one I think is worth teaching because it's extremely easy to lose meaning during aggregation.

Security analysts love `summarize`.

I love `summarize`.

It's one of the reasons KQL is so expressive.

But every time you summarize, you're making a decision about what detail you're willing to throw away.

So here's a useful mental model:

## Every `summarize` is lossy compression.

Maybe that's exactly what you want.

But know what you're discarding.

And a classic example is the difference between `make_set()` and `make_list()`.

If I do:

```kusto
| summarize Actions = make_set(ActionType) by DeviceId
```

I'm asking:

> Which action types appeared?

That's membership.

That's useful.

But it is **not chronology**.

If I later look at that array and start telling a story:

> First this happened, then this, then this...

I may be inventing order that the query never established.

`make_set()` is not a timeline.

If order matters, I need to think about ordering before aggregation and use an appropriate structure.

`make_list()` can preserve the ordering of its input when the input is deliberately ordered.

But even then, **I** have to establish that ordering.

Kusto does not know which sequence is meaningful just because I put events into an array.

So:

## Membership is not chronology.

That's a tiny KQL distinction with a huge analytical consequence.

Because once again, a query can contain all the right events...

and still tell the wrong story.

---

## [CHANGE TO SLIDE 22 — TELEMETRY TRUST STACK]

# 36:00 — THE TELEMETRY TRUST STACK

So after reviewing enough of these detections, I started thinking of trust as a stack.

Not a score.

Not some magical trust indicator.

Just a sequence of questions.

At the bottom:

### Observation.

Could the sensor observe the behavior at all?

Then:

### Collection.

Did the evidence successfully make it from the source into the platform?

Then:

### Population.

Do the table and fields actually contain the information the detection depends on?

Then:

### Representation.

Am I detecting the underlying meaning, or just one encoding of it?

Then:

### Identity.

Do the rows I'm correlating actually belong to the same entity?

Then:

### Relationship.

Does the correlation I created represent the relationship the behavior requires?

And finally:

### Claim.

What am I actually entitled to conclude from this evidence?

That last one matters a lot.

Because we spend enormous effort validating the query.

But the query is only one part of the chain.

You can write perfect KQL against the wrong sensor.

Perfect KQL against an empty field.

Perfect KQL against one representation.

Perfect KQL correlating unrelated machines.

Perfect KQL detecting one stage and claiming the whole attack.

The engine cannot save you from a bad premise.

And neither can AI.

---

## [CHANGE TO SLIDE 23 | NOW GIVE THE BAD PREMISE A GPU]

# 38:00 — NOW GIVE THE BAD PREMISE A GPU

And this is where I want to bring AI back into the discussion.

Because everything we've talked about so far existed before generative AI.

Humans have been making bad assumptions for quite a long time.

We're excellent at it.

AI did not invent that.

What AI changes is **scale**.

If I manually write one flawed detection this week, that's one problem.

If I encode the same assumption into a system capable of generating thirty detections a week...

now I have a different problem.

I've automated the assumption.

And I think that's a piece we sometimes miss in the AI conversation.

We talk about:

Speed.

Scale.

Agents.

Autonomy.

Productivity.

And those are real benefits.

I use them.

But every multiplier multiplies both the good work...

and whatever mistakes survive your controls.

So:

## Automation does not remove assumptions.

## It industrializes them.

And that is why I think assumption management becomes **more important** as automation gets better.

Not less.

---

## [CHANGE TO SLIDE 24 | OBVIOUSLY WRONG VS PLAUSIBLY WRONG]

# 40:00 — KQL DETECTION OF THE WEEK EXISTS BECAUSE THE MACHINE NEEDED A REVIEWER

This is really why Detection of the Week exists.

The pipeline was producing KQL.

And the interesting failures weren't usually ridiculous.

They weren't:

> `DeviceMagicUnicornTable`

Those are easy.

An invented table name gets caught.

Syntax errors get caught.

What scared me were detections that were **95 percent reasonable**.

Because plausible output is much more dangerous.

A field that probably contains the data you want.

A parent-child relationship that usually looks that way.

An IP comparison that works for the obvious representation.

A sequence that looks correct unless you check the entity boundary.

A join that proves proximity but not causality.

Those survive casual review.

Especially when you're busy.

Especially when the system has been correct twenty times in a row.

Especially once the automation earns trust.

And I think successful automation actually creates a new failure mode:

## Trust accumulation.

The machine works.

Then it works again.

Then again.

Eventually you stop interrogating it as aggressively.

You skim.

You approve.

You assume.

And now the same system that made you more productive is also capable of propagating a bad premise farther and faster than you could manually.

That's why I like this line:

## Obviously wrong is cheap.

## Plausibly wrong is expensive.

The obviously wrong thing announces itself.

The plausible mistake integrates beautifully into production.

---

## [CHANGE TO SLIDE 25 | HUMAN IN THE LOOP ≠ RUBBER STAMP]

# 42:00 — HUMAN IN THE LOOP DOES NOT MEAN HUMAN RUBBER STAMP

And this is where I think we need to be careful with the phrase:

> human in the loop.

Because technically having a human click Approve does not mean you have meaningful oversight.

If your system generates a hundred outputs...

and the human approves ninety-nine...

eventually the hundredth gets approved largely because the previous ninety-nine were fine.

That's not really review anymore.

That's ceremony.

A checkbox is not governance.

The point of the human isn't:

> Somebody has to press the green button.

The human has to own the assumptions that matter.

What behavior are we detecting?

Which telemetry sources are required?

What fields must exist?

What relationships are being inferred?

What could make the detection silently fail?

What could make it produce a false relationship?

What claim is this output actually making?

And that doesn't mean the human should manually redo all of the AI's work.

That defeats the point.

I want AI doing the expensive repetitive work.

Read the fifty reports.

Cluster the stories.

Extract the behaviors.

Draft the KQL.

Critique the KQL.

Generate test cases.

Find alternative representations.

Ask adversarial questions.

Absolutely.

The machine handles scale.

The human retains judgment and accountability.

---

## [CHANGE TO SLIDE 26 | AI CAN CHALLENGE. REALITY RESOLVES.]

# 44:00 — AI CAN REVIEW AI, BUT REALITY GETS THE FINAL VOTE

And AI can absolutely be part of the review process.

I think that's actually one of the best uses for it.

Take a generated detection and ask another stage:

What assumptions does this query make?

Which columns could be sparsely populated?

What alternate representations might bypass this logic?

Does this join really prove the relationship being claimed?

What benign activity could look the same?

What negative tests should we run?

That's useful.

Use one model to criticize another.

Generate adversarial test cases automatically.

Use AI to help surface the assumptions humans should spend their limited attention on.

But there's an important boundary.

The system that generated the conclusion should not be treated as the ultimate authority certifying its own conclusion.

Even if another model agrees.

Two LLMs agreeing with each other does not mean your environment agreed.

At some point something has to touch reality.

Run the query.

Inspect actual telemetry.

Measure field population.

Create the test condition.

Validate the entity relationship.

Check the sensor.

AI might correctly tell me:

> This detection assumes `RemoteUrl` is populated.

Great.

But AI doesn't get the final say on whether `RemoteUrl` is populated in my environment.

The environment does.

So I think of it this way:

## AI can challenge the assumption.

## Reality resolves it.

---

## REVIEW IS A CADENCE, NOT A GATE]

# 46:00 — REVIEW IS A CADENCE, NOT A GATE

And this is another reason I don't think human review can happen once.

Generate detection.

Review detection.

Approve detection.

Finished forever.

No.

Because everything underneath the detection changes.

Telemetry changes.

Products change.

Schemas change.

Parsers change.

Agents change.

Attackers change.

Models change.

Prompts change.

Sources feeding the pipeline change.

You can make no change whatsoever to the KQL and still have the meaning of the KQL change underneath you.

That's why I like periodic review.

Per-output review where consequences are important.

Scheduled review for detections.

Review after telemetry changes.

Review after model changes.

Review after prompt changes.

And importantly:

Review some of the outputs that **look successful**.

Don't only investigate failures.

Because the dangerous output may be the one that looks completely normal.

If I generated a hundred apparently good detections this month, I want to sample some of those and tear them apart.

Not because I expect them all to be wrong.

Because I want to know whether there is a systematic assumption spreading through the pipeline.

Again:

Automation makes assumption management more important.

Not less.

---

## [CHANGE TO SLIDE 27 | READ THE DETECTION BACKWARDS]

# 48:00 — HOW I READ A SECURITY QUERY NOW

So now when I read a detection...

whether a human wrote it or an AI wrote it...

I don't really start at the top.

I read it backwards.

I start with the conclusion.

What is this detection claiming?

Then:

What evidence supports that claim?

What relationship between those events am I assuming?

What entity boundaries matter?

What representations am I expecting?

Which fields need to be populated?

Which event types must exist?

Which tables need to exist?

Which collection path feeds those tables?

Which sensor observed the original behavior?

And then I ask the nastiest question I know:

## What would make this query return zero even if the attack happened?

That one question catches an unbelievable amount.

Maybe the attacker changes an IP representation.

Maybe the process ends up one generation deeper.

Maybe the field isn't populated.

Maybe the telemetry comes from the wrong sensor.

Maybe the events cross an entity boundary.

Maybe the detection sees one stage when what we actually care about is the chain.

And sometimes the answer is:

> I cannot make this claim reliably with the telemetry available.

Fine.

That is a valid engineering conclusion.

I'd rather ship an honest limitation than fictional certainty.

---

## [CHANGE TO SLIDES 28-29 | 6 Questions to Steam & SECURITY-ENGINEER KQL MINDSET]

# 50:00 — THE SECURITY-ENGINEER KQL MINDSET

And I think this gets to the biggest point I want people to take away.

There is a different mindset when you use KQL as a security engineer.

You're not merely querying a database.

You're reconstructing an event from fragments.

Fragments produced by systems that may be incomplete.

Noisy.

Delayed.

Misconfigured.

Version-dependent.

Normalized.

Filtered.

And occasionally manipulated by somebody who really does not want you reconstructing the event correctly.

So yes:

Learn the syntax.

Learn `summarize`.

Learn `join`.

Learn `leftanti`.

Learn `serialize`.

Learn `prev()`.

Learn `make_set()`.

Learn every clever operator you want.

But the deeper skill is learning to ask:

## What does this result actually allow me to say?

And then having the discipline not to say more than that.

Because:

A row appearing does not automatically prove your theory.

It proves the row appeared.

Now you have to earn the interpretation.

And zero rows does not mean nothing happened.

It means Kusto found no matching rows under the conditions you provided.

Everything beyond that...

is your responsibility.

---

The dangerous detection is the one that runs perfectly...

returns exactly what you expected...

gets approved because it looks reasonable...

gets automated because it keeps looking reasonable...

and quietly answers the wrong question at scale.

**The query ran successfully.**

That's the problem.

Thank you.
