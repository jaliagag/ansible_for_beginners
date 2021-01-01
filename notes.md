# Ansible for beginners

## Inventory

inventory file: information about target systems. default inventory /etc/ansible/hosts (if no inventory file is specified).

```ini
server1.company.com
server2.company.com

[mail] # < groups / aliases
server3.company.com

[db]
server4.company.com
server5.company.com

# another way of creating aliases / groups is through the use of the ansible_hosts parameter

web ansible_hosts=server6.compnay.com
mail ansible_hosts=server7.company.com

# other parameters
# ansible_connection --- ssh/winrm/localhost
# ansible_port --- 22/5986
# ansible_user --- root/admin
# ansible_ssh_pass --- pass
```

```yaml
# Sample Inventory File

# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# Database Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!

[web_servers]
web1
web2
web3

[db_servers]
db1

[all_servers:children]
web_servers
db_servers

#### exercise

sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

[db_nodes]
sql_db1
sql_db2

[web_nodes]
web_node1
web_node2
web_node3

[boston_nodes]
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes
```

```yml
#Array
Fruis:
- Orange
- Apple
- Banana
#Dictionary/map
Banana:
  Calories: xx
  Fat: asdf
  Carbs: asfd
  Colors: 		# Dictionary within dictionary
   tropical: blue
   european: yellow

# List of dicitonaries
- color: blue
  model:
    name: corvette
    model: 1995
  transmission: manual
  price: $20,000
- color: white
  model:
    name: t-cross
    model: 2020
  transmission: manual
  price: $12,000
```

- Dictionary: unordered

```yml
Banana:
  Calories: 222
  Fat: 999
  Carbs: 00

# is the same as 

Banana:
  Carbs: 00
  Calories: 222
  Fat: 999
```

- List: ordered

```yml
Fruits:
  - Orange
  - Apple
  - Banana

# Is NOT the same as

Fruits: 
  - Orange
  - Banana
  - Apple
```

```yml
#List of dictionary
employees:
    -
        name: john
        gender: male
        age: 24

# list of dictionary in dictionary
employee:
    name: john
    gender: male
    age: 24
    address:
        city: edison
        state: 'new jersey'
        country: 'united states'
    payslips:
        -
            month: june
            amount: 1400
        -
            month: july
            amount: 2400
        - 
            month: august
            amount: 3400
```

## 5. ansible playbooks

a playbook is a single yaml file containing a set of plays. a play defines a series of activities to be run on a single/group of hosts. a task is an action to be formed on the host (executing a command/script, shutdown/reboot, install a package...).

```yaml
name: Play 1
hosts: localhost
tasks:
  - name: execute command 'date'
    command: date

  - name: install httpd service
    yum:
      name: httpd
      state: present
```

modules: the different actions run by tasks. Examples: command, yum.

ansible: ansible <hosts> -a <command>

```yml
-
  name: Test connectivity to target servers
  hosts: all
  tasks:
    - name: Ping test
      ping:
```

```yml
-
    name: 'Stop the web services on web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Stop the web services on web server nodes'
            command: 'service httpd stop'
-
    name: 'Shutdown databases services'
    hosts: db_nodes
    tasks:
        -
            name: 'Stop the db services on web server nodes'
            command: 'service mysql stop'

-
    name: 'Restart all servers'
    hosts: all_nodes
    tasks:
        -
            name: 'Restart all servers at once'
            command: '/sbin/shutdown -r'

-
    name: 'Start db services on db nodes'
    hosts: db_nodes
    tasks:
        -
            name: 'Start db services on db nodes'
            command: 'service mysql start'
            
-
    name: 'Start the web services on web server nodes'
    hosts: web_nodes
    tasks:
        -
            name: 'Start the web services on web server nodes'
            command: 'service httpd start'
```

## 6. modules

- system
- command

```yml
- name: display contents
  command: cat resolv.conf chdir=/etc
- name: same thing, good old fashion way
  command: cat /etc/resolv.conf
- name: create a folder
  command: mkdir /folder creates=/folder
```

  - script: run a local script on the remote nodes: 

```yml
-
  name: play 1
  hosts: localhost
  tasks:
    - name: run local script on remote server
      script: /local/script.sh -arg1 -arg2
```

  - services: `service: name=httpd state=started` 
  - lineinfile: add information to the file 

```yml
- lininfile:
    path: /etc/resolv.conf
    line: 'nameserver 192.168.0.1'
```

Ensures that there is a line on the file

- files
- database
- cloud
- windows

## 7. variables

we can either declare variables on a separate file or on the same file:

```yml
- name: bla bla bla
  hosts: localhost
  vars:
    dns_server: 123.456.7.89 		# declaration
  tasks:
    - lineinfile:
      path: /etc/resolv.conf
      line: 'nameserver {{ dns_server }}' 	# usage of variable || jinja2 templating
```

- **WRONG** source: {{ dns_server }}
- **CORRECT** source: '{{ dns_server }}'
- **CORRECT** source: something.{{ dns_server }}Something

```yml
-
    name: 'Update nameserver entry into resolv.conf file on localhost'
    hosts: localhost
    vars:
        car_model: "BMW M3"
        country_name: "USA"
        title: "Systems Engineer"
    tasks:
        -
            name: 'Print my car model'
            command: 'echo "My car''s model is {{ car_model }}"'
        -
            name: 'Print my country'
            command: 'echo "I live in the {{ country_name }}"'
        -
            name: 'Print my title'
            command: 'echo "I work as a {{ title }}"'
```

## 8. conditionals

```yml
---
- name: install nginx
  hosts: all
  tasks:
    - name: isntall nginx on debian
      apt:
	name: nginx
	state: present
      when: ansible_os_family == "Debian" and
	    ansible_distribution_version == "16.04"
    - name: install nginx on redhat
      yum:
	name: nginx
	state: present
	# yum: name=nginx state=present
      when: ansible_os_family == "RedHat" or
	    ansible_os_family == "SUSE"
```

```yml
---
- name: install software
  hosts: all
  vars:
    packages:
      - name: nginx
	required: True
      - name: mysql
	required: True
      - name: apache
	required: True
  tasks:
    - name: isntall "{{ item.name }}" on debian
      apt:
	name: "{{ item.name }}"
	state: present
      when: item.required == True
      loop: "{{ packages }}"
```

Each variable item, when expanded, has an **item** value

```yml
- name: check status of a service and email if it's down
  hosts: localhost
  tasks:
    - command: service httpd status
      register: result 			# creates a variable called result
    - mail:
	to: me@dxc.com
	subject: service xxxxx is down
	body: this shit is down
	when: result.stdout.find('down') != -1
```

```yml
-
    name: 'Add name server entry if not already entered'
    hosts: localhost
    tasks:
        -
            shell: 'cat /etc/resolv.conf'
            register: command_output
        -
            shell: 'echo "nameserver 10.0.250.10" >> /etc/resolv.conf'
            when: command_output.stdout.find('10.0.250.10') == -1
```

## 9. loops

```yml
-
  name: create users
  hosts: localhost
  tasks:
    - user: name='{{ item.name }}' state=present uid='{{ item.uid }}'
      loop:
	- name: joe
	  uid: 1010
	- name: george
	  uid: 1011
	- name: pau
	  uid: 1012
	- name: simba
	  uid: 1013
	- name: windows
	  uid: 1014
	- ...
```

```yml
-
    name: 'Print list of fruits'
    hosts: localhost
    vars:
        fruits:
            - Apple
            - Banana
            - Grapes
            - Orange
    tasks:
        -
            command: "echo\" {{ item }}\""
            with_items: "{{fruits}}"

-
    name: 'Install required packages'
    hosts: localhost
    vars:
        packages:
            - httpd
            - binutils
            - glibc
            - ksh
            - libaio
            - libXext
            - gcc
            - make
            - sysstat
            - unixODBC
            - mongodb
            - nodejs
            - grunt
    tasks:
        -
            yum: 'name={{ item }} state=present'
            with_items: "{{ packages }}"
```

## ansible galaxy

- ansible-galaxy init mysql
- ansible-galaxy search <term>
- ansible-galaxy install <role> >> this will go to the default location (specified on the stdout). to use it:

```yml
- 
  name: instlal and configure mysql
  hosts: db-server
  roles:
    - geerlingguy.mysql

##########################
- 
  name: instlal and configure mysql
  hosts: db-server
  roles:
    - role: geerlingguy.mysql
      become: yes
      vars:
	mysql_user_name: db-user
```
 
- which roles are installed: ansible-galaxy list
- where are ansible roles: ansible-config dump | grep ROLE
- isntall on the current directory: ansible-galaxy install geerlingguy.mysql -p ./roles

## commands

- ansible-doc -l
- ansible-playbook <playbook-file-name> -i <inventory-file>







