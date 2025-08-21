```bash
vim ~/.gitconfig

[user]
	name = dien.vo
	email = dien.vo@andpad.co.jp
[url "ssh://git@github.com/"]
    insteadOf = https://github.com/

## then
ssh-add -D && ssh-add ~/.ssh/id_ed25519
```


https://code.tutsplus.com/quick-tip-how-to-work-with-github-and-multiple-accounts--net-22574t