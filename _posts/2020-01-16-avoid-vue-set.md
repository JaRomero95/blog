---
title: Avoid Vue.set - Reactivity and immutability
author: Juan Antonio Romero Molero
date: 2020-01-16 19:00:00 +0200
categories: [Frontend]
tags: [javascript, vue, immutability]
description: "Learn about the alternatives to Vue.set and why you should choose them"
---

## Introduction

Vue uses reactivity to listen for changes in component data and update the view if necessary after a change.

To achieve this, Vue adds getters and setters that wrap each property that we define. When we modify a value, we are calling the setter method that was created earlier. Through this setter, Vue can notice the change.

However, there are some situations where we find that our component does not update after a value changes.

## Reactivity problems

Let's look at the following component:

```js
export default {
  data: () => ({
    post: {
      title: 'The Last Wish'
    }
  }),
  methods: {
    toggleFav() {
      this.post.fav = !this.post.fav;
    }
  }
}
```

And its template:

```html
<div>
  <h1>
    <span v-if="post.fav">♥</span>
    Hello!
  </h1>
  <button @click="toggleFav">Fav</button>
</div>
```

If we run this example and try to use the button, we'll see that the page does not change. However, if we debug the component with the browser development tools, we'll see that the `fav` value changes even if the view does not reflect it.

We commented before that Vue adds getters and setters to the properties we defined in our component. The `fav` field, as we didn't declare it within the *post* model in the data of our component and we add it later, it has not been initialized correctly so it does not have these methods and Vue does not detect any change when we modify it.

Fixing this problem would be as easy as setting an initial value for the `fav` field:

```js
// ...
data: () => ({
  post: {
    title: 'The Last Wish',
    fav: false
  }
})
// ...
```

There are some situations where Vue cannot detect changes:

- New properties added to an existing object
- Properties removed from an object
- Set an element to a specific position in an array
- Changes in the length of an array

> To avoid problems with common situations such as mutations in arrays, Vue transparently overwrites some of its methods, since otherwise, it would not be able to detect the mutations made when we invoke them. The methods Vue modify so that you can use them without worrying about reactivity are: `push`, `pop`, `shift`, `unshift`, `splice`, `sort` and `reverse`. But I recommend you don't use them too, for the same reasons I explained in this article.

## Vue.set

Probably you already knew about reactivity in Vue and these specific cases, but I wanted to reintroduce them to talk about one of the proposed solutions for this problem, which is the `set` method of Vue.

Here are some examples:

```js
// Vue.set(object, propertyName, value)

Vue.set(this.post, 'fav', !this.post.fav);

this.$set(this.post, 'fav', !this.post.fav);

this.$set(this.posts, 1, post);
```

`Vue.set` simply ensures that the new value is set as a reactive property, solving the problem of not having declared it initially.

Is a quick and easy solution, but in my opinion, we should avoid this method.

## Why avoid Vue.set

Reactivity problems in Vue usually stem from a poorly designed component and come to light when we try to mutate an object. Using a brute force method like this prevents us from doing the right thing: refactoring.

Our components should be as small as possible, because the easier it is to extract the component code to another place, the easier it will be to add or modify functions. Otherwise, this method couples the code to the Vue API, forcing us to change the code before we can move to a non-component file.

Finally, avoiding using this method forces us to know more about immutability and its benefits in JavaScript, knowledge that will help us write pure and reusable functions and avoid having to face a tough debugging process due to unexpected mutations.

## Vue.set alternatives

If we are faced with one of the cases listed above, in which Vue cannot detect changes in our components, we have different options depending on the situation.

### New properties added to an existing object

In this case, we'll choose to create a new object containing all previous properties plus the one we are adding. Then we can change the reference to our new object. Vue will easily detect the change and update the view based on the new value.

For this, we have two options. First, using `Object.assign`:

```js
toggleFav() {
  this.post = Object.assign({}, this.post, {fav: !this.post.fav});
}
```

And second, the one I prefer, using [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax):

```js
toggleFav() {
  this.post = {...this.post, fav: !this.post.fav};
}
```

### Properties removed from an object

If what you want is not to add, but remove a property of an object, we can also do it without mutating it. We have several options, one of these will be to clone the object, delete the property and, as in the previous example, replace the object.

```js
deleteFavProperty() {
  const post = {...this.post};
  delete post.fav;
  this.post = post;
}
```

> Be careful with objects that contain another object in some of their properties, because if you clone them in this way, child objects won't be cloned and you could unintentionally mutate them. Use another method of cloning or avoid working with complex objects if you can.

We have another alternative. Here we can make use of [rest/spread](https://github.com/tc39/proposal-object-rest-spread), which again is the one I prefer.

```js
deleteFavProperty() {
  const {fav, ...rest} = this.post;
  this.post = rest;
}
```

### Set an element to a specific position in an array

If we try to do the following:

```js
replacePost(index, newPost) {
  this.posts[index] = newPost;
}
```

We'll see that Vue does not detect it either. The simplest option to avoid using `Vue.set` would be the `Array.splice`. Let's see how it looks:

```js
replacePost(index, newPost) {
  this.posts.splice(index, 1, newPost);
}
```

But yes, this **method mutates the array**. We mentioned earlier that Vue replaces some `Array` methods to be able to detect changes, and this is one of them.

However, we prefer to avoid mutations because some higher or lower components in our composition could be using this data. So we can apply one of the following options:

```js
replacePost(index, newPost) {
  // A
  this.posts = this.posts.map((post, postIndex) => {
    return postIndex === index ? newPost : post;
  });

  // B
  this.posts = Object.assign([], this.posts, {[index]: newPost});

  // C
  this.posts = [
    ...this.posts.slice(0, index),
    newPost,
    ...this.posts.slice(index + 1),
  ]
}
```

### Changes in the length of an array

Finally, we have the least common case, the modification of the length of an array:

```js
incrementPostsLength(numberOfEmptySpaces) {
  this.posts.length += numberOfEmptySpaces;
}
```

If we test this case, we'll see that Vue does not detect it. There are a few options to do this. One of them will be to create a new array with the number of elements that we want to add and concatenate it at the end of the original, giving rise to a new one:

```js
incrementPostsLength(numberOfEmptySpaces) {
  const emptySpaces = new Array(numberOfEmptySpaces);
  this.posts = [...this.posts, ...emptySpaces];
}
```

## Conclusion

I prefer to take advantage of the options the language offers than use the Vue API or any other third-party code when it's not really necessary, avoiding adding dependencies to my code.

Don't get me wrong, third party libraries are useful, but we don't have to abuse them. [The abuse of "lodash" is a clear example of this](https://you-dont-need.github.io/You-Dont-Need-Lodash-Underscore/#/).

If you choose these options, your code will be more portable and you'll learn how to take advantage of immutability, a knowledge that you can apply in your next JavaScript development, whatever framework you use.