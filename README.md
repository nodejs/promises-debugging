# promises-debugging

Promises have been available to Node.js developers for several years in various
forms (as a built-in feature of the language or as npm modules). However,
debugging errors created from promises tasks can still present some challenges
in some use cases.

As a result, an _informal_ working group was formed with the goal to make the
experience of Node.js developers less challenging when debugging such errors.

## Goals

The goals of this working group are:

- Identifying, investigating and proposing solutions for gaps in promises error
  debugging use cases.

- Documenting best practices about debugging promises errors.

## Scope

Promises debugging and error handling can cover a large surface area of the
Node.js runtime. We want to make sure this working group is focused on just the
critical parts of that surface area so that we can make concrete progress as
quickly as possible.

Other individuals or groups can tackle the challenges that are out of scope for
this group.

### In scope

- Ability to use post-mortem debugging techniques (using both OS level core
  dumps and other forms of artifacts) to debug unhandled errors created by
  promises tasks.

- Ability to use live debugging techniques to debug unhandled errors created by
  promises tasks.

### Out of scope

Any aspect of debugging promises that is not critical to _enabling_ debugging
use cases is not in scope. For instance:

- the performance of promises when using live debuggers is not relevant for this
  working group, even if it is very important for Node.js users.

- the user experience of using promises, even when it relates to providing a
  safer API, is not relevant to this working group. For instance, whether
  resolving or rejecting a promise that has already been resolved triggers an
  error is not relevant to this working group.

## Use cases (TODO)

### Post-mortem debugging (TODO)

#### Debugging unhandled rejections using OS-level core dumps (TODO)

#### Debugging unhandled rejections using other types of artifacts (TODO)

### Live debugging (TODO)

## Current state

A _huge_ amount of work has been done in the past few years to try to address
some of these challenges. This section tries to document those and give pointers
that allow to get more information about them.

The section is split into two sub-sections: the first about work that is
ongoing, and the second about work that has been completed or abandoned.

### Active work (TODO)

#### [src: exit on gc unhandled rejection](https://github.com/nodejs/node/pull/20097)

This PR adds support for various modes of error handling when a promise
rejection is not handled.

##### What needs to be done

* There seems to be no consensus around the suggested new `'ERROR_ON_GC'` mode.
  Objections have been presented for that mode and would need to be addressed:
    - https://github.com/nodejs/node/pull/20097#issuecomment-398838519
    - https://github.com/nodejs/node/pull/20097#issuecomment-398851206
    - https://github.com/nodejs/node/pull/20097#discussion_r196603315

### Completed or abandoned work (TODO)

#### [lib,src: "throw" on unhandled promise rejections](https://github.com/nodejs/node/pull/6375)

This PR implemented a similar approach to the active PR above ("src: exit on gc
unhandled rejection). It was closed in favor of
https://github.com/nodejs/node/pull/12010.


