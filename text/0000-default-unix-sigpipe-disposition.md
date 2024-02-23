I used to be in the "change default in a new edition" camp. After investigating more in depth, I have changed my mind. I now think we should keep the current default and use `unix_sigpipe` to cover all use cases.

## Arguments against changing the default

* I think the strongest argument against changing the default is that it would make Rust code less portable across platforms. If we changed the default, a test like https://github.com/rust-lang/rust/blob/master/tests/ui/process/sigpipe-should-be-ignored.rs would need to have "if unix: // run-fail, if windows: // run-pass".

* Another strong argument against changing the default is that if `SIG_IGN` is active in a program that needs `SIG_DFL`, there are clear indications (`BrokenPipe`) of what the problem is that makes it easy search for solutions. This makes `unix_sigpipe` discoverable. If `SIG_DFL` (or "inherit") is the default, the symptoms are much more subtle and harder to diagnose. The program just suddenly terminates without any output.

* Another argument against changing the default is that according to the [2023 survey](https://blog.rust-lang.org/images/2024-02-rust-survey-2023/technology-domain.png) Rust is used more for network/cloud programming than for writing textual command line tools. And networked code want `SIG_IGN`.

* Another argument against changing the default is that the default has been around since 2014. We should not underestimate the churn a new default would create.

## Solutions to problems related to not changing the default

* For users writing unix-y command line apps, it is easy to discover and use `#[unix_sigpipe = "sig_dfl"]`.

* For users that want that want minimal seccomp filters, or for users that want to know the original `SIGPIPE` disposition, or for users that on the basis of principle don't want the Rust runtime to change process-global state, it is easy to use `#[unix_sigpipe = "inherit"]`.

* For users that use a non-Rust main and links in Rust library code, we don't have an elegant solution I'm afraid. But since these will typically be C and C++ apps, it will be easy for them to adjust the `SIGPIPE` signal disposition themselves. And by using a non-Rust main, those users are already in some sense on their own and have opted out of the nice and cozy pure Rust world.

[1]: https://blog.rust-lang.org/images/2024-02-rust-survey-2023/technology-domain.png

## Proposal of next steps

I think we should
- [ ] Stabilize `#[unix_sigpipe = "sig_dfl"]`: https://github.com/rust-lang/rust/pull/120832
- [ ] Document the implications of the current default of. I think https://github.com/rust-lang/reference/pull/1465 is sufficient.
- [ ] Stabilize `#[unix_sigpipe = "inherit"]` after [bike-shedding](https://github.com/rust-lang/rust/pull/120832#issuecomment-1949830375) then name a bit more.
- [ ] Possibly stabilize `#[unix_sigpipe = "sig_ign"]` if there turns out to be a need for it. It is not clear to me there is a need since that is the default.
- [ ] Close this issue as resolved.

# Remarks

> This dates back to when Rust had a runtime and a green-threads implementation, and more magic handling of I/O.

I would like to point out that the fix was not made because of green-threads or to make I/O handling magic. It seems to me that it mainly was made to make Rust code more cross-platform. Making the behavior more in line with libgreen was more of a bonus outcome worth a remark, by my estimation.

> Rust currently resets the signal mask in a child process

> The same goes for signal masking. IMO this behavior should just be removed.

The code that messed around with the signal mask was [removed](https://github.com/rust-lang/rust/pull/101077) 14 months ago, so this is no longer a concern.






I used to be in the "change default in a new edition" camp. After investigating more in depth, I have changed my mind. I now think we should keep the current default and use `unix_sigpipe` to cover all use cases.

# Arguments against changing the default

* I think the strongest argument against changing the default is that it would make Rust code less portable across platforms. If we changed the default, a test like https://github.com/rust-lang/rust/blob/master/tests/ui/process/sigpipe-should-be-ignored.rs would need to have "if unix: // run-fail, if windows: // run-pass".

* Another strong argument against changing the default is that if `SIG_IGN` is active in a program that needs `SIG_DFL`, there are clear indications (`BrokenPipe`) of what the problem is that makes it easy search for solutions. This makes `unix_sigpipe` discoverable. If `SIG_DFL` (or "inherit") is the default, the symptoms are much more subtle and harder to diagnose. The program just suddenly terminates without any output.

* Another argument against changing the default is that - according to the 2023 survey [1] - Rust is used more for network/cloud programming than for writing textual command line tools. And networked code want `SIG_IGN`.

* Another argument against changing the default is that the default has been around since 2014. We should not underestimate the churn a new default would create.

# Solutions to problems related to not changing the default

* For users writing unix-y command line apps, it is easy to discover and use `#[unix_sigpipe = "sig_dfl"]`.

* For users that for ideological reasons don't want the Rust runtime to change process-global state at all, or for users that want minimal seccomp filters, or for users that want to know the original `SIGPIPE` disposition, it is easy to use `#[unix_sigpipe = "inherit"]`.

* I think TODO sufficiently documents the current default?

* For users that use a non-Rust main and links in Rust library code, we don't have an elegant solution I'm afraid. But since these will typically be C and C++ apps, it will be easy for them to adjust the `SIGPIPE` signal disposition themselves. And by using a non-Rust main, those users are already in some sense on their own and have opted out of the nice and cozy pure Rust world.

[1]: https://blog.rust-lang.org/images/2024-02-rust-survey-2023/technology-domain.png

# Proposal of what the next steps are

I think we should
- [ ] Stabilize `#[unix_sigpipe = "sig_dfl"]`: https://github.com/rust-lang/rust/pull/120832
- [ ] Document the implications of the current default of. I think https://github.com/rust-lang/reference/pull/1465 is sufficient.
- [ ] Stabilize `#[unix_sigpipe = "inherit"]` after [bike-shedding](https://github.com/rust-lang/rust/pull/120832#issuecomment-1949830375) then name a bit more.
- [ ] Possibly stabilize `#[unix_sigpipe = "sig_ign"]` if there turns out to be a need for it. It is not clear to me there is a need since that is the default.
- [ ] Close this issue as resolved.

# Remarks

> This dates back to when Rust had a runtime and a green-threads implementation, and more magic handling of I/O.

I would like to point out that the fix was not made because of green-threads or to make I/O handling magic. It seems to me that it mainly was made to make Rust code more cross-platform. Making the behavior more in line with libgreen was more of a bonus outcome worth a remark, by my estimation.

> Rust currently resets the signal mask in a child process

> The same goes for signal masking. IMO this behavior should just be removed.

The code that messed around with the signal mask was [removed](https://github.com/rust-lang/rust/pull/101077) 14 months ago, so this is no longer a concern.







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
