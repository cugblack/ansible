#ansible 

---

ansible [host-pattern] [-f forks] [-m modules] [-a args]

ansible 主机/主机组 -i hosts  -m 调用的模块名称 

 ####example:
    ansible hosts-name -i  inventory/hosts -m shell /
      -a "ls -al  /home/ubuntu/cAdvisor-ansible.sh"
 
    ansible-playbook test.yml -i inventory/develop  -f 10
    



默认forks:5 

默认模块：-m command 可省略不写

ansible-doc -s module-name 可查看模块参数

cron :执行定时任务 

    ansible docker -m cron -a "name="cron" minute=/3 hour= day=* mounth=* weekly=* job="/usr/bin/ntpdate 192.168.1.2""

group :新增组 ansible all -m group -a "gid=306 system=yes name=mysql"

yum : state:present/latest 安装 absent 卸载 ansible all -m yum -a "state=present name=corosync"

service:

    ansible all -m service -a "state=start/stop name=httpd enabled=yes"

file