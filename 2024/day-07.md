**Author:** duc

---

# Insights Into Browser Behavior During URL Changes From A Bug

## Introduction

![](https://www.notion.so/front-static/logo-ios.png)

Original article:  
https://nhducit.notion.site/Insights-Into-Browser-Behavior-During-URL-Changes-From-A-Bug-155adcd74d9980f38ce9ee90aa8afede#155adcd74d9980e1b13fe39aa67acbbe

![](https://prod-files-secure.s3.us-west-2.amazonaws.com/3f3d2bbc-c9cf-4c94-8f72-3e6b75f8e0cc/5f4fab5c-c58f-44f5-82a5-ec24e81eb47a/SocialMediaPreviewImage.png?table=block&id=155adcd7-4d99-80f3-8ce9-ee90aa8afede&spaceId=3f3d2bbc-c9cf-4c94-8f72-3e6b75f8e0cc&width=2000&userId=&cache=v2)

---

## Problem

---

## Context

Our application was written in [Angular](https://angular.dev/) and [ngRx](https://ngrx.io/), but the demo application will use React and simplified logic to make it easier to understand.

Users add products to the shopping cart and select items for checkout, which creates an order. Before completing the checkout, users may need to solve an MFA challenge to prevent fraud.

> Loading Mermaid code...

Here is the simplied logic that is implemented with rxjs.

---

## Investigation

I placed the debugger before navigating to the MFA web app in two flows.

Here is the result:

1. `"navigating to /mfa with type cash_redemption"` is logged in the console  
2. Chrome DevTools pause the application at the debugger within the cashback redemption logic.  
3. `"navigating to /mfa with type shopping_cart"` is **not** logged in the console  
4. Chrome DevTools doesn’t pause the application at the debugger within the shopping cart logic.

---

## Interesting bug

This is interesting: the DevTools didn’t pause at the Shopping Cart debugger, e.g. the:

```js
window.location.href = "/mfa?type=shopping_cart";
```

is not executed but the opened URL was:

```text
/mfa?type=shopping_cart
```

instead of:

```text
/mfa?type=cash_redemption
```

I thought that after the:

```js
window.location.href = "/mfa?type=cash_redemption";
```

line, the FE application was destroyed and the current browser tab loaded the new URL/application.

![](https://prod-files-secure.s3.us-west-2.amazonaws.com/c15e2d47-e034-46ca-8ca6-8717fa9b2844/3e8ac52f-5dc9-47f7-8807-27016352ad2c/CleanShot_2024-12-07_at_16.57.28.png)

But this was not the case; let’s confirm the assumptions:

Let’s check if commenting out:

```js
window.location.href = "/mfa?type=cash_redemption";
```

will make the:

```js
window.location.href = "/mfa?type=shopping_cart";
```

be executed.

Yes, it will.

> Loading JavaScript code...

That means this assumption is incorrect and the FE application is not immediately destroyed after the current tab URL is changed, e.g.:

```js
window.location.href = "new_url";
```

> I thought that after the  
> `window.location.href = "/mfa?type=cash_redemption"`  
> line, the FE application destroyed.

![](https://prod-files-secure.s3.us-west-2.amazonaws.com/c15e2d47-e034-46ca-8ca6-8717fa9b2844/6eb2bddd-e9c9-402a-982d-78b5e5b989dd/CleanShot_2024-12-07_at_16.58.46.png)

I created a PoC web app to confirm this. [You can try it here](https://stackblitz.com/edit/sb1-v28dgzyb?file=src%2FApp.tsx). After clicking on the **“Open slow web site”** button, the counter continues running; that means the web application is not destroyed until the new URL returns the response.

**Source code**

Loading...

---

## Conclusion

The solution for the original bug is quite simple. You can try to fix it by cloning this repo:  
[![aos-chrome-devtool](https://we-build-vn.notion.site/images/external_integrations/github-icon.png) aos-chrome-devtool ![](https://avatars.githubusercontent.com/u/4246176?v=4)](https://github.com/nhducit/aos-chrome-devtool)

The FE application is not immediately destroyed after the current tab URL is changed, e.g.:

```js
window.location.href = "new_url";
```

but the DevTools were detached from the current application, which is why any debugger after the:

```js
window.location.href = "new_url";
```

will not pause the application.

This is a more concise example of the issue:

> Loading JavaScript code...

---

## Resources

- Demo application: https://aos-chrome-devtool.vercel.app/
- GitHub repo:  
  [![aos-chrome-devtool](https://we-build-vn.notion.site/images/external_integrations/github-icon.png) aos-chrome-devtool ![](https://avatars.githubusercontent.com/u/4246176?v=4)](https://github.com/nhducit/aos-chrome-devtool)