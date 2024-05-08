---
dg-publish: false
---
## ansible是什么

是RedHat提供的一个工具: [ansible.com](http://ansible.com)

**快速入手文档**：[https://getansible.com/demo](https://getansible.com/demo)

Ansilbe是一个**部署一群远程主机**的工具。远程的主机可以是远程虚拟机或物理机， 也可以是本地主机。
Ansilbe通过**SSH协议实现远程节点和管理节点之间的通信**。理论上说，只要管理员通过ssh登录到一台远程主机上能做的操作，Ansible都可以做到。

简单的说 就是我们在 `control node(控制节点/管理节点)` 上安装`ansible` ,通过配置 `control node` 和 `managed nodes(受管节点/远程节点)` 基于key（**无密码的方式**）的**SSH连接**，达到通过`control node` 批量控制 `managed nodes`

## 基本概念

1. **Control node**
	The machine from which you run the Ansible CLI tools (`ansible-playbook` , `ansible`, `ansible-vault` and others). You can use any computer that meets the software requirements as a control node - laptops, shared desktops, and servers can all run Ansible. Multiple control nodes are possible, but Ansible itself does not coordinate across them, see `AAP` for such features.
2. **Managed nodes**
	Also referred to as ‘hosts’, these are the target devices (servers, network appliances or any computer) you aim to manage with Ansible. Ansible is not normally installed on managed nodes, unless you are using `ansible-pull`, but this is rare and not the recommended setup.
3. **Inventory**
	A list of managed nodes provided by one or more ‘inventory sources’. Your inventory can specify information specific to each node, like IP address. It is also used for assigning groups, that both allow for node selection in the Play and bulk variable assignment. To learn more about inventory, see [the Working with Inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#intro-inventory) section. Sometimes an inventory source file is also referred to as a ‘hostfile’.

## 使用步骤
### 1.  安装Ansible
在 `control node`上安装ansible
```
$ # Redhat/CentOS Linux上，Ansible目前放在的epel源中
$ # Fedora默认源中包含ansible，直接安装包既可
$ sudo yum install epel-release 
$ sudo yum install ansible -y 
```
### 2. 配置连接
配置从**管理节点到远程主机**之间基于key（无密码的方式）的**SSH连接**
```
$ # 生成ssh key
$ ssh-keygen
$ # 拷贝ssh key到远程主机，ssh的时候就不需要输入密码了
$ ssh-copy-id remoteuser@remoteserver
$ # ssh的时候不会提示是否保存key
$ ssh-keyscan remote_servers >> ~/.ssh/known_hosts
```
验证SSH配置: 在管理节点执行下面的ssh命令，既**不需要输入密码**，也**不会提醒你存储key**，那就成功啦。

```
$ ssh remoteuser@remoteserver
```

被管理的远程主机不需要安装特殊的包，只**需要python>2.4**，RedHat Linux一般安装方式都是默认安装的。

**ansible中的概念映射到CICD流程中：** 
![ansible中的概念映射到CICD流程中](assets/ansible中的概念映射到CICD流程中.svg)

### 3.  Ansible管理哪些主机（主机清单管理）
ansible通过`Inventory(清单)`知晓自己需要管理哪些主机  这个清单默认存放在 `/etc/ansible/hosts`  目录下
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```
3.  使用Ansilbe命令行管理主机（Ad-hoc command）
4.  使用Ansilbe脚本语言管理主机（脚本语言Playbook）
