---
title: "Clean Code"
subtitle: "You will not be the last person to see your code"
date: 2021-12-03T13:08:06-08:00
lastmod: 2022-01-03T13:08:06-08:00
draft: false
description: ""
tags: []
categories: [Development, Design]
series: []
series_weight: 1
seriesNavigation: true
featuredImage: "featured-image.jpg"
featuredImagePreview: ""
lightgallery: true
outdatedArticleReminder:
  enable: false
share:
  enable: true
---
## Why is Clean Code and Standards Important?

In the last few years, I have spoken with developers that are new to BC. A lot of them have done lots of dotnet. Most of them all look at me confused when I talk to them about Development Standards - Especially when it comes to Variable Naming etc. I likewise look baffled when they don't understand the importance of this.

To me, the most important reason is:

> You will NOT be the last person to see that code.

And while that was truer in the NAV world, where the code was saved in the Database, I still believe this to be true in Business Central. Only difference is that Your code will now be saved in a git repository, while the source code may be hidden in your .app file that You distribute. If You do not follow commonly agreed upon standards, you are ending up spending additional time while you maintain the code - especially if a co-worker needs to participate.

As someone who have worked with NAV and BC for over 20 years, I have seen my share of code created by someone else - and some of my own for that matter - getting frustrated that I wasn't able to skim the code due to inconsistent ways of writing the code. I used to work with a guy, who in every piece of code would have a variable `Aux`. Sometimes this variable was a Customer Table, other times an Integer, decimal ... You get the idea. Fact was, trying to debug his code ALWAYS ment adding extra time to figure out what actually went on.

## Clean Code and Design Patterns isn't a new thing

The first time I really heard about it was when I attended a session at [NAVTechDays](https://bctechdays.com/) in 2014 when I heard someone say

> A Codeunit should only have one global function,

and

> A function should not have more than 20 lines of code

My first reaction was: "*What have they been smoking?*"

It wasn't until I "got home" and tried to implement some of these ideas at customer sites myself. I, to this day, still remember the implementation I worked on, where I created a `Sales Post - Hook` codeunit. Initially I thought: "This is dumb - I am creating an extra object, just to transfer some fields from the `Sales Line` to the `Item Journal Line`, and from the `Sales Header` to the `General Journal Line`. But "This is the way, now" so I gave it a chance. Lo and behold - a month or so later, the client wanted some additional fields, and I realized that, I no longer had to change Codeunit 80 - `Sales - Post`, but just changing "my" codeunit `Sales Post - Hook`! By creating the hook, I had encapsulated this modification, so they were easy to update and maintain!

From then on, I fully embraced these design patterns, and I tried to educate the developers I work with to do the same. Granted it can be difficult to teach an old dog new tricks, but the team is slowly getting there.

Just recently, I had a Developer do some code review on my code, and he said:

> it's reads like you're having a conversation with NAV and teaching it what needs to be done

And this is exactly what I try to do. The code should really look like a table of content.

## Clean Code makes debugging and fixing code easier

We have all done code cloning, where we have repeated code inside a `CASE` statement (If you say you haven't, you are lying!). I myself once was trying to debug some code that I wrote. The customer had reported back that under certain circumstances one piece of the code wasn't executed. This code was written just like "we" always had, namely hundreds on lines inside one function. Trying to read all of the code, I finally decided to refactor and break the code into pieces.

By moving all of the code to discrete functions, and then removing those blocks, it became obvious that I had forgotten to copy some code to that scenario.

```AL
IF ("Method Type" IN ["Method Type"::Authorize, "Method Type"::"Voice Authorize", "Method Type"::"Return Authorize"]) AND
   ("Transaction Status" IN ["Transaction Status"::Declined,"Transaction Status"::Failed,"Transaction Status"::Error]) AND
   ("Declined Card Action" <> "Declined Card Action"::Nothing)
THEN BEGIN
  Success := FALSE;
  MODIFY;
  OrgEFTTrans := Rec;
  CASE "Declined Card Action" OF
    "Declined Card Action"::"Try Other Non-Expired Cards" : Success := SubmitTransactionWithAlternateCards(OrgEFTTrans, AlternativeTran, FALSE);
    "Declined Card Action"::"Extend Expired Card" : Success := SubmitTransactionWithExtendedExpirationDate(OrgEFTTrans, AlternativeTran, CardWasAutoBlocked);
    "Declined Card Action"::"Extend Expired Card; Then Try Other Non-Expired Cards" :
      BEGIN
        Success := SubmitTransactionWithExtendedExpirationDate(OrgEFTTrans, AlternativeTran, CardWasAutoBlocked);
        IF NOT Success THEN
          Success := SubmitTransactionWithAlternateCards(OrgEFTTrans, AlternativeTran, FALSE);
      END;
    "Declined Card Action"::"Extend Expired Card; Then Non-Expired Cards; Then Extend Other Expired Cards" :
      BEGIN
        Success := SubmitTransactionWithExtendedExpirationDate(OrgEFTTrans, AlternativeTran, CardWasAutoBlocked);
        IF NOT Success THEN
          Success := SubmitTransactionWithAlternateCards(OrgEFTTrans, AlternativeTran, FALSE);
        IF NOT Success THEN
          Success := SubmitTransactionWithAlternateCards(OrgEFTTrans, AlternativeTran, TRUE);
      END;
    "Declined Card Action"::"Try Other Non-Expired Cards;Extend Expired Card; Then Extend Other Expired Cards" :
      BEGIN
        Success := SubmitTransactionWithAlternateCards(OrgEFTTrans, AlternativeTran, FALSE);
        IF NOT Success THEN
          Success := SubmitTransactionWithExtendedExpirationDate(OrgEFTTrans, AlternativeTran, CardWasAutoBlocked);
        IF NOT Success THEN
          Success := SubmitTransactionWithAlternateCards(OrgEFTTrans, AlternativeTran, TRUE);
      END;
  END;
  IF Success THEN
    Rec := AlternativeTran;
  COMMIT;
END;
```

I tend to stress to folks, that a Procedure should do ONE thing and ONE thing only - And the Procedure Name should tell You what it does. The added benefit is that there are no doubts as to what the INTENT of that code is. If something doesn't work, and a function is called `GetOutstandingItemSalesLines` and the code looks like this:

```AL
local procedure GetOutstandingItemSalesLines(SalesHeader : Record "Sales Header"; var SalesLine : SalesLine);
begin
    SalesLine.SetRange("Document Type", SalesHeader."Document Type");
    SalesLine.SetRange("Document No.", SalesHeader."No.");
    SalesLine.SetRange(Type, SalesLine.Type::Item);
    SalesLine.Setfilter("Qty. Outstanding", '%1',0);
end;
```

it is easy for anyone to understand that the filter `SalesLine.Setfilter("Qty. Outstanding", '%1',0);` is incorrect - The procedure name shows the intent of retrieving outstanding sales lines.

## What's next

Following a Dynamics Con session "How I Manage My Team for Product and Customer Development?", Waldo  told the audience that he had some standard his company follows, and he remined us, that there was a Design Pattern initiative from the NAV days, that had since been largely abandoned. Waldo felt that that was sad. I Immediately followed up, with a message to Waldo, saying I would love to partake in a community effort in bringing this back to life. After some time, Waldo reached out, and we now had a team of four - which after Microsoft caught wind of the effort was expanded with a Microsoft Team Member.

https://alguidelines.dev is the result of this - And this is by no means complete, and we want YOU to take part of this as well. Participating is as easy as going to the [github repository](https://github.com/microsoft/alguidelines) and start, or participate, in a Discussion. You are also encouraged to Create a Pull Request. 

This site is meant to be the 'Generally Agreed upon patterns and standards', and having Microsoft involved, we hope this will be to where all developers are looking for best practices. If we all follow the same methods and standards, quite frankly, we all win.

## For Your Viewing Pleasure

The Nav TechDays sessions that started this for me :
{{< youtube 7ZUceRrbNAo >}}

Waldo's Session at Dynamics Con :
{{< youtube TQcyAnBaTgw >}}
