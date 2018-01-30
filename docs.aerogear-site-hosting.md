# Hosting of docs.aerogear.org web site

## Introduction

Previously all AeroGear docs were hosted on a file server and associated with the domain. We'd like to take advantage of github for hosting for reliability and ease of use.

## Problem Description

Hosting html (and associated files) for docs.aerogear.org is essential for communication with users. Need to automatically publish any changes to mobile-docs repo to docs.aerogear.org.

## Expectations

- If we take an approach used by https://github.com/debezium/debezium.github.io where asciidoc and html files are both in same repo (adoc in develop branch, html in master branch), we can anticipate the repo getting large over time with history of html changes mixed with history of adoc file changes.
- There won't be a version associated with combined components of mobile.next, ie there won't be an Aerogear v1.0 release

  
## Proposed Solution

Create a mobile-docs-html repo and use it to host docs.aerogear.org html files.

Use a CI service (eg circleci) to run asciibinder and push the output to the mobile-docs-html repo.

