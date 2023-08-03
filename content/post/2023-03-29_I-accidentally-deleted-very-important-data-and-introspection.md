---
title:       "I Accidentally Deleted Important Data... What Happened and Introspection"
subtitle:    ""
description: "An introspection on my recent work fault"
date:        2023-03-29
author:      "Marco"
image:       ""     
categories:  ["General"]
---

## What Happened
A few days ago in my job, I was patching a bug that touches fairly fundamental level of our code in a production site.
Normally, the process is we will wait for our changes to be approved by our peers and then one of us will jump on the production
site with the bug and patch it, so instead of a brand-new release, customers get their specific bug fixed straight away. And 
to be honest, we have patched numerous bugs throughout the year with many customers and everything has been fine. So, that day
I was following the normal routine, got approval from the team, patched the code and restarted the app. I normally would like
to check if the patch is working fine and check the logs to confirm there is no drama. However, when I was checking the log
after my patch that day, I saw the logs was spammed with api update, in the form of 
`PATCH api/xxx {"IMPORTANT_DATA_A": null, "IMPORTANT_DATA_B": null }`
My first instinct was ... "this doesn't look right". I then checked our internal indicator, and saw everything went red.
Ah oh...

![I am scared](/scared-sponge-bob.gif)

No lie I am scared, frustrated and shocked. I quickly imagined what could happen, and scenes about developers delete the database and run away
floated in my mind and even worse, it was 4:50 in the afternoon when I saw this happened and I was about to finish the day. Eventually I settled down after 10 minutes,
I told myself this kind of thing will happen, and let's face it and try to minimise this issue as much as possible. Good news is
this production site is 16 hours behind us, so no one will likely to be on and check at least within 5 or so hours, so we should
be fine if we can get everything reverted before they get up and like whattttt... I looked through my changes and found out 
what went wrong and worked together with one of my senior colleagues and eventually got everything reverted after about 
an hour.

![relief](/relief-mr-bean.gif)

Roughly the steps were
- Revert the patch back to its original, so the reverted changes will not get deleted too
- Put together some SQL query and revert data back from the audit log
- Check if everything has reverted, with the help of the log and some SQL queries

## Introspection
I guess this kind of things will happen, so it's probably okay and was a perfect experience for me to improve (thought probably
too costly) Hopefully, at least I will be calmer next time and get the issue fixed ASAP.

So what have I learned / can be improved

1. Django auditlog definitely helped a lot on the reverting process, it keeps a track of api setting an item from A -> B and that was
how we managed to restore the deletion (luckily all the deletion happened from api instead of direct database deletion, which
probably can only be restored if we keep a database snapshot or some other way I am not aware of)
2. Previously, the patches were mainly targeting some high level, or some separated code for customers. Next if we were
going to patch something at a fundamental level, we should be more careful, and setup some tests locally and make sure
there is no weird behaviour before patching
3. This things happens... I will need to calm down and face it. This time I was very lucky to get help from a senior colleague and
mentored me on the revert process even after hours. I was much more relaxed and confident
4. Practice on my SQL skills - looking at the SQL query we put up for the revert process, I reckon I should be able to 
work out, but probably will take a much longer time
5. How can we prevent this from happening on the internal system side? I guess there is no simple way to do this, but 
I would raise this up with the team if anyone has good suggestions