# Snowflake IDs

<!-- Fill these out once the PR has been created. Remember to remove comments once they're no longer applicable. -->

- Status: Pending
- Start Date: <!-- Insert today's date here, YYYY-MM-DD -->
- RFC PR: [#0](https://github.com/loki-chat/rfcs/pull/0) <!-- Update with link to PR after merging. -->
- Tracking issue: N/A <!-- When this RFC is made active, add link in the form of [<repo name>#0](https://github.com/loki-chat/<repo name>/issues/0) if applicable. -->
- Supercedes: N/A <!-- Path to the RFC(s) that this one made obsolete, if applicate. Format: [0000](../obsolete/0000-name.md) -->

## Summary

This RFC specifies a singular ID format to be used across Loki, for things like messages and users, called snowflakes. These are proven to work well for avoiding ID collisions, even across distributed systems, by their usage at companies like Twitter and Discord.

## Basic example

The ID 36889232862682412 may be deconstructed as follows:

| Timestamp in milliseconds, UTC (42 bits)   | Worker ID (10 bits)  | Counter (12 bits) |
| ------------------------------------------ | -------------------- | ----------------- |
| 000000001000001100011001111111100100010100 | 1110010110           | 111100010110      |
| 8798075156                                 | 918                  | 3862              |

We can get the Unix time of the ID by adding the Unix time of our epoch, which is midnight, January 1st, 2023. 8798075156 + 1672531200000 gives us 1681329275156. This gives us a correct time of April 12th, 2023, at 19:54:35.156 (7:54 PM), in UTC.

## Motivation

Standardizing on a single ID format ensures consistency, it means that existing ID-related code is more easily reusable, and it prevents us from reinventing the wheel in separate parts of the codebase.

An ID's job being to identify a certain thing, and only that thing, it should be unique. This should ideally also be scalable, such that two workers have zero chance of assigning two separate things the same ID and causing problems.

Finally, an ID beginning with the timestamp can easily be sorted by, and provide a reference point to get messages before and after it.

## Detailed design

Snowflakes are stored in 64-bit unsigned integers. The client may only depend on the timestamp, as the other parts may be changed without warning.

### Timestamp

Bits 63 through 22 are used for the timestamp. The epoch for this timestamp is midnight, January 1st, 2023. The Unix timestamp for this epoch is 1672531200000. This means that to convert to Unix time, the timestamp of the epoch should be added, and vice versa. This has space for times up to May 15th, 2162.

The timestamp of an ID may never be less than that of an ID generated earlier with the same worker ID. IDs across separate workers should be sufficiently close for the purpose of sorting, ideally less than 1 second.

In cases where only signed integers are available, the first bit of the timestamp may be ignored by the client, to account for the sign bit.

### Worker ID

Bits 21 through 12 are used for the worker ID. This is an internal ID assigned to the specific process that generated the ID by the backend, and must be unique for each process that generates IDs. The manner in which this is assigned is unspecified.

### Counter

Bits 11 through 0 are used for the counter. The counter increments each time an ID is generated. It may never decrease, except for when the clock used for the timestamp ticks to the next millisecond, in which case it is reset to zero. The counter may never overflow. If it reaches the 12-bit limit of 4095 before the timestamp clock has ticked to the next millisecond, no more IDs may be returned until this happens.

## Drawbacks

- 64 bits may be considered too little space. It should however be possible to migrate to longer IDs later while preserving then-legacy ones.
- The possibility of having federation in the future means that instances might end up sharing IDs, and this format does not identify the origin instance. This could mean a large change in how IDs work and/or are managed in case this is implemented.
- Clients have to keep track of which instance an ID belongs to.

## Alternatives

What other solutions have been considered? Why this solution rather than them? What is the impact of not doing this?

- A custom format was proposed, though it was discarded for being unproven and providing little benefit over snowflakes.
- Including a hash of the origin instance's domain name was considered, assuming federation would be supported, though it was dismissed for its high risk of hash collisions, dealing with which would mean taking bits away from the other parts of the ID that provide more effective means of ID collisions.
- UUIDv7 was proposed, though it does not appear to have much benefit over snowflakes, while being twice as long.
- Not specifying an ID loses the benefits mentioned in the [Motivation](#motivation) section.

## Prior art

[Twitter](https://developer.twitter.com/en/docs/twitter-ids), [Discord](https://discord.com/developers/docs/reference#snowflakes), [Instagram](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c) and [Mastodon](https://github.com/mastodon/mastodon/blob/main/lib/mastodon/snowflake.rb) all use snowflakes with a few variations here and there, demonstrating that they are proven and reliable.
