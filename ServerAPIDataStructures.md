## Table of contents
* [Index](https://righttoaskorg.github.io/righttoask-docs/index)
* [Features](https://righttoaskorg.github.io/righttoask-docs/Features)
* [Screens and Usage](https://righttoaskorg.github.io/righttoask-docs/ScreensAndUsage)
* [Security and privacy](https://righttoaskorg.github.io/righttoask-docs/SecurityAndPrivacy)
* [Direct messaging](https://righttoaskorg.github.io/righttoask-docs/DMs)
* [Discussion and further questions](https://righttoaskorg.github.io/righttoask-docs/DiscussionAndFurtherQuestions)
* [Prototype](https://righttoaskorg.github.io/righttoask-docs/Prototype)
* [API](https://righttoaskorg.github.io/righttoask-docs/API)

# Server API Data Structures



<a id="data_structures_overview"/>
## Main Data structures

The server's internal data structures correspond closely to the data structures posted on the Bulletin Board.  However, since many public structures such as questions, tallies and public keys can be updated, the server needs to keep track of when multiple different bulletin board entries correspond to the same data item, for example updates or tallies for a question.  

Each data structure therefore has specific immutable fields which define it - for example, an entity is defined by its name. The hash of the defining data is returned when the data is first posted on the Bulletin Board, then used as an index for subsequent updates.

**This repo contains intended descriptions for things not yet built. For existing server API data structures, including users, questions, and bulletin board data structures, see the [rust docs for the server repo](https://github.com/RightToAskOrg/right_to_ask_server/).**

### Ballot

We will use the same ballot data structure as ElectionGuard.  (VT: Note, these have changed - get updates.) Something like this:

```javascript
ballot = {
    ballot_style: "ballot-style-1", 
    contests:
        [
            {
                object_id: "contest-1-id"
                ballot_selections:
                    [
                        {
                            object_id: "contest-1-yes-selection-id",
                            vote: 1
                        },
                        {
                            object_id: "contest-1-no-selection-id",
                            vote:0
                        }
                    ],
            }
        ]
}
```

**VT: the following are very negotiable, with the clear intention of making sure EG compatibility is easy, esp including John Caron's verifier**

The **contest-id** should be a hash of the question - H(Q).  We will only ever have two selections: up and down.  The selection IDs can therefore be very simply encoded using only one more bit (or perhaps an 'up' or 'down' string if that seems helpful). The hash should (perhaps) include the date, but should _not_ include other data such as links and tags, because these can change after the question is posted.

We won't need an explicit 'abstain' option - we'll be able to tally the upvotes and the downvotes (regardless of whether they are an explicit downvote or a swipe-aside).  Then it will be immediately evident how many votes there were with two zeros, since the overall total votes will be obvious.

**Question / thing to check: Can EG and/or JC's verifier aggregate some, but not necessarily all, contests on a ballot? And can it aggregate some, but not all, of the votes already received?** I assume the answer must be yes, because many US jurisdictions give slightly different ballots to different voters according to different addresses, but we may have an extreme version because each participant might vote on a completely different subset of questions. I think this effectively means each participant makes up their own "ballot-style."

We'll want to be able to take a _contest_ and choose to aggregate and decrypt 
its votes, which may be spread across a lot of different positions on a lot of different ballots.

Every submitted ballot is signed.  The hash of the ballot and its signature, H(B | Sig) is inserted in the Merkle Tree / BB.



### Tallies

A tally is:

- A claimed total,
- Decryption proofs from each trustee,
- A clear statement of which votes were aggregated.  (**Check how EG does this - or does it assume that you always want to decrypt everything?)

Again, this is hashed, stored in the database, and posted on the BB.


