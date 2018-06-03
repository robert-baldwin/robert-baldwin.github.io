---
layout: post
title: "Vim Leader Commands"
date: 2018-06-01 11:06:03 -0800
tags: [git, vim]
excerpt_separator: <!--more-->
---

Defining custom leader commands in `vim` are easy and powerful. This gist from Github user stantok serves as a demonstration<sup>[1](https://gist.github.com/stantonk/9f4f103e473fa00567c4)</sup>. Let's build on this to see how the visual selection evolves over time with `git log`.

<!--more-->

## Analyzing the Command

The command uses git blame<sup>[2](https://www.git-scm.com/docs/git-blame)</sup> on the current file, filtering out lines outside of the visual selection with `sed`.

{% highlight viml %}
vmap <Leader>g :<C-U>!git blame <C-R>=expand("%:p") <CR> \| sed -n <C-R>=line("'<") <CR>,<C-R>=line("'>") <CR>p <CR>
{% endhighlight %}

With a visual selection we can press the key combination \ followed by g to execute:

{% highlight bash %}
git blame <current_file> |
sed -n <first_line_number>:<last_line_number:>p
{% endhighlight %}

Let's break this down piece by piece.

### Assigning the keymap

{% highlight viml %}
vmap <Leader>g
{% endhighlight %}

`vmap` denotes a visual+select mode command<sup>[3](http://vimdoc.sourceforge.net/htmldoc/map.html#mapmode-v)</sup>. This means the `<Leader>g` key combination will trigger the following command only if in visual mode with a visual selection.

`<Leader>` is the currently defined mapleader<sup>[4](http://vimdoc.sourceforge.net/htmldoc/map.html#mapleader)</sup>. It is set to `\` by default. You can check yours with `:echo mapleader` or set it to a new value for example `:let mapleader=","`. In our case we'll keep the default and trigger our command with the key combination \ followed by g.

### Shelling out

Now that we have our keymap set up we need a way to evoke the `git blame` command and pass it values from vim.

{% highlight viml %}
:<C-U>!git blame <C-R>=expand("%:p") <CR>
{% endhighlight %}

`:` enters "last-line mode" where vim expects a command.

`<C-U>` clears the command line. In visual mode vim automatically inserts `'<,'>` so the executed command applies only to the visual selection. `<C-U>` clears that out leaving us with `:`<sup>[5](https://stackoverflow.com/questions/13830874/why-do-some-vim-mappings-include-c-u-after-a-colon)</sup>. The visual selection's line numbers later in the `sed` command.

`!` lets us shell out of vim. We'll use it to execute our `git blame` piped to `sed`.

`<C-R>=` inserts the result of an expression at the cursor. `expand("%:p")` returns the full path of the current file. `<CR>` is a carriage return, the equivalent of pressing the enter key to execute the expression. Together will insert the full path of the current file at the current cursor.

### Filtering with sed

Now that we can shell out and pass the current path to `git blame` we want to pipe the output to `sed` to tailor our output to the current visual selection.

{% highlight viml %}
sed -n <C-R>=line("'<") <CR>,<C-R>=line("'>") <CR>p
{% endhighlight %}

What it looks like with a visual selection selection of lines 22 and 31 inclusive after substitution:

{% highlight bash %}
sed -n 22,31p
{% endhighlight %}

Using the -n option we can suppress printing unless explicitly requested<sup>[6](http://www.grymoire.com/Unix/Sed.html#uh-31)</sup>. Then we explicitly request the inclusive range between the first ('<) and last ('>) lines of our visual selection.

## Making improvements

The `git blame` command takes an argument `-L` that directly annotates a given range of lines<sup>[7](https://www.git-scm.com/docs/git-blame#git-blame--Lltstartgtltendgt)</sup>. This allows us to strip `sed` from our command entirely.

{% highlight viml %}
vmap <Leader>g :<C-U>!git blame -L <C-R>=line("'<")<CR>,<C-R>=line("'>")<CR> <C-R>=expand("%:p")<CR><CR>
{% endhighlight %}

## Using git log -L

With only some minor tweaks we can adapt this for `git log`<sup>[8](https://www.git-scm.com/docs/git-log#git-log--Lltstartgtltendgtltfilegt)</sup>.

{% highlight viml %}
vmap <Leader>dl :<C-U>!git log -L <C-R>=line("'<")<CR>,<C-R>=line("'>")<CR>:<C-R>=expand("%:p")<CR><CR>
{% endhighlight %}

Using `getline` in place of `line` we can make use of the regex option if it is more appropriate.

{% highlight viml %}
vmap <Leader>dr :<C-U>!git log -L '/^<C-R>=getline("'<")<CR>$/','/^<C-R>=getline("'>")<CR>$/':<C-R>=expand("%:p")<CR><CR>
{% endhighlight %}

Three keystrokes can now give a glimpse at the individuals who wrote the lines in the selection at what times, as well as a history of how the selection evolved over time.

