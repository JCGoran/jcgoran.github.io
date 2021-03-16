---
layout: post
title: Creating standalone Latex math images
tags: latex scripting
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

Just like many others working in the natural sciences, I too have been using Latex since forever, just because it's the only mature solution for typesetting math in a more or less sane way.

For the longest time though, I've been looking for a way to create just images of math expressions in a simple way, without necessarily having to go through this tedious process:

1. make a Latex document, with all the boilerplate code included
2. compile the document into a PDF or PostScript format
3. convert the result into an image file somehow (usually with something like [ImageMagick][imagemagick-wiki])

This is a nightmare to do on a regular basis, and, ever since I've discovered that my email client (Thunderbird) has support for inline images, I've ventured far and wide looking for some kind of a solution to make the above process more bearable, because there _has_ to be a better way to do this!
At one point I even wrote my own bare-bones shell script to do it, but it was a major hassle to use the Latex compiler, with a lot of arcane flags and whatnot, so I didn't pursue it further.
Online solutions seem like a viable alternative, but I'd prefer not to have to open a new browser tab each time I'm in need of something seemingly simple as this, especially as I already have about 5 GBs worth of Latex packages just sitting there on my computer.

After some time, I've stumbled upon [this tex.SE answer][tex-se-pnglatex], which pointed me to the amazing [pnglatex][pnglatex-repo] script, that basically does _exactly_ what I want - namely, makes a PNG image from a Latex math expression!

Using the script is straightforward, just pass whatever Latex math expression as an argument to the `-f` option, and the script will return the path to where the image is saved, like so:

```bash
user@host:~$ ./pnglatex -f '\displaystyle \int\limits_a^b f(x) \mathrm{d}x = F(b) - F(a)'
/home/user/fXYZ.png
```

The `XYZ` are 3 random characters that the script chooses on its own to prevent overwriting any existing files in the current directory.

The output image is shown below:

<img src="/assets/617aaeaf00b34a009c17b7b91ffe489d.png" class="center" width="40%">
_Example output_

Don't mind the blurriness, we'll get to that later.

The filename can of course be customized, by using the `-o [FILENAME]` option, where `[FILENAME]` is the path where you'd like to save the image.
The usual Latex rules apply, such as escaping the comment character (so `\%` outputs a literal percent sign, and `%` doesn't output anything), as well as shell expansion, which is why you should use single quotation marks (`'like this'`) instead of double ones.

This solution is _almost_ perfect, but it still leaves (by default) a file in the current directory, which I afterwards probably won't be using anyway - if I'm writing a paper or similar, I may as well put the equation directly in the source file, and if it's for something else, it's almost always for a one-time use.

As a result, I've decided to implement a feature in [my own fork][pnglatex-branch] of `pnglatex` to allow outputting the PNG file directly on `stdout` by specifying `-o -`, so that I can finally do this (requires the `xclip` package to work):

```bash
./pnglatex -f 'E = m c^2' -o - | \
	xclip -selection clipboard -t image/png
```

This way, I can directly paste the image anywhere using <kbd class="key">Ctrl</kbd>+<kbd class="key">V</kbd>, without having to open a folder to retrieve the image (since it's not saved to disk anyway).

I'm currently using the following wrapper function for the above, which has some nicer defaults:

```bash
pnglatex(){
    if [ $# -ne 1 ]
    then
        printf "usage: pnglatex [MATH EXPRESSION]\n"
        printf "Automatically pipes to the clipboard\n"
        return 1
    fi

    bash /path/to/pnglatex \
        -f "$1" \
        -d 200 \
        -b Transparent \
        -o - | \
        xclip \
            -selection clipboard \
            -t image/png

    return 0
}
```

The `-d 200` option specifies the [DPI][dpi-wiki] to be 200 (as shown earlier, the default image is a bit too low-resolution), while the `-b Transparent` makes the image background fully transparent.

Here's a sample input to the wrapper:

```bash
pnglatex 'G_{\mu\nu} = 8 \pi G T_{\mu\nu}'
```

and the corresponding output is (obtained by directly pasting from the clipboard):

<img src="/assets/14134adf623d4a6fad45709df1790820.png" class="center">
_Example output from the wrapper function_

So far, I'm very pleased with the script, and have been using it on a regular basis to quickly generate equations for email correspondences and Google Docs presentations (there's apparently an [addon][auto-latex] for Latex rendering in Docs, which I have yet to try out), though in principle it should work in any other software/website that allows pasting of PNG images.

[tex-se-pnglatex]: https://tex.stackexchange.com/a/279405
[pnglatex-repo]: https://github.com/mneri/pnglatex
[pnglatex-branch]: https://github.com/JCGoran/pnglatex/tree/stdout-fix
[auto-latex]: https://sites.google.com/site/autolatexequations/home
[imagemagick-wiki]: https://en.wikipedia.org/wiki/ImageMagick
[dpi-wiki]: https://en.wikipedia.org/wiki/Dots_per_inch
