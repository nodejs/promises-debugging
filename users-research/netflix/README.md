# Netflix promises error handling survey

This document presents the context and the findings of [a recent
survey](https://docs.google.com/forms/d/1lM4TdTJcRbW2CDZw7Yj4Su75joGvK6DIkD95CenVRmk/viewanalytics)
about promises error handling use cases that was done internally at Netflix.

The goal of that survey was to get a better understanding on what promises error
handling modes [recently proposed for the Node.js
runtime](https://github.com/nodejs/node/pull/20097) are relevant for users, and
how they would impact their ability to run applications reliably at Netflix
while providing a good developer and operators experience.

We hoped that it would help us inform the conversation in that PR, especially on
aspects that did not reach broad consensus yet.

The first section describes how this survey was conducted. Then, we provide a
summary of the results that highlights what we think is relevant and
interesting, but without any interpretation.

The third section interprets those key items that we found interesting and
presents what we think the key learnings can be.

Finally, we conclude with the set of recommendations we want to make for the
Node.js PR mentioned above and what other action items we think may be worth
tackling in the future.

## Disclaimer

We're aware that this survey doesn't follow best practices in its methodology
and implementation. We didn't expect this survey to provide us with a
statistical confidence in its results.

This is OK because the goal of this survey was to get a _general sense_ of what
promises users inside of Netflix care about when it comes to error handling use
cases, not an accurate picture of what all users inside and outside of Netflix
actually rely on.

## Methodology

We used Google forms to conduct the survey. The total number of respondents is
32 so far. The survey had been open for > 2 weeks at the time we collected and
interpreted results.

No question was mandatory, which explains why some questions have more answers
than others.

In addition to that, the question about asynchronously adding catch handlers to
promises was added when the survey had already been filled out by half of the
current number of  respondents. We consider this to be acceptable, and we’d
rather have some data about this question than no data.

The survey didn't target any specific population of JavaScript users at Netflix.
This was a conscious decision, as one of the goals was to get feedback from
JavaScript users that had diverse technical backgrounds and that didn’t have the
same biases as the users with which the team that conducted the survey interact
on a day to day basis.

In parallel of this survey, we did a small number of 30 minutes in-person
interviews with four selected respondents to go deeper into some of the answers
that were given when we thought that they could give us more insights.

Those interviews were not recorded. We understand that this is a missed
opportunity and that it lowers the credibility of this document, particularly
when it comes to conclusions that have been significantly influenced by those
interviews. We will strive to do better in the future.

## Summary of survey results

This section summarizes in prose the data that was collected by the survey and
that we consider interesting. The next section (Interpretation of survey results
and interviews) focuses on what we think that data means.

For raw data, please consult the [survey
results](https://docs.google.com/forms/d/1lM4TdTJcRbW2CDZw7Yj4Su75joGvK6DIkD95CenVRmk/viewanalytics).

### Promises usage

- Most of the respondents (90%) use promises. This could be because the survey
  was presented as a "promises survey", and thus users who didn't use promises
  may have self-selected out. There are at least three counter examples of this
  though, so this concern might not be valid.

- Most respondents of promises seem to use native promises, but a non-negligible
  number of respondents use non-native promises. Among non-native promises
  implementations, Q and Bluebird seem to have the most users.

- All respondents who are not using promises are using Rx observables instead.
  It's not clear whether the two serve identical use cases, but those users seem
  to at least have identified some overlap between the two and preferred Rx
  observables for several reasons:

  - Rx observables seem to be cancellable while promises are definitely not.
  - Historical context: promises were not necessarily available when those users
    had to pick a similar abstraction.
  - Issues with the error handling model of promises.

### Async/Await usage

- A significant number of respondents (~70%) seem to use async/await.

- For others, reasons not to use async/await include:
  - Issues with the error handling model.
  - Runtime not supporting them (old version of JavaScript Core).

### Promises error handling

- A significant portion of respondents seem to differentiate between fatal and
  non-fatal errors when using promises.

- A significant number of respondents seem to use different classes of errors to
  represent the difference between fatal and non-fatal errors.

- A significant number of respondents consider that they should always attach a
  catch handler to a promise, however most of them don't seem to enforce it via
  tooling (e.g. static analysis tools).

- Only one respondent mentioned that they add catch handlers asynchronously.
  When discussing their use case for it, they mentioned that they could achieve
  similar functionality differently (e.g. by emitting/listening on a 'error'
  event), and that the ability to add catch handlers _asynchronously_ was only a
  matter of user experience.

- A significant number of respondents seem to consider that exiting the Node.js
  process on an unhandled promise rejection is a desirable behavior.

- Most of the respondents that add an `'unhandledRejection'` event listener use
  it to log and exit the process.

- However, most respondents seem not to use it, and a non-negligible part of
  them were not aware that the `'unhandledRejection'` event exists.

- More respondents seem to be adding a listener for the `uncaughtException`
  event, and those handlers also almost always log messages and exit the
  process. However, a non-negligible number of respondents are not aware that
  this event exists.

- No respondent seems to consider that exiting the Node.js process once a
  promise with an unhandled rejection is garbage collected is a desirable
  behavior.

## Interpretation of survey results and interviews

This section presents _our interpretation_ of the data that was collected and
the four interviews we had with a selected sample of respondents.

### Promises are used significantly at Netflix

It is clear that a lot of JavaScript developers use promises at Netflix,
including when writing Node.js applications. It also seems like a significant
number of users are at least considering to use async/await.

### Promises are confusing for users

However, it's also clear that promises are confusing, even for developers who
have a lot of experience with them. Every single one-on-one conversation we
had with promises users highlighted at least one area of confusion with regards
to how different JS runtimes implement promises and how they handle their
errors.

### Developers don't use static analysis tools to validate promises usage

So far, we have seen that a significant number of users do not rely on static
analysis tools to help them validate their usage of promises. As a result, it
seems they _exclusively rely on runtime checks_ to perform this validation.

### Exiting Node.js process on unhandled rejection seems to be considered as valid behavior

A lot of respondents seem to be comfortable with Node.js exiting when a promise
rejection is unhandled. The motivation for this behavior includes having better
visibility on potentially hidden errors, and ensuring the process' state is
consistent.

The fact that the process exits synchronously or not does not seem to matter
much for users.

Some users are not comfortable with having the runtime dictate the behavior of
the process when an unhandled rejection occurs, but they seem to be fine with
being able to override the default behavior.

### Exiting on garbage collection doesn't seem to be considered as a desirable behavior

As part of the survey, no respondent chose this mode as a desirable behavior of
the runtime. Moreover, without giving any hint, every interview highlighted that
this was considered a potential source of confusion. We failed to find evidence
of respondents who considered this was a desirable behavior of the runtime.

### Attaching catch handlers asynchronously doesn't seem to be a required use case

Only one respondent mentioned it as an important use case. However, after
discussing this use case with them, they mentioned that the ability to
asynchronously add catch handlers to promises wasn't a functional requirement,
and that it was only a nice to have in order to provide better UX.

## Conclusions

In this section, we present what we think are relevant conclusions based on this
survey that can inform future discussions in the [PR recently proposed for the
Node.js runtime](https://github.com/nodejs/node/pull/20097) and more generally
on the topic of promises debugging and error handling.

### No evidence that exiting on GCed unhandled promise is desirable

We haven't managed to find evidence of exiting on GC being a valid use case. As
a result we are still leaning towards recommending not to add this behavior to
the Node.js runtime.

### Exiting on unhandled rejections could be well accepted by Node.js users

We haven't found any evidence that, when providing the ability to override this
behavior, having the runtime defaults to exit on unhandled rejections is
undesirable.

As a result, we will continue to lean towards recommending this as the default
behavior for the Node.js runtime, as well as providing the ability to add a
custom override.

### Advocating for using static analysis tools could help

It seems that using static code analysis tooling is not a common way for
developers to validate the robustness of their code regarding promises error
handling.

If such static analysis tooling was broadly used, we could at least inform
runtime behavior changes based on behavior that is enforced at build time.

As a result, it seems worth it to explore advocating for using useful static
code analysis tooling that encourages the best practices that we’d like the
broader community to use and that are compatible with the debugging tools we’d
like to use.

### Focus on developers education may help reach consensus in the future

All our conversations with promises users showed that there are a lot of
different perspectives on why to use promises, how they work, and what best
practices to use.

Most of the time though, when digging deeper and when providing additional
insights, it seemed that all interviewees reached a common ground on those
topics.

Our takeaway is that socializing challenges and best practices around using
promises and their error handling model could go a long way towards building a
shared understanding about promises and reaching a consensus on those topics.

We think reaching a broader consensus will help steer the Node.js platform
towards a more consistent behavior with regards to how promises are supported.
We hope to be able to work on enabling those conversations in the near future.

