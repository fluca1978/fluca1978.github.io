---
layout: post
title:  "Developers On Fire Interview"
author: Luca Ferrari
tags:
- opensource
- interview
- postgresql
- university

permalink: /:year/:month/:day/:title.html
---
On October 22nd I was interviewed by Dave Rael for "Developers On Fire" podcast.

# Developers On Fire Interview

I was contacted by Dave Rael, who is the author of [Developers on Fire podcast](https://developeronfire.com/podcast/episode-449-luca-ferrari-focused-practice). He is a very smart and kindly guy, and I think the work is doing with his podcast is simply amazing.
<br/>
<br/>
He told me someone suggested my name as a possible target for the podcast, and I was really proud about it. However, I was too busy back at that time, so I kindly asked Dave to delay the interview at the end of October, and he agreeded.
<br>
<br/>
So here it is, **[my very first podcast interview](https://developeronfire.com/podcast/episode-449-luca-ferrari-focused-practice)**: I found myself talking about a lot of different things, with particular regard to achery and university, but was not prepared at all. I'm quite happy with the fact I was really spontaneous.
I hope someone can enjoy the interview, and I hope to have another chance in the future to be on another episode.
<br>
<br>
<br/>
<center>
<a href="https://developeronfire.com/podcast/episode-449-luca-ferrari-focused-practice" target="_blank_">
<img src="/images/posts/developersonfire/developersonfire.png"  />
</a>
</center>

<br>
<br>
In the following, you can find some parts I would like to share about this experience.


## The UTC Problem

A few days before the interview I asked for confirmation and we had a little discussion about the difference of timezones. Dave is located at `UTC-6` and I'm in `Europe/Rome`, so I queried PostgreSQL to show me the times. That lead to some wrong results:

```sql
template1=# select 
  '2019-10-22 16:00:00' at time zone 'Europe/Rome' as my_time
, '2019-10-22 16:00:00' at time zone 'America/Denver' as what_should_be
, '2019-10-22 16:00:00' at time zone 'UTC-6' as utc_minus_6
, '2019-10-22 16:00:00' at time zone 'UTC+6' as utc_plus_6;

-[ RECORD 1 ]--|--------------------
my_time        | 2019-10-22 16:00:00
what_should_be | 2019-10-22 08:00:00
utc_minus_6    | 2019-10-22 20:00:00
utc_plus_6     | 2019-10-22 08:00:00
```

As usual, I was not aware of how well PostgreSQL is behaving and [thanks to a quick reply by Tom Lane](https://www.postgresql.org/message-id/12910.1571580645%40sss.pgh.pa.us) I learn something new: POSIX use positive offset for west timezones while ISO use negative ones. 
<br/>
*Sounds to me people doing POSIX and ISO don't agree on how the earth spins!*
<br/>

## Before the Interview

I have to admit I was a little nervous about the interview, mainly because I never did a real interview before. Well, I did an interview once for a local television when, as a fourteen archer, I won a competition. Since then, I never did an interview anymore.
<br/>
<br/>
Dave has been a great host, letting me to feel very comfortable and exeplaining to me how that was going to be. I really enjoy his attitude and way of *making the interview a real conversation between friends*, and that's how I felt actually: as I was speaking to a very old friend in front of a beer (or two).

## The Interview

Having set up the environment, the interview begins and again, as I wrote before, **it was much more like talking to a friend instead of doing a real interview**, and credits for this to Dave who is a real smart and friendly guy, and that means he is also a great host.
<br/>
<br/>
What I appreciated the most was the interest Dave had in listening to me and my stories, that by contrast I found by myself quite boring (but that's probably my biased opinion).

## After the Interview

After the interview we spend a few minutes talking about the result, when and how it will be published and usual greetngs and thanks.
<br/>
I ended up the whole thing with a feeling of calm and happiness.
