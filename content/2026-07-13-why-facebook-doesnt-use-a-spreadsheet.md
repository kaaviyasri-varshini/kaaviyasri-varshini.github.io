Title: Why Facebook Doesn't Use a Spreadsheet (And Why That Matters)
Date: 2026-07-13
Category: Developer Tools
Tags: graph database, databases, data, explainer, backend
Slug: why-facebook-doesnt-use-a-spreadsheet

## Have You Ever Wondered How Instagram Always Knows?

How Instagram knows to suggest "people you may know" — and somehow it's *always* eerily accurate? Or how a fraud team at your bank can spot a suspicious ring of transactions in seconds, when the connections span dozens of accounts?

The secret, a lot of the time, is something called a **graph database**. It sounds technical, but the idea behind it is refreshingly simple — and once you get it, you'll start noticing it everywhere.

## The Problem With Rows and Columns

Most databases you've heard of store data like a giant spreadsheet: rows and columns, neatly organized into tables. That works great for a lot of things — inventory lists, customer records, sales totals.

But spreadsheets get clumsy the moment you ask a *relationship* question. Try modeling "who is friends with who, who is friends with *their* friends, and so on" using rows and columns. You'd need to constantly cross-reference table after table, and the more "hops" away you go (friend of a friend of a friend...), the messier and slower it gets.

That's the exact problem graph databases were built to solve.

## So What Is a Graph Database, Really?

Picture a corkboard with photos pinned to it, and pieces of string connecting the photos that are related to each other. That's basically a graph database.

**The photos are called "nodes."** Each one represents a *thing* — a person, a product, a city, whatever you're tracking.

**The strings are called "edges."** Each one represents *how two things are connected* — "is friends with," "bought," "lives in."

Both the photos and the strings can carry extra details. A "person" node might have a name and age. A "bought" edge might have a date and price attached to it.

That's really the whole idea: **things, and the connections between them, stored as equally important pieces of the puzzle.**

## Why This Makes Certain Questions So Much Easier

Here's the everyday magic of it: because relationships are built right into the structure, the database doesn't have to do a bunch of extra work to figure out how things connect. It just follows the string.

Think about it like planning a road trip using an actual map versus a spreadsheet of coordinates. The map *shows* you the roads. The spreadsheet makes you calculate them. Graph databases are the map.

This is exactly why they shine at things like:

- **"People you may know"** — following the strings between your friends' friends
- **Fraud detection** — noticing when a handful of transactions form a suspicious loop
- **Recommendation engines** — "people who bought this also bought that"
- **Mapping out how anything is connected** — company org charts, supply chains, even Wikipedia links

## A Quick Real-World Analogy

Imagine you're at a party and someone asks, "Do you know anyone here who works in marketing?" You don't scan a spreadsheet in your head — you instinctively think through your social connections: "Sam knows Priya, and Priya works in marketing." That mental hopping from person to person? That's exactly what a graph database is built to do, just at massive scale and in milliseconds.

## The One Thing Worth Remembering

You don't need to memorize any technical terms to walk away with the important idea here:

Some data is really about the connections, not just the things themselves — and graph databases are simply a way of storing information that treats those connections as first-class citizens, not an afterthought.

Next time an app "just knows" who you might want to connect with, or a bank flags a sketchy transaction ring in real time, there's a good chance a graph database — and its web of nodes and strings — is quietly doing the work behind the scenes.
