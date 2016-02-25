---
layout: post
title:  "Write expressive code (II)"
date:   2013-08-13
categories: '2013' 
tags: other 
permalink: write-expressive-code-II
---
# Express your intention (II)

Folowing on [the previous article]({{ site.github.url }}/write-expressive-code-I) about expressive code, let's see how you can clarify those typically nested javascript callback functions.

> *Be expressive, my friend.*
> *(Unknown Jedi Fu Master)*

## Refactoring nested callbacks

Given an array of users included in one group, I want to get only those who don't belong to any other. I wrote this:

```javascript
function usersOnlyInThisGroup(users, groups) {
  return _(users).reject(function (user) {
    return _(groups).any(function (group) {
      return group.includeUserInSelfAndDescendants(user);
    });
  });
}
```

In order to clarify it, I did this refactoring:

```javascript
function usersOnlyInThisGroup(users, groups) {
  return _(users).reject(isIncludedInOtherGroup);

  function isIncludedInOtherGroup(user) {
    return _(groups).any(includesTheUser);

    function includesTheUser(group) {
      return group.includeUserInSelfAndDescendants(user);
    }
  }
}
```

I think the refactored version express the intention a way better than the first one. 

## To keep in mind
The more expressive your code is, the less effort to understand it.
