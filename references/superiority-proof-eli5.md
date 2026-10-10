# LAFS over K&R, explained like you're 5

This is the simple version of `superiority-proof.md`. It follows the same sections in the same order, so you can jump between the two, but it has no math symbols and no jargon. If something here feels too simple, the full version has the exact statement and the proof.

## The one picture: matching shoes

Every `{` and `}` is a pair of shoes: a left shoe that opens a block and a right shoe that closes it. Reading code means finding which right shoe goes with which left shoe. The whole proof is about where the two shoes stand.

```
// K&R: the left shoe hides at the end of the line
if (SomethingVeryLongHere) {
    DoThing();
}

// LAFS: both shoes stand in the same column
if (SomethingVeryLongHere)
{
    DoThing();
}
```

## §0 How to read the proof

The proof labels every sentence so you can tell what kind of sentence it is.

- **D (definitions)** are the rules of the game.
- **T (theorems)** are things you can check by counting. You don't have to trust anyone.
- **A and P (axioms and premises)** are things we choose to believe. You are allowed to say no.
- **C (corollaries)** are what follows if you say yes to the premises.

The proof makes four claims.

- **C0:** K&R's left shoe always stands off to the side, and no tool can fix that without turning K&R into something else.
- **C1:** LAFS pays a small, fixed price to line the shoes up, and it has tricks to win that price back.
- **C2:** If you believe lining up matters most, K&R never wins. If you only trade it off against saving lines, K&R can win on short heads.
- **C3:** "Worse is Better" explains why K&R is popular. It doesn't prove K&R is better.

The proof also says the word "never" is only exactly true in one sense. That's the sense in C2, and §6 below lists where it gets weaker.

## §1 The rules of the game

**The graph paper (D1).** Imagine every character sits on a square of graph paper. We can say exactly which row and column each shoe is in.

**Tall and flat (D2).** A pair of shoes is *flat* if both are on the same line. Nothing to hunt for, so it doesn't matter. A pair is *tall* if they're on different lines. Only tall pairs matter.

**The head (D3).** The head is the words in front of the left shoe, like `if (Ready)`. How wide the head is turns out to matter a lot.

**Groups of shoes (D4).** Sometimes shoes stand side by side, like `({`. We call that a group, and we measure from group to group. The measurement we care about is the **sideways gap**: how many columns apart the two groups of shoes are. Zero means they stand in the same column.

**Three ways to place the shoes (D5).** Each way pays a price for matching the shoes:

| Style | Where is the left shoe? | Where is the right shoe? | What you pay |
|---|---|---|---|
| K&R | End of the head line | Under the first letter of the head | A sideways gap as wide as the head |
| LAFS (Allman) | Alone on the next line, in the column | Same column | One extra line |
| Hybrid | End of the head line | Slid over to stand right under it | Everything inside shifts right by the head's width |

**Two annoyances (D6).** There are two things that might bother a reader: one extra line, and each step of sideways walking. We don't know how much each one bothers a particular person, so the proof works for any amount.

**The fair-fight rule (the Steelman Protocol).** Before comparing, I gave K&R every trick a formatter can add: squishing small things onto one line, one item per line, and so on. That is like letting the other team use their best player. After that, the only difference left between K&R and LAFS is where the shoes stand.

## §2 Things you can check by counting

### T1: The sideways gap

In K&R the sideways gap equals the width of the head plus one space. A head like `if (Record->IsActive)` is 21 characters, so the gap is 22 columns. In LAFS the gap is zero. This is a counting fact, and it comes straight from where each style puts the shoes.

**Why K&R can't be fixed.** K&R *means* "left shoe at the end of the head line, right shoe under the head's first letter". If you move the left shoe under the head, that's LAFS. If you slide the right shoe over to the left shoe, that's Hybrid. Either way you've stopped doing K&R. So no tool, option or future formatter can give K&R a zero gap. The gap is what K&R *is*.

The proof is careful about one thing. It proves K&R can't reach zero. It does not prove zero is worth wanting. That comes later, as a choice (A1 in §4).

### T2: Standing in a line

In LAFS the head, the left shoe and the right shoe all stand in one straight column, like three people queued up. In K&R only the head's first letter and the right shoe do, and the left shoe is off to the side.

This gives a second counting fact. In LAFS the left shoes can only stand on the "staircase" of indent steps, so there are at most as many spots as the code is deep. In K&R the left shoes land wherever the words happen to end, so there can be one spot per different head width. (Hybrid is excluded because the language forces its left shoe to stay inline.)

### T3: Three ways to pay

Every style pays to match the shoes, but they pay with different coins.

- **K&R pays in sideways walking.** The longer the head, the longer the walk.
- **Hybrid pays in pushing everything inside to the right.** Same: a longer head pushes it further.
- **LAFS pays in extra lines.** One extra line, sometimes two or three, and it doesn't matter how long the head is.

A walk that keeps growing will always lose to a price that stays the same. Pick any reader, however much they dislike extra lines, and there is a head width beyond which K&R costs them more. The reverse is also true: K&R can only win on containers whose sideways gap is smaller than the reader's price for extra lines.

This table shows what that looks like on the 8 tall pairs measured in §3:

| An extra line bothers you as much as walking this many steps | LAFS is cheaper when K&R's gap is bigger than | How many of my 8 examples |
|---|---|---|
| 4 | 4 | 8 of 8 |
| 8 | 8 | 6 of 8 |
| 16 | 16 | 5 of 8 |
| 32 | 32 | 3 of 8 |

This is also why the Hybrid rules cap the head width. Hybrid pays in the growing coin, so the spec keeps the head short.

### T4: Mirror versus pile-up

In LAFS, closing shoes are the exact reverse of the opening shoes, standing in the same column. If a group opens with `({`, it closes with `})` right below it. You match them by looking, with no counting needed.

In K&R, closing shoes from different places can pile up on one line:

```
// K&R: three closing shoes pile up in one spot. Which belongs to which?
Items.Filter((Item) => Item.IsActive).ForEach((Item) => Process({
    // Ids only: payload is fetched lazily
    Id: Item.Id
}));
```

The `}));` sits at the left edge. Its partners are way over at columns 45 and 63. The closing shoes' positions don't tell you where their partners are, because the partners' positions depend on the words before them. The only way to pair them is to count from the inside out. LAFS gives each group its own line instead:

```
// LAFS: each closing group stands under its own opening group
Items.Filter((Item) => Item.IsActive).ForEach
(
    (Item) => Process
    ({
        // Ids only: payload is fetched lazily
        Id: Item.Id
    })
);
```

### T5: Nesting dolls

Say you have k containers nested inside each other with only one thing in each, like nesting dolls. Naive Allman gives every shoe its own line, so it uses 2k − 1 extra lines compared to K&R. That would be a real problem for deep nesting, and it is the fair objection to "always put every brace on its own line".

LAFS fixes this by grouping the dolls' shoes together:

```
Wrap
([{
    Item
}]);
```

Now the overhead is exactly 1 extra line, no matter how deep the dolls nest. The cost of deep nesting is gone.

### T6: Counting lines

Small blocks get squished onto one line in LAFS (the Monolith), and each squished block saves at least 2 lines compared to K&R, which spreads it over 3 or more. Each tall pair LAFS has to detach costs at most 3 extra lines. So LAFS uses fewer lines overall whenever the saved lines beat the spent lines. The proof turns that into a simple test: 2 × (squished blocks) > 3 × (detached pairs). You can run that count on your own code to see which side you land on.

The comparison above is against K&R as it's usually written. If K&R also squishes small things (the fair-fight version), the squishing cancels out and K&R keeps a small lead in lines on tall containers. T3 is what prices that lead.

### O1: K&R's own book breaks its own rule

This is a fact about a book, not a counting result. In *The C Programming Language* the function's opening brace goes on its own line, like LAFS. The braces of `if`, `while` and `for` hug the head, like K&R:

```
int Max(A, B)
int A, B;
{
    return A > B ? A : B;
}

while (Count > 0) {
    Count = Next(Count);
}
```

The usual explanation is historical: old C put the parameter declarations between the `)` and the brace, so the brace couldn't hug. That reason is gone in modern C, but the exception stayed. So K&R already uses LAFS-style placement for its biggest containers, and it has two rules where LAFS has one.

## §3 The real numbers

I took four snippets, wrote each in K&R (with the fair-fight tricks) and in LAFS, and measured them with a script. Both versions of every snippet use the exact same tokens, so only the whitespace is different.

- **A:** a function with a comment and an if/else. The comment forces the function open. The if/else fits on one line, so LAFS squishes it.
- **B:** an array passed to a function. LAFS groups the `([` shoes.
- **C:** a function call with a callback. LAFS puts each argument on its own line.
- **D:** the pile-up example from T4.

Here is A, so you can see what's being compared:

```
// K&R: 9 lines, sideways gaps of 74, 22 and 7
int ProcessAccountRecord(Account_Record *Record, Process_Options Options) {
    // Archived records are read-only: never update them
    if (Record->IsActive) {
        Update(Record);
    } else {
        Archive(Record);
    }
    return 0;
}

// LAFS: 6 lines, sideways gap of 0
int ProcessAccountRecord(Account_Record *Record, Process_Options Options)
{
    // Archived records are read-only: never update them
    if (Record->IsActive) { Update(Record); } else { Archive(Record); }
    return 0;
}
```

| Case | Lines in K&R | Lines in LAFS | Total sideways gap in K&R | Total sideways gap in LAFS |
|---|---|---|---|---|
| A | 9 | 6 | 103 | 0 |
| B | 6 | 7 | 20 | 0 |
| C | 8 | 10 | 18 | 0 |
| D | 4 | 8 | 108 | 0 |
| All four | 27 | 31 | 249 | 0 |

What to take from it:

- LAFS used **4 more lines** in total (31 against 27). It lost lines in B, C and D, and won them back only in A.
- In exchange, the sideways gap went from **249 columns down to 0**.
- These four snippets are *illustrations*, not a sample of real code, and they were not picked to make LAFS look good. Three of the four cost LAFS lines.
- If you delete the comment from case A, the whole function squishes onto one line that is 155 characters wide, which is 1 line against 8.

## §4 What we choose to believe

Counting can't tell you whether sideways walking actually bothers a human reader. For that, the proof leans on three beliefs.

**P1: eyes like knowing where to look.** Finding something in a spot you already know is easier than hunting for it where it might be. In LAFS the shoes always stand on the indent staircase. In K&R they could be anywhere along the line.

**P2: things standing in a line look like one thing.** Two shoes in the same column, one at the top and one at the bottom, look like a pair, the way two goalposts look like a goal. This is the old Gestalt idea of closure and symmetry.

**P3: code is read much more than it is written.** A storybook is written once and read a hundred times. So a saving that is collected every time someone reads beats a cost paid once while writing. The proof says this is a common rule of thumb, not a measurement.

If you accept those, three things follow:

- **C-1, matching is easier.** In K&R finding a partner takes two steps: go to the right row, then hunt along it for the hidden shoe. In LAFS you stop at the first step. And K&R's second step gets longer with the head.
- **C-2, pile-ups cause mistakes.** Lines like `}));` make you count, and counting is where mismatches happen.
- **C-3, the saving adds up.** Every reading collects the saving, while the extra-line price is capped at three lines.

**Why "lining up comes first" (A1).** A1 is the choice that lining up the shoes comes first, and saving lines comes second. It's like cleaning your room: neat first, then see how much space you saved. The proof gives two reasons this is sensible, even if you'd trade lines for neatness at some rate:

1. **The prices are lopsided.** The extra-line price is capped at about three lines. The sideways walk it removes has no cap. So A1 only costs a few lines on short heads and costs nothing on long ones.
2. **You can get lines back but not the gap.** Squishing small blocks and grouping nesting dolls win lines back. Nothing can ever win K&R's gap back (T1).

## §5 "Worse is Better"

**5.1, what the idea says.** It's an old essay about two ways to build things. One says make it simple to *build*, even if it's less correct, because simple things spread fast. C and Unix are the usual examples. The idea is about what *survives*, and the author later wrote replies disagreeing with his own idea.

**5.2, formatting has no "correct" to give up.** The idea trades being easy to build against being correct. Layout can't make that trade, because every way of arranging the same code means exactly the same thing to the computer. It's like writing the same sentence on different paper. So "worse" can only mean "harder on the reader", and "simpler" can only mean "easier for the person building the tool".

**5.3, the bridge and the toll.** Building a better formatter costs something once, like building a bridge. Reading costs something every time, like paying a toll on every crossing. If crossing the bridge saves you anything at all, then with enough crossings the bridge pays for itself. The proof states this as a simple inequality, and it holds as soon as the number of readers times readings gets big enough.

**5.4, switching is not that hard.** The idea says a worse system gets stuck because switching is expensive. But changing layout doesn't change what the code does. A tool can tidy a whole project in one go, like a robot tidying every toy box at once. The one mess is the "who changed what" history, and Git has a feature (`blame.ignoreRevsFile`) that hides a tidy-up commit from that history. Switching isn't free, but it's a one-time job a machine can do.

**5.5, popular isn't the same as good.** Even if you reject all of the above, the idea only says the worse design *can* spread. K&R being everywhere fits that. But a snack being sold everywhere doesn't make it the healthy one. The geometry results in §2 don't mention popularity at all.

So the conclusion has two halves. If someone says "K&R is popular, so it's better", that's mixing up *surviving* with *being easy to read*. If someone says "K&R is popular because simple things spread", that's fine, and it explains the popularity without justifying it.

## §6 Where the proof is weaker

The proof lists five places where the strong "never" gets weaker. This is on purpose, so nobody can say it was hidden.

**B1, locked doors.** Some languages (Go is the main one, and Python at the top level of a file) won't let the left shoe move to the next line. That door is locked. There LAFS uses Hybrid: the left shoe stays at the end of the head line and the right shoe slides over to stand under it. That keeps the gap at zero, but everything inside shifts to the right. If it shifts too far, the specification itself (its H6 rule) goes back to a K&R-shaped layout. So in those languages, the honest claim is only that K&R is never *preferred* when the door is open.

**B2, short heads.** For a short head like `else` or `try`, the sideways gap is small, and LAFS's extra lines can cost more than the gap saves. A1 says to pay anyway, but someone who doesn't accept A1 can keep K&R for short heads.

**B3, squished things unfold.** LAFS squishes small blocks onto one line. When one grows too big, it suddenly unfolds from 1 line into many, and the change shows up as a big difference in version control. K&R, which doesn't squish blocks, never has that jump. The limit of 4 statements makes it smaller but doesn't remove it.

**B4, everything already assumes K&R.** Editors, linters, build checks and new teammates all default to K&R-shaped output. Switching is a real, one-time cost, even though a machine can do it.

**B5, nobody ran the experiment.** The counting results (T1 to T6) are solid. The beliefs about human eyes (P1 to P3) are beliefs, and the proof cites no study. If those beliefs are wrong, the counting still holds as geometry, but it stops being a reason that readers should care.

## §7 How you could prove it wrong

A claim that can't be wrong isn't really a proof, so the proof describes the experiment.

1. Take real programs and write each one in both styles. Because the tokens are identical, the only thing changing is where the shoes stand.
2. Ask people to do three jobs: find the partner of a marked closing shoe, check whether a block contains a certain line, and spot a planted mistake in a pile-up.
3. Time them and count their mistakes. Test different nesting depths and different head widths, and mix up which style each person sees first.
4. Predictions: LAFS should be faster at finding partners, and the longer the head, the bigger the gap. Flat containers should show no difference. K&R might be faster at checking a block at shallow depth because it uses fewer lines. And K&R should get more mistakes as more closers pile up.
5. If LAFS isn't faster, or the gap doesn't grow with longer heads, then P1 and P2 are wrong. The counting stays true, but it would no longer be a good reason to prefer LAFS.

## §8 The scoreboard

Here is every claim in the proof, with how sure we are:

| What it says | How sure? |
|---|---|
| K&R's sideways gap is the head's width plus one, and LAFS's is zero | Proved by counting |
| K&R can't get to zero without turning into something else | Proved by counting |
| LAFS lines head, left shoe and right shoe up in one column | Proved by counting |
| LAFS pays a fixed price and K&R pays a growing one | Proved by counting |
| LAFS closers mirror their openers, and K&R's can pile up | Proved by counting |
| Grouping nesting dolls costs 1 extra line however deep | Proved by counting |
| LAFS uses fewer lines when saved lines beat spent lines | Proved as a test you run on your own code |
| K&R's book puts function braces on their own line | A fact about the book |
| Matching is easier, pile-ups cause mistakes, savings add up | Only if you believe P1 to P3 |
| K&R is never best where the left shoe may move | Follows if you choose A1 |
| "Worse is Better" doesn't make K&R better | Argued in §5, not counted |
| The places where the claim gets weaker | Listed in §6 |

For the exact statements, proofs and numbers, read `superiority-proof.md`.
