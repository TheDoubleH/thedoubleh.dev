---
title: "Performance vs User-Experience"
subtitle: "What is more important to the user?"
date: 2022-01-03T16:06:20-08:00
draft: false
authors: [Henrik]
description: ""
tags: []
categories: [Development, Design, Job Queue]
series: []
series_weight: 1
seriesNavigation: true
images: ['featured-image.jpg']
featuredImage: "featured-image.jpg"
featuredImagePreview: ""
lightgallery: true
outdatedArticleReminder:
  enable: false
share:
  enable: true
toc: 
  auto: false  
---

## Perception is Reality

As a Magic Lover and member of the World-Famous Magic Castle in Hollywood, California, I have taken some classes in the art of conjuring. In Magic we often rely on the fact, that _Perception is Reality_. That is - If People _perceive_ a fact, then it must be so!

The same is true when it comes to users using NAV or Business Central! Over the years, I have often heard people complaining about slow systems, and while in many cases, there are low hanging fruits that can resolve the issues, there are also often times a different approach to the whole thing: Change the User Experience!

## Real World Issue

Many years ago, I worked with a Medical Device Company. We had built a custom configurator, and for each Sales Order, we planned a production order for a Hearing Aid. Each hearing aid was custom made for the patient, and components were based off of answers from the configurator. Anyone who have worked with MRP and Production Orders in NAV/BC knows that that planning part can be rather cumbersome and time-consuming.

The process we had was:

1. Ear Mold was received in the morning from Currier
2. User Scanned the inbound Tracking No. and selected Order Type (New vs Repair)
3. Sales Order was created, and a label was printed. Ear Mold was placed in a tote and placed on conveyor
4. Order Entry Person Scanned the Order Label. That opened the Sales Order Page, and order details was entered.
5. Order Entry Person 'Released' the Sales Order. This resulted in Production Planning to take place.
6. Once the Production Order was created, a Production Traveler report would print, and the order would go to production

We had anywhere between 2,000 and 2,500 orders a day.

It wasn't long before we ended up with Deadlocks and users waiting for the Production Order Planning - and users would just sit a wait for up to two minutes, and soon enough:

> The System is Slow today!

was a common complaint. We ended up inserting KPI records so we could find the culprits and to confirm or dispute the users' claims. After some initial optimization the system ran pretty stable. We still had deadlocks caused by the Production Planning and we still had Order Entry users sit and wait.

## How we solved the issue

"Outbound Warehouse Request" - No Really - In Standard NAV/BC the Release function creates a record in the `Outbound Warehouse Request` table, if the Location Code is set up for Warehouse. We realized that we didn't have to actually complete the production planning. We just needed to inform a process that the order needed Production Order Planning and then have another process pick it up!

We created a "Sales Order Release Request" table. We then had the release function create an entry (after some rudimentary tests passed), and the user now pressed 'Release' and were ready for the next order.

The process (from the users perspective) became

1. Ear Mold was received in the morning from Currier
2. User Scanned the inbound Tracking No. and selected Order Type (New vs Repair)
3. Sales Order was created, and a label was printed. Ear Mold was placed in a tote and placed on conveyor
4. Order Entry Person Scanned the Order Label. That opened the Sales Order Page, and order details was entered.
5. Order Entry Person 'Released' the Sales Order.

The NAS then took over and did:

1. Production Planning.
2. Once the Production Order was created, a Production Traveler report would print and the order would go to production

> The User EXPERIENCE was now that the system was super-fast

The user was no longer waiting for the system! We then created a NAS (precursor to Job Queue) and it plowed through the requests at its leisure. An added benefit, was that the Production Planning was now faster, as it now handled one order at a time, and gone were the deadlocks.

If I were to do this today, I would probably "just" create a `Job Queue` and then create a `Job Queue Entry` for each Sales Order. The `Job Queue` and `Job Queue Entry` handles a lot on the plumbing we had to create back then.

## Take-aways

This experience taught me, that while You always want to optimize Your code, spending hours and hours on gaining a second might not always be the best approach. To me the User *Experience* is much more important!
