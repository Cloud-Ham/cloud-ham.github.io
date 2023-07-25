---
title: "Competing for a Cybersecurity Role"
date: 2023-07-20T17:13:05Z
draft: false
---

- [Introduction](#introduction)
- [Getting into Cybersecurity](#getting-into-cybersecurity)
- [Competition for the Role](#competition-for-the-role)
- [Boosting your Chances](#boosting-your-chances)
  * [Get a Referral](#get-a-referral)
  * [Experience](#experience)
  * [Certs and Degrees](#certs-and-degrees)
  * [Sharpen and Focus your Resume](#sharpen-and-focus-your-resume)
  * [Volunteer Work](#volunteer-work)
- [What certs should I get?](#what-certs-should-i-get)
- [No Response from the Employer](#no-response-from-the-employer)

# Introduction

I spend a good amount of time in discords where people ask about what it takes to join a SOC, I figured it might be best to just write out all of my recommendations in one big post and ship it over. These recommendations are based on my experience between the two companies I've worked for in SOCs, and some of the ones I've done some light consulting for.

# Getting into Cybersecurity

One of the most common misconceptions out there is that because there are people needed in Cybersecurity, that hitting the bare essentials of requirements is enough to transition to a Cybersecurity role. And often, people consider that it could be their first IT job. This is possible, but unlikely for a couple of reasons:

* Cybersecurity is a very wide field with various sub-functions that you might be interested in. A SOC role tends to be a common starting point, but it doesn't have to be.
	* A SOC would expect you to know common terminology (IoC, MITRE ATT&CK, PCAP, AV/EDR, IDPS) and common SOC functions (SIEM/SOAR use, PCAP analysis, Malware analysis, threat hunting).
	* Application Security would be focused more on strengthening applications by limiting attack surfaces (containerization, converting IaaS to PaaS) and hardening the attack surface (Vulnerability management, security policies). 
	* Forensics would be more specialized in using proper tooling to build evidence and perserve the environment (Write-blocking tools, KAPE).

But the commonality between all of these is that to understand the Cybersecurity aspect (How do we defend/analyze this?), you have to understand what is operationally normal. An application that generates and provides overly-permissive and non-restricted SAS tokens at a very frequent basis to other applications could be totally normal for its use, as insecure as that might be. Or a website may write to the Apache log that each request was a 200 OK, even though the site directed them to a 404 page.

To secure, or to detect, a system or an event, is to know what is operationally normal. This is very hard to do in a homelab, and why most employers look for some previous IT experience before putting you in a Cybersecurity role.

# Competition for the Role

Now that we've cleared the basics, it's important to stress what happens when you apply. You update your resume, you walk through the company's hiring portal and fill in the fields. What happens now?

For starters, there *are* some systems in use that discard resumes that don't match certain criteria. I wouldn't be overly pessimistic about this happening as often as you may think, as with the wide scope of technology & systems, it'd be hard to account for when you wouldn't be a good candidate based off of keyword matching. For example, if you were hiring for a SOC Analyst with experience with KQL for Sentinel, you might search Sentinel on the resume. However, QRadar Suite (What used to be Cloudpak) can also run KQL queries federated to Azure. The analyst may not put Sentinel in their resume but still have familiarity with the language.

Ok, so after you make it past any possible resume dropping function, you are now in the resume list. If you are coming in without a referral, you lose some priority in this. In every instance I've witnessed, a referral is guaranteed an interview. More on this in the section "Boosting your Chances".

So now you're in the Battle Royale for the interview. If you are at the minimum requirements for the posting, your chances aren't great. But that shouldn't discourage you, apply anyways! You never know what could happen. You don't know what the employer has to review. But if you're applying for a remote role, your competition is most likely gigantic. They're some of the most desirable roles you can get.

# Boosting your Chances

Your resume is in the great pool of candidates. What can you do for yourself?

## Get a Referral

Mentioned above, this is huge. This tends to bypass a lot of requirements in getting a job (almost) anywhere. If the company trusts the employee submitting the referral and are on good terms, you're on track for at least an interview. 

Where can you get a referral?

* Go to a Cybersecurity Conference and meet people. People from all kinds of security roles from all kinds of companies will be there. Talk with them! Ask them what they think of their current job, and what might be required to get in. See what the options are. Some vendors may also be there specifically to recruit. Talk with them!
* Check with friends and family. This might seem basic, but I've had a decent number of people ask about how to get in somewhere, and later mention that they've got relatives in a company they'd be interested in working at.
* Look for job postings on Linkedin, specifically on your wall. Once you build out your professional network on LinkedIn, you should start seeing people promote positions on your home feed. Reach out to the person who posted it to see if you can get more detail or schedule a 30-minute chat. You may not get a referral out of this, but you will at least get some credit for being a self-starter.
* Go to a local game session. My recommendations are: Dungeons and Dragons sessions, Card game tournaments, or video game tournaments. Most of us in IT are nerdy and will be present. 

## Experience

You typically *need* experience in a prior company to have the type of experience that a company ma be looking for, but you can still drive your own experience and show your willingness to learn. 

My main suggestion here, until you get a role where you can develop professional experience in a subject, is to build a project site. Some ideas:

* Start a GitHub account. Work on projects, place it in a repository, make it public. Put your GitHub link on your resume. Or Gitlab, whatever works.
* Start a personal/tech blog. Document your work. What did you learn? What did you practice with?

I personally LOVE reading your projects on the resume, and I feel like it really highlights your efforts better than bullet-points on job descriptions.

## Certs and Degrees

I see the question of "Should I focus on a degree or certs?" quite often, and the answer is always: "Don't consider this the battle between the two. Consider both as qualifications." And even with that, experience is king. Real-world experience on systems and technology will almost always beat out the value of a certification or degree.

The biggest strength I see from a college degree is that you're a well-rounded, learned person. You have some writing skills, some math skills, and you were determined to make it through the schooling. Even if your degree isn't IT or directly Cybersecurity related, I'd give a fair amount of credit to it.

My issue with a college degree is that I don't really know what you learned. A Cybersecurity-focused degree between two different colleges fields two different results.

The biggest strength I see from certifications is the opposite of a degree. You didn't take classes on writing, math, or science here, but I can read the objectives list for the exam and be familiar with what you know about. And sometimes that translates directly into job functions. If we use Azure PIM on a daily basis and you have the Azure Security Engineer cert, I have a strong feeling you will know what to do when you need to claim a privileged Azure role.

Also with all of this said, it's worth noting that I do not have a degree and do not require any candidates I interview to have one. Or even certs. It's about putting together all of the applicant's capabilities between Experience & Qualifications.

## Sharpen and Focus your Resume

At where I currently work, the process is that our recruiter will send over a big batch of resumes and applicants to my manager. My manager will do the initial filtering, dropping people who are too green or too new to the field. He'll send me a shorter list and then I'll review and potentially request for some of them to be dropped as well.

Here are some reasons that the resume may get dropped, and although some of these may sound really picky, we're going through hundreds of them pretty often:

* Resume composition. A core component to our job is effective writing; short, clear and concise. If your resume is 5 pages long (For a non-government job), full of typos, broken formatting, it becomes grating to read. Aim for 1-2 pages, run spell check, and have some peers review with you. If you send me a copy of your resume, I will do my best to provide feedback as well.
* The resume has us asking a LOT of questions. You should hand your resume to a peer and ask them, "Do you know what I did at this company?" A lot of them talk about accomplishments but not the work or project asssociated with completing them. If you saved the company $20,000 in a project, that's good, but what was the project? Or if you list every programming language known to man in your "Skills" section and there's nothing that describes you ever using any of them, the resume doesn't tell us what you actually know or do.
* You aren't familiar with the topics on your resume. This has happened at least 4-5 times in my time with the current company. It makes you look like you're spamming words to bypass potential keyword filters when applying. Even if you had the best intent, please make sure you are ready to speak to points on your resume. You will most likely fail the interview otherwise.

## Volunteer Work

While this won't remove your need to have experience or qualifications, it is a boost to your resume. It tends to show a couple of key values of the human relationship: compassion, understanding, sympathy & empathy, and personal bonds. If you spend your time working with your community or participating in charity events, put that on your resume somewhere.

People underestimate the human bond & relationship needed in the field. No one wants to work with someone who is constantly detached, robotic, unkind, or disinterested. When you truly have a love for the human connection **and** the work required of the job, you will succeed the most.

# What certs should I get

This comes up a lot, and while certs aren't the only distinguishing factor to your skillset, it's probably worth just going over a handful. And these aren't necessarily recommendations for everyone. Consider what you're most interested in and pursue those.

* CompTIA Network+: Networking will follow you everywhere in IT, regardless of role. Whether you're a sysadmin, DBA, AppDev, SOC Analyst, Security engineer, Helpdesk analyst, etc. you will need understanding of networking.
* CompTIA Security+: Almost a pre-requisite at this point. It's very uncommon to get a SOC position with just this, but at Derbycon one year, there was a recruiter looking for people who met this requirement only.
* CompTIA CySA+: An extension of Security+. Easy enough to run through the material after Security+, but may not return much value.
* CompTIA Pentest+: My personal favorite to take. CEH and OSCP still kind of domiante the pentest cert area, but this was a lot of fun to take.
* CEH/OSCP: Including both of these in one as I haven't taken either of them, but they're commonly on pentest/red-team oriented job postings. Do some research to see if the roles you're interested in are looking for either of these.
* Cloud Certifications: Almost every company is using one or more public cloud providers. And each public cloud provider has their own certification path. For my last two companies, cloud experience has been a *must*. 

# No Response from the Employer

After you submit your application & resume, if things go quiet, I cannot stress this enough...

**Do NOT let it discourage you.**

There are a multitude of reasons why, at the company, they decided to pass on you and you will almost never know the answer. A couple of examples I've seen or heard from others in the industry:

* Department cutbacks occur during a hiring wave. Hiring for the spot is closed after opening.
* Internal politics leads to another internal candidate getting the spot.
* A referral came in from internal to the company, and the referred person gets the spot.
* There was another, more aligned candidate for the business needs.

Personally, I've had several situations where I've been either declined within 24 hours of applying, or ghosted after several communications. And funny enough the last time this happened, I ended up finding a better job than any of the other companies that I spoke with.

I want to reach back to that last point. It's not always what candidate is "more skilled" or "more experienced". A company may require experience exclusively with IBM QRadar products, and might turn down a SOC analyst of 30 years who has never used QRadar, for someone who's been an analyst for 4 years and is familiar with the entire QRadar suite. It's completely possible that you are skilled enough for a position, but it does not align with what the company wants.
