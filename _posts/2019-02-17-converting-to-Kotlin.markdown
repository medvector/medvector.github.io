---
layout: single
title:  "TIL: how to convert to Kotlin preserving git history"
date:   2019-02-17
categories: Kotlin
---
When we started actively using Kotlin in our project, we also started actively converting existing Java classes
to Kotlin.

It's very easy to do in IntelliJ IDEA because there is a special action that does everything for you: Code -> Convert Java File to Kotlin File.
However, there is one problem related to git: conversion is treated as .java file removal and .kt file
creation. This results in git history of the original .java file being lost.

A workaround is also quite simple, but requires some boring manual steps:
1. Rename .kt file to .java file 
2. Commit
3. Convert .kt file to Kotlin
4. Commit

This way there are two commits instead of one, but the history of the file in preserved.

There are some ways to automate it including IJ plugin, but TIL that IJ actually can do it for you without any plugins. You just need to check one little checkbox in a secret place :)

Let's convert a file to Kotlin using the action. Now when one commits the change from IJ one can notice 
that there is an additional checkbox in commit dialog:

<img src="../../assets/images/convertToKotlin.png">

Magic! We have our preserved git history without any effort from our side.