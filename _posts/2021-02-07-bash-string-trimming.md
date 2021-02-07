---
layout: post
title: Trimming a string with Bash
tags: bash scripting
mathjax: false
---
{% include mathjax.html %}


### The problem

Let's say we have a piece of text that looks like this:

```plaintext

			 
this    string    	
  has  	 whitespace

 everywhere 
 why	 	
		 
```

and the task is to delete all of the leading and trailing whitespace characters, that is, trim the string, using Bash.

To make it clear which characters are actually present in the above, here's the original string with [escape sequences][escape-wikipedia] shown:

```plaintext
\n\t\t\t \nthis    string    \t\n  has  \t whitespace\n\n everywhere \r\n why\t \t\n\t\t 
```

The reason why I'm restricting this problem to Bash is because this can be done in Python in one line using the built-in `strip` method:

```python
s.strip() # prints 'this    string    \t\n  has  \t whitespace\n\n everywhere \r\n why'
```

so let's see how we can do it in Bash.

### Finding solutions online

Googling the term 'trim string bash' gives [this article][trim-string] as the first hit, which offers a couple of solutions.

Much to my surprise, after running their code on my seemingly simple example, none of them worked as expected, and either removed too few, or too many characters.
Other solutions I've encountered seem to be assuming we're dealing with strings that don't have any newline characters (`\n`).
Additionally, they often use other utilities (commonly `sed`, `awk`, and `xargs`), so it would be useful to have a solution using only Bash-isms instead.

### String removal

In Bash, we can remove a given character or string from the front and the back of an input string using the following syntax:

```bash
x='some string'
echo "${x#s}" # will print 'ome string'
echo "${x%g}" # will print 'some strin'
```

We can also remove any one of the following characters in the square brackets as demonstrated in [this SO answer][so-whitespace] by using:

```bash
# removes the specified (whitespace) characters from the beginning
echo "${x#[$'\r\t\n ']}"
```

Note that the list of characters to be removed is specified as a string with a `$` prefix, because then the characters are properly escaped, as explained [in this Unix.SE answer][dollar].

Digging through the Bash `man` page reveals that it's instead possible to just use the keyword `[:space:]` to explicitly specify the entire class of whitespace characters, so the generalization of the above is then:

```bash
# removes _any_ whitespace character from the beginning
echo "${x#[[:space:]]}"
```

### Test code

Since we can get the size of a string using `${#variable}`, our task is straightforward - keep removing whitespace characters until there's nothing else to remove, i.e. until `size(string before trimming) == size(string after trimming)`, which is achieved with the following code:

```bash
s='		some string  '

size_before=${#s}
size_after=0
while [[ ${size_before} -ne ${size_after} ]]
do
	size_before=${#s}
	s="${s#[[:space:]]}"
	s="${s%[[:space:]]}"
	size_after=${#s}
done

echo "${s}" # prints 'some string'
```

Note that using something like `${s##[[:space:]]}` won't work properly.
According to the Bash manual, this would remove the _longest substring matching the pattern_, which means if we our original string was, say, `\t\t\n\t actual string`, it would just remove `\t\t`, and leave the rest as-is.

### Putting it all together

To make things handy, we can put everything in a function called `trimstring`, which can then be added to a `~/.bashrc` file or similar:

```bash
trimstring(){
    if [ $# -ne 1 ]
    then
        echo "USAGE: trimstring [STRING]"
        return 1
    fi
    s="${1}"
    size_before=${#s}
    size_after=0
    while [ ${size_before} -ne ${size_after} ]
    do
        size_before=${#s}
        s="${s#[[:space:]]}"
        s="${s%[[:space:]]}"
        size_after=${#s}
    done
    echo "${s}"
    return 0
}
```

After a lot of testing, below is a table of commonly used shells in which the above works and doesn't work.

[//]: # (make the table overflow if it's too wide to display; usually on mobile devices)
[//]: # (see:)
[//]: # (https://www.w3schools.com/css/css_overflow.asp)
[//]: # (https://stackoverflow.com/a/36507265)
[//]: # (https://stackoverflow.com/a/41085526)

<style type="text/css">
.table-wrapper {

  overflow-x: auto;

}
</style>

<div class="table-wrapper" markdown="block">

| **Shell** | Ash | Bash | Dash | Csh | Tcsh | Ksh | Zsh | Fish |
| -- | -- | -- | -- | -- | -- | -- | -- | -- |
**Works?** | <span style="color:green;font-size:25px">✓</span> | <span style="color:green;font-size:25px">✓</span> | <span style="color:green;font-size:25px">✓</span> | <span style="color:red;font-size:25px">✗</span> | <span style="color:red;font-size:25px">✗</span> |  <span style="color:green;font-size:25px">✓</span> | <span style="color:green;font-size:25px">✓</span> | <span style="color:red;font-size:25px">✗</span> |

</div>

### Appendix: solution using GNU Sed

After some more googling, the Sed-based solution from the article mentioned above works if we restrict ourselves to using GNU Sed, which has a `-z` option[^sed-z] that treats the null character as the end of a line instead; this means that `$` will only match the end of the whole text stream instead of individual newline (`\n`) characters, while `^` will match the beginning of the stream.
This allows us to make the following script:

```bash
trimstring_sed(){
	s="${1}"
	s="$(printf "${s}" | sed -z 's/^[[:space:]]*//')"
	s="$(printf "${s}" | sed -z 's/[[:space:]]*$//')"
	echo "${s}"
	return 0
}
```

The above basically matches zero or more instances of any whitespace character at the beginning and end of the input string, and removes them.

Comparing the timings of `trimstring` with `trimstring_sed` gives `trimstring` an obvious edge when it comes to speed though:

```bash
string="$(cat test.txt)" # contains the initial string

time for i in {1..1000}; do trimstring "${string}" > /dev/null; done
real	0m0.222s
user	0m0.217s
sys	0m0.006s

time for i in {1..1000}; do trimstring_sed "${string}" > /dev/null; done
real	0m3.521s
user	0m2.853s
sys	0m1.270s
```

Thus, there's at least an order of magnitude difference between just using built-in Bash-isms vs. using an external tool like Sed; of course, the Python solution is by far the fastest, but only if we're already inside the interpreter, and aren't using a shell in the first place.

[escape-wikipedia]: https://en.wikipedia.org/wiki/Escape_sequences_in_C#Table_of_escape_sequences
[so-whitespace]: https://stackoverflow.com/a/19347380
[dollar]: https://unix.stackexchange.com/a/48122
[wiki-whitespace]: https://en.wikipedia.org/wiki/Whitespace_character#Unicode
[trim-string]: https://linuxhint.com/trim_string_bash/
[^sed-z]: see [this SO answer](https://stackoverflow.com/a/52538392) and/or [this Unix.SE answer](https://unix.stackexchange.com/a/525524)
