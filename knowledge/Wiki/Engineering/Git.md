---
title: "Git"
date: 2026-01-14
tags:
  - engineering
  - git
---

Git is a version control system that is used for tracking changes in computer files. It is mainly used for software development but it can be used for any type of file. Git stores and detects changes in files using a system called hashing.

When a file is added to Git, it is assigned a unique hash code that represents the contents of the file at that time. When changes are made to the file, Git generates a new hash code that represents the new version of the file. Git then stores both versions of the file in its database.

When you want to view the changes made to a file, Git compares the hash codes of the current version and the previous version of the file. Git then shows you the differences between the two versions of the file. This process is known as "diffing" and it is what allows Git to keep track of changes made to files over time.

In addition to storing and detecting changes in files, Git also allows users to revert changes that have been made to a file. This is done by checking out a previous version of the file using its hash code.

Overall, Git's ability to store and detect changes in files makes it an essential tool for any software development team. It helps teams to collaborate effectively and ensures that everyone is working with the most up-to-date versions of files.

  

## How git store change

Git uses various techniques, such as binary-diffing and compression algorithms, to further minimize the storage space required