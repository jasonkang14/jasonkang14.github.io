---
title: React - Using Set as a State
date: "2019-07-16T17:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/using-set-as-a-state"
category: "React"
tags:
  - "React"

description: "How to use a 'set' as a state and update the state"
---

I got an interview question, which asked me to create a shopping cart where a user can add or remove an item. And when an item is added to a cart, I was supposed to put a checkmark to the item.

I first tried to implement this by using a `Boolean` state like below.

```typescript
consturctor() {
    super()

    this.state = {
        addedToCart: false
    }
}
```

However, when I did this, every single item got a checkmark, so I decided to use an `array` and add an index of the item to the array when an item is added to the cart, and put a checkmark to the items of which index exists in the array like below.

```typescript
consturctor() {
    super()

    this.state = {
        addedToCart: []
    }
}
```

But I realized that the values in the array could not be repeated. There is no repeated item in the list of items. so I decided to use a `set` instead of an `array` for efficiency.

You can use a `set` when...

1. there is no duplicate in your data--or when you try to remove duplicates
2. a fast look-up is required.
3. the order of data does not matter.

I thought my case suits all three. There were only 12 items, so maybe a fast look-up was not required, but it was an interview question, so I kinda wanted to show off that I know my stuff. And this is how I did it;

```typescript
constructor() {
    super();

    this.state = {
      productInCart: new Set(),
    };
  }
```

I set a `set` as a state like above, and updated, or `setState` likd below;

```typescript
handleProductClick = (idx) => {
  if (!this.state.productInCart.has(idx)) {
    this.addToCart(idx);
  } else {
    this.removeFromCart(idx);
  }
};

addToCart = (idx) => {
  this.setState(({ productInCart }) => ({
    productInCart: new Set(productInCart).add(idx),
  }));
};

removeFromCart = (idx) => {
  const updateCart = new Set(this.state.productInCart);
  updateCart.delete(idx);
  this.setState({
    productInCart: updateCart,
  });
};
```

When you add an item to a `set`, you can just add it directly to the current state. However, when you remove an item from a `set`, you have to declare a separate variable like you would do with an array like `const tempArray = this.state.array.slice()`, and then delete an item from the set and `this.setState` in order to update.
