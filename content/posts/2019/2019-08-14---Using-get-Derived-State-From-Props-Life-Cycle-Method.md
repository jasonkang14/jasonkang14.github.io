---
title: React Lifecycle - Using getDerivedStateFromProps with React Native and MobX
date: "2019-08-14T14:27:37.121Z"
template: "post"
draft: false
slug: "/posts/react/react-life-cycle-get-derived-state-from-props-with-mobx"
category: "React"
tags:
  - "React"

description: "How to use getDerivedStateFromProps instead of componentDidUpdate"
---

I am sure someone out there--or maybe you who are reading this post--has/have experienced infinite loop while trying to use `componentDidUpdate()`. Or you might have experienced that `this.props` and `prevProps` turned out to be same, so your `setState` inside `componentDidUpdate()` won't trigger.

I personally struggled a lot with `this.props` and `prevProps` being equal, but couldn't solve the problem. Adn the worst part was that it would work in some screens, but won't in others.

The solution that I have found is using `getDerivedStateFromProps()`. While `componentDidUpdate()` compares `this.props` with `prevProps` or `this.state` with `prevState`, `getDerivedStateFromProps()` compares `this.props` and `this.state` which are written as `props` and `state` respectively.

Look at my code below

```typescript
static getDerivedStateFromProps(props, state) {
        if (toJS(props.MainScreenStore.snsPostArr[0]).length !== state.prPostArr.length) {
            return {
                prPostArr: toJS(props.MainScreenStore.snsPostArr[0]),
                likedPostSet: new Set(toJS(props.MainScreenStore.snsPostArr[1]))
            }
        }
        return null;
    }
```

In the code above, I am trying to `setState` `prPostArr` and `likedPostSet` by using `props.MainScreenStore.snsPostArr`. I am fetching data from a server and storing the information into an `observable` called `snsPostArr` in a `MobX store`. The screen receives `observable` as a `props` from `MainScreenStore`, which handles all the requests made in `MainScreen`

![console logs from getDerivedStateFromProps](https://scontent-icn1-1.xx.fbcdn.net/v/t1.0-9/69336415_10219573739995003_5495412085456109568_n.jpg?_nc_cat=107&_nc_eui2=AeHeNzCxq53g06myCRPZzTYxjOPVBfNNOXSFhqfm7hXjR57BI6yyNUUt6gOxPfaZRp2ET59PczlEK707VjsINBi3Ro8DifNNkrXbVs870wTPTg&_nc_oc=AQmuevn132BoKchtXMlt-yPVxxZ6pWYPDGS2_CLlvSmbMwqDeNCc4bJCW2iuiV483iQ&_nc_ht=scontent-icn1-1.xx&oh=4635c18a9e8eb6058c336e3d8519ca80&oe=5E1606EA)

Since both parameters are arrays, you cannot compare them by simply using `===`, I am comparing the length of the two arrays. Initially both of them have the length of zero, but when the store fetches data from the server, the array from `props` changes, thus changing the length.

![iPhone Simulator](https://scontent-icn1-1.xx.fbcdn.net/v/t1.0-9/68536951_10219573747355187_4747634622542643200_n.jpg?_nc_cat=109&_nc_eui2=AeGhYQeQJYys75YQjbuhqREOamP-ebJ-BO_-n32QJn-yvOOKKollYbMxo9G7ymGayhAO4rCtd8PhBq6ocoHd5OzzCZBuxmUvkmb20DPEYTTmNQ&_nc_oc=AQlHNJT-h8gYLBqESNBg0F9y1DX5Pf5DGmGWOJccOXpAOmSyPFNw_-AJoI1lNa08wIU&_nc_ht=scontent-icn1-1.xx&oh=463d669a534169eb83cf512188228cff&oe=5DDB3625)

I will write about changing the timestamp in a later post
