---
layout: post
title:  "Custom Automcompete in Bash"
date:   2015-09-03 14:56:28
categories: autocomplete custom bash bashrc 
---

This post is about adding custom autocomplete to your bashrc file. One use case for this can, suppose you need to ssh at servers and the name of servers are long. So this can be helpful at that point of time.

Let us create a file named ```.server``` which will contain the list of your server names and place it in home folder. It is not necessary that you create a hidden file. Choice of file and location has nothing to do with this. You will just need the location of the file with the contents
in it.

Now open your ```.bashrc``` file and at the end write

```
complete -W "$(<~/.servers)" ssh
```

Here ssh is the command after which you will get the list of autocomplete.

