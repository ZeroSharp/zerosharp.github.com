---
layout: post
title: "Top tip for viewing CSV files in Excel"
date: 2015-05-08 11:32
comments: true
description: A quick hack to open CSV files with the wrong separator.
categories: [csv, excel]
---
Here's a quick hack when your CSV file has a different separator than Excel is expecting.

Add this on the first line of the CSV file:

    sep=;

or

    sep=,

This will override system setting for list separator character and Excel will open the file correctly.

<sup>Note: Excel expects the separator to match the one defined in the _Control Panel/Region/Formats/Additional settings/List Separator_. On a French system, it expects a semi-colon.</sup>