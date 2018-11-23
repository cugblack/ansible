# playbook使用技巧

---

## 任务计时

   将此插件[任务计时插件](plugins/task__profile_time.py)放置在playbook/callback_plugins路径下即可。

---

## 获取执行命令的输出 --Register

    在刚开始使用 ansible-playbook 做应用程序部署的时候，因为在部署的过程中有使用到 command 或 shell 模块执行一些自定义的脚本，而且这些脚本都会有输出，用来表示是否执行正常或失败。如果像之前自己写脚本做应用程序部署的，这很好实现。但现在是用 Ansible 做，那么要怎么样做可以获取到 ansible playbook 中 command 模块的输出呢？ Ansible 也提供的解决办法，这时我们就可以通过使用 register 关键字来实现，register 关键字可以存储指定命令的输出结果到一个自定义的变量中，我们通过访问这个自定义变量就可以获取到命令的输出结果。Register 的使用很方便，只需要在 task 声明 register 关键字，并自定义一个变量名就可以。

    - name: echo date 
      command: date 
      register: date_output 
 
    - name: echo date_output 
      command: echo "30"
      when: date_output.stdout.split(' ')[2] == "30"
    
---
## 加速ansible执行速度  ssh pipelining

    SSH pipelining 是一个加速 Ansible 执行速度的简单方法。ssh pipelining 默认是关闭，之所以默认关闭是为了兼容不同的 sudo 配置，主要是 requiretty 选项。如果不使用 sudo，建议开启。打开此选项可以减少 ansible 执行没有传输时 ssh 在被控机器上执行任务的连接数。不过，如果使用 sudo，必须关闭 requiretty 选项。
    pipeline=False  
    修改为：
    pineline=True

---
## Delegate_to（任务委派功能）

    场景介绍：在对一组服务器 server_group1 执行操作过程中，需要在另外一台机器 A 上执行一个操作，比如在 A 服务器上添加一条 hosts 记录，这些操作必须要在一个 playbook 联动完成。也就是是说 A 服务器这个操作与 server_group1 组上的服务器有依赖关系。Ansible 默认只会在定义好的一组服务器上执行相同的操作，这个特性对于执行批处理是非常有用的。但如果在这过程中需要同时对另外 1 台机器执行操作时，就需要用到 Ansible 的任务委派功能（delegate_to）。使用 delegate_to 关键字可以委派任务到指定的机器上运行。
    
    - name: add host record 
      shell: 'echo "192.168.1.100 test.123.com" >> /etc/hosts'
    - name: add host record to center server 
      shell: 'echo "192.168.1.100 test.123.com " >> /etc/hosts'
      delegate_to: 192.168.1.1
  
> 任务委派功能还可以用于以下场景：

      在部署之前将一个主机从一个负载均衡集群中删除。
      当你要对一个主机做改变之前去掉相应 dns 的记录
      当在一个存储设备上创建 iscsi 卷的时候
      当使用外的主机来检测网络出口是否正常的时候    
---

本地操作功能 --local_action

>Ansible 默认只会对控制机器执行操作，但如果在这个过程中需要在 Ansible 本机执行操作呢？细心的读者可能已经想到了，可以使用 delegate_to( 任务委派 ) 功能呀。没错，是可以使用任务委派功能实现。不过除了任务委派之外，还可以使用另外一外功能实现，这就是 local_action 关键字。

    - name: add host record to center server 
      local_action: shell 'echo "192.168.1.100 test.xyz.com " >> /etc/hosts'

>当然您也可以使用 connection:local 方法，如下：

    - name: add host record to center server 
      shell: 'echo "192.168.1.100 test.xyz.com " >> /etc/hosts'
      connection: local
      
---