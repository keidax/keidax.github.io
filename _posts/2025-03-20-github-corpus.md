---
layout: post
title: "Creating a language corpus from GitHub"
date: 2025-03-20 13:21:00 -0400
tags: update
---

Ever wanted quick access to a wide selection of source code files in a particular programming language, but finding and downloading individual projects felt like too much work?

In this post, I'll present an approach to efficiently automate the creation of a programming language corpus.

### Background

A `corpus` is a linguistic dataset consisting of many examples of language in a particular form.
The term often refers to natural human language, but we can apply it to programming languages as well.
So a corpus of code will include many source files written in a particular language.
Ideally these files are written by many authors and drawn from many codebases.
(There's no expectation this collection of source code executes together in a meaningful way.)

For my current use case (working on [tree-sitter-crystal](https://github.com/crystal-lang-tools/tree-sitter-crystal)), I want a large sampling of Crystal code found "in the wild" to see how frequently some particular syntax is used.
I don't particularly care where this code is coming from or which specific codebases are included.

Other use cases that come to mind:
* testing how a language parser or formatting tool works on a large sample size
* doing research on other people's code, like "What's the average line length in Rust?"

### Locating target repositories

The easiest way to find code for most languages is searching GitHub, and the easiest way to automate searching GitHub is with the `gh` CLI tool.
`gh` should be [available for your operating system](https://github.com/cli/cli#installation).

> After installing, you'll need to go through a one-time authentication process with `gh auth login`

We can directly search the GitHub API for codebases written in Crystal, for example:

```bash
gh api '/search/repositories?q=language:crystal'
```

Any [language supported by linguist](https://github.com/github-linguist/linguist/blob/main/lib/linguist/languages.yml) can be searched, from Ada to Zig.

You may have noticed how verbose the results are:
```
{
  "total_count": 9002,
  "incomplete_results": false,
  "items": [
    {
      "id": 6887813,
      "node_id": "MDEwOl...",
      "name": "crystal",
      "full_name": "crystal-lang/crystal",
      "private": false,
      "owner": {
        "login": "crystal-lang",
        "id": 6539796,
        "node_id": "MDEyOk...",
        .....
```

Let's trim to just the git URL, and load the maximum number of items:

```bash
gh api '/search/repositories?q=language:crystal' --jq '.items[].ssh_url' --paginate > crystal_urls.txt
```

In my testing, this always capped out at 1000 repositories --- more than enough for our needs.

### Efficient source code checkout

At this point we _could_ just loop through the list of repositories, clone each one, and call it a day.
But we would be downloading and storing a lot of extra data.
Remember: we're only interested in source code in our target language, not documentation, build files, or anything else.

For each repository, we first want to initialize a _shallow clone_.
```bash
git clone --no-checkout --depth=1 --filter=blob:none <url>
```

Essentially, this sets up the minimal git metadata for the project, without downloading any source files.
`--no-checkout` means we won't be populating the repo with source files.
`--depth=1` means the commit history is trimmed to the latest commit.
`--filter=blob:none` means we delay downloading git's internal data on each file until that file is checked out.

Next, we want to set up a "sparse checkout".
This is an experimental git feature that restricts the files present in a repository to specific patterns or sub-directories.

```bash
git sparse-checkout set --no-cone '*.cr'
```

`--no-cone` means we're operating on a list of file patterns, rather than directories.
`*.cr` limits the file set to just Crystal source code.

> Non-cone mode is deprecated, so it's possible this will stop working in future git versions. [See here for more details.](https://git-scm.com/docs/git-sparse-checkout#_internalsnon_cone_problems)

Finally, run `git checkout` to download all the missing blobs that match our sparse checkout patterns.

### Wrapping up

I've turned this process into a [convenient Ruby script](https://gist.github.com/keidax/e9a514fa5aef93e98eedf1665e923329).

Usage:
```bash
./load_repos.rb --language crystal --pattern '*.cr' --limit 200
```

`--pattern` may be passed multiple times.

`--limit` determines how many repositories are downloaded. The max is 1000.

`--threads` controls how many download threads are used. The default is 1.

Here's how you might quickly download a wide selection of C code:
```bash
./load_repos.rb --language c -p '*.c' -p '*.h' --limit 1000 --threads 20
```

And there's no need to use wildcard patterns if your use case is more narrow:
```bash
./load_repos.rb -l typescript -p 'package.json' --limit 50
```

I hope this is helpful!

---
