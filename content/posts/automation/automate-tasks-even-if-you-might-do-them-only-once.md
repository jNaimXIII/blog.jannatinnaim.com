---
draft: false
title: Automate Tasks Even If You Might Do Them Only Once
description: >-
  Automation is the key to ultimate productivity but, there lies a fine line
  between time and efficiency gains and a time sink.
date: "2023-04-04T03:22:43+06:00"
tags: ["automation", "cli"]
categories: ["story"]
---

## TLDR

[CSpell](https://github.com/streetsidesoftware/cspell/) in VSCode is a one click
setup process but not for Neovim and CLI use. The VSCode's distribution of
CSpell comes pre-configured with sane defaults whereas the CLI is bare-bones. To
take the CLI's capabilities to feature parity with the one in VSCode I wrote a
script that installs the necessary language packages for `cspell` to properly
lint code.

## The Problem

I was setting up my NeoVim config the other day and there's a plugin called
[`null-ls`](https://github.com/jose-elias-alvarez/null-ls.nvim) that aims to
provide formatting, linting, diagnostics support for languages that don't come
with a native LSP implementation.

One such solution was provided for `cspell`, a spell checker with dictionaries
specific to development needs. So, unlike regular dictionaries that can only
check if you've used words from a specific language or not, `cspell` can take
multiple dictionaries which includes valid words in software development but
maybe not a common word in the English language.

I am mainly a VSCode user and have been using the extension for `cspell` in
VSCode for a long time. As I mainly code in JavaScript, and some old projects
don't have proper linting set up, this spell checking at least somewhat
mitigates the amount of errors I get due to making some dumb typo.

To use it inside VSCode, it's as simple as installing the extension and you're
good to go. It comes pre-configured with support for all the languages you'll
ever use and the words that don't come bundled, you can just add them to the
whitelist of words. It's a seamless experience overall.

Let's now talk about using `cspell` in Neovim. If you're using something
[`Mason`](https://github.com/williamboman/mason.nvim) you can install `cspell`
as a linter for your IDE. Installing it like this is perfectly fine and it will
start linting your code for spelling mistakes just as you'd expect. However, I
didn't want to add it this way. Since `cspell` is not a _language server_, I
felt reluctant to add it via Mason. I strictly use Mason for adding language
servers such as `tsserver`, `lua_ls` or `rust_analyzer`. So, for linters and
formatters I use `null-ls` which comes with builtins for most of the widely used
linting / formatting packages. Among those builtins is `cspell`. To install
`cspell`, it's as simple as adding a new line to the `null-ls` config and you're
done. Similar to how you would've gotten diagnostics from `cspell` if you had
installed it via Mason, you'll get the same from installing it like this. In
`null-ls` you can even specify if you want to have only the diagnostics or code
action capabilities as well. That's why for things like linters and formatters,
`null-ls` is a better way to install them.

```lua
return {
  "jose-elias-alvarez/null-ls.nvim",
  opts = function(_, config)
    local null_ls = require "null-ls"
    config.sources = {
      null_ls.builtins.code_actions.cspell,
      null_ls.builtins.diagnostics.cspell
    }
    return config
  end,
}
```

With this, I had what I needed. I get warnings on possible spelling mistakes and
suggestions for what the possible fix might be. It's all good and the experience
is the same as VSCode except for one things. It seems that some words which
although, are correct in the programming language's context are being marked as
errors.

| ![VSCode Code Screen Shot](https://cdn.discordapp.com/attachments/1092561957195874406/1092576814775410708/image.png) |
| :------------------------------------------------------------------------------------------------------------------: |
|                                              _VSCode Code Screen Shot_                                               |

| ![Neovim Code Screenshot](https://cdn.discordapp.com/attachments/1092561957195874406/1092576496121565225/image.png) |
| :-----------------------------------------------------------------------------------------------------------------: |
|                                              _Neovim Code Screenshot_                                               |

The comparison above shows words being considered as errors for `cspell` running
inside Neovim and those same words being marked as correct in VSCode.

Why is this happening you might ask? It's because the base `cspell` package
essentially is no different than your system's spell checker. It ships with only
the English dictionary and the words in it which it lints your code with. So, it
still doesn't have any knowledge of the words that are valid in the context of
this language.

## The Solution

So, how do we fix this? Well, `cspell` ships all their dictionaries as _node
modules_ installable via `npm`. `cspell` as a CLI tool can be used in many ways.
One of them is to use it as a dev dependency for a Node.js project and have it
lint the code in CI/CD pipelines as well as the dev environments. So, shipping
their dictionaries as node modules makes sense. The installed dictionaries are
added to the `cspell` config file i.e `.cspell.json` in the project root.

```json
{
  "$schema": "https://raw.githubusercontent.com/streetsidesoftware/cspell/main/cspell.schema.json",
  "version": "0.2",
  "language": "en",
  "words": ["jannatin", "naim"],
  "dictionaries": ["en"]
}
```

These packages are meant to be added to individual dependencies to the project
which cspell can look for and use to properly lint the files. However, these
packages can be installed globally to use across the entire system. In my case,
I used a global `.cspell.json` file that would be used for all my projects as I
didn't want to add extra dependencies to my projects and VSCode didn't require
configurations like this.

`cspell` hosts all of their packages in their GitHub monorepo. The dictionaries
for all available languages can be found there as well. There's just one issue,
they have about a hundred of such dictionaries and each of them needs to be
individually installed.

Sure, I could've just copied and pasted the install commands manually and be be
done with it but, that's not an option with the sheer scale of the task. It
would've been fine if there were like 5 or 6 of them but, not when there are so
many.

Thankfully, the documentation has all of the available dictionaries in an
organized list which we can take advantage of. The dictionaries are listed in a
table with the package name, a description and the filetypes it should be used
in.

![CSpell's Documentation Page](https://media.discordapp.net/attachments/1092561957195874406/1092574354308603944/image.png?width=1320&height=828)

## The Process

Let's see how we can automate the installation of these packages.

The general idea I had was as follows.

- Get the README file with the package names and parse its contents.
- Filter out the section that contains the names of the packages.
- Accumulate all the package names from the parsed data.
- Use a template to inject the package name into a command that installs the
  package.
- Write all the installation commands into a script.
- Write all the packages installed into a file for reference.

With that rough set of instructions in mind, I went ahead and implemented the
code that does what I want.

At first I needed to call GitHub's API for the file. To do this, I used GitHub's
JavaScript library called [`octokit`](https://github.com/octokit/octokit.js). I
got my GitHub Personal Access Token (PAT) to authorize the library to fetch data
from GitHub. Although this wasn't a necessary step as the repository was
publicly accessible.

```javascript
const { Octokit } = require("octokit");

const http = require("https");
const fs = require("fs");

const octokit = new Octokit({
  auth: "GITHUB_PERSONAL_ACCESS_TOKEN",
});

async function main() {
  const res = await octokit.rest.repos.getReadme({
    owner: "streetsidesoftware",
    repo: "cspell-dicts",
  });

  const remoteFile = fs.createWriteStream("README.md");
  http.get(res.data.download_url, function (response) {
    response.pipe(remoteFile);
  });

  const file = fs.readFileSync("README.md").toString();
}

main();
```

Here I fetched the file from GitHub and read its contents into a string. I faced
my first hurdle over here which was that I didn't know `RegEx`. Since I had to
extract parts of a string, `RegEx` is the ideal thing to use in this case. So, I
had to improvise.

Luckily, the list was part of a markdown table which meant that each line
started with a `|` followed by a space and then the name of the package. I used
this knowledge to filter out only the strings matching that criteria. After some
more string replacing and filtering I was left with the desired array of package
names.

Now, all I had to do was map over these names and generate a template string to
install the inserted package and link it to `cspell`. Every package gets
installed via the same `npm install -g <package-name>` command format and to
link it is the same syntax of `cspell link <package-name>`. So, by putting these
two strings in a template literal I could pass in the package name to have them
be in the format I want.

```javascript
const lines = file.split("\n");

const dictLines = lines.filter((line) => line.startsWith("| [@cspell/dict"));
const dicts = dictLines.map((line) =>
  line.split("|")[1].split("]")[0].replace(" [", "")
);

// "@cspell/dict-ada/cspell-ext.json" -> "ada"
const importLines = dicts
  .map((dict) =>
    dict.replace("@cspell/dict-", "").replace("/cspell-ext.json", "")
  )
  .map((dict) => `"${dict}"`)
  .join(",\n");

// "@cspell/dict-ada/cspell-ext.json"
// ->
// npm install -g @cspell/dict-ada && cspell link add @cspell/dict-ada;
let installLines = dicts.map(
  (dict) => `npm install -g ${dict} && cspell link add ${dict};`
);
installLines.unshift("#!/bin/bash");
installLines = installLines.join("\n");
```

Since this was a shell script that I was writing, I used the `&&` and `;`
operators to separate the commands and instructions. The resulting array is a
list of commands that will install the packages for me. So, I wrote the contents
of the array into the `install.sh` file with each item starting in a new line. I
also prepended the output with a `#!/bin/sh` which signifies that the file is an
executable. That took care of my installation script. For the reference
document, I could directly write the contents of the list without any sort of
templating and that's exactly what I did.

```javascript
fs.writeFileSync("install.sh", installLines);
fs.writeFileSync("packages.txt", importLines);

fs.rmSync("README.md");
```

## The Results

After all that effort, what I was left with were 2 files; a `install.sh` file
and a `packages.txt` file. I then used the `chmod +x ./install.sh` command to
make the script executable. To run the script I did a simple `./install.sh` and
all the packages were being installed one by one without me having to do
anything.

All I had to do now was to take the package names outputted in the
`packages.txt` file and put them in the `.cspell.json` config file.

Finally, my Neovim's spell checking capabilities are at feature parity with the
one in my VSCode setup. Sure, it was quite some task but, the end result is
worth it. `null-ls` now marks most of the common words as correct and the ones
it doesn't mark as correct, I can add them to the whitelist with a simple
command.

| ![Neovim Code Screenshot](https://cdn.discordapp.com/attachments/1092561957195874406/1092576975916384266/image.png) |
| :-----------------------------------------------------------------------------------------------------------------: |
|                                              _Neovim Code Screenshot_                                               |

## Lessons Learned

The urge I had to automate such a small task took me this far into writing this
article and spending a ridiculous amount of time behind this but, what I gained
was not a simple task done. It was much more in terms of self reflection and the
things I now see I lack in.

Throughout this ordeal I realized that my string manipulation and parsing
techniques are lacking. Sure, I got what I needed done but, it wasn't the most
elegant of solutions. Using something like `RegEx` or maybe a better tool or
even a better programming language could've got it done in a more sophisticated
way I believe. However, this is an early stage for me in this domain and I wish
to explore CLI productivity and automation in depth in the coming years. With
that, I wish to improve my automation skills and tool utilization capabilities.

## Conclusion

Automating things as a developer is fun. This is not the first time I automated
a task but, it's one of the more substantial ones I've done. I had to do
something similar when I fetched all my YouTube subscriptions and listed them in
a file or the other time when I used a script to mass rename files.

In conclusion, automating tasks, even those you may only do once, can save you
time and effort in the long run. As shown in this example, setting up a spell
checker for Neovim required some additional steps beyond what was necessary for
VSCode. By taking the time to write a script to automate the installation of
necessary language packages for cspell, I was able to achieve feature parity
with the VSCode extension and improve my spell checking capabilities in Neovim.

It's always worth taking the time to automate even seemingly small tasks, as it
can make your workflow more efficient and allow you to focus on the more
important aspects of your work.
