```
$ python -c "import pty; pty.spawn('/bin/bash')"
$ ruby -e "exec '/bin/bash'"
$ perl -e "exec '/bin/bash';"

--------

$ Ctrl+Z
$ stty raw -echo && fg
$ export TERM=xterm-256-color
```
