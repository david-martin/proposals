# PR testing guidelines and checklists 

## Introduction

This proposal should cover the PR testing of the repositories concerning the new Aerogear/Feedhenry developments.
It was based on a discussion between Adam Saleh, Vitali Chepeliuk, Pavel Sturc and Vojtech Sazel.

## Problem Description

We currently don't have a proper set of guidelines for our PR checks and previously we used them mostly just to produce the build-artifacts
in some fashion. The quality checks have often been added in ad-hoc manner and then disregarded/forgotten about.

## Expectations

If we create a set of coherent guidelines and model our automated PR checks by them we expect
- to catch regressions early
- to maintain a good quality of incoming code

## Terms

* relevant repository - repository that could influence ship-able artifacts by it's contents in a bad way. If we really don't want to follow these guidelines, just claim that the repository is currently irrelevant for our project.
  
## Proposed Solution

These proposals only concern relevant repositories.

### Repository should have a mandatory review by two other developers

This stops rushed PRs and ensures knowledge of the change is spread across the team
In github it is easy to enforce at least one reviever as necessary. It is possible to implement multiple-reviewers policy through API,
but it might not be worth the effort.

There will be some decisions that we don't want to automate on purpose, for example, we will probably display code-coverage prominently,
but we won't have any automatically enforced that would stop PR from getting merged due to low coverage.

In the future, we might have optional integration tests triggered by comments or specific tags on the PR. This would be hard to enforce automatically,
so mandating those would fall as a responsibility of the reviewer.

There could be a template with a checklist delivered by a .github/PULL_REQUEST_TEMPLATE.md
Checklist would include:

* Read through the code and ask about any unclear code
* Evaluate if the new code is sufficiently tested, potentially consulting the code-coverage
* Evaluate if there are any follow-up tasks to be logged
* more?

The reviewer shouldn't need to checkout and run the code, we would mandate that our CI can cover this case automatically.

### All of the automated testing should finish in reasonable time, defined as in less than 10 minutes

Looking at our previous PR checking attempts, even fast checks take around 2 minutes, due to inconsistencies in time of spawning our on-demand slaves.

Our longest PR builds can take around 20 minutes.

Any time, there is PR check that consistently hits the 10 minute run-time, it should become an issue in our backlog to be solved.

### There should be a consistent and enforced style for our code

Maintaining a repository where multiple developers collaborate should be helped by ensuring consistent style.
While this often leads to a lot of bike-shedding, and changing the style after it has been applied can be tedious,
but the style enforcement still seems to be quite useful, especially for new collaborators.

For configuration files and dynamic languages, style checking can often find basic problems that would be discovered in compiled languages with type-checker.

For go, we will be using
* go vet
* go lint
* go fmt 
* errcheck

For jaascript, we will base our lint on the origin-web-console
* https://github.com/openshift/origin-web-console/blob/master/.jshintrc

We should decide how to enforce the style for:
* ansible, if some check exists
* bash (proposed by Pavel Sturc)

### If there is a build-able artifact, it should be built in reasonable subset of configurations

This mostly goes for binaries, such as our go-based cli tool, or compilable libraries for our mobile devices.
If compiler can build for multiple OSes or ARCHs, we should build all that we plan to release,
i.e. for go this would mean:
* linux
* windows
* mac

We wouldn't require that we need to run tests against all of the architectures.

We probably want to include things like Docker-files here as well.

### Unit tests should be run, results and coverage logged

As we mentioned previously, coverage level shouldn't be enforced automatically, but reviewers should look at the coverage and test quality in general.

### There should be at least some amount of integration testing, if at all possible

 We should run our code in some sort of staging environment that would be reasonable compromise between development and production environment, with emphasis on speed of running of the test, i.e. we don't want to wait 20 minutes for openshift to spawn.

### If there is an executable it should be executed outside of unit tests

This should be considered a minimal example of previous point. Unit-tests themselves often test only isolated bits of code in abstracted environment, we should ensure that the main entry-point to the application works.

### If there is installer, it should be run in a minimal, sanity-checking configuration

Another example of an integration-style test. Because previously in Feedhenry, we have had problems with installability of our dev-environment, if there is an install-script to be run, we should run it as a part of our PR check.

### If there is a deploy-able artifact, it should be deployed in a sanity-checking fashion

This can be considered another follow-up to the integration tests example. Because we will have many deploy-able artifacts such as Docker-files and Ansible playbook bundles, we should ensure the artifacts can actually be deployed.

### Guidelines for the speed/stability/comprehensiveness of the tests

With regards to testing and coverage
* Stable is more important than Fast or Comprehensive
* Fast and Comprehensive is equally important, let us be Pareto-efficient there ;)

Previously we have often fallen into the trap where we considered Comprehensive to be more important than Stable or Fast. This lead to test-suites that weren't trust-worthy and couldn't be run often enough.

### If there is an escape, regression or a bug found with a blocker priority, there should be a test that covers it in a PR-check, if possible.

There might be use to cover it with more tests, or more comprehensive tests in other test suites.

It might be impossible to track catch the high-priority bug in PR-check testing, in that case we should have a contingency to test this at release.
(Release checks are beyond scope of this proposal and should have their own proposal.)

## To be discussed

### What are other things that should be checked on specific types of repositories?

We have outlined some specific checks for several types of repositories, i.e.

* deployment tests for APBs
* installability tests for i.e. mobile-core

We haven't discussed 
* specific checks for our mobile-related repositories
  * mobile templates/quick-starts
  * mobile sdk's
* specific check for the UI-centric/browser based repositories
  * i.e. should we use selenium?

### How much should we enforce these guidelines?

There are probably more important and less important parts of these and for every repository the end-result w.r.t. PR checking will look different.

It still might be useful to have some of these enforced on global, organization-wide scale. We might use i.e. must/should priorities for this?

i.e for a project to be releasable it 
* MUST have mandatory review on PR's
* MUST have a mandatory PR-checking job
* all the build-able artifacts MUST be built
* all the unit-tests MUST be run
* some integration tests SHOULD be run

### How important is to be able to conditionally execute some more thorough tests?

We have previously discussed adding a capacity to run differently scoped tests for different PR request, based on either comments on PR or tags when the PR is created.

We currently don't have this capability, so either
* this is a high priority and we need this asap
* it is a nice to have for some things, like deploying new staging grids, that we definitely don't want to do on every PR
* we don't really want this, because we want to be forced to have consistent testing across all PR's, where no PR checking test would go stale.
