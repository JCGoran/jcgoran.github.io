---
layout: post
title: Using recoll on Android
tags: software recoll android termux
---
<style>
.page-content {
padding: 0;
line-height: 1.7em;
}

.post .post-content h2, .post .post-content h3, .post .post-content h4, .post .post-content h5, .post .post-content h6{
margin: 30px 0 19px;
}

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

</style>

_If you just want to skip to the TL;DR version of running recoll on Android, click [here](#tldr-version)_.

I have used the [recoll][recoll-wiki] information retrieval software for quite some time now for searching through my ever-growing collection of documents (mainly PDF and DJVU files) on the desktop, and it has proven to be quite reliable in quickly finding a paper or book which contains a certain phrase.
Ever since I got my Android-powered e-ink reader, I wanted to replicate the same workflow as I have on my desktop, but sadly, recoll was not available for Android...until now!

Thanks to the valuable [contributions][recoll-termux] by various users and developers, recoll is now available as a package on [termux][termux-wiki], the terminal emulator for Android, and works more-or-less the same way as its CLI desktop counterpart.

As recoll also provides a [web frontend][recoll-webui], below I will describe in detail how one can get it to work on Android devices.

## Installation

### F-droid, the alternative app store

As mentioned above, recoll is now available as a package on termux, a terminal emulator application for Android devices.
Unfortunately, the [Play Store version][termux-play-store] of termux has not been updated in a while, so one needs to acquire a more recent one from elsewhere.  
As Android allows alternative application stores, we can get it from [f-droid][fdroid-page], an app store containing only free, open-source Android applications.
To install it, go to [the f-droid page][fdroid-page] using your Android device, click on "Download f-droid", and open the downloaded file.
Note that you may need to enable installation from "unknown sources" to successfully install it.

### Termux

As mentioned earlier, termux is a terminal emulator for devices running Android, which brings a lot of the command-line functionality found on Linux to Android.
To install termux, open the f-droid app, click on the find button (🔍 icon), search for "termux", and finally click on the "Install" button (⭳ icon).

After the installation, open up the termux app, and write `pkg upgrade -y`, followed by <kbd class="key">Enter</kbd> (or <kbd class="key">⏎</kbd>) on the virtual keyboard to update all of the currently-installed packages on termux.
Recoll, along with its Python interface, can then be installed by running `pkg install recoll-python`.

### Additional abilities

Recoll is able to index many kinds of files (the full list can be found [here][recoll-doctypes]), but in order to be able to index the most common ones (such as PDFs), it requires some additional termux packages.
Below I will list the most common formats, and the additional packages required to index them, in the format `[FORMAT]: [PACKAGE(S)]`:

- PDF: `poppler`
- djvu: `djvulibre`
- Postscript: `ghostscript`
- Image tags: `exiftool` and `perl`

Each of the above can of course be installed using `pkg install [PACKAGE]`.

**Note**: I highly recommend installing the `aspell-en` package (and any other language packages you may use) for handling spelling approximations (see [here][recoll-aspell] for an explanation why it is useful).

## Indexing

Recoll requires an index in order to make any search queries;
to create an index of the default path(s) (the home directory), you can run:

```sh
recollindex
```

Depending on the size and number of files you are indexing, this operation may take some time;
don't worry, the command is fairly verbose and will let you know what it's currently doing.

<div style="border-width:3px; border-style:solid; border-color:indigo; border-radius:1em; margin:0 2px 1em 1em;">
<div style="margin: 0 2px 2px 1em;">
#### (_Optional_) customizing directories

By default, termux does not have access to any of the "common" directories on your device;
in order to do circumvent this, you need to run `termux-setup-storage`, and allow access to it in the dialog that appears.
This creates a [symbolic link][symlink-wiki] to the actual storage in your termux `$HOME` directory.  
Unfortunately, recoll does not follow symbolic links by default;
this can be circumvented by either:

1. setting `followLinks = 1` in `recoll.conf`
2. explicitly setting `topdirs` in `recoll.conf`

To do either of those, you should do the following:

- create a dir `~/.recoll`: `mkdir -p ~/.recoll`
- open the file `~/.recoll/recoll.conf`: `vim ~/.recoll/recoll.conf`
- depending on what you would like to do:
    - add the line `followLinks = 1` to the file
    - add the line `topdirs = [DIR1] [DIR2] ...` to the file, where `[DIR1]`, `[DIR2]`, etc. are the absolute paths to the directories which you want to index

</div>
</div>

Subsequent runs of the above `recollindex` command will only update the index, so they should finish fairly quickly.

In case you want to build a completely new index (while deleting any previous ones), use:

```sh
recollindex -z
```

There are quite a few additional options, for which I will just refer you to the [man page][recollindex-man].

## Running the web frontend

<div style="border-width:3px; border-style:solid; border-color:indigo; border-radius:1em; margin:0 2px 1em 1em;">
<div style="margin: 0 2px 2px 1em;">
#### (_Alternative_) installing from PyPI

Instead of the below, you can use the packaged version of it, which I've made available on [PyPI][recollwebui-pypi].
It allows you to install the project with only:

```sh
pip install recollwebui
```

Running the web UI is then accomplished with:

```sh
recollwebui
```

In case you would like to specify a port (see also [Troubleshooting: Python reports `address already in use`](#python-reports-address-already-in-use)), you can use:

```sh
recollwebui --port [PORT]
```

instead.

</div>
</div>

To run the web frontend, you need to first obtain the code;
this is most easily performed using git, which can be installed in termux by running:

```sh
pkg install git
```

Then, you should clone the directory using:

```sh
git clone https://framagit.org/medoc92/recollwebui.git
```

Furthermore, the web UI requires some additional Python packages, which can be installed with:

```sh
pip install waitress
```

After this, you can switch to the `recollwebui` directory with:

```sh
cd recollwebui
```

and finally run the web UI with:

```sh
python webui-standalone.py
```

after which you can switch to your web browser app, and navigate to the address `http://localhost:8080`.

The UI can be stopped by pressing the <kbd class="key">Ctrl</kbd> key, followed by the <kbd class="key">C</kbd> key on the on-screen keyboard.

Below is an example of the web UI running on my Onyx Note 3.

<img src="/assets/recoll_webui_screenshot.png" class="center" width="80%">
_The Recoll web UI running in the [EinkBro][einkbro] browser_

## Troubleshooting

### Python reports `no module named recoll`

In case you encounter this error, run the following:

```sh
export PYTHONPATH="${PYTHONPATH}:${PREFIX}/lib/python3/dist-packages"
```

and then run either `python webui-standalone.py` (if you installed with git) or `recollwebui` (if you installed from PyPI).

**Note**: you can also put this in your `~/.bashrc` file so you don't need to re-do it every time you open a new termux shell, using:

```sh
echo 'export PYTHONPATH="${PYTHONPATH}:${PREFIX}/lib/python3/dist-packages"' >> ~/.bashrc
```

### Python reports `address already in use`

You can specify the port number when you start the web UI using:

```sh
python webui-standalone.py --port [PORT]
```

where `[PORT]` is some number between 1024 and 65535.

### Python reports a `ResourceWarning` about an unclosed file

According to the developers, this is completely harmless ([source][waitress-resources]).

## TL;DR version

- install [f-droid][fdroid-page] on your device
- update the f-droid repositories (go to the "Updates" tab and swipe down)
- find (using the 🔍 icon) and install the termux app (using the ⭳ icon)
- open the termux app
- update the packages on termux by running: `pkg upgrade -y`
- install the recoll package and its common plugins on termux by running: `pkg install -y recoll-python aspell-en poppler djvulibre ghostscript exiftool perl`
- update the recoll index by running: `recollindex` (optionally [customizing the directories to index](#optional-customizing-directories))
- install the web user interface by running: `pip install recollwebui`
- run the following command in termux: `recollwebui --port 9999`
- switch to a web browser app
- enter the following address in a new tab: `http://localhost:9999`

[recoll-wiki]: https://en.wikipedia.org/wiki/Recoll
[recoll-termux]: https://github.com/termux/termux-packages/issues?q=recoll
[termux-wiki]: https://en.wikipedia.org/wiki/Termux
[recoll-webui]: https://framagit.org/medoc92/recollwebui/
[recollwebui-pypi]: https://pypi.org/project/recollwebui/
[waitress-resources]: https://github.com/Pylons/waitress/issues/264#issuecomment-615054610
[fdroid-page]: https://f-droid.org/
[termux-play-store]: https://play.google.com/store/apps/details?id=com.termux&hl=en_US&gl=US
[recoll-doctypes]: https://www.lesbonscomptes.com/recoll/pages/features.html#doctypes
[einkbro]: https://github.com/plateaukao/einkbro
[symlink-wiki]: https://en.wikipedia.org/wiki/Symbolic_link
[recollindex-man]: https://linux.die.net/man/1/recollindex
[recoll-aspell]: https://www.lesbonscomptes.com/recoll/usermanual/usermanual.html#RCL.INTRODUCTION.RECOLL
