---
layout: post
title: "ansible_with_vagrant"
date: 2014-08-12 01:03:44 +0900
comments: true
published: false
author: xengineer01
categories: 
---

はい、前回の宣言通り、ansibleエントリを書く[@xengineer01](https://twitter.com/xengineer01)です。  

## ansible とは？
---
[公式サイト](http://www.ansible.com/)曰く。

Ansible is the simplest way to automate IT.
つまり、IT を自動化する、一番簡単な方法だよーん。と。

海外のサイトの説明ってこんな感じな気がする。他のも調べてみよう・・・ 

#### chef
Automation for Web-Scale IT.  
お、ちょっとわかりやすい。

#### apache
The Number One HTTP Server On The Internet  
あ、すごい直球だった。

#### MySQL
The world's most popular open source database  
うむ。ちょっと有名どころすぎるとわかりやすくなりすぎる。

#### github
Build software better, together.  
コンセプトわかりやすい。

さて、茶番はこの辺にして本題へ。

### Inventory
ansibleの [document](docs.ansible.com) を読み始めて、最初にぶちあたるansible用語。

ansible_ssh_host
  The name of the host to connect to, if different from the alias you wish to give to it.
ansible_ssh_port
  The ssh port number, if not 22
ansible_ssh_user
  The default ssh user name to use.
ansible_ssh_pass
  The ssh password to use (this is insecure, we strongly recommend using --ask-pass or SSH keys)
ansible_sudo_pass
  The sudo password to use (this is insecure, we strongly recommend using --ask-sudo-pass)
ansible_connection
  Connection type of the host. Candidates are local, ssh or paramiko.  The default is paramiko before Ansible 1.2, and 'smart' afterwards which detects whether usage of 'ssh' would be feasible based on whether ControlPersist is supported.
ansible_ssh_private_key_file
  Private key file used by ssh.  Useful if using multiple keys and you don't want to use SSH agent.
ansible_shell_type
  The shell type of the target system. By default commands are formatted using 'sh'-style syntax by default. Setting this to 'csh' or 'fish' will cause commands executed on target systems to follow those shell's syntax instead.
ansible_python_interpreter
  The target host python path. This is useful for systems with more
  than one Python or not located at "/usr/bin/python" such as \*BSD, or where /usr/bin/python
  is not a 2.X series Python.  We do not use the "/usr/bin/env" mechanism as that requires the remote user's
  path to be set right and also assumes the "python" executable is named python, where the executable might
  be named something like "python26".
ansible\_\*\_interpreter
  Works for anything such as ruby or perl and works just like ansible_python_interpreter.
  This replaces shebang of modules which will run on that host.

### Lists
----------

1. list1
   - list1
   - list2
   - list3
   - list4
1. list2
1. list3
1. list4

### Table
----------

左揃え | 中央揃え | 右揃え
:----- | :------: | -----:
1111   | 2222     | 3333 
4444   | 5555     | 6666
7777   | 8888     | 9999  

<!-- more -->  

### Codes
----------
`$ command !!`

    $ ls -l
    $ ls -ltr
    $ sl -l
    $ sl -al

    param1 :
    説明  

### Links
************

[google test link](http://google.com "google")  
[google test link wo title](http://google.com)

### Images
************

![Root4Logo](http://root04.github.com/images/email.png)  
source/images配下にcommitしてpushした画像は、http://branch4.github.com/images/xxx.xxx  
でみれる。


### Emphasize
----------
*EMPHASIZE*  
_EMPHASIZE_  
**EMPHASIZE**  
__EMPHASIZE__  

### Quote
************
> a said a  
>> b said ba  
>>> c said ca  
  
<script type="text/javascript" language="javascript">
  num = Math.floor( Math.random() * 6 );
  document.write( aff[ num ]);
</script>
