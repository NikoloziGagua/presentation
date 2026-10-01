# Video script: Boeing 737 MAX, Bug or Business Decision?

**Target length:** about 4 minutes (3:36 to 4:24 allowed). Roughly 645 words, so keep a steady pace (about 150 words a minute).
**Audience:** other software developers (imagine a company training course).
**The story in one line:** a plane fights its pilots, and we work out whether the code was broken or doing exactly what it was told.

**Zoom cues** are in *[brackets]*. Don't read them out.

---

## 1. The cockpit (0:00 to 0:35)

*[Start on the full poster, then zoom into the header photo]*

October 2018. A brand-new Boeing 737 MAX takes off from Jakarta. Minutes later, the nose starts pushing down. The pilots pull it back up. It pushes down again. And again.

They're fighting something, but they don't know what, because nobody ever told them it existed.

Thirteen minutes after take-off, Lion Air flight 610 hits the sea. 189 people die.

*[Zoom onto the 346 badge]*

Five months later, it happens again in Ethiopia. 346 people in total. And the thing pushing the nose down wasn't a mechanical fault. It was software.

So here's my question for you, as developers: was it a bug?

## 2. Why the code existed (0:35 to 1:25)

*[Zoom into Panel 2, Figure 1]*

To answer that, we need to know why this software was written in the first place.

Boeing wanted the MAX to burn less fuel, so it fitted bigger engines. They had to sit further forward on the wing, and that made the nose want to pitch up.

Boeing could have redesigned the plane. Instead, it added a piece of software called MCAS, which quietly pushes the nose back down. That way the MAX would feel just like the old 737.

And that feeling was worth a lot of money. If it flew like the old plane, pilots wouldn't need new simulator training. Boeing even promised Southwest Airlines it would pay a million dollars per plane if its pilots ended up needing that simulator training.

So MCAS had one job: make a new plane feel like an old one.

## 3. Four decisions (1:25 to 2:35)

*[Zoom into Panel 3]*

Now look at how it was built. There are four decisions here, and each one sounds reasonable on its own.

*[Point to the first icon]*

One: MCAS read just one angle-of-attack sensor. The plane has two, but the code never compared them. That's a single point of failure in a system that controls the nose.

*[Zoom into Figure 3, the chart]*

Two: the requirement changed. The safety analysis the FAA saw said MCAS could move the tail by 0.6 degrees. The version that actually flew used 2.5, more than four times as much, and the analysis was never brought up to date.

*[Point to the eye icon]*

Three: pilots weren't told. MCAS wasn't in their manuals. The light that warns when the two sensors disagree only worked if the airline paid for an optional extra.

*[Zoom into Figure 2, the code]*

Four: it didn't stop. If the pilot trimmed back, MCAS fired again. So one faulty sensor meant the plane kept pushing the nose down until the pilots ran out of time.

## 4. What happened next (2:35 to 3:05)

*[Zoom into Panel 1, the timeline]*

After the second crash, the 737 MAX was grounded across the world. In the US it stayed on the ground for twenty months, until a software fix. In 2021, Boeing paid 2.5 billion dollars to settle a US fraud charge.

This is an American story: Boeing and its regulator, the FAA, are both US-based. But the lesson isn't only about aviation.

## 5. The answer (3:05 to 3:35)

*[Zoom into the red box in Panel 3]*

So, was it a bug?

I'd say yes. But not the kind of bug you'd find in a code review. MCAS matched its specification. The spec itself was wrong: one sensor, a requirement that changed without anyone re-checking it, and a feature kept hidden to save on training costs.

Software can pass every test against its specification and still do something nobody intended. Nobody intended MCAS to fly a plane into the ground. That's a bug, even if it's in the spec and not in the code.

*[Zoom into the quote in Panel 4]*

The US House committee called it "a horrific culmination of a series of faulty technical assumptions", along with "a lack of transparency" and "grossly insufficient oversight".

## 6. Your turn (3:35 to 4:05)

*[Zoom into the families photo, then the questions]*

Behind those words are the families in this photo.

So I'll leave you with three questions. Would you ship code that trusts one sensor? If the requirement changes, does your safety analysis change too? And if users don't know your code exists, how can they fight it when it goes wrong?

Most of us will never write flight software. But we all write code that trusts one input, under a deadline, for someone's budget. "It matches the spec" doesn't mean "it's not a bug".

Thanks for watching.

---

## Notes (not part of the narration)

- Time yourself on one read-through. If you're over 4:24, cut the Southwest sentence in part 2. If you're under 3:36, slow down and pause after "was it a bug?" both times it comes up.
- Record each numbered part as its own clip, then join them. A mistake then only costs you one short section.
- Read it through and change any phrasing that doesn't sound like you before you record.
