+++
date = '2016-09-21T16:38:13-05:00'
title = 'Spell Checking in Vim'
summary = """In the fast paced world of text editing it really helps to have a spell checker to \
catch those simple mistakes"""
tags = ['vim']
+++

In the fast paced world of text editing it really helps to have a spell checker to catch those simple 
mistakes. In the old days of vim 5, one would have to pipe their file through a command line spell 
checker like below.

```vim
:w!
:!aspell -c %
:w %
```

In vim 7 you have another choice. You can highlight spelling mistakes via the spell option.

```vim
:setlocal spell spelllang=en_us
```

This command will activate the spell option and specifies to check against US English. It is important 
to note that vim does not check grammer.

That’s it for now. Cheers!

