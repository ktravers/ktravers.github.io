---
layout: post
title: Notes on Building Git
tags: [book report]
---

I'm reading ["Building Git" by James Coglan](https://shop.jcoglan.com/building-git/) along with some fellow and former co-workers. Here's my notes on each chapter.

## Chapter 1: Intro

- Git as a decentralized version control model
- Git has a bad rap (hard to learn)
  - Confusing user interface
  - Many ways of doing the same thing
  - Unclear separation of responsibilities between commands
  - Unclear what's going on under the hood
- What are we gonna learn?
  - Unix system concepts
    - How to work with files
    - How processes communicate
    - How to present output in a terminal
  - Data structures used by Git to keep history companct + searchable
  - Diffing and merging algorithms
  - How Git implements a form of "distributed database"
  - How Git supports concurrent editing
  - Why merge conflicts occur (and how the prevent race conditions)

### Questions

- What are some of the most confusing parts of the Git interface?

### Resources

- This talk by Emily Xie is basically the cliff notes version of this book: https://youtu.be/Y2Msq90ZknI

## Chapter 2: Getting to know .git

- Contents of a `.git` repository
  - Works just fine w/o most of the files created on `git init`
  - So why create extra unnecessary stuff?
  - `.git/config`
    - filemode: whether the file is executable
    - reflog: log changes to files in `.git/refs`
      - refs: pointers to objects in `.git/objects` database
  - `.git/HEAD`
    - current commit as an id or symbolic reference (symref) to current branch
    - defaults to symref to master branch
  - `.git/info/exclude`: like a private .gitignore
  - `.git/hooks/`: custom hooks for custom behavior
  - `.git/objects`: Git's database
    - Git compresses every file it stores using DEFLATE compression algorithm (from zlib)
  - `.git/logs`
    - text files
    - log every time a ref changes value
      - make a commit
      - check out a branch
      - merge
      - rebase
- Commits
  - "root commit": commit with no parents
  - `.git/index` is in binary
  - `.git/logs` are text files
  - `git show` command: prints out info about the `HEAD` commit
  - `git cat-file -p <oid>` command: show an object from Git database
  - Trees
    - tree id: hex value built from file contents, so same regardless of timestamp, committer, etc
    - Creates one tree for every diretory in your project, including root
    - Each entry is anotehr tree (aka subdirectory) or a blob
    - file mode: numeric representation of its type and permissions
    - Git stores literal contents for blobs (so `git cat-file` command returns the actual contents, not a tree id)
  - Git stores blobs by:
    - Prepending w/ the word "blob"
    - Then a whitespace character
    - Then length of blob
    - Then a null byte
    - Then compress with zlib
  - Git stores trees by:
    - Prepending w/ the word "tree"
    - Then a whitespace character
    - Then length of string below
    - Then string made up of:
      - Mode in text
      - Then a whitespace character
      - Then filename
      - Then a null byte
      - Then its ID packed into binary
  - Git stores commits as series of headers followed by commit message
- How does Git calculate the filename for each object?
  - Uses SHA-1 hash of uncompressed file content
  - Effectively UUIDs
  - 2005: theoretical attack against SHA-1 "proven"
  - 2006: started discussing moving to SHA-256
  - 2017: Google announced an attack on SHA-1
    - "SHAttered": a method for manufacturing collisions (so you could target a specific commit/file)
    - Easy to detect, and Git has issued patches to check for manufactured collisions and reject them
  - Git maintainers insist it doesn't rely on SHA-1 as "proof of trust"
    - It's an integrity check, not a security measure
- Handy command line tools:
  - `ruby -r zlib -e "STDOUT.write Zlib::Inflate.inflate(STDIN.read)"`: inflate zlib compressed file
  - `wc`: count bytes
  - `hexdump`: print out hexadecimal representation of bytes in a file

### Questions

- Why does `git init` create files that's aren't necessary for a fully functioning git repository?
- Do we agree with Git maintainers that "security is a social problem" and not something they need to build into their hashing function?

### Discussion notes

- More on SHA-1 vs SHA-256: https://lwn.net/Articles/811068/
- Tags and git repo validity
  - "Valid" from the standpoint of which tool?
  - Git fails pretty gracefully, as opposed to GitRPC library
- No tests, strange for a technical tutorial book?
  - Do we need them?
  - Would it be detrimental to learning? More effective to dive into Git and plain ol' Ruby right away, as opposed to setting up a testing framework (time intensive, possibly distracting from core concepts)
  - Book's philosophy is learning through doing
  - Tests would have made the book less flexible, harder to maintain, more like a "build your own production-ready Git competitor"
- Coding along in another language?
  - Python is a good choice
  - JS puts you at risk of ending up in callback hell
  - Some of us tried Elixir, some Rust, but ended up spending more time learning about the language itself instead of Git. Distracts from the content of the book itself.

## Part I: Storing Changes

## Chapter 3: The first commit

OH boy we building stuff.

What we will build:  
- Just enough code to store a valid Git commit.

What we won't build (yet):  
- Subdirectories
- Executable fiels
- Symlinks
- `add` command
- index
- Command line argument processing
- Named branches

Only 3 items in `.git` are essential:  
- `objects` directory
- `refs` directory
- `HEAD` file (symref to file in `refs`)
  - Note: valid to just contain a commit id

- Helpful to know where those low level Ruby error names come from (ex. `Errno::EACCES`)
- Book throws some shade at Ruby: "[Ruby's filesytem interfaces File, Dir, Pathname, FileUtils design] is a little haphazard, and there is not a clear separation between methods for manipulating paths and methods for reading and writing files..."
  - Why is it designed this way?

### Questions

- What's another example where you'd receive relative information from a user then want to convert it to an absolute value before further processing?
  - Time (convert to UTC)
- Why didn't we TDD this first part?
- Did anyone else run into issues w/ piping `cat ../COMMIT_EDITMSG` to a commit message?

### Discussion notes

- Why are we skipping symrefs? Are they difficult to implement?
  - https://git-scm.com/docs/git-symbolic-ref
- Should I write this in Elixir instead? Then I can't cheat. Update: see my awkwardly named [JitEx](https://github.com/ktravers/jitex) project.

## Chapter 4: Making history

- Goal: put the commits in order
  - Create "causal" relationship between commits, not just temporal
  - More efficient to use parent oid field than timestamps (aka the "parent chain", aka a linked list).
    - Git db isn't indexed, so very slow to query especially in projects with lots of commits
    - Adding in remotes gets tricky too, because then you're loading and sorting commits from the source and remote
    - Plus there's the whole system time thing, where distributed committers most likely won't have synchronized clocks
    - Think of timestamps as just more metadata
    - Think of it as a series of patches
- Speaking of efficiency: "[Git] only creates new blobs when files have their content changed. Files with the same content will point to the same blob, even when those files appear in many trees across many commits."
- Updating `.git/HEAD`:  
  - Need to avoid dirty reads / writes
  - Need to at least appear to be atomic
  - Lockfile only used for `HEAD` (for now). Will we use it for refs in the future?
- Interesting comparing this hash-based relationship/object building versus something like SVN (Subversion) that uses numerical ids. Cool, clever, and elegant that Git builds the id as a hash of the entire commit contents, including the parent id. Seems less error prone as well (confirmed by SVN users in the club)
- Staff engineer convo: git actually does have indices it uses for git rev-list and git log? AND the whole linked list structure is an abstraction, and it actually does sort by timestamp!! We might get to this in later chapters.


### Questions

- How strong is the guarantee that this Lockfile gets us?
  - Only as good as how we check for it
    - Good approach: system level calls (machine can't lie to itself)
    - Bad approach: put the check at the application level; now you're at the mercy of race conditions between instances
  - What are other bad approaches you could take?
  - Plus locking is optional! There's a flag you can pass to skip locks. So how necessary is the lockfile?
- Where did this concept of a Lockfile originate? Something Linus thought up, or insired by something else?
- How much of a performance hit are we getting from implementing Lockfile?

### Bookclub discussion prompts

1. How are commits linked? Why is this better/more performant? (Why not sort by timestamps?)
1. In distributed systems, we often need to think about race conditions. How does Git handle the potential for multiple processes to write to the same HEAD file?
1. Compare/contrast: what happens when a file is changed in a commit? what happens when a file is unchanged in a commit? (w/r/t git database and trees)

## Chapter 5: Growing trees

- Time to run some executables.
- An interesting but dangerous shell option: `export RUBYOPT="--disable gems"` (skips Ruby gems on path lookups)
- Both Git and blockchain use merkle trees, in that they're trees made out of a hash of their own contents. Change one node, and it changes everything. Sign the entire history with each addition.
  - Git doesn't expend energy on proof of work, though, like blockchain does
- From friend of the author
  - Author likes to avoid concatenating strings (one of his bugbears)
  - Uses value objects as a thin but helpful wrapper (safer, more ergonomic)
  - Avoid passing around primitives, like strings
  - Difficult, since Ruby does have lots of helpful methods on its primitives (Array, String, Hash), so tempting to lean on those
- Order that you iterate over keys in a hash is the order in which those keys have been added to the hash
  - Is that dependable?
  - See https://ruby-doc.org/core-2.5.1/Hash.html ("Hashes enumerate their values in the order that the corresponding keys were inserted.")

### Questions

- Why are we storing things in octals? (and by "things", I mean file modes). More compact than a string, sure.
  - Author says it's "easier to read"... for computers, maybe?
  - Also kind of inefficient? Storing as binary is more space efficient, since the non-ASCII encoded type is only 1 bit. Doesn't matter too much, because of zlib compression, but still.
- ["Sticky bits"](https://en.wikipedia.org/wiki/Sticky_bit)... who named this?
- Need to research Merkle trees. Behind Git and blockchain.
- Can someone explain the `Z*` bit in the tree entry format? "use Z* in the format string to serialise
this combined field followed by a null byte."
- Did anyone implement the unideal Workspace refactors? (bottom of page 70) Learn anything interesting?
- Why isn't `lockfile.rb` organized under `database` directory?
- Why is `author.rb` organized under `database` directory?

### Bookclub discussion prompts

1. What's cool about Merkle trees?
1. Why must git's tree objects have their entries be sorted?
1. What do you think about the author's approach?
1. What do you think about the structure of our code so far? What would you change if anything?
1. Any stories to share about git snafus or race condition conundrums?

## Chapter 6: The index

- Time to address performance concerns (reading and hashing every file in a project on every commit is wasteful, for `git status`, `git diff`, and `git commit`)
- The `add` command and index. Why pair these two? How does one relate to the other? Answer: `add` adds files to the "index" aka staging area.
- [`SortedSet`](https://ruby-doc.org/stdlib-2.6.3/libdoc/set/rdoc/SortedSet.html): this seems like an object I should be using more often
- States:
  - "untracked file": not in index and not in latest commit
  - "changes to be committed": in index with blob id that differs from blob id in latest commit, aka "staged"
  = "changes not staged for commit": in index with metadata and blob id that differ from what's in index and latest commit, aka "unstaged"
- Index file format:
    ```
    A 12-byte header consisting of
      - 4-byte signature. The signature is { 'D', 'I', 'R', 'C' } (stands for "dircache")
      - 4-byte version number. The current supported versions are 2, 3 and 4.
      - 32-bit number of index entries.
    ```
- Call ``stat()` on index entry, see ten 4-byte numbers:
    ```
    32-bit ctime seconds, the last time a file's metadata changed
    32-bit ctime nanosecond fractions
    32-bit mtime seconds, the last time a file's data changed
    32-bit mtime nanosecond fractions
    32-bit dev, id of the hardware device file lives on
    32-bit ino, number of the inode storing file attributes
    32-bit mode
    32-bit uid, file user id
    32-bit gid, file group id
    32-bit file size
    ```
- Calling `git add` on file adds it to `.git/objects` and `.git/index`
- Helpful command: `git ls-files --stage`. Lists all files in the index with file mode, object id, and stage status
- Writing some code:
  - `Index::Entry` class separate from `Entry`.
    - Pro: Classes are cheap, so might as well make a new one, since they're different enough in functionality
    - Con: Naming confusion
  - Index stores file modes as numbers, not text. Trees store file modes as text. Why?
- Typo on p 81: "There must be at *lease* one null" ("lease" instead of "least")
- Typo on p 86: `OrderedSet` instead of `SortedSet`

### Questions

- Naming: why "index" instead of staging area? How is "index" an index in the traditional sense?
- Why does the index store file modes as numbers, versus trees that store them as text?
- Can someone explain the index path length scanning stuff on page 79-80 to me like I'm 5 years old? Excerpt: "The path length is stored to make parsing easier: rather than scanning forward byte-by-byte until it finds a null byte, the parser can read the length and jump forward that many bytes in one go."

### Bookclub discussion prompts

1. In what ways does our new `git index` behave like an index? In what ways is it more like a "staging area"?
2. Is the git `add` command something that's "nice to have" or absolutely necessary?
3. Is the git `add` command useful on its own without an index?
4. Did anyone spot any potential problems with our index implementation? Any edge cases we aren't accounting for?

### Discussion notes

- This chapter was very helpful for getting a sense of staging area in a physical/spatial sense. As an "area" separate from the git object data store.

## Chapter 7: Incremental change

- Index isn't really useful on its own
  - Need to use it to compose commits. Commit command doesn't read from the index. It still reads from the working tree.
  - Need to update it incrementally. Right now, our index overwrites itself. We need to be able to make incremental changes and not lose everything.
- We initialize a new index every time we call `add`... seems like that needs to change.
- But WAIT, we can rely on our Lockfile. Nice!
- Really like having a proper `Index::Checksum` class for the checksum behavior.
- We were able to simplify our Tree code by moving sorting logic into Index.
- Edge case: delete file, add directory with the same name. Now our index is borked:
    ```
    $ rm alice.txt
    $ mkdir alice.txt
    $ touch alice.txt/nested.txt
    $ jit add .
    $ git ls-files
    alice.txt
    alice.txt/nested.txt
    bob.txt
    ```
- Turns out our index implementation was too naive.
- So NOW we start a test suite. Better late than never?
- Typo on page 89: "wil signal whether..."
- Interesting edge case on p107-108: "I'm not entirely sure why Git adds this object to the database with no references in .git/index or .git/refs pointing to it"

### Questions

- I don't understand why we're using `Index#clear`... we initialize a new `Index` instance everytime we call `add`, so shouldn't we already have a fresh state?
- Why wait until now to start testing? This was the first time we tried to break the system, but surely we could have broken it much sooner (in ways we'd want to protect against with tests). Or was this the "right time" to introduce tests?

### Bookclub discussion prompts

1. We're using the pessimistic locking strategy for updating our index lockfile. What other locking strategies (if any) could work here?
2. Why was this chapter the time to introduce tests?
3. What other edge cases could we write tests for at this point? So far we've implemented `git init`, `git commit`, and `git add`.
4. Does anyone know why the behavior described on p107-108 is in place? Specifically, when you try to `add` an unreadable file, "Git adds this object to the database with no references in .git/index or .git/refs pointing to it".
5. Does anyone know why the lockfile conflict error message is "curiously...misleading"? (see p109) The author speculates that "this message was once true in some version of Git, but at some point the behaviour was changed and this message was not updated", but curious if anyone has more insight.
    - Source: https://github.com/git/git/blob/ddb1055343948e0d0bc81f8d20245f1ada6430a0/lockfile.c#L149-L162

### Discussion notes

- How similar is Git to Bitkeeper (the old Linux version control)? 
- Multiple datastores -- often helpful! Runs counter to the dogmatic advice of having a "single source of truth". But it's implemented well here and clearly helps us.
- Another "counter intuitive" thing: using exceptions as flow control. Not always the best design, but here, these are truly exceptional, so seems very useful.
- ActiveSupport has an atomic write operation that works the same way as this lockfile approach!
- Unreachable objects (regular refs, tracking refs, etc) *can* cause problems. We've seen some issues at scale with repos with excessive/pathological number of unreachable refs

## Chapter 8: First-class commands

- Progress check: we have `init`, `add` and `commit`
- Time to abstract!
- Abstraction + decoupling from global objects/system calls will also make this more testable
- Repository:
  - Interface for getting Database, Index, Refs, and Workspace
  - Knows filesystem locations
  - Use memoization for consistency (not performance, in this case) because some of the objects are stateful (Index)
- DRY is not eliminating literal duplication. It's about removing opportunity for maintainer to introduce inconsistency (change an implicit agreement that's not enforced)
  - Memoization for consistency > performance
- Hooray! `Command` classes!

### Questions

- What are other things we could test at this point?
  - Example: added files are written to the database (because we can't read from the database yet... is that true?)
  - Error paths in `add` command
- What assumptions made by the code aren't expressed in the test suite?
- What did you think of the refactor? Would you have done anything differently?
- Did the eventual structure of the code benefit from writing it in one "undifferentiated lump" and then refactoring?
- How 'bout that `throw`/``catch`, huh?
- Anything you don't like about the refactored code?
  - Locking code is a little spread out. Could we use a block or something similar instead?
- Can we think of alternative names for "Repository"?
  - Lump
  - "Code mausoleum" 😄
  - 


### Discussion

- There's a git implementation in awk! https://github.com/djanderson/aho
- Interesting to redefine `exit` in a Ruby class
- VERY interesting to redefine `puts`
  - Reminds book clubber of an early Ruby Tapas episode about `bare` words
  - 


## Chapter 9: Status report

- Using TDD to implement `git status --porcelain`. Why didn't we use TDD from the start? Because "[t]he add and commit commands...are fairly straightforward tools whose code we execute frequently while developing the codebase. We wrote tests for the edge cases but by-and-large if these commands were not working we would quickly notice. In contrast, a
command like status needs to deal with lots of possible combinations of states, not all of which we'll just bump into in the normal course of our work."
- Remember: only write as much code as you need to make the tests pass. Cleanup later. Bad code that makes the tests pass can also inspire additional tests.
- Next up: finally reading objects out of the `.git/objects` database

### Questions

- What was the difference between `Index#load` and `Index#load_for_update` again? Answer: locks.
- Was using TDD in this chapter beneficial to the code (or the books narrative) compared to the approach in previous chapters?
- Did anything surprise you about how status was implemented?
- Let's talk about strace!
- Index updates from running git status?!

### Discussion

- Many of the cited sources are from Wikipedia. Is that cool these days? Why not link to a non-Wiki source?
- Pyramid of tests: https://corgibytes.com/images/Pyramid-of-Tests.jpg
- Look into Gary Bernhardt's testing something something imperative shell
  - Test the core super thoroughly
  - Test the boundaries between services/dependencies super thoroughly
  - Don't worry too much about full end-to-end integration tests. Focus on the core and boundaries.
- Dave Thomas approach
  - TDD during development
  - Then throw it all away. Just keep the acceptance tests
  - Otherwise test suite turns into concrete, slows you down
- More testing resources
  - https://www.youtube.com/watch?v=URSWYvyc42M
  - https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell
  - https://www.destroyallsoftware.com/talks/boundaries
  - https://gist.github.com/jah2488/869de7482f668857d8f2c95ae8ef8ad2 (coin changer kata)
  - https://github.com/garora/TDD-Katas/tree/master/src
  - https://arlobelshee.com/what-makes-a-good-test-suite/corey-haines/


## Chapter 10: The next commit

- Previously on Building Git: we got most of the way through a fully functioning `git status` command.
- But still need to be able to display difference between last commit and current index
- To do so, we need to be able to:
  - Read the complete tree associated with `HEAD` commit
  - aka get the blob id for every item in the `HEAD` commit tree
  - Then compare to entries in index and detect differences
- Step 1: script that prints all the files in `HEAD` commit
- Finally, we'll be able to read from the db (`.git/objects`)
- Oh boy, escape codes. I'm well familiar with those from this post: [How My Bash Color Settings Broke edeliver](https://kate-travers.com/debugging-edeliver/)
- End of TDD? Huh.

### Questions

- Why do we need the blob ids to detect differences? I feel like I'm missing something fundamental about how this all works. Answer: oh yeah, because the `oid` changes when there's a change to the blob.
- Why haven't I used Ruby StringScanner before? Seems like a helpful library.
- Are the "detect changes" methods order dependent? What happens if we change that order?

### Discussion notes

- Building objects from parsed strings always makes me nervous, but I guess in this case it's fine, because the format is so strictly enforced.
- Separate Tree objects for Index and Database. Good for duck typing, hiding "write" methods from Index. Any potential for confusion/unexpected breakages?
- Does Ruby have an OrderedHash? Answer: nope, Rails: https://api.rubyonrails.org/v3.2/classes/ActiveSupport/OrderedHash.html
- Should I be using more Structs in my Ruby code? Reaching for classes too soon?

## Chapter 11: The Myers diff algorithm

- What we're missing:
  - See what lines we've changed from last commit
  - See commit log
  - Create and merge branches
- High level diff summary
  - Display smallest number of edits possible
    - Easier to read
    - Less likely to produce conflicts
  - Deletions followed by insertions (minimize interleaving)
  - Align with code structure (example of where the `end` shows up when adding a new ruby method)
- Myers algorithm is fast and greedy
  - Prefers deletions over insertions
  - Consumes as many lines that are the same before making a chnage (avoids the "wrong `end`" problem)
  - "SES": shortest edit script
    - Can be modeled as a graph search

### Questions

- We need tests for this. More than anything else. Something to confirm we're returning the diff we expect.

### Discussion notes

- More Git resources: [Oh Shit Git](https://ohshitgit.com) zine by Katie Sylor-Miller and Julia Evans
- [Xgit](https://xgit.io/): Git actually implemented in Elixir (with helpful blog posts about the process)

## Chapter 12: Spot the difference

- More refactoring lessons

### Discussion notes

- Covered side project that applies these learnings to parsing GEDCOM files ("Genealogical Data Communications")

## Part II: Branching and merging

## Chapter 13: Branching out

- Didn't take notes while reading :(

### Discussion notes

- Nothing too surprising in this chapter

## Chapter 14: Migrating between trees

- Right now, we can create branches that are just pointers to a commit. Not super useful
- Checkout command
  - Apply changes to the repo so that workspace, index, and HEAD are restored to state at the commit the branch points to
  - First time Jit makes changes to the workspace and index on behalf of the user (not via explicit command)
  - We want to be efficient when switching from tree to tree (commit to commit). Only change the files we need to, so will need to calc the diff
  - Throw error if there's a conflict
- Migration: used to plan changes. Smart.
- "Inspector"?
- I know the perils of self-hosting all too well.

### Questions

- Branch that's just a pointer to a commit: is that basically a Tag?
- "Git does not preserve all untracked changes": is this surprising?
- "In Jit's case, the code for checkout does still exist somewhere inside .git/objects, and it might be a fun exercise to try and retrieve it without using the command itself. But, I'll leave that for you". Did you figure this one out?

### Discussion notes

- Maybe next week we can pair on retrieving "lost" code. Spoiler: we did!
- Would have been nice to have included an example of git clobbering your changes on `git checkout`
- Suggestion: use `git switch` instead, it has options for how to handle changes
- Changing expectations around "auto-save"
  - Didn't used to be expected
  - Now it's basically required, from a UX/product perspective
- Lots of error handling code: truism or code spell?
  - We think truism
  - Elixir makes this easy and concise, nice when a language helps you out like that


## Chapter 15: Switching branches

Current state of things:

> When we run `checkout bar`, we copy the contents of `.git/refs/heads/bar` into
`.git/HEAD`, and we don't store any representation of `bar` being the current branch.

- Branch pointers need to follow the HEAD of the branch, otherwise we'll "lose" commits
- Why were we copying contents again? Why didn't we implement a "pointer" from the beginning?
- In Git, "`HEAD` is strictly a reference to the branch pointer `bar`, rather than a reference directly to commit B."
- "detached `HEAD`": when HEAD points to a commit instead of a symref
  - Described as "detached" beacuse any commits will move HEAD but not the branch pointer
- TIL: deleting a branch only deletes its pointer in `.git/refs/heads`, not its commits
- TIL about Ruby's `OptionParser` for parsing command line options. So helpful!

### Questions

- Is there a one-line command for turning detached `HEAD` into a properly tracked branch? `git branch qux && git checkout qux`. Wait, duh: `git checkout -b qux`
- Why wait until now to introduce `OptionParser`? String parsing is fun and all, but kinda seems like something we could have skipped.

### Discussion notes

- Have you looked through `git` source code? https://github.com/git/git
- We solved last week's challenge! Look in `.git/refs/heads/master`, it has the latest sha.
- Fun exercise: build your own Ruby option parser. We built one for EU vs US date formatting!

```
require 'optparse'
require 'date'

# DateTime formatter
#
# Supports flags for US vs Euro date format
#
# Usage:
#
# --us
# => outputs date formatted MM/DD/YYYY
#
# --intl
# => outputs date formatted DD/MM/YYYY

options = {}

OptionParser.new do |opts|
  opts.banner = "So you've asked for help with this parser..."

  opts.on("--us", "--usa", "optional help text") do
    options[:format] = "%m/%d/%Y"
  end

  opts.on("--intl", "--eu", "--euro", "optional help text") do
    options[:format] = "%d/%m/%Y"
  end
end.parse!

p DateTime.now.strftime(options[:format])
```

- Symrefs can point to other symrefs, which can point to other symrefs, etc. Does Git ever use that in practice? Should it? Should we?!
- The difference between `git branch qux` (make a branch, but don't update `HEAD`) and `git checkout -b qux` (make a branch & update HEAD) in a detached `HEAD`. The Git CLI sure is a mess sometimes!
- The `SymRef` & `Ref` classes that start out as single-field Structs to wrap around Strings was a cool concept
- Naming things is hard! `@current_oid` is the … previous OID?
- `File.join("refs", "heads", something)` seems to show up in a lot of places – should this knowledge of building paths be centralised somewhere?


## Chapter 16: Reviewing history

TODO: re-read this chapter.

### Questions

What happened at the end of this chapter? I was fine until the last few pages, then completely lost.

### Discussion notes

Logging. Turns out to be more complicated than I expected.


## Chapter 17: Basic merging

Note: read with an eye for some sort of pairing exercise based on the chapter content.

- Git branches are just pointers, so what actually happens when you merge?
- Simplest case: merging single commits
  - Merge commit has two parents
  - Finds "best common ancestor" aka "merge base"
  - Does the diff starting from best common ancestor

### Questions

### Discussion notes

- These example file names and contents could be better.
- "Virtual commits" - let's step through what's happening there.

## Chapter 18: When merges fail

- Typo? "focussed"
- Merges are "trivial" when changes don't overlap
- Right now, `Command::Merge` class has a monolithic `run` method. Time to extract some stuff out
  - Start by separating the inputs of the merge from its execution
    - Inputs:
      - Names and ids of the branch heads
      - Ids of the best common ancestors
    - Execution:
      - Changes to the index and workspace
  - Create `Merge::Inputs` object
  - Create `Merge::Bases` to find BCA
  - Create `Merge::Resolve` object that applies changes
- What if the incoming commit is an ancestor of the current HEAD?
  - aka "Null Merge"
  - Happens when commit is already reachable from current HEAD, aka has already when merged
  - Confirm by checking if BCA == incoming commit
- What is the incoming commit is a descendent of current HEAD?
  - aka "Fast Forward Merge"
  - Happens when BCA is equal to current HEAD
  - No changes on HEAD that aren't already reachable from incoming commit
  - Just update HEAD to incoming commit, since they already share existing history
  - Nuance: use `--no-ff` option to construct a new merge commit instead of fast forwarding
  - Confirm by checking if base oid == left oid (which in this case is the HEAD oid)
- How does git know it's in a merge conflict state? `.git/index` non-zero stages for entries
  - The index should never contain both zero- and non-zero-staged entries for the same path
  - So it's important to key entries in the index by path AND stage
- If your changes do not commute, then git flags as a conflict and forces the user to fix manually, rather than guess
  - Changes to different files commute (can be applied in any order)
  - Changes to same file often doesn't commit (order matters, so needs to be fixed manually to prevent data loss)
- Types of conflict:
  - "add/add": base does not contain path, and branches add an entry at the same path with different contents
  - "content": base contains path, and branches both modify same path
  - "modify/delete": base contains path, and one branch modifies and one branch deletes path

### Questions

- What's a commit "name"? How is it different from the oid?
  - Answer: it's a "revision" string (but now I'm not sure what that is)
- In what scenario would you want to use `--no-ff`?
  - You still want the change reflected in the commit history
  - Makes it easier to revert change, since you have a merge commit you can revert
  - Personal preference
- "...you might be wondering why we don't just use the Merge::Resolve class which effectively does the same tree-diff and migration work. The answer is that, in order to handle merge conflicts, that class is about to get a lot more complicated."
  - Why not go down that path then show the refactoring work?
  - Sometimes the author does this; sometimes he doesn't. Interested in how he decides.
  - In this case, seems like an example of avoiding the wrong abstraction (or DRYing code up too soon). "Just because two things happen to look the same, doesn't mean they really represent the same concept. In this case, by saying precisely that we just want to check out the merged commit and nothing more, we avoid opting in to any new behaviour the Resolve class might gain that we don't want. Having small building blocks is an important step in avoiding reusing code beyond its intended scope"
- I wish `merge3` had more of a descriptive name. Maybe `validate_three_way_merge` or something like that.


### Discussion notes

- Bitshifting...something I rarely do in Ruby.
- I think I'd understand this better if we were TDDing it. More tedious, but it'd help it sink in.
- From @talum:
  - What do we like/dislike about refactoring to `Merge::Inputs`? [Commit](https://github.com/jcoglan/jit/commit/c9c3ea9714d6f24007cdb0f26258e6f09ea49407)
  - Truly did not appreciate what fast-forward merges were before this chapter. Are there times when y'all don't like a `fast-forward` merge? Why or why not?
  - There was a nice quote here: "Just because two things happen to look the same, doesn't mean they really represent the same concept." It reminded me of Sandi Metz's take on finding the right abstractions. Any elaborations? How do you know when you've found the right abstraction? When do you think it's OK to duplicate code?
  - We returned to hexdump land and inspected some stage flags. Stage 0 means the file is clean, any non-zero number means there are conflicts.
  - The end sections about detecting conflicts got VERY wordy, not so much a question as a comment. [commit](https://github.com/jcoglan/jit/commit/fe3702f7d6b929af0612da7597a395178920ab33)
  - The conflict detection is part of `Merge::Resolve` and the author alludes to the fact this expansion is coming. Is this a good place for the detection?
  - When there is a conflict, we're [currently merging both blobs together with the header/separators](https://github.com/jcoglan/jit/commit/fe3702f7d6b929af0612da7597a395178920ab33#) and not doing anything more sophisticated.  Wonder when we'll get to that!
  - Was there anything else you would changed in the implementation?


## Chapter 19: Conflict resolution

- Mostly logging?
- "Combined diff" shows diffs from stages 2 AND 3

### Questions

- The `on_process(&block)` for logging messages seems overly complicated.
- Why separate hashes for `CONFLICT_LONG_STATUS` and `CONFLICT_SHORT_STATUS`? Same keys for both...

### Discussion notes

- Cool diff flags:
  - `git diff -cc`: See combined diff
  - Specify stage
    - `git diff --base` / `-1` (base)
    - `git diff --ours` / `-2` (left aka HEAD)
    - `git diff --theirs` / `-3` (right, incoming commit)

## Chapter 20: Merging inside files

### Questions

### Discussion notes

From @aharpole:

- It's not possible to just look at line numbers to figure out the correct merge
- `diff3` algorithm aims to solve this
- `diff3` tries to create blocks of things that changed, while also keeping around the unchanged chunks even when they aren't on their original lines
- `Diff3` merges work with our existing Diff object that emits sets of Edit objects
- We produce match sets, which are a hash that maps a line number on the original to its matching line in the compared file
- `<part where my mind kind of glazed over as the looping/parsing algorithm got discussed>`
- `Diff3` is sensitive to the underlying diff algorithm! A different diff algorithm might lead to a different merge outcome.
- The diff format is different when describing a diff with multiple parents, it's the cc format (I couldn't find what the cc stands for, but I think one of them means "combined")
- Process of building the combined diff from a left and right diff: shift off deletions from the a diff into the combined diff with `nil` in second column, shift off changes in both a and b diff in pairs, and put deletions from the right diff into the combined list with a `nil` in the first column, then take the edits with a b line that can be paired up and add those to the combined diff.
- Combined diffs similarly get combined into hunks just like regular diffs
- Adjusting the diff generating code to work with multiple parents is an instance of a common modification pattern we find ourselves running into when programming: moving from a situation where we always have a single "thing" to having one or more of those things at a time.

Group discussion:

- Someone tried to produce an incorrect sorting algorithm, then actually [made a correct one](https://cs.paperswithcode.com/paper/is-this-the-simplest-and-most-surprising)
- When the chapter pointed out diff3 is a Unix utility, thought we were just going to shell out to some Unix command, not implement it ourselves.
- Merging is not a solved problem! Or at least, we haven't "perfected" it yet
- What about stacked diffs? Commit is the unit of work, not the Pull Request.
  - https://jg.gg/2018/09/29/stacked-diffs-versus-pull-requests/
- ["Language aware merging"](http://www.cs.kent.edu/~jmaletic/cs63902/Papers/Hunt02.pdf)

## Chapter 21: Correcting mistakes

### Questions


### Discussion notes

- I don't use the `git rm` command. Maybe I should start...
- TIL `ORIG_HEAD`

From @talum:

This chapter's all about correcting mistakes: removing files form the index, resetting the index, discarding commits, and aborting a merge.

- A nicety about `git` is that is tries really hard to prevent data loss. So there are a lot of confirmations when you want to `rm` a file.
- There are also a lot of options. `--cached` `--forced` for people who wanna skip safety
- The `reset` command resets by inserting tree entries from head commit into the index. Walking the tree to find entries...kinda neat too.
- Notably, the author has a complaint for maybe the first time!? The tricky parsing problem.
> This is a bad state of affairs: we cannot tell from the syntax of the command itself whether the word "readme" denotes the name of a file, or the name of a branch or other kind of commit identifier, and that means Git has to guess.
  - A sharp edge that hasn't been fixed over time
  - Hard to change at this point
  - Counterpoint: is it user-friendly? Probably introduced to prevent user from having to think too hard.
  - API "robustness" principle: "conservative in what you do, liberal in what you accept from others"
- Many ways of discarding commits: `soft`, `mixed`, `hard` . A "false economy" to try to make `hard` reuse the logic for `mixed`
- Real cool that aborting the merge reuses the reset hard logic.
- The fact that reset keeps a reference to the previous head commit in `.git/ORIG_HEAD` is quite nice too.

## Chapter 22: Editing messages

- Currently we can use `reset` to:
  - Drop commits from current branch and redo them
  - Squash multiple commits into single commit
- Not perfect, though, because you "keep all the tree content, but reset throws away the commit messages", which means we can't edit old messages yet
- Step 1: make setting the message easier. Time to add support for `--message` and `--file` options to `git commit` and `git merge`
- Step 2: show the original message for editing before making the commit
  - Method that prompts user for input via their editor
  - Set initial text seen in editor
  - Initial text will have helper comments that get stripped out
  - Separate `.git/MERGE_MSG` and `.git/COMMIT_EDITMSG` files - I like the choice not to try to re-use/share a single file.
  - Reminder: when we amend or change commits, we're not mutating anything (not possible). Instead, "we're adding new commits and changing what the refs point to"
- Step 3: add support for `git commit --amend`
  - First tool we've added for modifying project history
- Step 4: add support for distinct commit author vs commit committer
  - New `committer` attribute
  - `Commit#time` is now the committer time (preserve correct sort order after amends, etc.)

### Questions

- Is a git commit message template a thing? Like, within git?
- How can we make it easy for people to have good commit message hygiene? "Good commit culture" on a team
- What is a "good" commit? Is that something universal we can actually define?
  - Probably varies per project
  - Different teams have different norms
  - Somewhat related: push protections

### Discussion notes

- Author's advice: design programs from the outside in, that way you discover what you actually need
- [`Shellwords.shellsplit`](https://ruby-doc.org/stdlib-2.5.1/libdoc/shellwords/rdoc/Shellwords.html#method-c-shellsplit) is new to me
- `system` method is super helpful:
    > "`system` does all this for us: it blocks until the child process exits. It also makes the child process use the same standard output/error streams as the Ruby process, so if you run a terminal-based editor like Vim7 or Emacs8, the editor will appear in your terminal and Ruby will wait until you quit it."
- `--reedit-message` and `--reuse-message` commit message flags are cool, but I find myself always defaulting to `git commit --amend`
- Leaky abstraction: nothing's getting "amended", because these objects are immutable. Terminology sort of obscures what's happening under the hood.


## Chapter 23: Cherry-picking

Oof, better read this chapter again.

- Didn't realize `.git/sequencer` was a thing. Very cool!

### Questions

- Not sure about re-using `RevList`. Why didn't we create a cherry pick specific object, like we did with `Merge::CherryPick`? Or at least a subclass?

### Discussion notes

## Chapter 24: Reshaping history

- `git commit --amend` uses `cherry-pick`? 🤯 Not really, according to [the docs](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---amend) it's closer to `reset`.
    > It is a rough equivalent for:
    >
    > ```
    > $ git reset --soft HEAD^
    > $ ... do something else to come up with the right tree ...
    > $ git commit -c ORIG_HEAD
    > ```
- Two very important things to remember:
  - "although it may only appear that we've modified commit B, we have in fact generated a whole new history that diverges from the parent of B."
  - "The original history still exists, but may no longer have any refs pointing at it."
- Reordering commits == cherry-pick 1 commit, then cherry-pick range 🤯🤯
- Clever; use branching to mimic `stash`ing commits
- Difference between `quit` and `abort`
- "Crash-only software"


### Questions

- Is everything that changes history a `cherry-pick` and/or `reset`?
- Does git have any commands that explictly reference `sequence`? Seems to be an important bit of plumbing, but not really referenced.

### Discussion notes

From @aharpole:

- I loved this chapter because cherry-picking always felt super conceptually simple to me (take this diff and plop it into HEAD), but knowing that Git doesn't store branches as stacks of diffs, but instead as pointers to trees, I knew it would work super differently!
- I was surprised but not that surprised to find out that it uses a merge, but how it uses a merge is :exploding_head:
- Anyone else struggle with this conceptually a little? In particular, the idea of performing a merge between C and E with D as the base, even though we actually don't want D at all, but this still results in a merge that applies E cleanly to C without D's changes (see pp 455-456).
- I like that we started with the single commit that dealt with how we conceptually deal with cherry-picks, then moved onto cherry-picking multiple commits.
- You can pick a series of individual commits, or ranges; it's pretty flexible!
- This idea of committing a series of commits feels like it'll prove really useful for rebases, especially some of the new primitives like the sequencer files
- Never heard the term "crash-only software" but I have seen the pattern before and I agree it's a really good pattern.
- TIL that you can both abort a cherry-commit, which cleanly reverts, but you can also quit a cherry commit, which stops the cherry pick but keeps the working tree as-is

## Part III: Distribution

## Chapter 25: Configuration

- Can't be avoided!
- Choosing CST ("concrete syntax tree") over AST ("abstract syntax tree") to preserve whitespace, comments, etc

### Questions

- Why would we be tempted to avoid configuration stuff? We're already handling a bunch of "annoying parsing problems", etc. Seems like adding config will probably make our lives easier.
- There's gotta be easier ways to parse this config file

### Discussion notes

## Chapter 26: Remote repositories

### Questions

### Discussion notes

Need to review parts about refspecs.

## Chapter 27: The network protocol

- `fetch` so far:
  - Gets list of refs and values from remote
  - Uses refspecs to decide which ones to download
  - "Just knows" what objects to copy form remote to local
- How does the data transfer happen?
  - e.g. how does `fetch` connect to a remote computer and communicate with remote repo?
- Git supports multiple protocols
  - SSH (ssh://)
    - Available on most machines out of the box
    - Encrypted adn authenticated connection
  - HTTP (http://)
    - Requires custom services running on remote computer (which ones?)
  - Git's own custom transport (git://)
    - Requires custom services running on remote computer
    - Git [daemon](https://github.com/git/git/blob/master/daemon.c)
- For Jit, we're punting on HTTP, viva SSH
- Git servers allow you to run the `git-upload-pack` program via SSH
  - `git-upload-pack` works as an agent, fetching info
  - Protective measure: you can communicate with `git-upload-pack` program only via SSH. It runs sanctioned git commands. You can't run those commands directly
  - Via SSH, `fetch` reads from output of `git-upload-pack` and writes to input of `git-upload-pack`
  - "While it's running, `git-upload-pack` acts as a server that the client, the fetch command, can talk to."
- Git protocol
  - Message ends with newline
  - Begins with length header (4 digit hexadecimal number)
  - Empty message is a "flush packet"
  - First line, HEAD ref, has list of supported capabilities (ex. multi_ack, thin-pack)
- Pack format: send stream of objects in as small a payload as possible, for faster transfers
- `fetch` needs to be able to parse this pack
- Punting on "delta compression" for now (see Chapter 30)
- Starting with building a pack writer class instead of reader
- Pack::Reader class
  - Author notes that the `input` must respond to certain methods. Good case for strong typing!
- Re: overfetching problem with the `Pack::Reader` class
  - We're reading 256 bytes at a time, so inevitably end up overfetching, then having to move the cursor to start read the next object at the correct starting point
  - Could pack's size header include length of compressed data?
  - Could we inflate entire stream first, then parse?

### Extra credit: Git docs

- [Git docs](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols) on protocols
  - Mentions "Local" protocol, for remotes on the same host (file://) with shared filesystem
  - Git protocol
    - Like ssh, but no auth
    - Read-only (not safe to allow pushing, since there's no auth)
    - Very fast, but unsecure
- [Git docs](https://git-scm.com/book/en/v2/Git-Internals-Transfer-Protocols) describe two transfer protocols: "dumb" protocol and "smart" protocol.
  - "Dumb"
    - Appropriate for read-only repos served over HTTP
    - No Git-specific code required
    - Can't handle writes
    - Difficult to secure, rarely used anymore
  - "Smart"
    - Requires "git aware" process on remote end
    - Sounds similar to what we built in this chapter

### Questions

- The author didn't go into much detail on "Git's own custom transport" protocol. Does anyone know more about it?
- Code design question: what are some alternatives to the "overfetching" approach in the `Pack::Reader` class?
  - Only a little complicated
  - Definitely better than checking one byte at a time (worthwhile tradeoff)
  - Why 256 bytes? Big vs little endian
  - If the length is variable, then we have to make some smart guesses as a compromise
  - Probably inescapable problem
  - Maybe we'll refactor once we get to delta compression?
- This chapter punted on "delta compression". How tough is that going to be to implement? Answer: see Chapter 30.

### Discussion notes

- 27.3 (p544) Link to another version of Adder.rb program
  - Tiny program translates input into output
- GitHub doesn't support ssh approach used in this chapter
- Big endian vs little endian
  - Macs are little endian
  - PCs are big endian
  - Comes down to capabilities of chips
  - Making variable length little endian --> applying computer science basics
- A [delightful comment](https://github.com/git/git/blob/master/daemon.c#L429-L430) in git daemon source


## Chapter 28: Fetching content

- Pack negotiation: produce the smallest possible pack to send over network
  - We only want the objects we don't already have
- Rebasing: cool when you're working solo, gets messy otherwise

### Questions

### Discussion notes

- Would there ever be a case where we'd "clear the books" and squash everything until a single commit?
- This chapter, we're writing client and server code
- Git is just a database
  - It has tools that allow us to do version control, etc
  - But at the end of the day, you put stuff in then fetch it back out
- What would GitHub look like if it, too, was a decentralized system? Everyone runs their own instance of GitHub, and you can fork, etc. across instances


## Chapter 29: Pushing changes

### Questions

### Discussion notes

From @jankoszewski:

- More of an observation, but explaining that the absence of a source ref in a push to i.e. `:my-feature-branch` is the reason why that branch is deleted was very much an "aha" moment for me. It made me wonder about a theoretical push with the absence of a / a nil target branch, i.e. `push origin master:`, and if that would effectively be any different than `push origin master`?
- The code written/outlined in 29.2.2 indicates that multiple refs can be updated for a given push, but I'm having trouble grokking how this can be given that I believe that the source and target refs for any push operation should be unique. Is it possible to match on ref path here? Or did I misunderstand the code and the code being used is being invoked in a broader/different manner?
- Is the ABA problem an acceptable tradeoff for push / optimistic concurrency control? and would stacked diffs be a magic bullet to solve or prevent this problem?

## Chapter 30: Delta compression

- "Myers algorithm performs well on line-oriented text files"
- XDelta algorithm for binary files
- No native Ruby support for XDelta algorithm, we're implementing it ourselves

### Questions

### Discussion notes

- This algorithm is a nice example of compromise. Trade-off between performance and legibility
- Zlib compression is baked into Ruby. Would it be feasible/reasonable to upstream delta compression into Ruby? Are there enough use cases?
- Tangent: [Strange Loop talk on compressing Super Mario video game](https://youtu.be/vH-D6GdkyCw)
- Demystify binary data by opening in a hex editor


From @aharpole:

- Git uses a variant of XDelta
- XDelta tries to represent a string as a series of operations to construct the string, with the hopes that this representation is smaller than the original string
- You can either copy data from another place or insert new data. Inserts are less desirable since you're inserting a literal chunk of text.
- XDelta begins by splitting the string into 16-byte chunks of data, then it performs a "sliding search" by grabbing 16 bytes starting at each part of the target string, then seeing if that 16 bytes is in the index for the source string.
- If the string isn't found, we take the first character, append to the insert buffer, then move the sliding window over one. We continue until we find a match
- When we find a matching starting block, we try to expand it by checking to see if the following letter is the same in both strings. If so, that gets included in the matching block. We'll also search backwards and do the same (but if you go backwards and match one, you remove that character from the end of the input buffer)
- We use this algorithm instead of diffs because diffs are optimized more for line-level changes, and diffs aren't suitable for binary files
- Diffs also expect things to be in the same order, but words can be ordered differently and still be useful for producing an optimized delta.
- This approach is really good overall for storing a series of commits that might act on the same file because you store an object, then the delta to produce a modified version of that object.
- For encoding deltas, we represent an Insert operation as a byte representing the size, followed by the actual contents. The size limit is 127 (0x7f)
- For a copy, we use 2 numbers, offset and size.
- Offset can be up to 4 bytes, size can be up to 3 bytes
- Git saves bytes by cutting out all the zero bytes.
- There is a header byte that represents whether the seven bytes are kept, and to indicate whether it's a copy. We use 0 to indicate that the byte was 0 and is excluded, and 1 if it wasn't excluded.
- That leaves the first bit in the header byte. It will be 1 if it's a copy, and 0 if it's an insert.
- I believe this means that for the insert, you start with 0 and then the other 7 bits represent the size, which is why the size limit is 127.


## Chapter 31: Compressing packs

- Ruby `Forwardable` module seems handy

### Questions

- What commands did he run to get that nice tree with all the files for each commit? And the nice neat lists for commit, type, size and path? `git ls-tree HEAD -r -l` gets you the list, but still not sure about that nicely formatted tree.

### Discussion notes

- In real Git, maybe there's something that looks for renames, since that's an easy compress given the content hasn't changed.
- Tangent: Dropbox used to use S3 for storage, but now has their own homegrown system. Maybe they're using delta compression, too? Might be something on their technical blog about this.
- "One observation we can make from the above data is that objects tend to compress either very
well or very poorly". Is this driven by user behavior? If we encouraged users to use Git differently, could we impact this?
- Nice summary on Git and compression from @matthewmccullough: https://gist.github.com/matthewmccullough/2695758
- "Ruby Garbage Collection in Under 2 Hours" book recommendation: https://flurly.com/p/ruby-garbage-collection

From @aharpole:

- When constructing packs, we find pairs of objects that would compress well together with XDelta called delta pairs.
- You can't just try every possible pair because that has O(n^2) complexity; if you have N objects to pack, it would take N^2 attempts to identify the best pack. No good.
- We make some educated guesses about objects. A blob is unlikely to be a good match with a tree or commit object, for instance.
- Objects with the same path are most likely to be similar to one another.
- Adjacent versions of a file will probably compress best since there won't be further changes. (of course, depending on the actual changes and what order you did them in, that might not be true!)
- It's not easy to order versions of the file because they could be on concurrent branches, but we can use file size as a proxy since it's common for files to grow over time.
- We sort the objects in reverse order by type (commit/tree/blob), name (just the file name, not full path) and size
also we skip really tiny files and just send them over the network as-is
- Because of how object data is stored in the database we're able to add a load_info method that can read the type and size without reading the full object.
- We use a sliding window to try to compress the objects. In this sliding window we will try to make pairs from objects.
- We keep a limited number of objects in memory to use to consider for making pairs. this also helps us have linear time performance
- When we make these chains of deltas, we want to make sure that the actual compression savings are worthwhile. In practice, the compression tends to either save a lot or very little with no middle ground.


## Chapter 32: Packs in the database

- Unexpected way to generate random ID for temp files:
    ```ruby
    TEMP_CHARS = ("a".."z").to_a + ("A".."Z").to_a + ("0".."9").to_a
    id = (1..6).map { TEMP_CHARS.sample }.join("")
    ```
- TIL [CRC32 algo](https://en.wikipedia.org/wiki/Cyclic_redundancy_check#CRC-32_algorithm)
- Re: hash table for git index. This is a new data store, and we can use binary search to get things. Efficient, because binary search is log time.
- Building hash table and fanout table are separate operations, but they do function essentially the same way. Different data structures, but they perform similar functions.


## Chapter 33: Working with remote branches

- Lots of nice "glue" code in this chapter
- Builds on stuff / reuses stuff we built chapters ago
- Nice feature add for safely deleting branches

### Questions

- Why wait to talk about remote branches now? Why not sooner?
  - Maybe this was a loose end we had to tie up?
  - Maybe it's a nice send-off? You're not tied to anything, Git is flexible, go build whatever extension you want.
- Now that we've read the book, did it inspire any ideas for contributing to Git?
  - Maybe something(s) to make it more user-friendly?
    - GitHub makes git more user-friendly. Maybe there's stuff we can port.
    - More GUI operations, printing out ASCII representations of things
    - Git Tower is a good example of how the interface can be improved.


## Chapter 34: ...and everything else

Not much to cover here. Just a few closing thoughts to send us on our way.
