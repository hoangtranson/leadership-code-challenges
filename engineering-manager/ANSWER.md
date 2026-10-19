# Part A - Your 90-day plan (max 3 pages)

1. **Diagnosis** - what is actually wrong here? Use the data above; we are looking for root patterns, not a restatement of the tables.

I see 5 patterns here:

- Money correctness depends on people noticing, not on guarantees
    - Incidents #1 (no idempotency), #2 (connection pool), #4 (float arithmetic) and 3 of #5–9 are all classic money-system mistakes that tooling and design rules should make impossible.
    - Code review is the only safety net, and it is leaking.
- Dependence on single people's knowledge
    - S. (wallet) and T. (pipeline) each have a bus factor of 1: they hold critical knowledge, which is risky if they resign, go on holiday or get sick. For example, incidents #10 and #11 took longer to resolve because T. was away.
    - "S.'s reviews make juniors cry": this stops juniors from learning.
- Launching a region is a one-off craft job, not a repeatable process
    - Incident #3 at the region-2 launch: "Config copied from region 1 with wrong provider credentials".
    - Region 3 took 4 months to launch instead of 6 weeks.
    - The post-mortem never happened, which means region 4 may repeat the same mistakes unless we create action items for them.
- Heavy dependency on QA
    - QA finds regressions late, so releases slip. Bigger, slower releases then carry more risk and produce more regressions.
    - The bug backlog grew 30%.
- No learning loop
    - No delivery metrics exist beyond a burndown chart nobody trusts.
    - There is no incident process and there are no post-mortems.
    - It is difficult to tell whether each sprint is getting better.

2. **Priorities and trade-offs** - you cannot fix everything while launching region 4. What do you do, in what order, and explicitly: what do you *not* do this quarter?

We have 2 important goals:
- Launch region 4 in 10 weeks
- Money-impacting incidents to zero

We have 90 days -> 3 months -> 12 weeks -> 6 sprints (1 sprint = 2 weeks)

- Week 1
    - Work with the teams and hold 1:1s with key members of each squad to understand what is going on, why the incidents and the delay in launching region 3 happened, and to collect any valuable feedback.
    - Review the product documents and try out the platform's key features.
    - Work with the CEO to see whether I can negotiate the 2 goals above
        - what we need to do to complete both goals
        - if I cannot achieve both, I can commit to "Money-impacting incidents to zero" and see whether I can cut scope from the region-4 tasks so we can still launch it in 10 weeks

- Weeks 2-3: stop the money bleeding
    - Audit every provider callback and payout path for idempotency.
    - Add a lint ban on floats in money code, or store amounts as integer minor units (cents) and do the math with a library like bignumber.js.
    - Build a daily automated reconciliation job (ledger vs. provider vs. wallet balances) with alerts
        - Ledger vs. wallet: for every wallet, the sum of its ledger entries must equal its current balance. A mismatch means some code changed a balance without recording it, or recorded it wrong.
        - Ledger vs. provider: every deposit and withdrawal in the ledger must match exactly one provider transaction, by provider transaction ID and amount, and the other way round. A mismatch means a duplicate, a missing callback, a wrong amount or a failed withdrawal we think succeeded.
        - Totals: the sum of all player balances must equal the funds we actually hold for players.
    - Stand up a lightweight incident process
        - define severity or priority levels
        - define who an issue is assigned to, how, and which cases go to the backlog
        - while fixing, the engineer records the timeline, impact and root cause, and we find out why we did not catch it
        - track MTTR, review the process ourselves to improve it, and take feedback from people, because those who work closely with the product every day have more context than me
    - Hold a 60-minute region-3 retro and feed it into the region-4 plan
        - focus on the reasons rather than on who messed up
        - turn the action items into a checklist for region 4
- Weeks 2/3/4 - 11 (we may start earlier depending on team resources): use region 4 to make launches repeatable
    - define a checklist for launching a new region, with clear go/no-go criteria
    - talk to key engineers in the team to see whether we can set up the common parts of a new region as code, to save time on future regions
    - Run a staging dry-run
        - deploy from scratch
        - money flows end to end, with provider sandboxes
        - check that region-specific rules work as expected
        - rollback steps in case of incidents
        - measure the time for the whole process, and list specific cases with the actions to take and who to reach out to
    - Roll out in stages
        - internal and test brands first
        - 1 or 2 brands, then the rest
    - Improve testing for critical parts that are missing tests, especially payment-provider integration logic
        - request and response
        - the same callback twice
        - callbacks (webhooks)
        - timeouts and retries
        - signature verification
        - amounts and rounding
        - etc. The idea is to test all the cases that caused incidents.
    - Remove single points of failure
        - document something every sprint, such as how to deploy and how to launch a new region
        - pair programming and knowledge sharing. For example, S. runs knowledge-sharing sessions for the "Payments & Platform" squad.
        - L. can also run sharing sessions instead of supporting quietly
        - T. needs to share knowledge and have at least one backup for when T. is not available
    - Introduce end-to-end tests so we can make sure old features, like money flows, do not break, and run them during implementation and deployment instead of at the last minute

- Week 12
    - prepare for the region-4 launch
    - work closely with BA or QA to recheck all happy paths
    - prepare the environments to make sure no environment variable has a wrong value

3. **Process changes** - concrete and few. For each: what problem it attacks and how you'll know it's working.

- Incident process changes -> rising MTTR and no data such as root causes -> MTTR falls, and we can look at the system (Jira) to see, for a given period, how many bugs there were per severity and the root cause of issues such as "Wrong rounding on currency conversion for one brand".
- QA process changes -> bug backlog +30% quarter over quarter -> AI tools like Claude are speeding up how fast we ship code, so I think engineers will have more time to test their own features. I will propose strong test ownership by engineers and give the 2 QAs time to test features end to end to make sure there are no side effects. I will measure whether the bug backlog grows by less than 30% this quarter.
- Code review changes -> many money-related issues -> for this kind of change, I will create a label or something similar so the PR is highlighted to the team and needs at least 2 approvals, plus a PR checklist. I will measure it by money-impacting incidents staying at 0.
- Region launch playbook -> fixes incident #3 and the delay in launching region 3 -> config as code, a checklist for launching a new region, and a staged rollout. I will measure it by region-4 milestones hit as planned, and by the number of launch issues staying under about 5, with no major issues.

4. **Metrics** - what you will measure to know the org is getting healthier. Justify each; no dashboard shopping lists.

- Money-impacting incidents
- MTTR by severity
- Release frequency
- Region-4 milestones, planned vs. actual
- Time to first review / PR merge time
- Defect escape rate

5. **People** - using the roster: who needs what from you (coaching, scope change, protection, challenge), and what you do about the two open headcounts and the QA bottleneck.

- S. is staff-level and the only person who truly understands the wallet engine
    - He is the most critical person to keep and also the biggest risk.
    - Give: formal ownership of money-safety standards and of the reviewer pool, plus architecture authority on the region-4 payments work.
    - Challenge: direct, specific feedback that his review tone is hurting the team. Redefine his success as "others can do what you do".
    - Protect: he must no longer be the sole approver.

- D. and the 2 other solid mids
    - Develop them into second wallet experts.
    - Let them lead the region-4 payment integration, with S. as advisor. This is a stretch assignment with a safety net.

- The three juniors
    - Assign each of them a buddy/mentor to support them at work: a mid-level engineer, or L. from "Web & Backoffice", since L. seems to enjoy mentoring.
    - Do not allow them to touch money code without a pool reviewer.

- Tech lead T.
    - Close to burnout. T. gets an on-call rotation inside the squad, runbooks, and a frozen backlog intake agreed with Product.
    - Ask T. what to drop and what to delegate to other people.
    - The second open headcount goes to T.'s area.

- L. is our most experienced people-manager
    - The highest-trust leader. Formalize the cross-squad mentoring (recognition, and a possible senior or people-lead path).
    - Make L. a co-author of the process changes from week 1, because L. understands the team and the product better than I do (the best person I can go to for advice).

- Two open backend headcount
    - I will find out why they have been open for 3 months, for example a difficult technical interview, a slow recruitment process, or the roles not being treated as urgent, and then act on it.
    - The risk around S. remains even while we train D. and the 2 other solid mid-level developers as his backups, so we still need one strong backend engineer with payments experience, plus one backend engineer with pipeline experience to reduce the risk around T.

- QA
    - Developers own their tests. QA moves into automation and quality coaching, with one QA embedded in Payments.

- Me
    - First 2 weeks: 1:1s with all 22 people to understand the team before making commitments to the CEO.
    - Understand what the CEO needs from me and what outcomes I can bring to the team.

## Part B - The PR (max 1 page of review + ½ page answer)

### Write your review

I will request changes on GitHub for this PR and send a Slack message (our team communication channel) to the PR owner about my comments.

- Review 1

```
+  const members = await Member.findAll(); // line 78
```

The description says it takes accountId so each brand can be run separately => I think this needs a filter by accountId here; as written, this line loads all members of all brands.

- Review 2

```
const bets = await Bet.findAll({
+        where: { memberId: member.id, createdAt: { [Op.gte]: weekStart } },
+      }); // line 86
```

We query data by member ID and createdAt greater than or equal to weekStart. The issue is that nothing prevents paying the same period twice through a retry, rerun or timeout. There is no idempotency pattern.

- Review 3

```
wallet.balance = String(parseFloat(wallet.balance) + cashback); // line 106
```

I think this is a float money issue, and we need to use bignumber.js to handle it.

- Review 4

```
+      const wallet = await Wallet.findOne({ where: { memberId: member.id } });
+      if (!wallet) {
+        continue;
+      }
+
+      wallet.balance = String(parseFloat(wallet.balance) + cashback);
+      await wallet.save();
+
+      await CashbackPayout.create({
+        memberId: member.id,
+        amount: cashback,
+        weekStart,
+      });
+
+      await WalletTx.create({
+        walletId: wallet.id,
+        type: 'CashbackCredit',
+        amount: String(cashback),
+      }); // line 101 - 119
```

This is not atomic. We save the wallet, then create the CashbackPayout and the WalletTx separately. A concurrent deposit can be overwritten, and a failure midway leaves the balance and the ledger out of sync.

- Review 5

```
console.log('cashback failed for member', member.id, err); // line 122
```

We only console.log on failure, which hides real errors at runtime. A better approach is to use a logger or record a failed status here.

- Review 6

```
adminRouter.post('/cashback/run', async (req, res, next) => { // line 145
```

Do we need middleware to protect the admin endpoint?

- Review 7

I see an N+1 query problem inside `runWeeklyCashback`: we query all members, and for each member we run 1 query for Bet, 1 query for Wallet and 3 writes.

This means 10,000 members can take 50,000 database round trips on a POST to /admin/cashback/run, and the caller waits for the whole loop to finish.

I suggest using one aggregate query, running the payout as a background job with a dry-run mode, and having /admin/cashback/run respond immediately.

- AI review

I can also make mistakes, so if the company allows it, I will use the simplify command or a Claude code review to check the changes again and make sure no issues are missed.

### The conversation

There are critical issues in this PR, so I will message S. and D. privately first and book a meeting about it. (I will make sure S. and D. understand that it is not about blame; it is about team collaboration.)

During the meeting, I will ask S. "Walk me through how you reviewed this" to understand why this PR was approved, and ask related questions (including why the tests hid all the issues above, and S.'s ideas for making the tests reliable). Then I will make sure D. understands the issues, and have S. and D. pair program on the fix.

Lessons learned
- Add a checklist to the PR template for payment-related logic and N+1 queries.
- Use AI for code review.
- If engineers use AI for coding, I will ask them to define rules for it so this issue does not repeat with other developers.
- Run a sharing session so everyone understands this.

## Part C - Two situations (max 300 words each)

### C1

What I think about in the first minutes:
- Wake S. (technical lead) and the on-call engineer. Leave T. on leave unless the data pipeline turns out to be involved.
- My first thought is that money is only lost when users withdraw or bet. So I will check with S. whether we can pause these in region 2, and whether there are any recent related changes we can revert as a quick fix immediately. Then we keep investigating what happened the next day, when the whole team is available.
- Reply to the CEO about the actions we are taking right now and the next actions to fix it the next day.

Next 12 hours
- Check recent code changes and logs with S. and the squad to find the root cause.
- Inform the customer support team so they know about this issue and are aware there may be many complaints from players.
- Keep the CEO updated (even while the CEO is asleep).

Following 12 hours
- Ship the fix and verify it.
- Discuss with the team how to recover the money, the risks and the time it will take.
- If the issue takes a long time, I will ask S. to rest and hand over the shift to another person, with notes on what is not finished.
- Send a written summary to the CEO.
- Schedule a meeting with the team about this incident to agree actions that prevent it in the future.

### C2

I will stay calm and not get defensive. Instead, I will acknowledge L.'s point.

But we created this process based on real incidents, and it helps prevent them from happening again. I will ask L. to go through it item by item with me privately today, so it does not take up the whole team's time and people are unblocked to get on with their work, and to bring the items L. thinks are theater. Then we agree on the changes and I will publish them.

In the meeting with L., I will listen first, because L. knows the team better than I do. Then I will be clear about which points I agree and disagree with, and say that telling the squads to ignore a safety control is not acceptable, and what my expectations are. I will also give L. more ownership, because people trust L. Then in the next team meeting, I will announce the changes and credit L.
