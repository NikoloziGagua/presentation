# Video script: Boeing 737 MAX, Bug or Business Decision?

**Target length:** about 4 minutes (3:36 to 4:24 allowed), read at a normal pace of roughly 150 words per minute.
**Audience:** other software developers (imagine a company training course).
**Visuals:** open on the full poster, then zoom into each numbered panel as you talk about it.

---

## [0:00 to 0:25] Hook (title and the 346 badge)

Here's a question for you as developers. If your code does exactly what the spec says, and people die, is that a bug? That's the question behind the Boeing 737 MAX. It's an American story, because Boeing and its regulator, the FAA, are both US-based. Three hundred and forty-six people died in two crashes, and a piece of software was at the centre of both.

## [0:25 to 1:00] Panel 1: What happened?

In October 2018, Lion Air flight 610 crashed into the sea off Indonesia, killing 189 people. Less than five months later, Ethiopian Airlines flight 302 crashed just after take-off, killing 157. Both were brand-new 737 MAXs. Three days after the second crash, the FAA grounded the MAX in the US, and it wasn't cleared to fly again until November 2020, after a software fix. In January 2021, Boeing paid 2.5 billion dollars to settle a US fraud charge.

## [1:00 to 1:50] Panel 2: The hidden software, MCAS

So what is MCAS? The MAX has bigger engines than the older 737, and they had to be mounted further forward. That made the nose want to pitch up. Instead of a bigger redesign, Boeing added software called MCAS, which pushes the nose back down automatically. The idea was that the MAX would handle like the old 737, so airlines wouldn't have to put pilots through simulator training.

Figure 2 is a simplified version. It's not Boeing's real code. If the angle-of-attack reading is too high and the flaps are up, trim the nose down by 2.5 degrees, and if the pilot trims back, do it again. Now look at the last line. The plane has two angle-of-attack sensors, like the one in Figure 1, but MCAS relied on just one at a time. So when that one sensor gave bad data, MCAS kept pushing the nose down.

## [1:50 to 2:50] Panel 3: Bug or business decision?

So was it a bug? I'd argue it wasn't, and that's the scary part.

First, one sensor with no cross-check is a single point of failure, in a system that can push a plane's nose down.

Second, the requirement changed. The safety analysis the FAA saw said MCAS could move the tail by up to 0.6 degrees. The final design used 2.5, over four times more, and that number was news to the FAA's own engineers.

Third, pilots weren't told. MCAS wasn't disclosed to them, and the warning that tells pilots the two sensors disagree only worked if the airline had bought an optional extra.

And why did all of this matter so much? Because training cost money. Boeing had promised Southwest Airlines a million dollars per plane if its pilots needed simulator training.

Put it together and the code did exactly what it was designed to do. The failure was in the decisions around it.

## [2:50 to 3:40] Panel 4: What should developers learn?

The US House committee that investigated the crashes called them "a horrific culmination of a series of faulty technical assumptions by Boeing's engineers, a lack of transparency on the part of Boeing's management, and grossly insufficient oversight by the FAA." Behind those words are the families in Figure 4, holding photos of the people they lost on Ethiopian 302.

So I'll leave you with three questions. Would you ship a safety feature that trusts one sensor? When the requirement changes, does your safety analysis change with it, or do the tests still check the old one? And if users don't know your code exists, how can they fight it when it goes wrong? The Lion Air pilots were fighting a system nobody had told them about.

## [3:40 to 4:00] Close

Most of us won't write flight software. But we all write code that reads one input and trusts it, and we all work under deadlines and budgets. The 737 MAX shows that "it works as designed" is not the same as "it's safe". Thanks for watching.

---

## Notes (not part of the narration)

- Read it aloud once with a timer. If you run over 4:24, cut the Southwest sentence. If you're under 3:36, slow down on the three questions, which is where the pauses help most.
- Say "Figure 1", "Figure 2" and so on as you zoom in, so viewers can follow the poster.
- Before recording, double-check the facts listed in the chat reply.
