---
title: What's the HSL?
author: huytd
date: 11-24-2022
---

There are many different ways to represent an RGB color when using CSS, for example, using hexadecimal colors (`#RRGGBB`) or using the exact RGB colors (`rgba(R G B alpha)`).

Another way to represent RGB values is using HSL, which stands for _Hue_, _Saturation_, and _Lightness_.

Where Hue is the angle of the desired color on the color wheel, and Saturation in percentage to describe how saturated the color will be. And lastly, the Lightness, also in percentage, tells how bright it will be.

![image](https://user-images.githubusercontent.com/613943/204167281-fb2f90a8-ae57-4681-bff6-cf129156519d.png)

For example:

```
// 90% satured red, full bright
hsl(0 90% 100%)

// somewhere between yellow and green, half bright
hsl(80 80% 50%)
```

One benefit of using HSL over hexadecimal or RGB values is it’s closer to the natural way humans think about colors. For example, using HSL, it’s easier to modify the color (make it more saturated or lighter/darker).

For example, take a look at the two shades of the Emerald color: `#6EE7B7` and `#059669`. I doubt you can find anything related just by reading the two hex values here.

Now, look again. This time, they’re in HSL: `hsl(156.2 71.6% 66.86%)` and `hsl(161.38% 93.55% 30.39%)`. First, look at the Hue value, and look at the color wheel again. You can tell the color is somewhere in between the Green and Cyan color, then you can tell how saturated the color is and how bright.
