---
layout: post
title: "Bash one-liner for renaming high-res image files to @2x"
date: 2012-03-21 13:54
comments: true
categories: bash, iOS, trick
---

This tip may be useful to iOS developers when they're given a bunch of high-resolution files and they need to rename them with the ***@2x*** filename suffix.

Supposing the files are all in the same directory (containing only the files you need to rename), just do this in the terminal:

	for file in *; do mv "$file" "${file%.*}@2x.${file##*.}"; done

As a result, the files will all be renamed following the pattern `<name>@2x.<extension>`.
