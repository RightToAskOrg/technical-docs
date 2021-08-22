## Table of contents
* [Index](https://righttoaskorg.github.io/righttoask-docs/index)
* [Features](https://righttoaskorg.github.io/righttoask-docs/Features)
* [Screens and Usage](https://righttoaskorg.github.io/righttoask-docs/ScreensAndUsage)
* [Security and privacy](https://righttoaskorg.github.io/righttoask-docs/SecurityAndPrivacy)
* [Direct messaging](https://righttoaskorg.github.io/righttoask-docs/DMs)
* [Discussion and further questions](https://righttoaskorg.github.io/righttoask-docs/DiscussionAndFurtherQuestions)
* [Prototype](https://righttoaskorg.github.io/righttoask-docs/Prototype)
* [API](https://righttoaskorg.github.io/righttoask-docs/API)



# Server API sketch

This file sketches the main ideas for a REST API for client-server interaction.  The key idea is to separate the data upload step from the inclusion-verification step.

We envisage a bulletin board (BB) which iteratively constructs a balanced Merkle Tree and periodically (upon request) produces a root hash, allowing for participants to compare their root hashes and verify the inclusion of data.

We will implement a Bulletin Board using the [rust Merkle Tree implementation by Andrew Conway](https://docs.rs/merkle-tree-bulletin-board/). The server communicates directly with this BB, and offers a public interface for some of its operations.

The BB builds a **Published root** of a Merkle tree when requested by the server.  

The BB has two classes of data, both represented by their hash values:

- **included values** are incorporated into a **published root**
- **pending values** are not yet included, but will be in the next published root.

The BB can execute (where n is the total number of leaves):

- order_new_published_root: a new **published root** - O(1)
- get_parentless_unpublished_hash_values: the set of (hashes of) **pending values** - O(log n)
- get_most_recent_published_root: the current **published root** - O(1)
- get_proof_chain: a proof of inclusion, for any **included value**, wrt the **published root** (even if the value was included several iterations in the past) - O(log n)
- a history proof, for any prior **published root**, of the inclusion of all of its data in the current **published root** - O(log^2 n) - combination of get_hash_info and get_proof_chain
- a history proof, for any prior **published root**, of the inclusion of that root in the current **published root** - O(|intermediate roots|) (This is not expected to be used much - get_hash_info suffices.)
- censor-leaf: removes a data item from the tree, while leaving its hash so that the removal is evident.

  These proofs work only on hashes of data, not on the posted data directly, because it is not required to be unique.

The main functionality of the RightToAsk client-server interaction is to post questions (and links, tags, or answers) or encrypted votes, and read others' questions and the decrypted tallies. A secondary function is to post and read public keys, for signing and verification of the uploaded data.  (At a later time, we may add public encryption keys for direct messages.) We also add verification of the BB state. 

(VT Note/TODO: Be consistent about whether a question or other data item is referred to by the hash of its data or the hash of its (data || sig).  (1) Consistency with Election Guard would be good, but (2) ease-of-use of BB would be good, which favours hashing (item+sig).  OTOH, if there are some items without sigs it might be annoying/complicated. Decision: H(data), no sig.)

First read the [Main Data structures](https://righttoaskorg.github.io/righttoask-docs/ServerAPIDataStructures).

## Post functions

Each upload is divided into two parts: data to be uploaded to the BB, and other optional metadata.  The BB data also needs the client's signature.  So a typical post, e.g. for account creation, question-posting, or voting, has this structure:

**Post**

- D = data
- CSig = signature<sub>Client</sub>(data)
- M = metadata

This receives an immediate acknowledgement from the server, of the form:

**Ack**

- SSig = signature<sub>server</sub>(h(D, CSig))

where h indicates a cryptographic hash function (e.g. SHA256), for use in the BB proofs.  This is a promise from the server that D, CSig  (or, more precisely, h(D,CSig)) will be included on the BB.

The role of the metadata M is to allow the client to upload extraneous info to the server without that info being posted on the BB.

At this point, the data should appear in BB.get_pending_hash_values.

(VT: not 100% clear what the data format should be.  Ideally it should allow for easy verification later, when the items are included in a published root.)

The exact requirements for the data obviously depend on the specific kind of message. For registration, data would include a name, public key and (optionally) electorate.  For voting, data would include vote ciphertexts with sufficient extra data to identify which questions they were answering.  For question-asking, the data is quite complex and would include the text of the question, the ID of the person asking, any tags or links, etc.  We will also add some more functionality for answering questions, marking them as answered, re-tagging them for other MPs, etc...

### New Question

- D =  Question
- M =  List(Enc(address, MP public key))

Responses: 

- Ack (with sig, as above), 
- Validity failure (details), 
- Permission failure (details). 
- Duplicate failure (h).

The server applies question's [validity and permission rules](#question_def) (which should also be checked by the client). 

(VT: Consider exactly how perfectly equal a duplicate should need to be in order to be automatically excluded.  I think the same Question_Text except whitespace, and the same other data. In particular, even an identical question with different background should probably be allowed. Update: this is determined by the Defining Fields - two questions are duplicates iff they  have the same defining fields.)

M is empty if the user has not opted in to sharing their address with their MP, or if they have not tagged their MP as either a MP_Who_Should_Ask_The_Question or Question_Answerer. Otherwise, M contains a list of ciphertexts containing the person's address, encrypted with the public encryption keys of each tagged MP.  (VT: Ask whether this is a useful enough feature for the bother. I suspect not, in which case M is empty.)

(VT: TODO: resolve inconsistencies with registration data structures - atm we don't have public encryption keys, just digital sig keys, but we'll need them for the DM feature. Need to think/ask about multiple public keys for staffers associated with one MP.)

### Edit Question

D includes:

- Question_Id: the hash used to identify the question, 
- Last_Update: the hash used to identify the last update.
- Edits: a list of (field, value) pairs, where the field is an element of the [question data structure](https://righttoaskorg.github.io/righttoask-docs/ServerAPIDataStructures) and the value is an update to be appended to the existing value. 

Responses: 

- Success: Question_Id, New_Last_Update, 
- Auth Failure (you're not permitted to do this update, which may be either because you don't have permission to update that field, or there are already too many answers of that kind), 
- Merge failure (you need to manually resolve X - for example, if two people have simultaneously updated a field that can only have one element).

All responses are signed by the server.

The server applies question's [validity and permission rules](#question_def) (which should also be checked by the client). 

(VT: Remember censorship when considering what conflicts with what - in some circumstances a moderator should be able to remove stuff. Definitely questions.  Possibly answers from MPs.)

Simultaneous update rules:

The merge rules for each field are:

- Question_Text, Person_Suggested_Question, Signature: Reject updates.
- Background: Reject updates unless currently blank.
- MP_Who_Should_Ask_The_Questions, Entity_Who_Should_Answer_The_Question: Eliminate duplicates (including with values already present). If the total number of values doesn't exceed the limit, accept. If the limit has already been exceeded, reject. If it hasn't, but would if this update was accepted, send a merge request back to the client (pick at most m out of the n you tried to submit...).  Note that this might cause cascading merges that need to be manually resolved, but that's less trouble than allowing locks.
- Answer: If there are simultaneous valid \& permitted updates, accept both (append), regardless of whether the MP is the person who was tagged.
- Answer_accepted: May be changed from false to true.
- Hansard link, Keywords, Topics: Same as MP_Who_Should_Ask_The_Questions, Entity_Who_Should_Answer_The_Question.

** Server Design **
A question that has been updated multiple times, e.g. by adding some background, an answer, or a hansard link, may have multiple different BB records.  It will be the server's job to keep track of all this.  Each Edit contains a Last_Update field, which points to the previous most-recent update for this question.  The server will store both the Question_Id (hashcode) and the Last_Update (hashcode), along with the aggregated current state of the Question. 

The Last_Update points to the most recent edit, which in turn points to the next-most-recent, etc, allowing the server iteratively to retrieve an inclusion proof for the complete question (initial post plus updates).

(TODO: update the inclusion proofs appropriately - an inclusion proof for one question may actually be an inclusion proof for multiple additions/edits, each represented separately on the BB.)

### Flag question
This is used to flag a question as deserving of censorship. Its intention is to allow people to inform the server of questions that are threatening, abusive, etc. Exactly how this translates into a censor instruction to the BB is undefined - for example, it could be automatic based on the fraction of viewers who flag it, or it could require human intervention.


D includes:

- Question_Id:

M includes:

- Info about the nature of the complaint?

The server should not publish who complained.


### New Registration

D includes:

- Name
- PK: public key
- (optional) electorate
- (optional) email domain

M includes:

- (optional) email name

(TODO: update registrations for better structure in ServerAPIDataStructures.)

### Edit Registration

D includes:

- Name
- PK: New public key
- Sig: signature on update
- bool: replace or add
- bool: can-endorse
- (optional) electorate
- (optional) email domain

M includes:

- (optional) email name

If the New public key is different from the old, there should be a signature to show the update is sound.

**TODO: Rather than invent own, it would be better to adopt an existing standard.  The two booleans are meant to indicate (a) whether the new registration replaces the old one or adds to it; (b) whether the new key is allowed to endorse other keys.  

**TODO: look at other PKI standards; adopt.

## Get functions

Get function ask for things without altering the server's state.

For each of these, in addition to the Response R, the server also sends:

- PR - most recent published root
- Pending (bool) - An indication of whether this value is still pending (i.e. not incorporated into PR)
- Ssig = signature<sub>server</sub>(h(Response))

(TODO: Make sure the signed Response data is formatted s.t. verifying later inclusion is easy.)

** Find-similar-questions **

Takes a draft question (and perhaps some other metadata), and returns a list of similar questions according to NLP analysis.

Input: 

- Question: q
- Int: num_responses (max MAX_RESPONSES, prob 50)

Response: 

- vec(Question, float): a vector num_responses long of the num_responses closest matches, with estimated similarity.

(TODO: Or error? Not clear what errors there could be. Even if num_responses too high, could just return fewer.)

(TODO: Add vote info - we want the person to see them in proportion to their upvotes. Likewise to find-matching.)

(TODO: Consider a slick way to get updates without getting a whole new list.  For example, should this return the perceptual hash of either the input question or its neighbours?)

Note: It's possible that we only want the string of the question text, not the whole question, but including the rest seems harmless at this point - it can always be left blank.

** Find-matching ** 

Takes a list of question metadata requirements (e.g. question-writer, who-should-answer, who-should-ask, keywords?) and returns all the (unexpired) matching questions

Input: 

- Question: q
- Int: num_responses (max MAX_RESPONSES, prob 50)

Response:

- Int: num_matches (the number that actually matched, of which the responses are the 'top')
- vec(Question, float): a vector num_responses long of the num_responses closest matches, with estimated similarity.

(TODO: Or error? Not clear what errors there could be. Even if num_responses too high, could just return fewer.)

Notes: This is in some ways the dual of Find-similar - again, we don't actually need the whole question - we don't use the question-text.

TODO: Might want to be able to get next 30, etc.  Might want a timestamp, so 
maybe have something a bit like "new questions are available - refresh?" to pick up things that have arrived since you started looking. But this has a different meaning from 'I'm still scrolling'

TODO: It's possible that we want a hybrid of these, though we certainly want the pure versions of each.


**Query key**
Client sends:

- ID

Server returns:

- ID
- PK<sub>ID</sub> = the public key associated with that ID


(**Question: Think about exactly what verification info we want with the PK - do we need the current root? Do we need the server to sign it? Should you get back either a sig or a BB inclusion proof depending on whether it has already been included?**)

(**Question: Earlier verisons had separate queries for questions/question_edits and for tallies.  I'm not sure this is useful, so I've deleted them. We could, however, consider various kinds of search-based updates, e.g. all the questions posted by one user, or for answers or tallies for the questions that you asked. This would be easy - the question is whether you would ever _not_ want everything else as well.**)

(**Question: Need to make it obvious/easy to check that something you were told is subsequently included. Need to think about the data structure a little.  It probably works out fine if we simply make a Merkle leaf out of each batch decryption - clients can then verify inclusion later, though they obviously can't verify if something was withheld. However, vote tallies are a little complicated.**)

(**Question: we want a way to query earlier questions if the client has been offline for a while. We could implement this as either (1) amending the above query to include a root hash R, i.e. interpret this as a query for the questions since R, or (2) change the verification f'ns below so that rather than asking for something specific, you're simply asking for inclusion-proven list of all questions (or keys) between two updates.  (2) seems better. Which then means we need to consider completeness proofs, i.e. is it possible, without seeing the whole tree, to verify that you've seen _all_ the questions or all the decrypted tallies?  I assume we can do this simply with a counter, but it's not obvious.**)

(VT: TODO: When the querier is an MP, the server should also return the list of encrypted addresses for any questions tagged for that MP. (Probably version 2.0.)

## Verification functions
There are four different things a client may wish to verify (though the first two are very similar):

- that certain previously-posted data is included on the BB,
- that a previous published root is properly included in the current published root,
- that a previously-given answer to a query is included on the BB (e.g., tallies, questions). 
- that the published root is properly constructed from all posted data.

Let H = h(D) be some data previously posted to the BB. This could be something posted by a user (as h(D,CSig)), by the server (as a tally update) or by the trustees (e.g. decryption proof). The requester may know about it because they submitted it, or because they received it in response to a query (above).

### Verification data structures

**(Update for Andrew's BB)**

### BB verification

The verification that the tallies are a valid decryption of the votes on the BB should be exactly the same as ElectionGuard.  **(Probably significant detail to go through carefully.)**


**Verify-inclusion**


(**VT: Note: We probably want a nice bulk way to verify the inclusion of everything you were told was pending.  Though of course it's completely fine if you can't do this until the next root hash is published, so the most efficient way to do it may simply be to download all the data for the just-published root, check its proper formation, and then (plainly) check that it matches what you were told. VT(2) Actually this is possibly the most important thing, and is quite easy given that a query gives you the current root.  So there's an obvious local verification function in response to a query, which is to check that the complete included data in PR is (a) properly included in PR and (b) matches any 'pending' information you were given in response to earlier queries.**)

**Verify-history**

A client can request proof that earlier root hashes are properly-included subtrees of the current one. 

**Verify-BB**

This function is intended to allow a complete download of the BB's contents, expecting that the client will want to verify all the vote aggregation and decryption as well.  Here we describe only the verification that the root hash is properly constructed.  Unlike all the other query functions, this function outputs data, not just the hashes.  The client sends:

- R1, R2: two root hashes from different epochs

and the server returns

- the complete BB contents between R1 and R2.

If R1 is omitted, the entire history since inception until R2 is returned.  If R2 is omitted, the current root hash is used.

There is no need for a history proof because the client can compute this themselves.

### Election Verification
(VT: TODO)

This will, as much as possible, match ElectionGuard's election verification, consisting of checking all the decryption proofs, vote aggregations, etc.

**Verify-crypto**
TODO


## Details and other questions
I'm not quite sure what M would be, but it seems a useful placeholder.  It is also possible that we should have a similar placeholder for the server's response.  e.g. it could tell you which epoch your data will appear in, to avoid ambiguity in the case where your upload occurred close to the boundary.

I've also generally left out data that didn't need to be there, such as the vote data already known to the client.  It's possible that it would be simpler to include this sometimes, rather than only the hash.

I have also omitted many of the details that need to be added for general good API design, such as a version number, to allow for later updates.

