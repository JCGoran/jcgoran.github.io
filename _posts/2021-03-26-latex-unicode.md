---
layout: post
title: Inserting uncommon characters in Xelatex projects
tags: latex fonts unicode
---

<style type="text/css">
kbd.key {
  border-radius: 3px;
  padding: 1px 2px 0;
  border: 1px solid black;
  background-color: #f2f2f2;
  box-shadow: 0px 1px 2px;
  font-family: "Noto Mono", sans-serif;
}
img.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
img + em {
  display: block;
  text-align: center;
}
p {
  page-break-inside: avoid;
}
</style>

This was enough of an annoyance for me that it warrants a short post in case anyone else encounters a similar issue.

## The problem

Let's say you want to (for whatever reason) insert a particular [Unicode][unicode-wiki] character into a Xelatex project, but when you compile the document, the compiler complains with something like this:

```plaintext
Missing character: There is no [CHARACTER] in font [FONT]!
```

To give an example, consider [this character][hieroglyph]: 𓀀.

In case the character isn't rendered properly on your viewing device, it should look something like this:

<img src="/assets/ac84ccb744e64d869eed82fd7b09cdc6.png" class="center" width="10%">

This character isn't available in the default font which Xelatex uses, Latin Modern; one option to render it correctly is to use an entirely different font for the whole document, but this may have some unintended consequences, such as _other_ characters not being rendered correctly, so let's think of something else.

## Listing available fonts

To start, we need to actually find out which fonts support this character.
On Debian, there's a handy command, `fc-list` (part of the `fontconfig` package), which lists all of the fonts installed on the computer, in a format that looks like this:

```plaintext
/usr/share/fonts/truetype/noto/NotoSansMono-ExtraBold.ttf: Noto Sans Mono,Noto Sans Mono ExtraBold:style=ExtraBold,Regular
```

Now, the first part is the actual path of the file, which can be opened using an application like the [GNOME font manager][font-manager], from which you can search for a character using <kbd class="key">Ctrl</kbd>+<kbd class="key">F</kbd>.
This is handy, but if the character is _really_ uncommon (like the one above), a guessing game of "is this character available in this font?" using the font manager is not really efficient.

## Finding the character in a font

A much better approach would be to query _all_ of the fonts installed to see if they support a given character.
Thankfully, there's a tool available that does exactly this, `hb-shape`, available in the `libharfbuzz-bin` package, which works like this:

```bash
hb-shape [FONTFILE] [CHARACTER]
```

If the character is not available in a given font, the output of the above will contain something like `[.notdef=0+250]`, or `[gid0=0+1000]`, or `[.null=0]`[^hb-output].

Since we're only interested in fonts which _do_ have support for the character, we can simply pipe the output to `grep`, and look for inverse matches with the `-v` flag:

```bash
hb-shape [FONTFILE] [CHARACTER] | grep -v '\.notdef\|gid0\|\.null'
```

Thus, if we want to query all of the installed fonts for a character, we can just run the above in a loop[^awk-ugly]:

```bash
for font in $(fc-list | awk -F: '{ print $1 }'); do
	# we're interested in the path to the font as well
	printf '%s: ' "${font}"
	hb-shape "${font}" [CHARACTER]
done | grep -v '\.notdef\|gid0\|\.null'
```

Here we're `grep`ing the whole output so we just call it once, rather than on each iteration, and the output should look something like this:

```plaintext
/usr/share/fonts/truetype/noto/NotoSansEgyptianHieroglyphs-Regular.ttf, [u13000=0+898]
```

## Using the character in the project

Now that we have a list of fonts supporting the character, we can put this somewhere in the preamble of the Xelatex project[^tex-se]:

```plaintext
\usepackage{fontspec}
\usepackage{newunicodechar}
\newfontface{\whatever}{[FONTNAME]}
{% raw %}\newunicodechar{[CHARACTER]}{{\whatever{[CHARACTER]}}}{% endraw %}
```

where `[FONTNAME]` is the name of the font file (without the extension).
The barrage of curly braces is necessary, while the name `\whatever` can be anything, as it's not used anywhere else.

Now we can directly input the Unicode character in the project, compile it with the Xelatex PDF compiler, and the character will be rendered correctly in the output!

<img src="/assets/9bbba6bae0994a4a9e7fa20c65d5d9c1.png" class="center" width="50%">
_Part of the PDF with the character from above_

Note that there may be issues with loading fonts due to spaces in the filename, but the idiosyncrasies of Latex have caused me to waste far too much time just on this one simple problem, so I'll leave the debugging of filenames as an exercise to the reader (a good starting point is [this tex.SE answer][tex-fonts]).

[unicode-wiki]: https://en.wikipedia.org/wiki/Unicode
[hieroglyph]: https://www.compart.com/en/unicode/U+13000
[tex-fonts]: https://tex.stackexchange.com/a/292810
[font-manager]: https://github.com/FontManager/font-manager
[^awk-ugly]: the man page of `fc-list` doesn't specify what are the available options for the `--format` flag so I've opted for using `awk` instead to clear up the path
[^tex-se]: credits: [this tex.SE answer](https://tex.stackexchange.com/a/531619)
[^hb-output]: it's possible that there are other codes that symbolize that the character is unavailable
