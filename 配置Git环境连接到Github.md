#配置Git环境连接到Github


很早就开始使用git来管理自己的项目了,以前为了偷懒,直接下了100多M的github客户端来就获取Github上面的代码,最近公司的版本管理工具开始从svn转移到git上面,现在重新补一下git的知识;

需要说明的是,我的ssh目录下面已经存在了一个id_rsa,现在是为了另外配置一个新的密钥用来验证github的连接的;遇到了三个问题:(归根结底都是因为自定义名词的公钥没有执行)

* Could not open a connection to your authentication agent
* No more authentication methods to try. Permission denied (publickey).
* Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.

##下面是完整的部署流程
###先用ssh-keygen生成一个新的密钥
```bash
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh-keygen -t rsa -C "wangmax0330@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Methew/.ssh/id_rsa): github_id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in github_id_rsa.
Your public key has been saved in github_id_rsa.pub.
The key fingerprint is:
SHA256:7vPPUF8ecpFgFCU3BH93P7wPWechrN/zOHY0rc/yezI wangmax0330@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
|            .B== |
|            . = o|
|               ++|
|            . . *|
|        S   .+ **|
|       .   ...+BO|
|        . ..  +++|
|       ..  o. E=+|
|        .o..oo.@@|
+----[SHA256]-----+
```
 


###查看生成的密钥列表
```bash
Methew@Methew-PC MINGW64 ~/.ssh
$ ls
github_id_rsa  github_id_rsa.pub  id_rsa  id_rsa.pub  known_hosts
```
复制github_id_rsa.pub 公钥到github账号里面的SSH key里面去,add an new ssh key
可以使用shell 命令: cat github_id_rsa.pub 获取里面的公钥的内容


###使用 ssh -T git@github.com 连接Github测试是否能通:
```bash
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.252.131)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.252.131' (RSA) to the list of known hosts.
Permission denied (publickey).
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh -T git@github.com
Warning: Permanently added the RSA host key for IP address '192.30.252.128' to the list of known hosts.
Permission denied (publickey).
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh -T git@github.com
Permission denied (publickey).
```
发现出错了,我们加上-v 参数进行调试,打印完整的信息
```bash
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh -v git@github.com
OpenSSH_7.1p1, OpenSSL 1.0.2d 9 Jul 2015
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: Connecting to github.com [192.30.252.131] port 22.
debug1: Connection established.
debug1: identity file /c/Users/Methew/.ssh/id_rsa type 1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_7.1
debug1: Remote protocol version 2.0, remote software version libssh-0.7.0
debug1: no match: libssh-0.7.0
debug1: Authenticating to github.com:22 as 'git'
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client chacha20-poly1305@openssh.comnone
debug1: kex: client->server chacha20-poly1305@openssh.comnone
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: Server host key: ssh-rsa SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8
debug1: Host 'github.com' is known and matches the RSA host key.
debug1: Found key in /c/Users/Methew/.ssh/known_hosts:2
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /c/Users/Methew/.ssh/id_rsa
debug1: Authentications that can continue: publickey
debug1: Trying private key: /c/Users/Methew/.ssh/id_dsa
debug1: Trying private key: /c/Users/Methew/.ssh/id_ecdsa
debug1: Trying private key: /c/Users/Methew/.ssh/id_ed25519
debug1: No more authentication methods to try.
Permission denied (publickey).
```
发现默认加载的密钥是id_rsa,我自定义名字github_id_rsa根本没有使用。同时可以使用如下命令查看密钥列表：
```bash
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh-add -l
Could not open a connection to your authentication agent.
```

如果执行ssh-add时提示"Could not open a connection to your authentication agent"，可以现执行命令：
```bash
Methew@Methew-PC MINGW64 /e/Workspaces/OwnWorkspace
$ ssh-agent bash
bash: __git_ps1: command not found
```
ssh-agent是一种控制用来保存公钥身份验证所使用的私钥的程序，其实ssh-agent就是一个密钥管理器，运行ssh-agent以后，使用 ssh-add将私钥交给ssh-agent保管，其他程序需要身份验证的时候可以将验证申请交给ssh-agent来完成整个认证过程。ssh-agent可以把自己的私钥加密缓存，ssh内部的机制可以在通迅过程中把缓存的私钥安全的带到目标机上,受管理的私钥通过ssh-add来添加，所以ssh-agent的客户端都可以共享使用这些私钥,大致使用它有两点好处:
好处1：不用重复输入密码。
	用 ssh-add 添加私钥时，如果私钥有密码的话，照例会被要求输入一次密码，在这之后ssh-agent可直接使用该私钥，无需再次密码认证。
好处2：不用到处部署私钥
	假设私钥分别可以登录同一内网的主机 A 和主机 B，出于一些原因，不能直接登录 B。可以通过在 A 上部署私钥或者设置 PortForwarding 登录 B，也可以转发认证代理连接在 A 上面使用ssh-agent私钥登录 B。

###添加github_id_rsa 密钥到agent 里面去

```bash
$ ssh-add -l
The agent has no identities.
bash: __git_ps1: command not found
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh-add ~/.ssh/
github_id_rsa      id_rsa             known_hosts
github_id_rsa.pub  id_rsa.pub
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh-add ~/.ssh/github_id_rsa
Identity added: /c/Users/Methew/.ssh/github_id_rsa (/c/Users/Methew/.ssh/github_                id_rsa)
bash: __git_ps1: command not found
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh-add -l
2048 SHA256:7vPPUF8ecpFgFCU3BH93P7wPWechrN/zOHY0rc/yezI /c/Users/Methew/.ssh/github_id_rsa (RSA)
bash: __git_ps1: command not found
```

再测试一下 ssh -v git@github.com
```bash
Methew@Methew-PC MINGW64 ~/.ssh
$ ssh -v git@github.com
OpenSSH_7.1p1, OpenSSL 1.0.2d 9 Jul 2015
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: Connecting to github.com [192.30.252.129] port 22.
debug1: Connection established.
debug1: identity file /c/Users/Methew/.ssh/id_rsa type 1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /c/Users/Methew/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_7.1
debug1: Remote protocol version 2.0, remote software version libssh-0.7.0
debug1: no match: libssh-0.7.0
debug1: Authenticating to github.com:22 as 'git'
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client chacha20-poly1305@openssh.comnone
debug1: kex: client->server chacha20-poly1305@openssh.comnone
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: Server host key: ssh-rsa SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8
debug1: Host 'github.com' is known and matches the RSA host key.
debug1: Found key in /c/Users/Methew/.ssh/known_hosts:2
Warning: Permanently added the RSA host key for IP address '192.30.252.129' to the list of known hosts.
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /c/Users/Methew/.ssh/github_id_rsa
debug1: Server accepts key: pkalg ssh-rsa blen 279
debug1: Authentication succeeded (publickey).
Authenticated to github.com ([192.30.252.129]:22).
debug1: channel 0: new [client-session]
debug1: Entering interactive session.
PTY allocation request failed on channel 0
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
Hi wangmax0330! You've successfully authenticated, but GitHub does not provide shell access.
debug1: channel 0: free: client-session, nchannels 1
Connection to github.com closed.
Transferred: sent 3388, received 1796 bytes, in 0.5 seconds
Bytes per second: sent 6244.4, received 3310.2
debug1: Exit status 1
bash: __git_ps1: command not found
```
Hi wangmax0330! You've successfully authenticated, but GitHub does not provide shell access.表示成功连接上githuble
现在clone 一个github项目到本地
```bash
Methew@Methew-PC MINGW64 /e/Workspaces/OwnWorkspace
$ git clone git@github.com:wangmax0330/pikia-test.git
Cloning into 'pikia-test'...
remote: Counting objects: 863, done.
Receiving objects:   7% (63/863), 2.80 MiB | 691.00 KiB/s
Receiving objects: 100% (863/863), 8.62 MiB | 372.00 KiB/s, done.
remote: Total 863 (delta 0), reused 0 (delta 0), pack-reused 863
Resolving deltas: 100% (276/276), done.
Checking connectivity... done.
bash: __git_ps1: command not found
```

本来到这一步就没问题了,但是后来我又发现如果我把git bash关闭(其实就是agent 服务kill掉了)重新打开clone另外的项目就会出现错误
```bash
Methew@Methew-PC MINGW64 /e/Workspaces/OwnWorkspace
$ git clone git@github.com:wangmax0330/spring-boot-examples.git
Cloning into 'spring-boot-examples'...
Permission denied (publickey).
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.
```
当然你也可以重启agent,然后把密钥加到agent里面,但是为了方便,我们需要设置一下配置config,这里面有三种config,
1. /etc/gitconfig 文件：包含了适用于系统所有用户和所有库的值。如果你传递参数选项’--system’ 给 git config，它将明确的读和写这个文件。 
2. ~/.gitconfig 文件 ：具体到你的用户。你可以通过传递--global 选项使Git 读或写这个特定的文件。
3. 位于git目录的config文件 (也就是 .git/config) ：无论你当前在用的库是什么，特定指向该单一的库。每个级别重写前一个级别的值。因此，在.git/config中的值覆盖了在/etc/gitconfig中的同一个值。

~/.ssh/config（用户专用）和/etc/ssh/ssh_config（全局共享）。要按照该顺序读取这些文件，对于给定的某个参数，它使用的是读取过程中发现的第一个配置。用户可以通过以下方式将全局参数设置覆盖掉：在自己的配置文件中设置同样的参数。在ssh或scp命令行上给出的参数的优先级要高于这两个文件中所设置的参数的优先级。

下面是config 的配置,配置好之后,就可以直接连接github了,不用启动agent,以后如果有其他地方需要密钥了,可以重新制定一个复制粘贴,改一些Host和IdentityFile
```bash
Methew@Methew-PC MINGW64 /e/Workspaces/OwnWorkspace
$ cat ~/.ssh/config
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```
注意: Host 查阅资料之后,发现是别名设置,用于设置ssh连接用的,但是发现配置github的时候,HostName是可有可无的,,但是Host是必须写成github.com,不然是会报错的;