# Git - I accidentally got into vi when I committed

It's generally easier to use the `-m` flag when doing a git commit, ie:

``` shell
> git commit -m "Description of changes"
```

but when you just type `git commit` you end up in vi (if that's your default text editor).

The way you finish off your commit is to:

1. Type in your commit message
2. Hit "Esc" to enter command mode ("i" to gets back to insert mode if you need to write some more)
3. Type `:wq` for 'write' and 'quit'

and that's it.

If you want to quit without saving it's `:q!`