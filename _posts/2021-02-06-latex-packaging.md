---
layout: post
title:  "Packaging a Latex project"
tags: latex arxiv
mathjax: true
---
{% include mathjax.html %}

## The problem

Whenever I'm working on a LaTeX project, I inadvertently end up with a bunch of files scattered throughout the project directory.
Usually, not all of them are actually used to build the final document, and I'd like a way to package the project without any unnecessary files, so I can submit it to arXiv or somewhere else.

### The not-so-good solutions

At first, I've thought of several naive ways to accomplish this:

1. Submit each file manually via the arXiv interface. This is quite tedious, error-prone, and time-consuming, and arXiv offers an option to upload an archive of the project (`tar.gz`, `zip`, or whatever) anyway, and thus, should only be used as a last resort.
2. Use `tar` to package just the necessary files. This basically has the same cons as the previous option, so it's also not optimal.
3. Using git, run `git archive` to export an archive of the project.

Since I've been using git for almost all of my personal and professional projects, I've hastily opted for option 3; unfortunately, while working on the project, I've accidentally tracked a bunch of PDF figures and data files as well, and, as a result, I still had to semi-manually (read: with a lot of experimentation with `grep` and various other utilities) prune the archived file.
This was still quite tedious, and I was hoping to use a less error-prone and more automated method in the future.

### The automated solution

Out of curiosity, I've decided to just google for any options that might be available, and I've come across a tool built to solve this exact issue: `bundledoc`.

This program is available on Debian/Ubuntu based Linux distributions as part of the `texlive-extra-utils` package (also as `texlive-core` in Arch), and can be installed by simply running:

```bash
apt install texlive-extra-utils
```

Note that `bundledoc` makes use of the `snapshot` LaTeX package, which is available in the `texlive-latex-extra` package, so this package should be installed as well if `apt` doesn't do it automatically.

On macOS, the `bundledoc` program and the `snapshot` package are available on [MacPorts][macports], in the packages `texlive-bin-extra` and `texlive-latex-extra`.

### Usage

Using `bundledoc` is simple - just insert the following in your primary `[NAME].tex` file, even before `documentclass`:

```tex
\RequirePackage{snapshot}
```

Then just rebuild the project (using whatever CLI/GUI build system you prefer, my new favorite is `latexmk`), and you should see an additional `[NAME].dep` file somewhere in your project directory.
To package everything up into a compressed archive (`tar.gz`), simply run:

```bash
bundledoc --localonly --manifest="" [NAME].dep
```

This will produce an archive `[NAME].tar.gz` which contains just the necessary files for rebuilding your _entire_ project from scratch.
The flags in the above command are not strictly mandatory, but without them, the created archive may be littered with a bunch of `sty` files, which are usually available on the arXiv server anyway.

For more customization options, take a look at the `man` page of `bundledoc`.
To verify `bundledoc` packaged everything correctly, just un`tar` the created archive in an empty directory on your computer (using `tar xf [ARCHIVE]`), and then rebuild it there (again, using your build system of choice).
One of the perks of using `bundledoc` is that you can readily submit this freshly-built archive to arXiv without the need for any additional changes.

### Notable issues

While `bundledoc` is pretty handy, there _is_ one problem I've encountered though: the resulting compressed archive has the following structure:

```plaintext
├── NAME
│   ├── file1.tex
│   ├── file2.tex
│   ├── file3.tex
...
```

As you can see, there is a top directory _inside_ the archive, which means this won't work:

```bash
tar xf [ARCHIVE] && [BUILD_SYSTEM]
```

but there needs to be a `cd` somewhere in between (or a flag given to `[BUILD_SYSTEM]` which tells it where to build it from).

This isn't a big deal on my own system, but can be an issue if the journal I'm submitting the project to has _very_ strict guidelines on what the layout of the archive should be to correctly process it.

Unfortunately, giving an empty top directory name to `bundledoc` doesn't work:

```bash
$ bundledoc --localonly --directory="" --manifest="" [NAME].dep
Option directory requires an argument
```

Using anything other than the empty string works, but hacks such as using `.` or `/` to prevent it from making an actual top level directory will just give an error: `bundledoc: File exists`.

The following Bash/Zsh script takes care of this particular problem by extracting the archive, _without_ the top level directory, in a temporary location, and then re-packaging it in a new archive[^1]:

```bash
# USAGE: remove_topdir [ARCHIVE]
remove_topdir(){
    if [[ $# -ne 1 ]]
    then
        echo "USAGE: remove_topdir [ARCHIVE]"
        return 1
    fi
    # the name of the archive
    archive="${1}"
    # check that the path exists and is a file
    if ! [[ -f "${archive}" ]]
    then
        echo "ERROR: ${archive} does not exist or is not a file!"
        return 2
    fi
    # the name of the output archive
    archive_new="${archive/.tar.gz/-trimmed.tar.gz}"
    # make a temp dir
    tmp_dir="$(mktemp -d)"
    # extract the contents of the archive in that dir
    # we remove the top level dir in the achive
    tar --strip-components=1 --extract --file="${archive}" -C "${tmp_dir}"
    # re-compress it all in the new archive
    tar cf "${archive_new}" -C "${tmp_dir}" .
    # error handling
    if [[ $? -ne 0 ]]
    then
        echo "Something went wrong!"
        return 3
    else
        echo "New archive is located at ${archive_new}"
        return 0
    fi
}
```

Appending this to your `~/.bashrc` file allows running the following to get the right directory structure:

```bash
bundledoc --localonly --manifest="" [NAME].dep && remove_topdir [NAME].tar.gz
```

which creates a file `[NAME]-trimmed.tar.gz` ready for uploading.

[astro-ph]: https://twitter.com/LeaksPh
[macports]: https://www.macports.org/
[^1]: for why you can't just pipe the uncompressed output directly back into tar, see [this AskUbuntu answer](https://askubuntu.com/a/587046)
