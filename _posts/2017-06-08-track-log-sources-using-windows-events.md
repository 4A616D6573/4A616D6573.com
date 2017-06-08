---
layout: post
title: Track Log Sources Using Windows Events
---

## Introduction

This post will show you how to alert or report on the creation and deletion of computer accounts within an environment by utilising [Windows Events](https://msdn.microsoft.com/en-us/library/windows/desktop/aa964766.aspx).

---

## Use Case

While most System Information and Event Management (SIEM) software today offers automatic detection and integration of log sources, it can be useful to have a secondary process in place to keep it in check or just have an analysts eyes cover the events to gain a better understanding of what exists within the environment.

Alternatively you may have processes in place to notify you when new endpoints and servers are added or removed in a network but again having a secondary process in place might be useful.

The proverb "Trust, but verify" defines this process nicely.

---

## Required Components

Below are the recommended components to integrate this use case:

- Domain Controller(s) (Configured with Active Directory Users and Computers)
- [Windows Security Event 4741 (A computer account was created)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4741)
- [Windows Security Event 4743 (A computer account was deleted)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4743)

If you have devices that are pre Windows 2003 use the following Windows Events:

- [Windows Security Event 645 (Computer Account Created)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=645)
- [Windows Security Event 647 (Computer Account Deleted)](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=647)

---

## Setup

Setup is really quite easy, using your tool of choice create a query or alert that will detect the above security events.

This can easily be done by monitoring for the Windows Event IDs `4741` or `4743` and summarising by the `Account Name` field under `New Computer Account` or `Target Computer` sections.

You can then output that query to an alert or report.

Here are some basic Splunk searches:

`source="WinEventLog:Security" "EventCode=4741" | eval Account_Name=mvindex(Account_Name, -1) | top Account_Name`

`source="WinEventLog:Security" "EventCode=4743" | eval Account_Name=mvindex(Account_Name, -1) | top Account_Name`

> **Note:** The section `eval Account_Name=mvindex(Account_Name, -1)` is used to ignore the first extracted value under `Account_Name` because it will always be the creator of the computer account.

I have two reports setup to run weekly to provide a summary of all computers accounts created and all computers accounts deleted in an environment from the previous week.

---

## Considerations

These events can occur multiple times per creation or removal so either use the report to provide you a summary or apply a threshold to your alert if possible.

You may only need to track log sources of a certain type i.e. Windows Server, so attempt to filter out all unwanted results.

---

## Closing

Knowing what is in your environment is extremely important, hopefully this post can help you achieve better visibility in your network or at least provide you with a secondary process for comparison purposes.

I hope you found this post informative. If you have any questions you can contact me via the [About](/about/) section.

---
