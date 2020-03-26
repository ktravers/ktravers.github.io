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
  - `git show` command: prints out info about the HEAD commit
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
- Should I write this in Elixir instead? Then I can't cheat.

## Chapter 4: Making history

### Questions

### Discussion notes

## Chapter 5: Growing trees

### Questions

### Discussion notes

## Chapter 6: The index

### Questions

### Discussion notes

## Chapter 7: Incremental change

### Questions

### Discussion notes

## Chapter 8: First-class commands

### Questions

### Discussion notes

## Chapter 9: Status report

### Questions

### Discussion notes

## Chapter 10: The next commit

### Questions

### Discussion notes

## Chapter 11: The Myers diff algorithm

### Questions

### Discussion notes

## Chapter 12: Spot the difference

### Questions

### Discussion notes

## Part II: Branching and merging
## Chapter 13: Branching out

### Questions

### Discussion notes

## Chapter 14: Migrating between trees

### Questions

### Discussion notes

## Chapter 15: Switching branches

### Questions

### Discussion notes

## Chapter 16: Reviewing history

### Questions

### Discussion notes

## Chapter 17: Basic merging

### Questions

### Discussion notes

## Chapter 18: When merges fail

### Questions

### Discussion notes

## Chapter 19: Conflict resolution

### Questions

### Discussion notes

## Chapter 20: Merging inside files

### Questions

### Discussion notes

## Chapter 21: Correcting mistakes

### Questions

### Discussion notes

## Chapter 22: Editing messages

### Questions

### Discussion notes

## Chapter 23: Cherry-picking

### Questions

### Discussion notes

## Chapter 24: Reshaping history

### Questions

### Discussion notes

## Part III: Distribution
## Chapter 25: Configuration

### Questions

### Discussion notes

## Chapter 26: Remote repositories

### Questions

### Discussion notes

## Chapter 27: The network protocol

### Questions

### Discussion notes

## Chapter 28: Fetching content

### Questions

### Discussion notes

## Chapter 29: Pushing changes

### Questions

### Discussion notes

## Chapter 30: Delta compression

### Questions

### Discussion notes

## Chapter 31: Compressing packs

### Questions

### Discussion notes

## Chapter 32: Packs in the database

### Questions

### Discussion notes

## Chapter 33: Working with remote branches

### Questions

### Discussion notes

## Chapter 34: ...and everything else

### Questions

### Discussion notes
