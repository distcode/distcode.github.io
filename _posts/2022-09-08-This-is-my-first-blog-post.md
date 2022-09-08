---
layout: post
title: How to use time and date in KQL 
tags: [KQL]
# color: rgb(42, 52, 68)
---

# How to use time and date in KQL

This article shows you how to work with date and time values in KQL.

## creating a datetime

The function `datetime()` allows you to create a datetime variable.

```kql
let myDate = datetime();
VMComputer
| where TimeGenerated >= myDate
| project
```

bla bla bla

# h1

## h2

### h3

#### h4

##### h5
