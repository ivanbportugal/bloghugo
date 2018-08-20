+++
author = "Ivan Portugal"
date = "2018-08-20T00:35:13+00:00"
post_date = "2018-08-19T04:00:00+00:00"
tags = ["git", "log", "graph"]
title = "Git Log Goodies"

+++
I have always found that learning Git was easier with a GUI but I always seemed to keep hitting some sort of "capability ceiling"... There are several out there and I particularly enjoyed SourceTree from Atlassian and GitKraken from Axosoft. Here are some others: [Git Guis](https://git-scm.com/download/gui/windows)

However, the command line is _significantly_ more powerful. Here is a handy-fancy git log with a graph. Add this to your `.gitconfig`:

```shell
[alias]
    lg1 = log --graph --all --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(bold white)— %an%C(reset)%C(bold yellow)%d%C(reset)' --abbrev-commit --date=relative
    lg2 = log --graph --all --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(bold white)— %an%C(reset)' --abbrev-commit
```

Then, try it out!

```shell
git lg1
```

![Git Log Graph Sample](/bloghugo/img/GitLogGraphSample.png)