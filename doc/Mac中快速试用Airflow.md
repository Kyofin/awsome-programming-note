# Mac中快速试用Airflow

## 部署环境

python3.7.4



## 虚拟化环境

创建目录`airflow_learn`。

进入该目录，创建python虚拟环境。

```
python3 -m venv .venv
```

执行成功后，该目录下会有`.venv`文件夹。

激活该虚拟环境。

```
source .venv/bin/activate
```

此时，试用python命令和pip命令都是使用该虚拟环境的。

## 安装airflow

创建airflow目录，用于后续安装airflow。

![image-20211116104659040](http://image-picgo.test.upcdn.net/img/20211116104659.png)

```SHELL
# Airflow needs a home. `~/airflow` is the default, but you can put it
# somewhere else if you prefer (optional)
export AIRFLOW_HOME=/Users/huzekang/study/airflow_learn/airflow

# Install Airflow using the constraints file
AIRFLOW_VERSION=2.2.2
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
# For example: 3.6
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
# For example: https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.6.txt
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# The Standalone command will initialise the database, make a user,
# and start all components for you.
airflow standalone

# Visit localhost:8080 in the browser and use the admin account details
# shown on the terminal to login.
# Enable the example_bash_operator dag in the home page
```

启动成功后，可以看到账号密码在命令行中

![image-20211116104830128](http://image-picgo.test.upcdn.net/img/20211116104830.png)

