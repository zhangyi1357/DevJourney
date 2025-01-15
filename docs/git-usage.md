# Git 使用指南

## Git 命令

配置用户名和邮箱：

````
git config --global user.name "zhangyi1357"
git config --global user.email "zhangyi_1357@sjtu.edu.cn"
````

设置默认远端为 origin，设置默认 push 策略为 current，即远端不存在同名分支自动创建新分支：

```bash
git config remote.pushDefault origin
git config push.default current
```

## GitHub 相关

### SSH 配置和使用

参考链接：

1. [Generating a new SSH key and adding it to the ssh-agent - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
2. [Adding a new SSH key to your GitHub account - GitHub Docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

命令行摘要如下：

```bash
# Generate ssh keys
ssh-keygen -t ed25519 -C "your_email@example.com"

# Paste the following content into personal github settings
cat ~/.ssh/id_ed25519.pub

# Add the secret key into ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519.pub
```

### SSH 连接报错

报错信息如下：

```bash
$ git push -u origin main
kex_exchange_identification: Connection closed by remote host
Connection closed by 198.18.0.175 port 22
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

原因：大概率是开启了 VPN 导致相应端口拒绝访问。

解决方案：根据 [stackoverflow 链接](https://stackoverflow.com/a/60994276/18147684)中给出的方法，将 GitHub 的 ssh 端口修改为 443 即可，具体在 ~/.ssh/config 中填写以下内容：

````
Host github.com
 Hostname ssh.github.com
 Port 443
````