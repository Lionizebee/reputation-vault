# ReputationVault — Presentation Script

*Read this out loud a couple times before recording/presenting. Pause where
you see (pause) — it helps you sound confident instead of rushed. Total
time: about 90 seconds for the main pitch, plus backup answers below.*

---

## The 90-second pitch

Hi, I built ReputationVault — a treasury vault that pays contributors
automatically, based on what they've actually done, not a fixed salary.

(pause)

Here's the problem I'm solving. Most DAOs and startups pay people one of
two ways: a flat salary regardless of effort, or one admin manually
deciding who gets what. Both are broken. Flat pay rewards inactive people
the same as your best contributor. Manual routing doesn't scale, and it
puts trust back in one person's hands — which defeats the point of
building on a blockchain in the first place.

(pause)

So here's how ReputationVault works. When money comes into the vault, it
instantly splits itself three ways, automatically — no human touches it.
Seventy percent goes into a contributor pool. Twenty percent goes into an
operational account the admin can spend anytime. Ten percent goes into a
locked emergency fund.

(pause)

Contributors earn reputation points when a completed milestone gets
logged — shipped a feature, hit a goal, whatever matters to that team. If
someone goes quiet, their points slowly fade. Then, roughly once a month,
the contributor pool pays out automatically — split based on everyone's
current reputation. Nobody approves it. Anybody can trigger it. It just
happens.

(pause)

And the emergency fund has real teeth — releasing money from it requires
multiple trusted signers to agree, and a mandatory waiting period. No
single person, not even the admin, can drain it alone.

(pause)

This is live right now on the Stacks testnet — five smart contracts,
fully deployed, and tested 36 different ways to prove the math and the
rules actually hold up. I'll pull up the explorer link so you can see it
for yourself.

---

## If they ask: "Why is this different from a shared bank account?"

A bank account doesn't know who did the work. This vault calculates that
automatically, every payday, based on real contribution history — nobody
has to decide it by hand.

## If they ask: "What stops someone from stealing the funds?"

Three separate locks. Regular spending needs the admin's own signature.
Contributor pay only moves through the automatic reputation calculation —
no one can just take it. And the emergency fund needs several trusted
people to agree, plus a mandatory wait — even the admin can't move it
alone or in a hurry.

## If they ask: "Is this actually working, or just a concept?"

It's live on testnet right now — not a mockup. Here's the link.
*(pull up: https://explorer.hiro.so/address/ST25HBNK2PKDND65BXTGV8AP6NHE967GFS7KGDVYG?chain=testnet)*
Five published contracts, real transactions, and a full test suite —
36 tests, all passing — proving the split math, the payout math, and
every access rule actually behave correctly.

## If they ask: "What's the actual innovation?"

Pay isn't a fixed number. It's alive — it goes up when you contribute and
fades when you go quiet. That's a genuinely different financial behavior
than "here's your paycheck," which is what almost everything else in this
space still does.

## If they ask: "What's next / how would you extend this?"

Three directions: letting other apps reuse the same reputation score for
DAO voting power, not just pay. Making the decay curve configurable per
DAO. And eventually replacing the trusted "reporter" who logs milestones
with an on-chain oracle, so milestone verification is fully trustless too.

---

## One-line answer if you're put on the spot and blank

"It's a vault that pays people based on what they've done, not a fixed
salary — and it's already live on testnet, so I can show you instead of
just telling you."
