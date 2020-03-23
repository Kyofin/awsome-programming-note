## 参考文档

[Example Ansible Playbook](https://gpdb.docs.pivotal.io/6-5/install_guide/ansible-example.html#Untitled1)

[Initializing a Greenplum Database System](https://gpdb.docs.pivotal.io/6-5/install_guide/init_gpdb.html#topic1)

[pgAdmin 4 with Greenplum Database](https://gpdb.docs.pivotal.io/510/admin_guide/access_db/topics/g-pgadmin-for-greenplum-database.html)



## 服务器准备

阿里云2台。

gp001为master节点

gp002为从节点



## 安装

### 下载rpm安装包

https://network.pivotal.io/products/pivotal-gpdb/#/releases/602746/file_groups/2478

greenplum-db-6.5.0-rhel7-x86_64.rpm



### 所有节点配置hosts文件

```
39.101.198.46 gp001
39.101.205.241 gp002
```



### 所有节点root账号都配置ssh免登陆



### 上传安装包到master节点/opt/gp目录



### 在master使用ansible初始化所有节点的安装环境

进入`/opt/gp`目录。

准备文件ansible-playbook.yml。

```yaml
---

- hosts: all
  vars:
    - version: "6.5.0"
    - greenplum_admin_user: "gpadmin"
    - greenplum_admin_password: "gpadmin"
    # - package_path: passed via the command line with: -e package_path=./greenplum-db-6.0.0-rhel7-x86_64.rpm
  remote_user: root
  become: yes
  become_method: sudo
  connection: ssh
  gather_facts: yes
  tasks:
    - name: create greenplum admin user
      user:
        name: "{{ greenplum_admin_user }}"
        password: "{{ greenplum_admin_password | password_hash('sha512', 'DvkPtCtNH+UdbePZfm9muQ9pU') }}"
    - name: copy package to host
      copy:
        src: "{{ package_path }}"
        dest: /tmp
    - name: install package
      yum:
        name: "/tmp/{{ package_path | basename }}"
        state: present
    - name: cleanup package file from host
      file:
        path: "/tmp/{{ package_path | basename }}"
        state: absent
    - name: find install directory
      find:
        paths: /usr/local
        patterns: 'greenplum*'
        file_type: directory
      register: installed_dir
    - name: change install directory ownership
      file:
        path: '{{ item.path }}'
        owner: "{{ greenplum_admin_user }}"
        group: "{{ greenplum_admin_user }}"
        recurse: yes
      with_items: "{{ installed_dir.files }}"
    - name: update pam_limits
      pam_limits:
        domain: "{{ greenplum_admin_user }}"
        limit_type: '-'
        limit_item: "{{ item.key }}"
        value: "{{ item.value }}"
      with_dict:
        nofile: 524288
        nproc: 131072
    - name: find installed greenplum version
      shell: . /usr/local/greenplum-db/greenplum_path.sh && /usr/local/greenplum-db/bin/postgres --gp-version
      register: postgres_gp_version
    - name: fail if the correct greenplum version is not installed
      fail:
        msg: "Expected greenplum version {{ version }}, but found '{{ postgres_gp_version.stdout }}'"
      when: "version is not defined or version not in postgres_gp_version.stdout"
```

准备文件hosts

```
gp001
gp002
```

执行ansible-playbook。

```shell
$ ansible-playbook ansible-playbook.yml -i hosts -e package_path=./greenplum-db-6.0.0-rhel7-x86_64.rpm

```



至此都成功的话，所有节点都有已经建好的可ssh的账号gpadmin，密码是gpadmin。

下面所有操作都是切到gpadmin账号后执行的。



### 配置账号gpadmin在节点间ssh免密登录



### 所有节点都修改环境变量

 vi ~/.bashrc

```shell
source /usr/local/greenplum-db/greenplum_path.sh
```



### 设置gp安装的相关配置

#### 准备gpinitsystem_config文件。

```shell
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config \
     /home/gpadmin/gpconfigs/gpinitsystem_config
```

参考如下：

其实要改的只有`DATA_DIRECTORY`和`MASTER_DIRECTORY`。

前者所有节点都需要先建好，用于存放数据文件。

后者只有master需要提前建好，用于放master的数据。

```properties
ARRAY_NAME="Greenplum Data Platform"
SEG_PREFIX=gpseg
PORT_BASE=6000 
declare -a DATA_DIRECTORY=(/data1/primary /data1/primary /data1/primary /data2/primary /data2/primary /data2/primary)
MASTER_HOSTNAME=mdw 
MASTER_DIRECTORY=/data/master 
MASTER_PORT=5432 
TRUSTED SHELL=ssh
CHECK_POINT_SEGMENTS=8
ENCODING=UNICODE
```

#### 准备hostfile_gpinitsystem文件。

  在/home/gpadmin/gpconfigs目录下自行新建。

```
gp001
gp002
```



### 正式安装

```shell
$ cd ~
$ gpinitsystem -c gpconfigs/gpinitsystem_config -h gpconfigs/hostfile_gpinitsystem
```

等所有配置都自动检测无误后，会提示是否安装，输入y即可。



## 测试使用

在master执行下面命令，看看是否能使用psql连上gp。

```shell
$ psql postgres
```

安装完成后需要配置外网访问，和修改gpadmin的账号。navicat才能连上。

### psql终端中修改默认密码

```
alter user gpadmin encrypted password 'gpadmin';
```



### 开启外网访问。

在master中找这文件pg_hba.conf和文件postgresql.conf。

```shell
[root@gp001 greenplum-db]# find / -name "pg_hba.conf"
/home/gpadmin/data1/primary/gpseg0/pg_hba.conf
/home/gpadmin/data2/primary/gpseg1/pg_hba.conf
/home/gpadmin/data/gpseg-1/pg_hba.conf

[root@gp001 greenplum-db]# find / -name "postgresql.conf"
/home/gpadmin/data1/primary/gpseg0/postgresql.conf
/home/gpadmin/data2/primary/gpseg1/postgresql.conf
/home/gpadmin/data/gpseg-1/postgresql.conf

```

这里只需要修改`/home/gpadmin/data/gpseg-1/pg_hba.conf`和`/home/gpadmin/data/gpseg-1/postgresql.conf`即可。因为这两个都是是master data里的配置。

![](http://image-picgo.test.upcdn.net/img/20200321235255.png)

![](http://image-picgo.test.upcdn.net/img/20200321235240.png)





## 常用命令

### 重新加载配置文件

```undefined
gpstop -u
```



### 其他启停命令

```bash
gpstart #正常启动 
gpstop #正常关闭 
gpstop -M fast #快速关闭 
gpstop –r #重启 
```



### gp自带工具建数据库

```
createdb -h gp001 -p 5432 mydatabase
```



### gp自带工具连接gp服务器

```
psql -p 5432 -h gp001 -d mydatabase -U gpadmin -W 
```



