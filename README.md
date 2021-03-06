Ansible
---
122
一、简介

Ansible is a radically simple configuration-management, application deployment, task-execution, and multinode orchestration engine.

Design Principles

	Have a dead simple setup process and a minimal learning curve
	Be super fast & parallel by default
	Require no server or client daemons; use existing SSHd
	Use a language that is both machine and human friendly
	Focus on security and easy auditability/review/rewriting of content
	Manage remote machines instantly, without bootstrapping
	Allow module development in any dynamic language, not just Python
	Be usable as non-root
	Be the easiest IT automation system to use, ever.



---
二、安装

ansible依赖于Python 2.6或更高的版本、paramiko、PyYAML及Jinja2。


2.1 编译安装

解决依赖关系

      yum -y install python-jinja2 PyYAML python-paramiko python-babel python-crypto
      tar xf ansible-1.5.4.tar.gz
      cd ansible-1.5.4
      python setup.py build
      python setup.py install
      mkdir /etc/ansible
      cp -r examples/* /etc/ansible


2.2 rpm包安装
      yum install ansible

注意：不同版本的ansible的功能差异可能较大。


---
三、简单应用

ansible通过ssh实现配置管理、应用部署、任务执行等功能，因此，需要事先配置ansible端能基于密钥认证的方式联系各被管理节点。



    ansible <host-pattern> [-f forks] [-m module_name] [-a args]
	  -m module：默认为command


    ansible-doc: Show Ansible module documentation
 	  -l, --list            List available modules
  	  -s, --snippet         Show playbook snippet for specified module(s)






---

四、YAML


4.1 YAML介绍

YAML是一个可读性高的用来表达资料序列的格式。YAML参考了其他多种语言，包括：XML、C语言、Python、Perl以及电子邮件格式RFC2822等。Clark Evans在2001年在首次发表了这种语言，另外Ingy döt Net与Oren Ben-Kiki也是这语言的共同设计者。

YAML Ain't Markup Language，即YAML不是XML。不过，在开发的这种语言时，YAML的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。其特性：

	YAML的可读性好
	YAML和脚本语言的交互性好
	YAML使用实现语言的数据类型
	YAML有一个一致的信息模型
	YAML易于实现
	YAML可以基于流来处理
	YAML表达能力强，扩展性好




4.2 YAML语法

YAML的语法和其他高阶语言类似，并且可以简单表达清单、散列表、标量等数据结构。其结构（Structure）通过空格来展示，序列（Sequence）里的项用"-"来代表，Map里的键值对用":"分隔。下面是一个示例。

name: John Smith
age: 41
gender: Male
spouse:
    name: Jane Smith
    age: 37
    gender: Female
children:
    -   name: Jimmy Smith
        age: 17
        gender: Male
    -   name: Jenny Smith
        age 13
        gender: Female

YAML文件扩展名通常为.yaml，如example.yaml。


4.2.1 list

列表的所有元素均使用“-”打头，例如：
    A list of tasty fruits
    
        - Apple
        - Orange
        - Strawberry
        - Mango

4.2.2 dictionary

字典通过key与value进行标识，例如：

---
An employee record

name: Example Developer

job: Developer

skill: Elite

也可以将key:value放置于{}中进行表示，例如：

---

 An employee record
 
{name: Example Developer, job: Developer, skill: Elite}


---
五、Ansible基础元素


5.1 变量


5.1.1 变量命名

变量名仅能由字母、数字和下划线组成，且只能以字母开头。



5.1.2 facts

facts是由正在通信的远程目标主机发回的信息，这些信息被保存在ansible变量中。要获取指定的远程主机所支持的所有facts，可使用如下命令进行：

    ansible hostname -m setup

5.1.3 register

把任务的输出定义为变量，然后用于其他任务，示例如下:

    tasks:
     - shell: /usr/bin/foo
       register: foo_result
       ignore_errors: True

5.1.4 通过命令行传递变量

在运行playbook的时候也可以传递一些变量供playbook使用，示例如下：

	ansible-playbook test.yml --extra-vars "hosts=www user=mageedu"

5.1.5 通过roles传递变量

当给一个主机应用角色的时候可以传递变量，然后在角色内使用这些变量，示例如下：

	- hosts: webservers
	  roles:
	    - common
	    - { role: foo_app_instance, dir: '/web/htdocs/a.com',  port: 8080 }


5.2 Inventory

ansible的主要功用在于批量主机操作，为了便捷地使用其中的部分主机，可以在inventory file中将其分组命名。默认的inventory file为/etc/ansible/hosts。

inventory file可以有多个，且也可以通过Dynamic Inventory来动态生成。

5.2.1 inventory文件格式

inventory文件遵循INI文件风格，中括号中的字符为组名。可以将同一个主机同时归并到多个不同的组中；此外，当如若目标主机使用了非默认的SSH端口，还可以在主机名称之后使用冒号加端口号来标明。

	ntp.magedu.com

	[webservers]
	www1.magedu.com:2222
	www2.magedu.com

	[dbservers]
	db1.magedu.com
	db2.magedu.com
	db3.magedu.com

如果主机名称遵循相似的命名模式，还可以使用列表的方式标识各主机，例如：

    [webservers]
    www[01:50].example.com

    [databases]
    db-[a:f].example.com

5.2.2 主机变量


可以在inventory中定义主机时为其添加主机变量以便于在playbook中使用。例如：

    [webservers]
    www1.magedu.com http_port=80 maxRequestsPerChild=808
    www2.magedu.com http_port=8080 maxRequestsPerChild=909

5.2.3 组变量

组变量是指赋予给指定组内所有主机上的在playboo中可用的变量。例如：

    [webservers]
    www1.magedu.com
    www2.magedu.com

    [webservers:vars]
    ntp_server=ntp.magedu.com
    nfs_server=nfs.magedu.com

5.2.4 组嵌套

inventory中，组还可以包含其它的组，并且也可以向组中的主机指定变量。不过，这些变量只能在ansible-playbook中使用，而ansible不支持。例如：

    [apache]
    httpd1.magedu.com
    httpd2.magedu.com

    [nginx]
    ngx1.magedu.com
    ngx2.magedu.com

    [webservers:children]
    apache
    nginx

    [webservers:vars]
    ntp_server=ntp.magedu.com

5.2.5 inventory参数

ansible基于ssh连接inventory中指定的远程主机时，还可以通过参数指定其交互方式；这些参数如下所示：

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

5.3 条件测试

如果需要根据变量、facts或此前任务的执行结果来做为某task执行与否的前提时要用到条件测试。

5.3.1 when语句

在task后添加when子句即可使用条件测试；when语句支持Jinja2表达式语法。例如：

tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -h now
    when: ansible_os_family == "Debian"

when语句中还可以使用Jinja2的大多“filter”，例如要忽略此前某语句的错误并基于其结果（failed或者sucess）运行后面指定的语句，可使用类似如下形式：

    tasks:
    - command: /bin/false
        register: result
        ignore_errors: True
    - command: /bin/something
        when: result|failed
    - command: /bin/something_else
        when: result|success
    - command: /bin/still/something_else
        when: result|skipped

此外，when语句中还可以使用facts或playbook中定义的变量。

5.4 迭代

当有需要重复性执行的任务时，可以使用迭代机制。其使用格式为将需要迭代的内容定义为item变量引用，并通过with_items语句来指明迭代的元素列表即可。例如：

    - name: add several users
      user: name={{ item }} state=present groups=wheel
      with_items:
        - testuser1
        - testuser2

上面语句的功能等同于下面的语句：

    - name: add user testuser1
      user: name=testuser1 state=present groups=wheel
    - name: add user testuser2
      user: name=testuser2 state=present groups=wheel

事实上，with_items中可以使用元素还可为hashes，例如：

    - name: add several users
      user: name={{ item.name }} state=present groups={{ item.groups }}
      with_items:
        - { name: 'testuser1', groups: 'wheel' }
        - { name: 'testuser2', groups: 'root' }

ansible的循环机制还有更多的高级功能，具体请参见官方文档（http://docs.ansible.com/playbooks_loops.html）。





---
七、ansible playbooks

playbook是由一个或多个“play”组成的列表。play的主要功能在于将事先归并为一组的主机装扮成事先通过ansible中的task定义好的角色。从根本上来讲，所谓task无非是调用ansible的一个module。将多个play组织在一个playbook中，即可以让它们联同起来按事先编排的机制同唱一台大戏。下面是一个简单示例。

	- hosts: webnodes
	  vars:
	    http_port: 80
	    max_clients: 256
	  remote_user: root
	  tasks:
	  - name: ensure apache is at the latest version
	    yum: name=httpd state=latest
	  - name: ensure apache is running
	    service: name=httpd state=started
	  handlers:
	    - name: restart apache
	      service: name=httpd state=restarted

7.1 playbook基础组件

7.1.1 Hosts和Users
	
	playbook中的每一个play的目的都是为了让某个或某些主机以某个指定的用户身份执行任务。hosts用于指定要执行指定任务的主机，其可以是一个或多个由冒号分隔主机组；remote_user则用于指定远程主机上的执行任务的用户。如上面示例中的
		-hosts: webnodes
		 remote_user: root

	不过，remote_user也可用于各task中。也可以通过指定其通过sudo的方式在远程主机上执行任务，其可用于play全局或某任务；此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户。

		- hosts: webnodes
		  remote_user: mageedu
		  tasks:
		    - name: test connection
		      ping:
		      remote_user: mageedu
		      sudo: yes

7.1.2 任务列表和action

	play的主体部分是task list。task list中的各任务按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个任务后再开始第二个。在运行自下而下某playbook时，如果中途发生错误，所有已执行任务都将回滚，因此，在更正playbook后重新执行一次即可。

	task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致。

	每个task都应该有其name，用于playbook的执行结果输出，建议其内容尽可能清晰地描述任务执行步骤。如果未提供name，则action的结果将用于输出。

	定义task的可以使用“action: module options”或“module: options”的格式，推荐使用后者以实现向后兼容。如果action一行的内容过多，也中使用在行首使用几个空白字符进行换行。
		tasks:
		  - name: make sure apache is running
		    service: name=httpd state=running

		在众多模块中，只有command和shell模块仅需要给定一个列表而无需使用“key=value”格式，例如：
			tasks:
			  - name: disable selinux
			    command: /sbin/setenforce 0

		如果命令或脚本的退出码不为零，可以使用如下方式替代：
			tasks:
			  - name: run this command and ignore the result
			    shell: /usr/bin/somecommand || /bin/true		

		或者使用ignore_errors来忽略错误信息：
			tasks:
			  - name: run this command and ignore the result
			    shell: /usr/bin/somecommand
			    ignore_errors: True		

7.1.3 handlers
	
	用于当关注的资源发生变化时采取一定的操作。

	“notify”这个action可用于在每个play的最后被触发，这样可以避免多次有改变发生时每次都执行指定的操作，取而代之，仅在所有的变化发生完成后一次性地执行指定操作。在notify中列出的操作称为handler，也即notify中调用handler中定义的操作。
		- name: template configuration file
		  template: src=template.j2 dest=/etc/foo.conf
		  notify:
		     - restart memcached
		     - restart apache	

	handler是task列表，这些task与前述的task并没有本质上的不同。
		handlers:
		    - name: restart memcached
		      service:  name=memcached state=restarted
		    - name: restart apache
		      service: name=apache state=restarted

	
案例：

	heartbeat.yaml
	- hosts: hbhosts
	  remote_user: root
	  tasks:
	    - name: ensure heartbeat latest version
	      yum: name=heartbeat state=present
	    - name: authkeys configure file
	      copy: src=/root/hb_conf/authkeys dest=/etc/ha.d/authkeys
	    - name: authkeys mode 600
	      file: path=/etc/ha.d/authkeys mode=600
	      notify:
	        - restart heartbeat
	    - name: ha.cf configure file
	      copy: src=/root/hb_conf/ha.cf dest=/etc/ha.d/ha.cf
	      notify: 
	       	- restart heartbeat
	  handlers:
	  	- name: restart heartbeat
	  	  service: name=heartbeat state=restarted


---
八、roles


ansilbe自1.2版本引入的新特性，用于层次性、结构化地组织playbook。roles能够根据层次型结构自动装载变量文件、tasks以及handlers等。要使用roles只需要在playbook中使用include指令即可。简单来讲，roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷地include它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中。

	一个roles的案例如下所示：
		site.yml
		webservers.yml
		dbservers.yml
		roles/
		   common/
		     files/
		     templates/
		     tasks/
		     handlers/
		     vars/
		     meta/
		   webservers/
		     files/
		     templates/
		     tasks/
		     handlers/
		     vars/
		     meta/

	而在playbook中，可以这样使用roles：
	---
	- hosts: webservers
	  roles:
	     - common
	     - webservers

	也可以向roles传递参数，例如：
	---

	- hosts: webservers
	  roles:
	    - common
	    - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
	    - { role: foo_app_instance, dir: '/opt/b',  port: 5001 }

	甚至也可以条件式地使用roles，例如：
	---

	- hosts: webservers
	  roles:
	    - { role: some_role, when: "ansible_os_family == 'RedHat'" }


8.1 创建role的步骤

	(1) 创建以roles命名的目录；
	(2) 在roles目录中分别创建以各角色名称命名的目录，如webservers等；
	(3) 在每个角色命名的目录中分别创建files、handlers、meta、tasks、templates和vars目录；用不到的目录可以创建为空目录，也可以不创建；
	(4) 在playbook文件中，调用各角色；

8.2 role内各目录中可用的文件

	tasks目录：至少应该包含一个名为main.yml的文件，其定义了此角色的任务列表；此文件可以使用include包含其它的位于此目录中的task文件；
	files目录：存放由copy或script等模块调用的文件；
	templates目录：template模块会自动在此目录中寻找Jinja2模板文件；
	handlers目录：此目录中应当包含一个main.yml文件，用于定义此角色用到的各handler；在handler中使用include包含的其它的handler文件也应该位于此目录中；
	vars目录：应当包含一个main.yml文件，用于定义此角色用到的变量；
	meta目录：应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系；ansible 1.3及其以后的版本才支持；
	default目录：为当前角色设定默认变量时使用此目录；应当包含一个main.yml文件；

---
九、Tags

tags用于让用户选择运行playbook中的部分代码。ansible具有幂等性，因此会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地长。此时，如果确信其没有变化，就可以通过tags跳过此些代码片断。


---
十、Jinja2相关

10.1 字面量
	表达式最简单的形式就是字面量。字面量表示诸如字符串和数值的 Python 对象。下面 的字面量是可用的:

	“Hello World”:
	双引号或单引号中间的一切都是字符串。无论何时你需要在模板中使用一个字 符串（比如函数调用、过滤器或只是包含或继承一个模板的参数），它们都是 有用的。

	42 / 42.23:
	直接写下数值就可以创建整数和浮点数。如果有小数点，则为浮点数，否则为 整数。记住在 Python 里， 42 和 42.0 是不一样的。

	[‘list’, ‘of’, ‘objects’]:
	一对中括号括起来的东西是一个列表。列表用于存储和迭代序列化的数据。例如 你可以容易地在 for 循环中用列表和元组创建一个链接的列表:

	<ul>
	{% for href, caption in [('index.html', 'Index'), ('about.html', 'About'),
	                         ('downloads.html', 'Downloads')] %}
	    <li><a href="{{ href }}">{{ caption }}</a></li>
	{% endfor %}
	</ul>

	(‘tuple’, ‘of’, ‘values’):
	元组与列表类似，只是你不能修改元组。如果元组中只有一个项，你需要以逗号 结尾它。元组通常用于表示两个或更多元素的项。更多细节见上面的例子。

	{‘dict’: ‘of’, ‘key’: ‘and’, ‘value’: ‘pairs’}:
	Python 中的字典是一种关联键和值的结构。键必须是唯一的，并且键必须只有一个 值。字典在模板中很少使用，罕用于诸如 xmlattr() 过滤器之类。

	true / false:
	true 永远是 true ，而 false 始终是 false 。

10.2 算术运算

Jinja 允许你用计算值。这在模板中很少用到，但是为了完整性允许其存在。支持下面的 运算符:

	+
		把两个对象加到一起。通常对象是素质，但是如果两者是字符串或列表，你可以用这 种方式来衔接它们。无论如何这不是首选的连接字符串的方式！连接字符串见 ~ 运算符。 {{ 1 + 1 }} 等于 2 。
	-
		用第一个数减去第二个数。 {{ 3 - 2 }} 等于 1 。
	/
		对两个数做除法。返回值会是一个浮点数。 {{ 1 / 2 }} 等于 {{ 0.5 }} 。
	//
		对两个数做除法，返回整数商。 {{ 20 // 7 }} 等于 2 。
	%
		计算整数除法的余数。 {{ 11 % 7 }} 等于 4 。
	*
		用右边的数乘左边的操作数。 {{ 2 * 2 }} 会返回 4 。也可以用于重 复一个字符串多次。 {{ ‘=’ * 80 }} 会打印 80 个等号的横条。
	**
		取左操作数的右操作数次幂。 {{ 2**3 }} 会返回 8 。

10.3 比较操作符

	==
		比较两个对象是否相等。
	!=
		比较两个对象是否不等。
	>
		如果左边大于右边，返回 true 。
	>=
		如果左边大于等于右边，返回 true 。
	<
		如果左边小于右边，返回 true 。
	<=
		如果左边小于等于右边，返回 true 。

10.4 逻辑运算符

对于 if 语句，在 for 过滤或 if 表达式中，它可以用于联合多个表达式:

	and
		如果左操作数和右操作数同为真，返回 true 。
	or
		如果左操作数和右操作数有一个为真，返回 true 。
	not
		对一个表达式取反（见下）。
	(expr)
		表达式组。







