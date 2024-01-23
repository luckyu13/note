# Description
## error
```
PS D:\note> git push
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
## solution
> ref: *https://blog.csdn.net/weixin_45637036/article/details/106560217*

edit the `~/.ssh/config` and and the follow content
```
Host github.com
  User git
  Hostname ssh.github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
  Port 443
  
Host gitlab.com
  Hostname altssh.gitlab.com
  User git
  Port 443
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
```

