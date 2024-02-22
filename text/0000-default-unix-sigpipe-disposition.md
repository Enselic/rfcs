I think we should keep the default, because

* It is easy to fix with `unix_sigpipe`
* It reduces inconsistencies between platforms
* It has been the default since 2014 and changing it would create churn that can't be justified
* 

Here is a summary of the pros and cons of changing the default. I intend to keep it up to date and exhaustive. Feel free to extend the lists if you have edit rights on comments.

Resolving this issue might need an RFC, but I think a summary like this is a good start.

My long term goal is to remove uncertainties related to https://github.com/rust-lang/rust/pull/120832 by doing necessary preparations to resolve this issue.

# Keep `SIG_IGN` as the default

## Pros

* If `SIG_DFL` behavior is wanted, it is easy to find `#[unix_sigpipe = "sig_dfl"]` by searching on the internet for ErrorKind::BrokenPipe
* Keeps Rust behavior more consistent across platforms. Wanting `SIG_DFL` is a in a way a niche unix-specific use case. It makes sense to opt-in to it.
* Avoid ecosystem churn
* Using Rust for network programming is more common than for command line utilities [1]
* It is easier to figure out that `SIG_IGN` is your problem than that `SIG_DFL` is your problem

## Cons

* Writing command line apps with textual output becomes annoying. (But easy to fix with the new attribute)
* Changing `SIGPIPE` disposition which is process global state is unexpected for a language that prides itself in not having a runtime
* Rust library code does not behave the same as Rust bin code. If you use non-Rust code that becomes your problem though.

# Reasons to not change `SIGPIPE` at all by default (in 99.9% of cases this means leaving it as `SIG_DFL`)

* Make Rust library code behave the same as Rust bin code
* It is harder to properly handle `SIGPIPE` when its disposition is `SIG_IGN` than when it is `SIG_DFL`
* seccomp filters don't have to allow `lib::signal`
* Applications can get the original disposition of `SIGPIPE`
* 

# Reasons to make `SIG_DFL` the default

* Fill in later, but I don't think this makes sense to do.


If we change the default in a new edition, it will be easy to opt-in to the current behavior by using the `#[unix_sigpipe = "sig_ign"]` attribute.


# Drawbacks of current default (`SIGPIPE` is set to `SIG_IGN`)

Roughly ordered by importance/relevance.

* This changes the default behavior of pipes from what people might expect when writing UNIX applications.

[1]: https://blog.rust-lang.org/images/2024-02-rust-survey-2023/technology-domain.png


> This dates back to when Rust had a runtime and a green-threads implementation, and more magic handling of I/O.

I would like to point out that the fix was not made to make I/O handling magic. Instead, it was considered undesired that a program could suddenly get killed just because you messed around with processes and pipes.
So my estimation is that SIGPIPE was not `SIG_IGN` _primarily_ because libgreen did so.

> Rust currently resets the signal mask in a child process

Not any more. TODO: link


There are some different outcomes here:

## If we decide to keep the default

* We can close this issue after adding documentation (maybe https://github.com/rust-lang/reference/pull/1465 is enough)
* We can stabilize `#[unix_sigpipe = "sig_dfl"]`
* We can stabilize `#[unix_sigpipe = "inherit"]` since that will be the only way to minimize `seccomp` filters. We might want to change name to `#[unix_sigpipe = "inherit"]` first
* We can maybe stabilize `#[unix_sigpipe = "sig_ign"]`. Let's discuss it separately.

## If we decide to change the default

* We can stabilize `#[unix_sigpipe = "sig_ign"]` since we need it to opt-in to the old behavior.
* We can maybe stabilize `#[unix_sigpipe = "sig_dfl"]`. Let's discuss it separately.
* We can maybe stabilize `#[unix_sigpipe = "inherit"]`. Let's discuss it separately.











- Feature Name: SIGPIPE disposition not changed by default
- Start Date: 2024-02-18
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

This RFC proposes to stop changing the disposition of `SIGPIPE` (to `SIG_IGN`) by default.

If accepted, the default will be changed in a new Rust edition, with an easy way to opt-in to the old behavior.

If accepted and fixed, we can close https://github.com/rust-lang/rust/issues/62569.

# Motivation
[motivation]: #motivation

Not everyone wants the default `SIGPIPE` disposition to be `SIG_IGN`. The current default was decided in 2014 when Rust still had green threads. There are good reasons to change the default.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Explain the proposal as if it was already included in the language and you were teaching it to another Rust programmer. That generally means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how Rust programmers should *think* about the feature, and how it should impact the way they use Rust. It should explain the impact as concretely as possible.
- If applicable, provide sample error messages, deprecation warnings, or migration guidance.
- If applicable, describe the differences between teaching this to existing Rust programmers and new Rust programmers.
- Discuss how this impacts the ability to read, understand, and maintain Rust code. Code is read and modified far more often than written; will the proposed feature make code easier to maintain?

For implementation-oriented RFCs (e.g. for compiler internals), this section should focus on how compiler contributors should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

# Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
- If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain?

# Prior art
[prior-art]: #prior-art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- For language, library, cargo, tools, and compiler proposals: Does this feature exist in other programming languages and what experience have their community had?
- For community proposals: Is this done by some other community and what were their experiences with it?
- For other teams: What lessons can we learn from what other communities have done here?
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

This section is intended to encourage you as an author to think about the lessons from other languages, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other languages.

Note that while precedent set by other languages is some motivation, it does not on its own motivate an RFC.
Please also take into consideration that rust sometimes intentionally diverges from common language features.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- What parts of the design do you expect to resolve through the RFC process before this gets merged?
- What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
- What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

# Future possibilities
[future-possibilities]: #future-possibilities

Think about what the natural extension and evolution of your proposal would
be and how it would affect the language and project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project and language in your proposal.
Also consider how this all fits into the roadmap for the project
and of the relevant sub-team.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
