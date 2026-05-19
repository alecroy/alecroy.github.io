---
layout:     post
date:       2022-02-22 20:59:45 -0500 EST
title:      "No Timezones"
categories: Math
---

I often wish we just dropped time zones, and agreed to use UTC for everything.  This is widely regarded as a "bad idea".  I sympathize with that viewpoint.  Still, I think the need to coordinate time across time zones outweighs the need to transition days at midnight.

With timezones: local time tells you the local day because the day transitions at midnight.  Remote time is given by some offset, which should give you the remote day.  You need to keep track of 2 offsets (local to UTC, UTC to remote), but that's kind of unavoidable.

Without timezones: local time is remote time, both of which tell you the London day because London days run 00:00–23:59.  Local day might be half a day ahead or behind the London day, but that offset is something you can get used to because it won't change.

If an event starts at 17:00 on a London Tuesday, there's basically no way 2 people around the world could be off, unless they get the week wrong.  It might be a local Monday night, or a local Wednesday morning, sure.  But it's less confusing.

At the same time, I think we should drop Daylight Savings.  I feel like that's less controversial.  Timezone offsets rarely change apart from Daylight Savings, and those changes are set up to cleverly avoid crossing midnight.  Our clocks roll forward & back at 2:00a to avoid rolling back (or forward) the day.  Still, it causes more confusion than it's worth.  If businesses want to shift their schedule over the year, they can always do that.

In short, I think knowing *when* something occurs is more important than knowing on *which day* it falls.  If we have to choose between knowing the remote time or the local day (and I think we do), we should choose local time = remote time.

That's all.
