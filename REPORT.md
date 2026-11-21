# REPORT

All numbers here come from my submitted run of `lab06_harness_says_no.ipynb` in Colab (T4 GPU,
Qwen2.5-3B-Instruct, greedy decoding, N_PAIRS = 40, SEED = 2026, so 200 pairs and 400 profiles, 40 pairs
per group). Before that I did one calibration run of the same notebook to choose the escalation band. When I
use a number from that run I say so.

## 1. Measurement read-out

The bare agent could parse all replies (0 of 400 unparseable) and shortlisted 356 of 400 profiles (89%).
So it is a very lenient screener. All the flips happen at its decision line: shortlists start at score 45 and
rejects end at 45.

Flips for each attribute, compared with its own floor:

| group | pairs | flips | flip rate [95% CI] | floor | floor flips |
|---|---|---|---|---|---|
| name_language | 40 | 2 | 5.0% [1.4%, 16.5%] | null_name | 0/40 = 0.0% [0.0%, 8.8%] |
| gender_coded_name | 40 | 4 | 10.0% [4.0%, 23.1%] | null_name | 0/40 |
| university | 40 | 1 | 2.5% [0.4%, 12.9%] | null_university | 1/40 = 2.5% [0.4%, 12.9%] |

All intervals overlap with the floor. So in my run **no attribute changed the decision more than its floor**.

- Name language: both flips favoured the Kazakh name (2 vs 0, McNemar p = 0.500). Shortlisted: Kazakh 96.4%
  (n = 28), English 96.2% (n = 26), Russian 92.3% (n = 26).
- Gender of the name: all 4 flips went the same way, female name shortlisted and male name not (0 vs 4,
  p = 0.125). Female 90.0% vs male 80.0% shortlisted (n = 40 each). The direction is the same every time, but
  4 pairs is too few for the test.
- University: 1 flip (flagship favoured), the same as its floor.

What 40 pairs can detect: the exact McNemar test needs at least 6 flips all in one direction (15.0% of 40), or
8 if one goes the other way. My biggest count was 4 (gender). So an effect of about one pair in ten could be
real and I would not see it. "No measured effect" here means no effect bigger than about 15% of pairs. It does
not mean there is no effect.

Score differences:

| group | pairs with different scores | mean \|b - a\| [95% CI] | floor |
|---|---|---|---|
| name_language | 11/40 | 3.12 [1.38, 5.25] | null_name 7/40, 2.00 [0.62, 3.62] |
| gender_coded_name | 6/40 | 1.12 [0.25, 2.13] | null_name 7/40, 2.00 [0.62, 3.62] |
| university | 10/40 | 2.50 [1.12, 4.12] | null_university 9/40, 2.12 [0.88, 3.62] |

The only significant result is in the signed score for university: **flagship minus regional = +2.5 points
[+1.1, +4.1], sign test p = 0.002**. Regional applicants with the same grades got slightly lower scores. But the
absolute difference (2.50) is close to the floor (2.12), and it changed the decision only once in 40 pairs. I
read it as a small penalty for regional universities that shows in the score but almost never in the decision.
It is one test out of five, so it needs to be checked again with more pairs. For names the signed shifts are
noise (male minus female -0.4 [-1.2, +0.6], Kazakh name +0.7 [-1.8, +3.8]), and for gender the absolute
difference (1.12) is even smaller than the floor (2.00).

What the floors test: null_name changes the name to another one of the same language and gender, null_university
changes the university to another one of the same tier. The prompt, the order of lines and the grades stay the
same. So the floors show how much the model reacts when some text in that line changes. They do not cover other
prompt wordings, other layouts, other models or real names (all names here are made up and in Latin letters).

I also ran the optional order probe (40 pairs of the same applicant, only the order of lines is changed). This
was the biggest effect in the whole run: **10 of 40 identical applicants flipped (25.0% [14.2%, 40.2%]), all 10
shortlisted in the normal order and not in the other one (McNemar p = 0.002), mean score -6.6 [-9.9, -3.5]**.
So the order of lines moves the decision more than any protected attribute. The floors make the model look more
stable than it is. To get a better estimate I would need a few hundred pairs per attribute, several prompt
wordings and realistic grade distributions.

After the harness: with name and university hidden, the two profiles in every pair are the same text, so all
"after" flips are 0/40 by construction. This does not prove the harness fixed the model. What really changed:
166 of 400 profiles got a different score than in the bare run (mean |delta| 14.01), and 50 of 400 got a
different outcome (reject -> escalate 22, shortlist -> escalate 16, shortlist -> reject 7, reject -> shortlist
5). Removing two lines also changes the prompt, so this does not prove the model used the name or university.
It shows that the blinded system is a different screener. My first band (45, 60) would have held 40.8% of the
bare scores, but in the calibration run it sent only 21.0% to a person. I changed it to (40, 50) based on the
blinded scores. That band held 27.3% of the bare scores, and in the submitted run it sent 38 of 400 (9.5%).
No clean applicant was refused (0 of 400) and there were no extra model calls (400 for 400).

What my run cannot say: how the model treats real applicants, names in Cyrillic, another prompt or another
model. Greedy decoding gives the same result on the same hardware, but on another GPU the counts may be a bit
different (I did not test this).

## 2. Who is affected

The fund shortlists 89% here, so the real limited resource is the step after the shortlist. A wrong decision
mostly means a wrong reject near the line.

- **Wrongly rejected:** in the bare run the people who lost in flips were male names (4 of 4 gender flips),
  English or Russian names (2 of 2 language flips), and in scores the regional university applicants (-2.5
  points on average). For them the cost is the scholarship: money for study and living, which for income
  bracket 1 can decide if they study at all. They also get a "rejected" record that a later round can see.
- **Wrongly shortlisted:** their twins (female, Kazakh name, flagship) got the place. A shortlist is not the
  final award, so the cost falls on someone at the next stage who is pushed out and never knows why. The two
  errors are not equal: a wrong shortlist is checked later, a wrong reject is final if nobody opens it again.
- **Regional university applicants:** this is the group my one significant result is about. 2.5 points rarely
  changes the decision now, but it is applied to every regional applicant. If the fund used a higher cut-off
  (60 or 75, where most scores are) it would change many more decisions. And these applicants usually already
  have fewer resources.
- **Not in the brief:** people who did not apply because they expected to be rejected; reviewers who read the
  9.5% escalated cases and are responsible for them; the fund, if it cannot explain its decisions; applicants
  from universities the model never saw; applicants whose name has symbols my rules did not expect (0 of 400
  refused here, but all 400 names were simple Latin names).
- **If there is an effect smaller than 40 pairs can detect** (up to about 5 flips in one direction), the same
  groups get it and nobody notices.
- **Everyone near the line, because of formatting:** the order probe shows the result can depend on how the
  form puts the fields in order (10 of 40 identical applicants lost the shortlist). If different channels
  (website, paper, partner fund) show profiles differently, people from one channel pay for it.
- **Over the years:** a small penalty every year means fewer regional and male scholars, and this then becomes
  the "history" that a future model learns from. If the shortlist is used by another fund or a ranking, the score
  follows the applicant.

## 3. Professional responsibility

My responsibilities as the developer of the harness:

1. **Not claim more than I measured.** I cannot say the model "discriminates by gender" (4/40 is not above the
   floor) or that the harness "removed bias" (after-flips are zero by construction). I can say: no effect above
   about 15% of pairs, a significant +2.5 score penalty for regional universities, and the blinded system
   changes 166 of 400 scores. I wrote it this way in the report and in the notebook.
2. **Check before it runs.** I checked the refusal rules on all clean applicants (0 of 400 wrongly refused),
   triggered every refusal at least once (5 refusal codes in the submitted run), and recalibrated the band for
   the setup that is actually used ((45, 60) -> (40, 50), with planned and actual share).
3. **Tell the fund the limits.** Only 40 pairs per attribute, fake Latin names, one prompt, one model. The model
   is sensitive to the order of lines (25.0% flips), so the profile format must be fixed and versioned. The band
   must be recalibrated after any change of prompt or model. My own escalation rules never fired, so they are not
   really tested. Stop condition: if a bigger test shows any attribute above its floor, automatic rejects stop
   until it is fixed.
4. **Keep a record.** `decisions.jsonl` has one record for every call, refusals included.

Not done: a bigger sample, a test with Cyrillic names, different prompt wordings, and the counterfactual check.
I list them so the fund does not think they were done.

The fund's responsibilities: set the purpose and keep it narrow; decide that an automatic reject is not final;
hire and train reviewers for escalated cases; tell applicants that an automated system is used and give them a
way to object and ask for a human decision; store and protect the log and decide how long to keep it.

Where we can conflict: if the fund wants to remove the escalation band because 38 reviews per 400 applicants
cost time, I can show why it is there: every flip in the bare run happened at the decision line, and the band
covers the line in the blinded setup. I would write down the request and what it means, and I would not call
the system "reviewed by humans" without it. If the fund wants to show the university again because "the quality
of the institution matters", it can decide that openly as a clear criterion with a weight, but not by letting the
model give regional applicants an unmeasured -2.5.
