---
author: kernyan
comments: true
date: 2019-08-09 07:47:39+00:00
layout: post
link: http://kernyan.com/2019/08/09/clang-format-style-option-listings/
slug: clang-format-style-option-listings
title: clang-format style-option listings
categories:
- Others
---




While perusing the [clang-format documentation](https://clang.llvm.org/docs/ClangFormatStyleOptions.html) today, I noticed some very exciting features such as "AllowAllArgumentsOnNextLine", " AllowAllConstructorInitializersOnNextLine", and "AllowShortIfStatementsOnASingleLine (ShortIfStyle)".







Eager to try out these new features, I setup a .clang-format file with the above style-options, only to find out that the clang-format binary on my machine wasn't up to date. Turns out that the clang documentation was actually for clang 9.0, for which the associated binaries haven't been officially released yet. 







Looking around the clang docs, I also realized that while each of the style-options supported across the clang versions are documented, there wasn't a single changelog-like document. 







Such an information would be particularly useful since aside from new style-options being added, existing ones could also 1) be renamed, or 2) have their option settings upgraded from boolean to enum. 







Perhaps there's already a style-option listing somewhere, if so please let me know. In the meantime, I wrote a python script to pull the style-option interface and curated them [[here]](https://github.com/kernyan/clang-format-style-options) (link tables new additions, changes to interface, and total listing of the clang-format style-options). 







For clang 9.0 in particular,







New for clang-format 9







  * AlignConsecutiveMacros (bool)
  * AllowAllArgumentsOnNextLine (bool)
  * AllowAllConstructorInitializersOnNextLine (bool)
  * AllowShortLambdasOnASingleLine (ShortLambdaStyle)
  * TypenameMacros (std::vectorstd::string)
  * NamespaceMacros (std::vectorstd::string)
  * SpaceAfterLogicalNot (bool)






Changed interface for clang-format 9







  * from: AllowShortIfStatementsOnASingleLine (bool)
  * to: AllowShortIfStatementsOnASingleLine (ShortIfStyle) 


