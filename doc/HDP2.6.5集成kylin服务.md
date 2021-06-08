#hdp-2.6.5集成kylin服务


## 环境
```shell script
HDP-2.6.5
kylin-3.1.1
```
## kylin部署包准备
```shell script
https://mirror.jframeworks.com/apache/kylin/apache-kylin-3.1.1/apache-kylin-3.1.1-bin-hbase1x.tar.gz
```

## hdp-stack目录创建kylin的
```
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
mkdir /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/KYLIN
```

## 上传ambari-kylin-service的压缩包到hdp集群
项目根目录执行编译脚本`build.sh`
```shell script
sh build.sh
```
编译成功后，可以看到`service.tar.gz`。上传该压缩包到服务器上。复制到ambari stack目录后解压。
```shell script
cp service.tar.gz /var/lib/ambari-server/resources/stacks/HDP/2.6/services/KYLIN
tar zxvf service.tar.gz
```
可以看到如下目录。
![](http://image-picgo.test.upcdn.net/img/20210601164450.png)


## Restart Ambari
```
ambari-server restart
```
重启ambari后，打开ambari页面增加服务，可以看到kylin服务。
![](http://image-picgo.test.upcdn.net/img/20210601164640.png)
![](http://image-picgo.test.upcdn.net/img/20210601165835.png)

## ambari安装kylin服务流程剖析
在ambari页面上安装kylin时，ambari-server会对stack目录下的kylin的package目录进行压缩生成archive.zip。
![](http://image-picgo.test.upcdn.net/img/20210607145406.png)
然后ambari-agent会将其缓存起来，解压到ambari-agent的缓存目录/var/lib/ambari-agent/cache/stacks/HDP/2.6/services/KYLIN/，之后在ambari页面上点击安装/启动操作，agent都会加载缓存目录下的脚本。
因此在agent缓存存在时，修改stack目录下的配置是不生效的，要先删缓存，等ambari重建缓存配置才会加载最新的。

## 离线环境下安装故障处理
在ambari中对自定义添加的kylin服务，一直无法安装。每次安装都在执行安装 epel-release（yum的拓展包）时报错。
![](http://image-picgo.test.upcdn.net/img/20210607145147.png)
解决的办法：

手动下载 epel-release 并安装到服务器上。
下载地址 ： https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
下载到服务器后，执行安装。
```rpm -Uvh epel-release*rpm```
在安装完 epel-release 后，就可以在ambari中重新执行安装kylin服务，此时就可以正常安装。

不过在安装完 epel-release 后，如果安装hdp自带的服务，如：`yum flume_2_6_5_0_292 `时由于离线环境会报错。错误如下：

```shell script
stderr:
Traceback (most recent call last):
  File "/var/lib/ambari-agent/cache/common-services/FLUME/1.4.0.2.0/package/scripts/flume_handler.py", line 122, in <module>
    FlumeHandler().execute()
  File "/usr/lib/ambari-agent/lib/resource_management/libraries/script/script.py", line 375, in execute
    method(env)
  File "/var/lib/ambari-agent/cache/common-services/FLUME/1.4.0.2.0/package/scripts/flume_handler.py", line 45, in install
    self.install_packages(env)
  File "/usr/lib/ambari-agent/lib/resource_management/libraries/script/script.py", line 821, in install_packages
    retry_count=agent_stack_retry_count)
  File "/usr/lib/ambari-agent/lib/resource_management/core/base.py", line 166, in __init__
    self.env.run()
  File "/usr/lib/ambari-agent/lib/resource_management/core/environment.py", line 160, in run
    self.run_action(resource, action)
  File "/usr/lib/ambari-agent/lib/resource_management/core/environment.py", line 124, in run_action
    provider_action()
  File "/usr/lib/ambari-agent/lib/resource_management/core/providers/package/__init__.py", line 53, in action_install
    self.install_package(package_name, self.resource.use_repos, self.resource.skip_repos)
  File "/usr/lib/ambari-agent/lib/resource_management/core/providers/package/yumrpm.py", line 264, in install_package
    self.checked_call_with_retries(cmd, sudo=True, logoutput=self.get_logoutput())
  File "/usr/lib/ambari-agent/lib/resource_management/core/providers/package/__init__.py", line 266, in checked_call_with_retries
    return self._call_with_retries(cmd, is_checked=True, **kwargs)
  File "/usr/lib/ambari-agent/lib/resource_management/core/providers/package/__init__.py", line 283, in _call_with_retries
    code, out = func(cmd, **kwargs)
  File "/usr/lib/ambari-agent/lib/resource_management/core/shell.py", line 72, in inner
    result = function(command, **kwargs)
  File "/usr/lib/ambari-agent/lib/resource_management/core/shell.py", line 102, in checked_call
    tries=tries, try_sleep=try_sleep, timeout_kill_strategy=timeout_kill_strategy)
  File "/usr/lib/ambari-agent/lib/resource_management/core/shell.py", line 150, in _call_wrapper
    result = _call(command, **kwargs_copy)
  File "/usr/lib/ambari-agent/lib/resource_management/core/shell.py", line 303, in _call
    raise ExecutionFailed(err_msg, code, out, err)
resource_management.core.exceptions.ExecutionFailed: Execution of '/usr/bin/yum -d 0 -e 0 -y install flume_2_6_5_0_292' returned 1.  One of the configured repositories failed (Unknown),
and yum doesn't have enough cached data to continue. At this point the only
safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=<repoid> ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable <repoid>
        or
            subscription-manager repos --disable=<repoid>

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=<repoid>.skip_if_unavailable=true

Cannot retrieve metalink for repository: epel/x86_64. Please verify its path and try again
stdout:
2021-06-03 15:08:55,823 - Stack Feature Version Info: Cluster Stack=2.6, Command Stack=None, Command Version=None -> 2.6
2021-06-03 15:08:55,828 - Using hadoop conf dir: /usr/hdp/2.6.5.0-292/hadoop/conf
2021-06-03 15:08:55,829 - Group['livy'] {}
2021-06-03 15:08:55,831 - Group['spark'] {}
2021-06-03 15:08:55,831 - Group['ranger'] {}
2021-06-03 15:08:55,831 - Group['hdfs'] {}
2021-06-03 15:08:55,831 - Group['zeppelin'] {}
2021-06-03 15:08:55,831 - Group['hadoop'] {}
2021-06-03 15:08:55,832 - Group['users'] {}
2021-06-03 15:08:55,832 - User['hive'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,833 - User['zookeeper'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,834 - User['ams'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,834 - User['ranger'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'ranger'], 'uid': None}
2021-06-03 15:08:55,835 - User['tez'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'users'], 'uid': None}
2021-06-03 15:08:55,836 - User['zeppelin'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'zeppelin', u'hadoop'], 'uid': None}
2021-06-03 15:08:55,837 - User['livy'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,837 - User['spark'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,838 - User['ambari-qa'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'users'], 'uid': None}
2021-06-03 15:08:55,839 - User['flume'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,839 - Adding user User['flume']
2021-06-03 15:08:55,914 - User['kafka'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,915 - User['hdfs'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hdfs'], 'uid': None}
2021-06-03 15:08:55,916 - User['sqoop'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,916 - User['yarn'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,917 - User['mapred'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,918 - User['hbase'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,919 - User['hcat'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': [u'hadoop'], 'uid': None}
2021-06-03 15:08:55,919 - File['/var/lib/ambari-agent/tmp/changeUid.sh'] {'content': StaticFile('changeToSecureUid.sh'), 'mode': 0555}
2021-06-03 15:08:55,921 - Execute['/var/lib/ambari-agent/tmp/changeUid.sh ambari-qa /tmp/hadoop-ambari-qa,/tmp/hsperfdata_ambari-qa,/home/ambari-qa,/tmp/ambari-qa,/tmp/sqoop-ambari-qa 0'] {'not_if': '(test $(id -u ambari-qa) -gt 1000) || (false)'}
2021-06-03 15:08:55,927 - Skipping Execute['/var/lib/ambari-agent/tmp/changeUid.sh ambari-qa /tmp/hadoop-ambari-qa,/tmp/hsperfdata_ambari-qa,/home/ambari-qa,/tmp/ambari-qa,/tmp/sqoop-ambari-qa 0'] due to not_if
2021-06-03 15:08:55,927 - Directory['/tmp/hbase-hbase'] {'owner': 'hbase', 'create_parents': True, 'mode': 0775, 'cd_access': 'a'}
2021-06-03 15:08:55,929 - File['/var/lib/ambari-agent/tmp/changeUid.sh'] {'content': StaticFile('changeToSecureUid.sh'), 'mode': 0555}
2021-06-03 15:08:55,930 - File['/var/lib/ambari-agent/tmp/changeUid.sh'] {'content': StaticFile('changeToSecureUid.sh'), 'mode': 0555}
2021-06-03 15:08:55,931 - call['/var/lib/ambari-agent/tmp/changeUid.sh hbase'] {}
2021-06-03 15:08:55,939 - call returned (0, '1014')
2021-06-03 15:08:55,940 - Execute['/var/lib/ambari-agent/tmp/changeUid.sh hbase /home/hbase,/tmp/hbase,/usr/bin/hbase,/var/log/hbase,/tmp/hbase-hbase 1014'] {'not_if': '(test $(id -u hbase) -gt 1000) || (false)'}
2021-06-03 15:08:55,945 - Skipping Execute['/var/lib/ambari-agent/tmp/changeUid.sh hbase /home/hbase,/tmp/hbase,/usr/bin/hbase,/var/log/hbase,/tmp/hbase-hbase 1014'] due to not_if
2021-06-03 15:08:55,945 - Group['hdfs'] {}
2021-06-03 15:08:55,946 - User['hdfs'] {'fetch_nonlocal_groups': True, 'groups': ['hdfs', u'hdfs']}
2021-06-03 15:08:55,946 - FS Type:
2021-06-03 15:08:55,946 - Directory['/etc/hadoop'] {'mode': 0755}
2021-06-03 15:08:55,959 - File['/usr/hdp/2.6.5.0-292/hadoop/conf/hadoop-env.sh'] {'content': InlineTemplate(...), 'owner': 'hdfs', 'group': 'hadoop'}
2021-06-03 15:08:55,960 - Directory['/var/lib/ambari-agent/tmp/hadoop_java_io_tmpdir'] {'owner': 'hdfs', 'group': 'hadoop', 'mode': 01777}
2021-06-03 15:08:55,975 - Repository['HDP-2.6-repo-1'] {'append_to_file': False, 'base_url': 'http://10.93.6.247/hdp/HDP/centos7/2.6.5.0-292/', 'action': ['create'], 'components': [u'HDP', 'main'], 'repo_template': '[{{repo_id}}]\nname={{repo_id}}\n{% if mirror_list %}mirrorlist={{mirror_list}}{% else %}baseurl={{base_url}}{% endif %}\n\npath=/\nenabled=1\ngpgcheck=0', 'repo_file_name': 'ambari-hdp-1', 'mirror_list': None}
2021-06-03 15:08:55,983 - File['/etc/yum.repos.d/ambari-hdp-1.repo'] {'content': '[HDP-2.6-repo-1]\nname=HDP-2.6-repo-1\nbaseurl=http://10.93.6.247/hdp/HDP/centos7/2.6.5.0-292/\n\npath=/\nenabled=1\ngpgcheck=0'}
2021-06-03 15:08:55,983 - Writing File['/etc/yum.repos.d/ambari-hdp-1.repo'] because contents don't match
2021-06-03 15:08:55,983 - Repository with url  is not created due to its tags: set([u'GPL'])
2021-06-03 15:08:55,984 - Repository['HDP-UTILS-1.1.0.22-repo-1'] {'append_to_file': True, 'base_url': 'http://10.93.6.247/hdp/HDP-UTILS/centos7/1.1.0.22/', 'action': ['create'], 'components': [u'HDP-UTILS', 'main'], 'repo_template': '[{{repo_id}}]\nname={{repo_id}}\n{% if mirror_list %}mirrorlist={{mirror_list}}{% else %}baseurl={{base_url}}{% endif %}\n\npath=/\nenabled=1\ngpgcheck=0', 'repo_file_name': 'ambari-hdp-1', 'mirror_list': None}
2021-06-03 15:08:55,987 - File['/etc/yum.repos.d/ambari-hdp-1.repo'] {'content': '[HDP-2.6-repo-1]\nname=HDP-2.6-repo-1\nbaseurl=http://10.93.6.247/hdp/HDP/centos7/2.6.5.0-292/\n\npath=/\nenabled=1\ngpgcheck=0\n[HDP-UTILS-1.1.0.22-repo-1]\nname=HDP-UTILS-1.1.0.22-repo-1\nbaseurl=http://10.93.6.247/hdp/HDP-UTILS/centos7/1.1.0.22/\n\npath=/\nenabled=1\ngpgcheck=0'}
2021-06-03 15:08:55,987 - Writing File['/etc/yum.repos.d/ambari-hdp-1.repo'] because contents don't match
2021-06-03 15:08:55,988 - Package['unzip'] {'retry_on_repo_unavailability': False, 'retry_count': 5}
2021-06-03 15:08:56,345 - Skipping installation of existing package unzip
2021-06-03 15:08:56,345 - Package['curl'] {'retry_on_repo_unavailability': False, 'retry_count': 5}
2021-06-03 15:08:56,440 - Skipping installation of existing package curl
2021-06-03 15:08:56,440 - Package['hdp-select'] {'retry_on_repo_unavailability': False, 'retry_count': 5}
2021-06-03 15:08:56,536 - Skipping installation of existing package hdp-select
2021-06-03 15:08:56,540 - The repository with version 2.6.5.0-292 for this command has been marked as resolved. It will be used to report the version of the component which was installed
2021-06-03 15:08:56,780 - Command repositories: HDP-2.6-repo-1, HDP-2.6-GPL-repo-1, HDP-UTILS-1.1.0.22-repo-1
2021-06-03 15:08:56,780 - Applicable repositories: HDP-2.6-repo-1, HDP-2.6-GPL-repo-1, HDP-UTILS-1.1.0.22-repo-1
2021-06-03 15:08:56,781 - Looking for matching packages in the following repositories: HDP-2.6-repo-1, HDP-2.6-GPL-repo-1, HDP-UTILS-1.1.0.22-repo-1
2021-06-03 15:08:59,747 - Adding fallback repositories: HDP-UTILS-1.1.0.22, HDP-2.6.5.0
2021-06-03 15:09:01,831 - Package['flume_2_6_5_0_292'] {'retry_on_repo_unavailability': False, 'retry_count': 5}
2021-06-03 15:09:02,010 - Installing package flume_2_6_5_0_292 ('/usr/bin/yum -d 0 -e 0 -y install flume_2_6_5_0_292')
2021-06-03 15:09:42,203 - Execution of '/usr/bin/yum -d 0 -e 0 -y install flume_2_6_5_0_292' returned 1.  One of the configured repositories failed (Unknown),
and yum doesn't have enough cached data to continue. At this point the only
safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=<repoid> ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable <repoid>
        or
            subscription-manager repos --disable=<repoid>

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=<repoid>.skip_if_unavailable=true

Cannot retrieve metalink for repository: epel/x86_64. Please verify its path and try again
2021-06-03 15:09:42,203 - Failed to install package flume_2_6_5_0_292. Executing '/usr/bin/yum clean metadata'
2021-06-03 15:09:42,360 - Retrying to install package flume_2_6_5_0_292 after 30 seconds
2021-06-03 15:10:52,581 - The repository with version 2.6.5.0-292 for this command has been marked as resolved. It will be used to report the version of the component which was installed

Command failed after 1 tries

```
这时候需要执行如下命令禁止epel源，并且重新制作本地源缓存：
```shell script
yum-config-manager --disable epel
yum clean all
yum makecache

```