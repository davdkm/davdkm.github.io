---
layout: post
title:  "Learning React - First Impressions"
categories:
---
I found the AngularJS lessons from the FlatIron School pretty interesting when I was moving through the coursework. After doing some research and talking with other developers, it seemed the two big front end technologies were Angular and React. It seemed to me React was growing in popularity so I wanted to see what the fuss was about. Luckily, the FlatIron school recently rolled out a React course. Nice!

From my initial understanding of React, is that it operates a bit differently from Angular under the hood but there's definitely some similarities. While Angular is a framework that operates on the client-side directly on the DOM, React apparently works on the server side and on a Virtual DOM. It takes a clever differencing approach to changing the views, and only applies the minimum chunks of data required to update the DOM.

I started React and noticed that if there were errors in my code that made my app fail, they'd show up during compile. If I required components that weren't being used, React would blow up. This was due to not requiring React at the top of every JS file that used React.

I was also introduced to coding in JSX, which is a kind of meld between javascript/html/xml. When creating components, React is expecting only one element to be returned. The element can have many children nested within it, but the return of React component must always a be a single element due to javascript works.

There's also two ways to create React elements: with `React.createElement()` syntax and JSX which looks like HTML. Although it's not required to use JSX with react, I think I will be writing with it as `createElement()` looks a little tedious to use.

I'm just getting my feet wet with React, but I think it will be pretty fun. Maybe even better than learning Angular, but we shall see!
