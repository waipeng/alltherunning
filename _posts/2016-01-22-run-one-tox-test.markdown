---
layout: post
title:  Run just one tox test
categories: notes
---
If you are using tox, you can run just one tox test by doing:

    tox -e py27 -- <TestClass>

This is particularly useful if you are doing development for big OpenStack
projects and you don't want to run all the tests over and over again.
