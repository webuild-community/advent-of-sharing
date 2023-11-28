---
title: Reduced Motion
author: nvh95
date: 11-28-2022
---

:runner: Animation/Motion is a very important part of web applications, making web pages more lively, interesting, and "real". However, many people may not be able to tolerate too much motion, which can cause dizziness, blurred vision, headaches, or nausea.

:brain: Normally, the vestibular system (including the inner ears and parts of the brain) helps us balance. However, for many people, animation may cause their vestibular system to become unbalanced and lead to vestibular disorders, creating discomfort for the user, with the most typical symptom being dizziness.

:question: Therefore, as a frontend developer, we need to pay attention to this detail so as not to create discomfort for the user. But we also cannot remove all motion from a web page, as for most users with stable vestibular systems, they will not have the best experience.

👩‍💻 Fortunately, CSS has a media query called "prefers-reduced-motion". This media query will check to see if the user wants to "reduce" the motion of the web page. This setting can be easily configured in the operating system (image 2). The syntax of "prefers-reduced-motion" is as follows:


```css
.box:hover {
  transform: scale(1.2);
}

@media (prefers-reduced-motion: reduce) {
  .box {
    transition: none;
  }
}

@media (prefers-reduced-motion: no-preference) {
  .box {
    transition: transform 300ms;
  }
}
```

:gift: In the code box above, when the user hovers over the (box), if they want to reduce motion, the box will increase to 1.2 times immediately. Normally, it will increase to 1.2 times in 300 ms. In this way, most users will experience the animation, while the minority of users who choose reduced motion will not get dizzy.

:movie_camera: The attached video describes this feature from a very familiar software, Slack. When we activate the reduced motion, if there is a new message, the notification will gradually appear. Otherwise, the notification will slide in from the right.

:sunglasses: These are small details, but they will be information that helps you become a better frontend developer. Always put yourself in the user's position (specifically in this case, users with vestibular disorders) as it will help you create good quality and user-friendly web applications.

https://user-images.githubusercontent.com/613943/204171890-1ff95622-a23c-45a0-8305-2a08d1ce339c.mp4

https://user-images.githubusercontent.com/613943/204171900-e0378bb5-8d68-4bb6-b8ff-73b2c9474226.mp4

If you are using a Mac, you can activate the Reduced Motion here:
(Choose the option in the image to reduce motion when using web applications)

![Navigating to Accessibility in Mac Settings to activate Reduced Motion](https://user-images.githubusercontent.com/613943/204170698-6f50d4aa-470d-4df6-ba87-991e8d95d806.png)
