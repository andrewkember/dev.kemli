---
title: "GMail Cleanup"
date: 2021-05-16T19:00:00Z
draft: false
summary: |
  Gmail has four tabs of email: Inbox, Social, Promotions and Updates. I've trained Gmail to put important messages in the Inbox, but Promotions and Updates are streams of ephemeral information that I don't want to maintain. I glance at new messages there, and read what I want. 

  I'm writing a Google Apps Script to regularly archive those email streams.
  
---

Gmail has four tabs of email: Inbox, Social, Promotions and Updates. I've trained Gmail to put important messages in the Inbox, but Promotions and Updates are streams of ephemeral information that I don't want to maintain. I glance at new messages there, and read what I want. 

I'm writing a Google Apps Script to regularly archive those email streams.

The logic is this:

* If it's in the Inbox
* ...And in Promotions or in Updates
* ...And it's not starred
* ...And it's older than 2 months
* Archive the message thread

(Note that all email in the four tabs is technically in the Inbox - Social, Promotions and Updates are sub-categories, which Gmail implements as labels.)

This gives me extra control: If there's an email in Promotions or Updates that I don't want to be archived, I star it. 

Here's the Apps Script code:

``` javascript
function cleanInbox() {
  // Try this search string in the Gmail search box until you're happy with the list of threads to be archived.
  const searchString = "in:inbox (in:promotions OR in:updates) NOT is:starred older_than:2m ";
  var batchSize = 10; // Archive this many threads at a time

  console.log("Running clearInbox");
  console.log("searchString: " + searchString);
  console.log("batchSize: " + batchSize);

  var threads = GmailApp.search(searchString);
  console.log("Found " + threads.length + " matching threads.");

  // Archive the threads in batches to reduce number of API calls
  for (var j = 0; j < threads.length; j+=batchSize) {
    GmailApp.moveThreadsToArchive(threads.slice(j, j+batchSize))
  }
  console.log("Threads archived.");
}
```

I created a project at https://script.google.com/home and pasted this code in the Files section. When I chose Run for the first time, I was prompted for access to my Gmail. 

Then I went to the Triggers section and set up a time-based trigger to run the function every week. 