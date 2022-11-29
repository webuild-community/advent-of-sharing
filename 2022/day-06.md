---
title: Calculating nested border radius
author: huytd
date: 11-29-2022
---
This is a short but very helpful tip for calculating border radius of some elements inside something that already have the border radius (think about a rounded button inside a rounded card).

You can’t just use the same value for the outer and the inner round because the result will not look good.

The formula for calculating the inner radius is:

```
Inner Radius = Outer Radius - Gap
```

![image](https://user-images.githubusercontent.com/613943/204413553-4ad63bb2-50eb-4f17-a019-f2f0c2bf7812.png)


It can be implemented in CSS like this:

```css
--outer-radius: 1em;
--padding: 0.5em;
--inner-radius: calc(var(--outer-radius) - var(--padding));
```

Source: 

- https://twitter.com/lilykonings/status/1567317037126680576
- https://cloudfour.com/thinks/the-math-behind-nesting-rounded-corners/
