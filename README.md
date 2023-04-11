# Loki RFCs

## What is an RFC?

A **R**equest **F**or **C**omments (RFC) is a proposal to specify some large decision made in relation to Loki. It allows us to establish common understanding and encourages documenting that certain decisions have been made.

Most things, like regular feature ideas, can be done through issues, and probably do not need RFCs.

## Lifecycle of an RFC

RFCs have four stages:

- **Pending:** The RFC has been submitted as a pull request.
- **Active:** The RFC pull request has been merged and will be implemented/enacted.
- **Landed:** The proposed RFC has been enacted or implemented in a shipped release.
- **Rejected:** The RFC pull request has been closed.
- **Obsolete:** The RFC was previously active or landed, but has been superceded or is otherwise no longer relevant.

## When to follow this process

You should create an RFC for substantial decisions. These are mainly decisions that affect a large portion of the project, such as the ID format. It is always advised to ask first in the Discord server.

Some decisions probably do not require an RFC, such as implementation details specific to one part of the project.

## The process

1. Discuss your proposal in the Discord server. This will help you see if your proposal is desired and agreed with, allow it to mature, and reveal potential altenatives.
2. Create your RFC.
	- Create a new branch, or fork the repo.
	- Copy `0000-template.md` to `0000-my-proposal.md`, replacing `my-proposal` with a name for your RFC. Don't assign a number just yet.
	- Write your RFC. Put care into this step. If you're unmotivated, show a lack of understanding, or are disingenuous about drawbacks or alternatives, your RFC is more likely to be rejected. Feel free to read existing active RFCs to gain a better understanding of what is expected.
	- Once you're done, submit a pull request outlining the basics of your proposal.
	- Now that you've made a pull request, fill out the fields at the top of it based on the instructions in the comments.
3. Receive feedback.
	- You can modify your RFC based on feedback from others, and changes may be requested.
	- Once the team feels the RFC has been discussed thoroughly and that it is at a point where it is largely agreed upon, a member may squash and merge the pull request. The pull request may also be closed if the RFC is deemed undesired, or a different RFC replaced it.

## Active RFCs

Once an RFC becomes active, a tracking issue may be created in the appropriate repository, if relevant. A link to this issue should then be added to the RFC. Once this issue has been marked as completed, the RFC may have its status changed to landed.

Small modifications to active RFCs can be done through followup pull requests. We should strive to write each RFC such that it reflects the final design of the feature, however, new ideas and such may come up that warrant changing an RFC. Major changes, to the extent that they would substantially change the RFC, might warrant a new RFC that would result in the original being marked as obsolete.

## Implementing an RFC

The author of an RFC is not obligated to implement it. Of course, they are welcome to, as is any other developer, once the RFC has been accepted.

If you want to implement an RFC, but are unsure about whether or not someone is already working on it, you should check the tracking issue for it to see if someone has been assigned. If not, you can ask on the issue or in the Discord server.

## Acknowledgements

This RFC process is heavily inspired by and based on those of [Vue](https://github.com/vuejs/rfcs) and [Rust](https://github.com/rust-lang/rfcs).