---
layout: post
title:  A quick vim tip
categories:
    - blog
---
At work I often have to work on large complex [mysql][mysql] queries, not wanting to leave [vim][vim] when I don't have to I have this great little shortcut that allows me to execute queries directly from [vim][vim]. All I need do is name the file the same as the database I want to execute the query against, save it (I use the .sql file extension - though it doesn't require it), then hit <em>leader m</em>. Assuming your setup properly, it will ssh into my dev-server and execute the query in [mysql][mysql].

Just put this in your ~/.vimrc

```
nmap <silent> <Leader>m :!ssh dev-server mysql %:r < %<CR>
```

[mysql]:    http://mysql.org
[vim]:      http://vim.org
