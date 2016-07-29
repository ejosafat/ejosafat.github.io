---
layout: post
title:  "Practical OOP"
date:   2016-07-29
categories: '2016' 
tags: books
permalink: practical-oop
---
# Practical OOP


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
