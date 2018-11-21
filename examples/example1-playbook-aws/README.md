---
# ansible on aws


> Command:

    sudo ansible-playbook -i hosts  playbook.yml -t "test"

    执行此命令后，首先会到roles/test/tasks 下，执行对应的任务，如果需要拷贝文件，会从roles/test/files找到需要拷贝的文件，执行完任务后，会找到roles/test/handlers下的句柄，执行。    

---

## roles角色目录结构：

tasks: 放置需要到目标主机执行的任务脚本
files：文件，存放需要拷贝到目标主机的文件
handlers：放置一些句柄，用于执行完tasks后做一些其他任务
```
roles/test
├── defaults
│   └── main.yml
├── files   
│   └── commitid
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
│   └── main.yml
└── vars
    └── main.yml
```

---

> 适用于目标主机使用ubuntu用户（非root），使用key登录

>> [目标主机名单](hosts)

>>[playbook](playbook.yml)