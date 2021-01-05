+++
title = "Handling duplicated routes in Vue.js"
date = 2021-01-01T10:29:49+01:00
draft = false
author = "Marc Benedi"
authorTwitter = "marcbenedi"
cover = "img/vue-router-duplicated-route-error.png"
tags = ["Vue", "Web"]
keywords = ["Vue", "Web"]
description = ""
showFullContent = false
+++

For one of my side projects, I'm using **Vue.js**[[2]] to develop a web client. Vue.js offers an official router for the framework called **Vue Router**[[3]].

In order to navigate between **Views**, the framework offers the following function:
```javascript
router.push({ name: 'home'})
```
But what happens if we are already in the view `home`? We get the following nasty error:

```javascript
NavigationDuplicated {
  _name: "NavigationDuplicated", 
  name: "NavigationDuplicated", 
  message: "Navigating to current location ("/home") is not allowed", 
  stack: "Error at new NavigationDuplicated (webpack-int…/node_modules/vue/dist/vue.runtime.esm.js:3876:9)"
}`
```

We could avoid this error by adding a `catch` handler to the `router.push({ name: 'home'})` and check there the exception type. This solution is fine but it starts to become a problem if we have multiple `router.push` in the application because it would end up with tons of **code duplication**.

Searching around StackOverflow, I found a solution[[4]] which showed me the way to proceed. The solution explained here is an adaptation to that answer, but instead of comparing `Strings` to check the error type, I use the built-in functions offered by Vue Router.

```javascript
const router = new VueRouter({
  mode: "history",
  base: process.env.BASE_URL,
  routes
});
```

```javascript
const originalPush = router.push;
router.push = function push(location) {
  return originalPush.call(this, location)
    .catch(err => {
      if (isNavigationFailure(err, NavigationFailureType.duplicated)) {
        // Handle duplicated navigation here
        console.warn(err)
      } else {
        // Otherwise throw error
        throw err
      }
    });
}
```

The solution consists in redefining the `push` function with a new one that behaves the same except when there is a `NavigationDuplicated` error. 

# References 📑

[1]: https://router.vuejs.org/guide/advanced/navigation-failures.html
[[1]]: https://router.vuejs.org/guide/advanced/navigation-failures.html

[2]: https://vuejs.org/
[[2]]: https://vuejs.org/

[3]: https://router.vuejs.org/
[[3]]: https://router.vuejs.org/

[4]: https://stackoverflow.com/a/62331463/7484221
[[4]]: https://stackoverflow.com/a/62331463/7484221