---
title: "Design performant dashboard"
date: 2025-07-17
---

I had to do an exercise to design the UI for a dashboard system.
At first the requirements were simple:

- you go to a URL with dashboard/:id
- you have a title
- you can select the time period over which to see the data
- you have a widget which displays a time series for a given metric
- you have a button to add more widgets
- you have two modes: a live mode which shows the latest data and refreshes periodically, an absolute mode to see the data for a given period

Questions are
what is the UI layout?
What is the backend API like? A widget can have many visualization types, and many time series

Then more complications arise

- How do you refresh the data in live mode

  - long polling: effectively setInterval -- more simple and actually enough for use case where the smallest granularity is past 5 min
  - Server sent event -- never actually used it

- Suppose you have 5000 widgets on the dashboard (Why would anyone want to do that???) - how do you handle it?

Render only the widgets that are displayed
We can use the intersection observer API

Which one you want to render first, which request should fire first?

Priority queue, dynamic size batching,
how to determine the priority?
We load the first in the page, we know or find out which metrics take longer to load,

load based on intention of user? (like the pagination case, if user go to second page, then go the next pages, it shows intention of wanting more)

Who handles the request? The container or the widget itself

Ideally it would be the container, but how does it know when a widget should fetch
Data is stored in the container
The widget notifies the container that it wants data by a subscription mechanism.

How to jump to a particular widget, using the browser CTRL+F command, then you need to render the widget, it's the data that is fetched only when you are in the viewport

Virtualize the widgets? Reduce the number of DOM nodes
Does it work in this case?

- For one visualization, you have an enormous amount of data to show, how to prevent the page from crashing because of Out Of Memory error

Free the memory on the client when not needed, set the widget data to null and let the garbage collector do its job.

The most insane thing I heard: store the data in a more efficient format than JSON i.e. in compressed binary. Then decompress and deserialize it only when actually need to read it. Trade off on the time vs memory.
