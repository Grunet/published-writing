## Table of Contents

- [What is a Change Plan?](#what-is-a-change-plan)
- [What Benefits Do Change Plans Bring?](#what-benefits-do-change-plans-bring)
   * [Force You To Think](#force-you-to-think)
   * [Peer Review](#peer-review)
   * [Drive Clarification](#drive-clarification)
   * [Facilitate Discussion](#facilitate-discussion)
   * [Discoverable](#discoverable)
   * [Auditable](#auditable)
- [A Change Plan Template in Detail](#a-change-plan-template-in-detail)
   * [Summary](#summary)
   * [Impact](#impact)
   * [Security](#security)
   * [Communication Plan](#communication-plan)
   * [Test plan](#test-plan)
   * [Before the change](#before-the-change)
   * [Steps](#steps)
   * [Monitoring](#monitoring)
   * [Backout plan](#backout-plan)
- [Takeaway](#takeaway)

## What is a Change Plan?

A change plan is a document describing a plan to make a nonstandard change to a production environment. For example, manually making changes to hand-curated virtual machines.

The outline of a change plan might look something like this

- Summary
- Impact
- Security
- Communication Plan
- Test Plan
- Before the change
- Steps
- Monitoring
- Backout Plan

Before diving into each of these sections, let’s discuss why change plans are helpful to begin with.

## What Benefits Do Change Plans Bring?

There are several distinct benefits change plans bring to the table.

### Force You To Think

Like any template, a change plan template prompts you to consider each section carefully, determine whether or not it’s applicable for this change, and then fill out the section if so.

Without a change plan template, it can be easy to forget important aspects of a change, e.g. having a backout plan.

### Peer Review

Just like for code review, peer review of change plans can be powerful. Not only do you get extra scrutiny of the plan, but you can also get bidirectional knowledge sharing between the participants.

Without a change plan, there is no knowledge sharing and there’s increased risk of the change going awry.

### Drive Clarification

The steps of a change plan need to be detailed down to a point where anyone could follow them. This forces any ambiguous language to be clarified, increasing the likelihood of the steps being followed correctly and with the correct outcomes.

Without a change plan, the steps might be determined on-the-fly and may result in mistakes being made.

### Facilitate Discussion

With a change plan external participants can comment on and engage in discussions about the change. For example, a Product Manager might request the date of a change be moved since it overlaps with a big feature release.

Without a change plan there’s no artifact to structure discussion around, and worse external stakeholders might not be aware a change is happening at all.

### Discoverable

With a change plan, the change exists in documented history and can be examined by people in the future. For example, people trying to make a similar change might review and learn from it.

Without a change plan, the details of the change are lost after it’s performed and no one other than the performer knows about it.

### Auditable

A change plan can serve as an artifact for auditors to confirm that you’re following change management procedures correctly.

Without a change plan some other artifact needs to be created for auditing purposes.

## A Change Plan Template in Detail

What follows is an example of a change plan template used at a previous job of mine.

### Summary

A few sentences to let the reviewer know what are we doing and why are we doing it.

### Impact

How many customers or internal users will be impacted if things go right? How about if things go south / pear shaped / blow up?

### Security

This change increases | temporarily decreases | decreases | has no impact to security. Another sentence or two to justify why temporarily or permanently decreasing security is a good idea.

### Communication Plan

How are you going to communicate to internal users, support, etc. that the change is happening? Consider that if the change impacts users or causes downtime, you may need to communicate the change weeks in advance.

### Test plan

If this is a high risk or complex change (or not easy to back out), how are you going to test this first? If you are not going to test it first, justify it was either easy to back out or otherwise low risk. A reviewer might have suggestions of what needs to be tested or how to test.

### Before the change

What steps are you going to take to prepare for the change or stage or test things before you do the change?

### Steps

Step 1

Backup or save current state…

Step 2

Do something

Run a command:

this is a command that you run that you will copy/paste during the change

Test that the change worked!

This is some expected output you should see

If it didn’t work, do the backout steps, try again… be specific as to what happens if things go wrong.

Any follow up or cleanup steps

### Monitoring

What can we monitor to know that this change worked?

How can check for unexpected side-effects we may have on the application?

What other parts of the application could be affected by this change?

### Backout plan

The same as Steps but specifically how you would back out changes. If you can’t back out the change, note it here.

Undo some stuff

Undo some other stuff

Check that the undoing worked

## Takeaway

Change plans are an excellent tool to help you manage your ever changing production environments. While they may seem like pure overhead at first, they shine when faced with uncertain, complex, or risky changes.