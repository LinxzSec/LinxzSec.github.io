---
layout: post
title:  "OSCP Experience & Tips"
categories: [Non-technical]
tags: [OSCP, Hacking, Courses, Certifications]
draft: false
---

# OSCP Experience

Hey - as most of you know I just recently passed my OSCP and I thought I'd be like everyone who does OSCP and make a blog post about my experience with both the exam and the course.

## PWK Experience

So, I was originally going to buy 60 days of lab time however I decided in the end that I would only buy 30 days, and to be honest, I'm glad I made that decision. I spent about 6 out of the 30 days not in the lab so let's say I spent 24 days in the lab, in that time I successfully rooted 30 boxes which is not bad going; I could've/should've done more perhaps but I did not want to burn myself out before the exam so I did a box or two a day.

I had a fair amount of experience going into the labs, I had already come very close to being Elite Hacker on HTB two times, which although it is not massively impressive considering I did some of the harder HTB boxes I was fairly confident in the lab, especially after seeing the difference in difficulty, I had also done various VulnHub machines in the run up to receiving my labs as well as taking advantage of [the list of OSCP like HTB boxes by TJNull](https://www.reddit.com/r/oscp/comments/alf4nf/oscp_like_boxes_on_hack_the_box_credit_tj_null_on/).

As I mentioned, the difference in difficulty between HTB and the PWK labs I was not expecting at all, the PWK labs were considerably easier than most of the boxes I'd done on HTB (minus a few) which gave me greater confidence. I will say that I, like others originally thought that the OSCP was some kinda' expert pen-testing level cert let me say; it's not! It really is ultimately an entry level pen-testing cert. I cannot go into too much details about the boxes however I will say that the list I mentioned earlier by TJ and the lists of VulnHub machines that are like PWK/OSCP machines is a really close fit so if you are nervous and want some extra practice, for sure you need to hit your head on those for a while.

## PWK Tips

1) Set a realistic time frame for yourself.
   I did 30 days because I used 16 days of holiday I had accumulated at work, if you mix this in with all the weekends I was pretty much not at work for an entire month, I had effectively 18 hours a day (most days) to study for this, I knew I had time and that's why I went with 30 days, not to mention I was fairly confident in my ability deep down (even though my surface level imposter syndrome suggested otherwise). So, if you have any other commitments and cannot book such a sheer amount of time off work like I did; really think about how much effort you can put in because I won't say this course is effortless. Did I think it was easy? Yes, I did think it was easy; it was easy but it still takes **time** & **effort** so set a realistic goal for yourself.

2) Watch all of the videos & read the PDF.
    A lot of the problems you will face in the labs is shown in significant detail in the course material and you don't appreciate that until you come to a box and go "oh, that was in the guide almost perfectly" that is what the guide is there for! I will admit the videos & PDF are boring, yes its boring, I really dislike learning that way but it was worth it for whenever I got stuck.

3) Read the writeup for the box Alpha on the forum.
    This lab box is given to you and you cannot use it in the lab report anyways (if you were planning on doing a lab report) so you may as well look at someone elses methodology, see how it compares to your own, etc, and get a feel for what the PWK boxes are like before you jump in.

4) If you're only going for 30 days lab time - don't do the lab report.
   Honestly, it's a waste of time when you only have 30 days. Wasting 4, 5, 6 days on the lab report for 5 points is just not worth it. I did not do a lab report and I relied soley on my skill alone to get the necessary points. If you feel that you really are weak and need a lab report to pass then I highly recommend going with 60 or 90 days instead of 30. I think if you're doing 30 days but relying on the lab report to scrape you by then you are out of your comfort zone and need to read tip numero uno of this post.

5) Don't read so many blog posts about peoples experience.
    I read A LOT of these in my run up to OSCP and I wish I hadn't because I psyched myself out and convinced myself this exam is harder than it actually is (I know this is subjective). If you feel you **must** read blog posts or **need** to read blog posts then take **ALL** of them with a pinch of salt (including this one).

## OSCP Experience

Going into the exam I had a bit of an awkward time, my exam started at 20:00 which of course is not that friendly especially given that it was on a Friday so I had to book the day off work. In the end I went to bed at around 7:00 on Friday morning (from staying up all of Thursday night) and I then slept until 15:30. I must admit waking up way too early for my exam put me on edge abit because I was worried I would not be able to make it through (spoiler: I did).

Before my exam began I decided instead of trying to last minute panic study I would chill out, I still used the computer but I mostly just did what I do best; shitpost on the Internet, procrastinate like an absolute God and more or less just "wasted time".I will be the first to say though that this procrastination and "time wasting" was something I appreciated so much in the exam because by the time it got to 20:00 I was fired up & ready to hack all the things.

Exam started at 20:00 and by 20:30 I had already done the buffer overflow, this was 25 points and I was super fucking happy at this point and knew I could do this. I had started my scans on all the other boxes while I was doing the BOF so once I finished that I already had all my scan results. I took a 15 minute break at this point and then started on a 20 point box. I got user on this box within about an hour but spent an hour or so stuck on the privilege escalation so I took another break and when I came back I knocked out the 10 pointer. Again after finishing the 10 pointer I took a break and got the root on the 20 pointer that I had user on; this put me at 55 points in roughly 5 hours. I knew I had a shitload more time left so I took on the other 20 pointer, got user in roughly 2 hours and decided to go for an hour break as I knew I was now super close to having enough points to pass. By the time I came back I was 8 hours in, I spent 3 more hours getting the priv-esc on the 20 pointer at which point I then had 75 points, so 5 points more than I needed for a pass. Here is where I really slowed down. I took a break for around two hours. I came back and got user on the final 25 pointer in roughly 30 minutes (due to the fact I still had a Metasploit usage) and I knew this would put me at about 80-85 points I was at this point 13 and a half hours into the exam and I had a pretty comfortable pass - I spent the next 4 hours banging my head into the priv-esc of this final 25 pointer and even with Metasploit I just could not get it, I was now 17 and a half hours in, I was super tired and didn't want to do anymore hacking so I thought "fuck it, I've got enough points I will just start the report, but first a 45 minute nap!" I took the nap and when I woke up I spent the remaining hours doing my report, I ended my exam 30 minutes early once I knew I had everything and then I went to sleep for 6 hours.

When I woke back up from my 6 hour sleep, I had luckily nearly finished the report from the time I had left in my exam so I banged out the rest of the report in about 3 hours, proofed it & proofed it again and then took another nap for 3 hours. When I woke back up I proofed my report one more time and finally sent it off and alas 6 days later I received my email from OffSec congratulating me on my pass, I was so fucking happy (kinda wish I had a picture of my reaction, I woke up to the email it was quite nice.)

## OSCP Tips

1) Document all the things!
   I cannot even begin to explain how much this helped me for the exam. I had spent so much time perfecting my notes & their structure that when it came to the exam report (although tedious) it was a breeze to just copy everything into it - really, document everything and do it well. I used CherryTree as I mentioned and I took advantage of the template in [James Halls' wonderful blog post on the OSCP](https://411hall.github.io/OSCP-Preparation/)

2) Record the exam!
   This one sounds a bit excessive or "weird" perhaps, but actually - it was super useful. I don't like breaking my workflow when I'm hacking. Especially not for menaial tasks such as taking screenshots. So in order to ensure I didn't have to do that, I recorded both of my VM screens for the exam using OBS, this meant that when it came to documenting my exam I could simply take screenshots from the video I made!

3) Don't panic!
   I know this is a somewhat "obvious" tip but, don't panic especially about the time; you have **plenty** of time. Really, plenty. You do not need to be concious of the clock, 24 hours doesn't seem like a lot of time but if you can get the BoF down as fast as possible then you'll realise just quiet how much time you have.

## Final Notes

Overall, I don't have anything else to say, this was pretty much my entire experience of the entire process from start-finish (well, minus the all of the practice I did before doing the OSCP). I hope this is helpful for someone looking to do the OSCP. I appreciate that there are **a lot** of blog posts out there surrounding this exam & the process however, I guess I just wanted to give my take on things because I'm an opinionated twat :D Thanks for reading! 