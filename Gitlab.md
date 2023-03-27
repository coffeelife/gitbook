# Gitlab CI/CD

## 概念

### DevOps

Development 和 Operations 的组合，传统的软件组织讲开发和 IT 运营以及质量保障设为分离的部门，Devops 是在敏捷开发模型上进一步发展而来的。满足了产品研发所要求的持续开发、持续测试、持续集成、持续部署。

### CI/CD

即 Continuous Integration（持续集成）和 Continuous Delivery（持续交付）Continuous Deployment（持续部署）。

### pipeline

配置好的 CI/CD 的流程，比如安装依赖包，进行代码 review、构建、编译、发布等一整套的流程。

### GitLab Runner

执行流水线的环境，所有 CI/CD 的任务都是在他内部去进行执行的，GitLab负责代码管理，Runner是流水线，CI/CD 的执行环境，它可以在在 GNU/Linux、K8s、Docker、macOS 和 Windows 上

## 工具

- GitLab-Runner

- Git

- Node

- npm or Yarn

## 安装docker以及gitlab-runner

连接服务器

```
ssh root@10.0.0.xxx
```

安装docker

```
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

开启服务和权限配置

```
sudo groupadd docker  # 创建 docker 用户组
sudo usermod -aG docker $USER  # 将当前用户加入到 docker 的用户组中
```

启动 Docker

```
sudo systemctl enable docker
sudo systemctl start docker
//验证 Docker 环境安装成功
docker info
```

安装GItlab

```
sudo docker run --detach \
  --hostname 115.159.52.223 \
  --publish 443:443 --publish 80:80 --publish 222:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

```
--detach：后台运行，如果去掉，会看到执行的整个过程日志。
--hostname：指定运行的 hostname，可以是域名也可以是 IP。
--publish：端口的映射，可以缩写成 -p 443 用于 HTTPS 协议访问，222 用户 SSH 协议访问，因为 22 端口已经被占用。
--name：容器的名称。
--restart：重启的方式，会自动重启。
--volume：指定本地卷，配置、日志、数据，使用本地卷后，删除容器，不会删除配置、数据。
```

```
no：默认策略，在容器退出时不重启容器
on-failure：在容器非正常退出时（退出状态非 0），才会重启容器
on-failure:3：在容器非正常退出时重启容器，最多重启 3 次
always：在容器退出时总是重启容器
unless-stopped：在容器退出时总是重启容器，但是不考虑在 Docker 守护进程启动时就已经停止了的容器
```

[Install GitLab Runner | GitLab](https://docs.gitlab.com/runner/install/index.html)

```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

安装`gitlab-runner`

```
curl -L --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64"
chmod +x /usr/local/bin/gitlab-runner
useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
gpasswd -a gitlab-runner docker
newgrp docker
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
gitlab-runner start
```

## 2.注册Gitlab-runner

路径 设置 > CI/CD > Runner > Expand 如图

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-29-17-15-35-image.png)

`docker`方式

```
docker run --rm -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register \
  --non-interactive \
  --executor "docker" \
  --docker-image node:16.15.1 \
  --url "http://gitlab.example.com/" \
  --registration-token "TOKEN" \
  --description "coronary_runner_138" \
  --tag-list "docker,coronary" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

进入Runner容器内

```bash
docker exec -it gitlab-runner bash
gitlab-runner register ...
```

普通方式

```
gitlab-runner register \
  --non-interactive \
  --url "http://gitlab.example.com/" \
  --registration-token "TOKEN" \
  --executor "docker" \
  --docker-image node:16.15.1 \
  --description "coronary_runner_138" \
  --tag-list "docker,coronary" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

### 取消注册runner

```
gitlab-runner unregister --url http://example.com/ --token xxxxxxxx
```

`url`修改成自己的`http://gitlab.exmaple.com/`

`registration-token`修改成自己的`TOKEN`

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-24-11-19-43-image.png)

gitlab后台注册的可用runner

除了 url 和 registration-token 还有几个属性：

1. executor：执行器，可选 docker、k8s、shell

2. description：runner 的描述

3. tag-list：runner 的 tag，使用逗号分隔，如果一个项目有多个 Runner，需要根据 tag 来指定使用那个 Runner 来运行任务

4. locked：是否锁定，锁定后，只能适用于被项目，不能被其他项目使用

### 关键词

script 被 runner 执行的 shell script
after_script 在 job 之后执的脚本
allow_failure 是否允许 job 失败
artifacts 在 job 成功后将一个文件列表或目录列表制成制品上传
before_script 在 job 之前执行的脚本
cache 缓存在随后的 job 的一些文件、目录
coverage 代码覆盖率设置
dependencies 依赖某个 job 的制品并下载到当前的 job 中
environment job 依赖的环境变量
except 限制那些情况 job 不会被触发
extends job 的继承的配置项
image 依赖的 Docker 镜像
include 运行 job 包含额外的 yml 文件
interruptible 定义当作业因较新的运行而变得冗余时是否可以取消。
only 限制 job 何时可以被触发
pages 上传 job 的结果到 极狐GitLab Pages
parallel 一个 job 应该并行运行多少个实例。
release 通知 runner 生成一个 release
resource_group 限制 job 的并发性
retry 是否重试 重试几次
rules 与 only/except 类似，限制 job 的调起
services 使用 Docker 镜像服务
stage 阶段
tags 选择 runner 的 tag
timeout 超时
trigger 定义下游 pipelines 触发器
variables 定义 job 级别的变量
when 何时触发任务

`only` 和 `except` 中可以使用特殊的关键字：

- 当一个作业没有定义 `only` 规则时，其默认为 `only: ['branches', 'tags']` 。
- 如果一个作业没有定义 `except` 规则时，则默认 `except` 规则为空。

| 关键字            | 描述释义                                                         |
| -------------- | ------------------------------------------------------------ |
| branches       | 当一个分支被push上来                                                 |
| tags           | 当一个打了tag标记的Release被提交时                                       |
| api            | 当一个pipline被第二个piplines api所触发调起(不是触发器API)                    |
| external       | 当使用了GitLab以外的外部CI服务，如Jenkins                                 |
| pipelines      | 针对多项目触发器而言，当使用CI_JOB_TOKEN， 并使用gitlab所提供的api创建多个pipelines的时候 |
| pushes         | 当pipeline被用户的git push操作所触发的时候                                |
| schedules      | 针对预定好的pipline计划而言（每日构建一类）                                    |
| triggers       | 用触发器token创建piplines的时候                                       |
| web            | 在GitLab WEB页面上Pipelines标签页下，按下run pipline的时候                 |
| merge_requests | 当合并请求创建或更新的时候                                                |
| chats          | 当使用GitLab ChatOps 创建作业的时候                                    |

### only 和 except高级用法

- `only` 和 `except` 支持高级策略，`refs` 、 `variables` 、 `changes` 、 `kubernetes` 四个关键字可以使用。
- 如果同时使用多个关键字，中间的逻辑是 `逻辑与AND` 。

```
only:
    refs:
      - master
      - schedules


 only:
    refs:
      - branches
    variables:
      - $RELEASE == "staging"
      - $STAGING

only:
    changes:
      - package.json


//结合使用
only:
    refs:
      - master
    changes:
      - package.json

//当md文件发生变化时，会忽略CI作业
except:
    changes:
      - "*.md"

//一旦合并请求中修改了js文件或者修改了utils目录下的文件，都会触发Docker构建
only:
    refs:
      - merge_requests
    changes:
      - /^*.js$/
      - utils/**/*
```

## gitlab-ci.yml文件

[阮一峰 YAML 教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)

#### branch分支触发构建流程

```
# 定义node镜像
image: node:16.15.1

before_script:
  - echo "每个job之前都会执行"
  - node -v
  - npm -v
  - git --version
  - npm install -g cnpm --registry=https://registry.npm.taobao.org
  # - yarn -v

variables:
  GIT_SUBMODULE_STRATEGY: recursive

# 阶段
stages:
  - install
  - init
  - build
  - deploy

cache:
  paths:
    - node_modules/
    - build/

# 安装依赖
install:
  stage: install
  # 此处的tags必须填入之前注册时自定的tag
  tags:
    - docker
  # 规定仅在package.json提交时才触发此阶段
  only:
    changes:
      - package.json
  # only:
  #   - dev-mguo
  #   - /^dev-*$/
  # 执行脚本
  script:
    - cnpm install

init:
  stage: init
  tags:
    - docker
  only:
    - dev-mguo
  # 执行脚本
  script:
    - git submodule init && git submodule update && cd framework && git checkout dev && git pull && cd ../

# 打包项目
build:
  stage: build
  only:
    - dev-mguo
  tags:
    - docker
  script:
    - npm run build
  # 将此阶段产物传递至下一阶段
  artifacts:
    paths:
      - build/

# 部署项目
deploy:
  stage: deploy
  only:
    - dev-mguo
  tags:
    - docker
  script:
    # 直接编译上传
    npm run deploy
  # when: manual


# tags 触发
```

#### tags触发流程

```
//新建tag
git tag -a v1.0 -m "Release v1.0"
//查看tag
git tag
//显示tag附注信息
git show v1.0
//提交本地tag到远程仓库
git push origin v1.0
//提交本地所有tag到远程仓库
git push origin --tags
//删除本地tag
git tag -d v1.0
//删除远程tag
git push origin :refs/tags/v1.0
//获取远程版本
git fetch origin tag v1.0



//git-ci.yml配置
only:
  - tags
except:
  - branches
```

#### 流水线触发

```
only:
  - triggers

except:
  - triggers
```

### 镜像使用

`image: node:16.15.1`表示使用nodejs 16.15.1版本最为基础镜像，`pipelines`触发后会pull Nodejs的镜像，`npm install`、`npm run build`、`npm run deploy`的使用需要node的镜像环境。

### 编写job

`before_script`代表是每个script脚本执行前的脚本，

`stage`定义的是阶段的执行顺序

```
# 阶段
stages:
  - install
  - init
  - build
  - deploy
```

执行顺序就是 install > init > build > deploy

```
# 打包项目
build:
  stage: build
  only:
    - dev-mguo
  tags:
    - docker
  script:
    - npm run build
  # 将此阶段产物传递至下一阶段
  artifacts:
    paths:
      - build/
  # when: manual
```

`build`代表的是job的名字

`stage`代表的是执行的阶段

`only`代表的是在什么分支来执行

`tags`代表的是执行使用的`runner`标签

`script`代表的是执行的脚本

`artifacts`代表执行的阶段缓存文件

`cache`代表的是执行过程中缓存的文件

`when: manual`代表的是手动触发

- `on_success` ：只有前面的阶段的所有作业都成功时才执行，这是默认值。
- `on_failure` ：当前面阶段的作业至少有一个失败时才执行。
- `always` : 无论前面的作业是否成功，一直执行本作业。
- `manual` ：手动执行作业，作业不会自动执行，需要人工手动点击启动作业。
- `delayed` : 延迟执行作业，配合 `start_in` 关键字一起作用， `start_in` 设置的值必须小于或等于1小时，`start_in` 设置的值示例： `10 seconds` 、 `30 minutes` 、 `1 hour` ，前面的作业结束时计时器马上开始计时。

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-29-18-04-48-image.png)

```
其中 cache 指的是缓存, 常用于依赖安装中, 如几个jobs都需要安装相同的依赖, 可以使用依赖, 此时可以加快依赖的安装进度;
对于artifacts则是将某个工件上传到GitLab提供下载或后续操作使用, 由于每个job启动时, 都会自动删除.gitignore中指定的文件, 因此对于依赖安装目录, 即可以使用cache, 也可以使用artifacts.

两个主要有以下几个区别:

cache不一定命中，artifacts肯定命中, 能否使用cache取决当当前机器是否生成过cache, artifacts则每次都会从GitLab下载
重新安装时因为使用的是缓存, 所以很有可能不是最新的
特别是开发环境, 如果每次都希望使用最新的更新, 应当删除cache, 使用artifacts, 这样可以保证确定的更新
4.artifacts中定义的部分, 会自动生成, 并可以传到下面的job中解压使用, 避免了重复依赖安装等工作
如果使用Docker运行Gitlab-Runner, cache会生成一些临时容器, 不容易清理
artifacts可以设置自动过期时间, 过期自动删除，cache不会自动清理
artifacts会先传到GitLab服务器, 然后需要时再重新下载, 所以这部分也可以在GitLab下载和浏览
```

## ![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-29-18-05-41-image.png)

        到这我们已经做好了项目的 CI持续集成，CD 持续部署还没有做，最常用的是将 build 后的静态文件放到服务器上或者做成Docker 镜像，可以运行在任何一个服务器上。

### 隐秘变量使用

在 .gitlab-ci.yml 中使用一些密码、私钥，直接使用时非常不安全的。极狐GitLab 提供了一种在 CI/CD 中安全使用隐秘变量的方式，添加变量后在 .gitlab-ci.yml 中使用 `$+变量名` 的方式来使用，如镜像名、运行的容器名、用户名、密码等。如下图添加变量

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-29-18-09-55-image.png)

## 部署方式

### docker方式

```
# 定义构建镜像
build-image:
  image: $DOCKER_IMG # 依赖的基础镜像
  stage: build
  script:
    - docker login -u $HARBOR_USERNAME -p $HARBOR_PWD $HARBOR_URL # 登录镜像仓库
    - docker build -t $APP_IMAGE_NAME . # 构建镜像
    - docker push $APP_IMAGE_NAME # 推送镜像
    - docker rmi $APP_IMAGE_NAME # 删除本地镜像
  only:
    - master

deploy:
  image: $DOCKER_IMG
  stage: deploy
  script:
    # 如果有容器名为$APP_CONTAINER_NAME 的容器在运行则强行上删除
    - if [ $(docker ps -aq --filter name=$APP_CONTAINER_NAME) ]; then docker rm -f $APP_CONTAINER_NAME; docker image rm -f $APP_IMAGE_NAME;fi

    # 登录镜像仓库
    - docker login -u $HARBOR_USERNAME -p $HARBOR_PWD $HARBOR_URL

    # 使用 8001 端口,镜像名为$APP_CONTAINER_NAME 的后台方式 运行一个镜像
    - docker run -d -p 8001:80 --name $APP_CONTAINER_NAME $APP_IMAGE_NAME
    - echo 'deploy product websit success'
  only:
    - master
  when: manual # 手动执行,需要点击
```

### ssh方式

Runner 跑在机器 A，应用部署在机器 B，那么 Runner 对前端项目 build 之后，怎样将 build 后的静态文件，上传到机器 B？在机器 A 和机器 B 之间创建一个免密登录，在机器 A build 项目之后，静默地调用指令将文件上传到机器 B，从而实现部署。

#### 创建 SSH key并上传

```
ssh-keygen -t rsa -b 2048 -C "email@example.com"
scp -r id_rsa.pub root@1.2.3.4:/root/.ssh/authorized_keys
```

将公钥 id_rsa.pub 上传到服务器 authorized_keys 文件中，第一次会保存 known hosts，并且要求输入密码，上传成功后，再次执行上面的命令就不需要输入密码了，免密构建成功，就要配置 CI/CD 了

创建一个 SSH_PRIVATE_KEY 的变量，将秘钥复制进去。接着修改.gitlab-ci.yml 文件

```
deploy-server:
  stage: deploy
  before_script:
    # 如果没有安装 `ssh-agent`,就安装，基于 RPM 的镜像可以将 apt-get 替换为 yum
    - 'command -v ssh-agent >/dev/null || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s) # 运行 ssh-agent
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -

    # 创建对应的目录并给相应的权限
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh

    - ssh-keyscan ipaddress >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
  script:
    - npm run build
    - scp -r dist/* root@ipaddress:/usr/local/www/hello-vue
  only:
    - master
```

代码解释：

任务是在 Docker 容器中运行的，所以要使用 ssh 必须要确保已经安装必要的软件。
如果没有安装 ssh-agent，就安装，基于 RPM 的镜像可以将 apt-get 替换为 yum。
注意目录 /usr/local/www/hello-vue 需要配置了 Nginx 映射才能够访问。

## trigger触发

| Token           | 描述        |
| --------------- | --------- |
| xxxxxxxxxxxxxxx | cicd-test |

### use cURL

```
curl -X POST \
     -F token=TOKEN \
     -F ref=REF_NAME \
     http://gitlab.example.com/api/v4/projects/projectId/trigger/pipeline


curl -X POST \
     -F token=TOKEN \
     -F ref=dev-mguo \
     http://gitlab.example.com/api/v4/projects/projectId/trigger/pipeline
```

### use .gitlab-ci.yml

```
script:
  - "curl -X POST -F token=TOKEN -F ref=REF_NAME http://gitlab.example.com/api/v4/projects/370/trigger/pipeline"

//添加变量
curl -X POST \
     -F token=TOKEN \
     -F "ref=REF_NAME" \
     -F "variables[RUN_NIGHTLY_BUILD]=true" \
     http://gitlab.example.com/api/v4/projects/370/trigger/pipeline
```

### Use webhook

```
http://gitlab.example.com/api/v4/projects/370/ref/REF_NAME/trigger/pipeline?token=TOKEN


http://gitlab.example.com/api/v4/projects/370/ref/dev-mguo/trigger/pipeline?token=TOKEN

//添加变量
http://gitlab.example.com/api/v4/projects/370/ref/REF_NAME/trigger/pipeline?token=TOKEN&variables[RUN_NIGHTLY_BUILD]=true
```

#### 子模块策略Git submodule strategy

- `GIT_SUBMODULE_STRATEGY` 类似于 `GIT_STRATEGY` ，当你的项目需要包含别的项目代码时，可以将别的项目作为你的项目的子模块，这个时候就可以使用 `GIT_SUBMODULE_STRATEGY` 。
- `GIT_SUBMODULE_STRATEGY` 默认取值 `none` ，即拉取代码时，子模块不会被引入。
- `GIT_SUBMODULE_STRATEGY` 可取值 `normal` ，意味着在只有顶级子模块会被引入。
- `GIT_SUBMODULE_STRATEGY` 可取值 `recursive` ，递归的意思，意味着所有级子模块会被引入。

子模块需要配置在 `.gitmodules` 配置文件中，下面是两个示例：

场景：

- 你的项目地址： `https://gitlab.com/secret-group/my-project` ，你可以使用 `git clone git@gitlab.com:secret-group/my-project.git` 检出代码。
- 你的项目依赖 `https://gitlab.com/group/project` ，你可以将这个模块作为项目的子模块。

子模块与本项目在同一个服务上，可以使用相对引用：

| 1<br>2<br>3 | [submodule "project"]<br> path = project<br> url = ../../group/project.git |
| ----------- | -------------------------------------------------------------------------- |

子模块与本项目不在同一个服务上，使用相对绝对URL：

| 1<br>2<br>3 | [submodule "project-x"]<br> path = project-x<br> url = https://gitserver.com/group/project-x.git |
| ----------- | ------------------------------------------------------------------------------------------------ |

详细可参考 [Using Git submodules with GitLab CI](https://docs.gitlab.com/ce/ci/git_submodules.html)

## Release功能

参考链接：[Releases API | GitLab](https://docs.gitlab.com/ee/api/releases/index.html)

[[gitlab] release功能](https://blog.csdn.net/a112626290/article/details/105404318)

```
//Create a release  POST /projects/:id/releases
curl --header 'Content-Type: application/json' --header "PRIVATE-TOKEN: <your_access_token>" \
     --data '{ "name": "New release", "tag_name": "v0.3", "description": "Super nice release", "milestones": ["v1.0", "v1.0-rc"], "assets": { "links": [{ "name": "hoge", "url": "https://google.com", "filepath": "/binaries/linux-amd64", "link_type":"other" }] } }' \
     --request POST "https://gitlab.example.com/api/v4/projects/24/releases"

//简单命令
curl --header 'Content-Type: application/json' --header "PRIVATE-TOKEN: XXXXXXXXXXXXX" --data '{ "name": "'release名称'", "tag_name": "'标签名'", "ref":"'标签名'" ,"description": "'描述信息'" }' --request POST http://ip地址/api/v4/projects/工程id/releases

//PRIVATE-TOKEN
//projectId
curl --header 'Content-Type: application/json' --header "PRIVATE-TOKEN: XXXXXXXXXXX" --data '{ "name": "'release名称'", "tag_name": "'标签名'", "ref":"'标签名'" ,"description": "'描述信息'" }' --request POST http://gitlab.example.com/api/v4/projects/工程id/releases
```

### PRIVATE-TOKEN

获取方式 ,参考`TOKEN`

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-30-14-07-20-image.png)

### projectId

每个gitlab中的项目都有一个唯一识别号，我们称之为project id

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-30-14-08-21-image.png)

### Tag

中文翻译为标签。某些程度上面，tag和release你都可以认为是快照的概念。
release基于tag，因此需要先打标签：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409102623650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ExMTI2MjYyOTA=,size_16,color_FFFFFF,t_70)

如果没有其他操作，生成release就是把某个版本牵出来，里面都是该版本的源码，将会生成4种文件:  `zip`, `tar.gz`, `tar.bz2` 和 `tar`

| Attribute                | Type            | Required                         | Description                                                                                                                                                               |
| ------------------------ | --------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                     | integer/string  | yes                              | The ID or [URL-encoded path of the project](https://docs.gitlab.com/ee/api/index.html#namespaced-path-encoding).                                                          |
| `name`                   | string          | no                               | The release name.                                                                                                                                                         |
| `tag_name`               | string          | yes                              | The tag where the release is created from.                                                                                                                                |
| `tag_message`            | string          | no                               | Message to use if creating a new annotated tag.                                                                                                                           |
| `description`            | string          | no                               | The description of the release. You can use [Markdown](https://docs.gitlab.com/ee/user/markdown.html).                                                                    |
| `ref`                    | string          | yes, if `tag_name` doesn’t exist | If a tag specified in `tag_name` doesn’t exist, the release is created from `ref` and tagged with `tag_name`. It can be a commit SHA, another tag name, or a branch name. |
| `milestones`             | array of string | no                               | The title of each milestone the release is associated with. [GitLab Premium](https://about.gitlab.com/pricing/) customers can specify group milestones.                   |
| `assets:links`           | array of hash   | no                               | An array of assets links.                                                                                                                                                 |
| `assets:links:name`      | string          | required by: `assets:links`      | The name of the link. Link names must be unique within the release.                                                                                                       |
| `assets:links:url`       | string          | required by: `assets:links`      | The URL of the link. Link URLs must be unique within the release.                                                                                                         |
| `assets:links:filepath`  | string          | no                               | Optional path for a [Direct Asset link](https://docs.gitlab.com/ee/user/project/releases/index.html#permanent-links-to-release-assets).                                   |
| `assets:links:link_type` | string          | no                               | The type of the link: `other`, `runbook`, `image`, `package`. Defaults to `other`.                                                                                        |
| `released_at`            | datetime        | no                               | The date when the release is/was ready. Defaults to the current time. Expected in ISO 8601 format (`2019-03-15T08:00:00Z`).                                               |

## 问题汇总

1.python

node镜像版本问题 image: node:15.16.1

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-24-16-01-39-image.png)

2.npm

需要安装node镜像 image: node:15.16.1

![](/Users/gm/Library/Application%20Support/marktext/images/2022-06-24-16-03-18-image.png)

[gitlab-runner的无权限问题 - 芭菲雨 - 博客园](https://www.cnblogs.com/bafeiyu/p/12538861.html)

```
ps aux|grep gitlab-runner  #查看当前runner用户

sudo gitlab-runner uninstall  #删除gitlab-runner

gitlab-runner install --working-directory /home/gitlab-runner --user root   #安装并设置--user(例如我想设置为root)

sudo service gitlab-runner restart  #重启gitlab-runner

ps aux|grep gitlab-runner #再次执行会发现--user的用户名已经更换成root了
```

3. fatal: could not read Username for 'https://gitserver.com/ ': No such device or address

   修改成相对路径解决问题（不知道为啥绝对路径会出问题）

   解决方案：[Gitlab CI 拉取 submodules](https://blog.csdn.net/wangjiang_qianmo/article/details/122691224)

4. 使用yarn报错

   尝试删除 `yarn.lock`
   尽量使用同一个源

   `yarn config set strict-ssl false`

   `yarn config set registry https://registry.npmjs.org`

   链接：

5.docker容器时间戳使用与本地不同问题

```
docker cp /etc/localtime <container_id>:/etc/
```

## 前端项目 CI/CD 最佳业务配置思路

一个好的 CI/CD 流程，应该包含以下几点

- 权限可控
- 缓存使用得当
- 分支作用明确
- 根据 tag 创建稳定的 release
- 极狐GitLab Runner 可伸缩
- 流水线异常多渠道通知
- 支持多个微服务一键同步部署
- 隐秘信息使用声明变量
- 使用镜像时要配置 .dockerignore

权限可控是指发布正式环境，或者预发布环境只有指定的人才能发布，合并代码必须要经过 CI 验证后才能合并。

缓存使用得当，可以加快流水线的构建，也使的发布流程更加顺畅，缓存可以使用第三方的，也可以用本地，另外需要指定缓存的删除策略，不然一直增加缓存机器会爆掉的。缓存可以使用单个文件的 hash 值来缓存，也可以直接缓存一个目录，分支作用明确之后才能规划分支的发布流程，有些分支只能发布开发环境，有些分支只能发布 test 环境，而且只有一个分支才能发布正式环境。

一个版本发布后需要给代码的打一个 tag，标注增加了那些功能，修复了那些问题，做了那些优化，然后发布一个 release。这部分也可以使用 CI/CD 自动构建，流水线异常多渠道通知是指，发布或构建过程中出错了，要能及时让人及时感知，快速解决问题，一个流水线构建有的需要 10 分钟，不可能每次都要一直等在哪里，所有可以接入钉钉通知、邮件通知，或接入第三方应用通知。

微服务的流行使 CI/CD 迎来了更大的挑战，目前 极狐GitLab 已经支持了多流水线构建，方便一次版本同时发布多个应用，避免有的应用丢失。使用.dockerignore 将不需要的文件在构建镜像时忽略，以加快构建速度。

## 预定义变量

There are also [Kubernetes-specific deployment variables](https://docs.gitlab.com/ee/user/project/clusters/index.html#deployment-variables).

| Variable                                 | GitLab | Runner | Description                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------------------------- | ------ | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CHAT_CHANNEL`                           | 10.6   | all    | The Source chat channel that triggered the [ChatOps](https://docs.gitlab.com/ee/ci/chatops/index.html) command.                                                                                                                                                                                                                                             |
| `CHAT_INPUT`                             | 10.6   | all    | The additional arguments passed with the [ChatOps](https://docs.gitlab.com/ee/ci/chatops/index.html) command.                                                                                                                                                                                                                                               |
| `CI`                                     | all    | 0.4    | Available for all jobs executed in CI/CD. `true` when available.                                                                                                                                                                                                                                                                                            |
| `CI_API_V4_URL`                          | 11.7   | all    | The GitLab API v4 root URL.                                                                                                                                                                                                                                                                                                                                 |
| `CI_BUILDS_DIR`                          | all    | 11.10  | The top-level directory where builds are executed.                                                                                                                                                                                                                                                                                                          |
| `CI_COMMIT_BEFORE_SHA`                   | 11.2   | all    | The previous latest commit present on a branch. Is always `0000000000000000000000000000000000000000` in pipelines for merge requests.                                                                                                                                                                                                                       |
| `CI_COMMIT_BRANCH`                       | 12.6   | 0.5    | The commit branch name. Available in branch pipelines, including pipelines for the default branch. Not available in merge request pipelines or tag pipelines.                                                                                                                                                                                               |
| `CI_COMMIT_DESCRIPTION`                  | 10.8   | all    | The description of the commit. If the title is shorter than 100 characters, the message without the first line.                                                                                                                                                                                                                                             |
| `CI_COMMIT_MESSAGE`                      | 10.8   | all    | 提交message。                                                                                                                                                                                                                                                                                                                                                  |
| `CI_COMMIT_REF_NAME`                     | 9.0    | all    | branch or tag名称                                                                                                                                                                                                                                                                                                                                             |
| `CI_COMMIT_REF_PROTECTED`                | 11.11  | all    | `true` if the job is running for a protected reference.                                                                                                                                                                                                                                                                                                     |
| `CI_COMMIT_REF_SLUG`                     | 9.0    | all    | `CI_COMMIT_REF_NAME` in lowercase, shortened to 63 bytes, and with everything except `0-9` and `a-z` replaced with `-`. No leading / trailing `-`. Use in URLs, host names and domain names.                                                                                                                                                                |
| `CI_COMMIT_SHA`                          | 9.0    | all    | 提交version                                                                                                                                                                                                                                                                                                                                                   |
| `CI_COMMIT_SHORT_SHA`                    | 11.7   | all    | `CI_COMMIT_SHA` 前8字符                                                                                                                                                                                                                                                                                                                                        |
| `CI_COMMIT_TAG`                          | 9.0    | 0.5    | The commit tag name. Available only in pipelines for tags.                                                                                                                                                                                                                                                                                                  |
| `CI_COMMIT_TIMESTAMP`                    | 13.4   | all    | The timestamp of the commit in the ISO 8601 format.                                                                                                                                                                                                                                                                                                         |
| `CI_COMMIT_TITLE`                        | 10.8   | all    | The title of the commit. The full first line of the message.                                                                                                                                                                                                                                                                                                |
| `CI_CONCURRENT_ID`                       | all    | 11.10  | The unique ID of build execution in a single executor.                                                                                                                                                                                                                                                                                                      |
| `CI_CONCURRENT_PROJECT_ID`               | all    | 11.10  | The unique ID of build execution in a single executor and project.                                                                                                                                                                                                                                                                                          |
| `CI_CONFIG_PATH`                         | 9.4    | 0.5    | The path to the CI/CD configuration file. Defaults to `.gitlab-ci.yml`.                                                                                                                                                                                                                                                                                     |
| `CI_DEBUG_TRACE`                         | all    | 1.7    | `true` if [debug logging (tracing)](https://docs.gitlab.com/ee/ci/variables/README.html#debug-logging) is enabled.                                                                                                                                                                                                                                          |
| `CI_DEFAULT_BRANCH`                      | 12.4   | all    | The name of the project’s default branch.                                                                                                                                                                                                                                                                                                                   |
| `CI_DEPENDENCY_PROXY_GROUP_IMAGE_PREFIX` | 13.7   | all    | The image prefix for pulling images through the Dependency Proxy.                                                                                                                                                                                                                                                                                           |
| `CI_DEPENDENCY_PROXY_PASSWORD`           | 13.7   | all    | The password to pull images through the Dependency Proxy.                                                                                                                                                                                                                                                                                                   |
| `CI_DEPENDENCY_PROXY_SERVER`             | 13.7   | all    | The server for logging in to the Dependency Proxy. This is equivalent to `$CI_SERVER_HOST:$CI_SERVER_PORT`.                                                                                                                                                                                                                                                 |
| `CI_DEPENDENCY_PROXY_USER`               | 13.7   | all    | The username to pull images through the Dependency Proxy.                                                                                                                                                                                                                                                                                                   |
| `CI_DEPLOY_FREEZE`                       | 13.2   | all    | Only available if the pipeline runs during a [deploy freeze window](https://docs.gitlab.com/ee/user/project/releases/index.html#prevent-unintentional-releases-by-setting-a-deploy-freeze). `true` when available.                                                                                                                                          |
| `CI_DEPLOY_PASSWORD`                     | 10.8   | all    | The authentication password of the [GitLab Deploy Token](https://docs.gitlab.com/ee/user/project/deploy_tokens/index.html#gitlab-deploy-token), if the project has one.                                                                                                                                                                                     |
| `CI_DEPLOY_USER`                         | 10.8   | all    | The authentication username of the [GitLab Deploy Token](https://docs.gitlab.com/ee/user/project/deploy_tokens/index.html#gitlab-deploy-token), if the project has one.                                                                                                                                                                                     |
| `CI_DISPOSABLE_ENVIRONMENT`              | all    | 10.1   | Only available if the job is executed in a disposable environment (something that is created only for this job and disposed of/destroyed after the execution - all executors except `shell` and `ssh`). `true` when available.                                                                                                                              |
| `CI_ENVIRONMENT_NAME`                    | 8.15   | all    | The name of the environment for this job. Available if [`environment:name`](https://docs.gitlab.com/ee/ci/yaml/README.html#environmentname) is set.                                                                                                                                                                                                         |
| `CI_ENVIRONMENT_SLUG`                    | 8.15   | all    | The simplified version of the environment name, suitable for inclusion in DNS, URLs, Kubernetes labels, and so on. Available if [`environment:name`](https://docs.gitlab.com/ee/ci/yaml/README.html#environmentname) is set.                                                                                                                                |
| `CI_ENVIRONMENT_URL`                     | 9.3    | all    | The URL of the environment for this job. Available if [`environment:url`](https://docs.gitlab.com/ee/ci/yaml/README.html#environmenturl) is set.                                                                                                                                                                                                            |
| `CI_HAS_OPEN_REQUIREMENTS`               | 13.1   | all    | Only available if the pipeline’s project has an open [requirement](https://docs.gitlab.com/ee/user/project/requirements/index.html). `true` when available.                                                                                                                                                                                                 |
| `CI_JOB_ID`                              | 9.0    | all    | job id                                                                                                                                                                                                                                                                                                                                                      |
| `CI_JOB_IMAGE`                           | 12.9   | 12.9   | 运行job的容器的image                                                                                                                                                                                                                                                                                                                                              |
| `CI_JOB_JWT`                             | 12.10  | all    | A RS256 JSON web token to authenticate with third party systems that support JWT authentication, for example [HashiCorp’s Vault](https://docs.gitlab.com/ee/ci/secrets/index.html).                                                                                                                                                                         |
| `CI_JOB_MANUAL`                          | 8.12   | all    | job是否是人工执行的                                                                                                                                                                                                                                                                                                                                                 |
| `CI_JOB_NAME`                            | 8.12   | all    | job名称                                                                                                                                                                                                                                                                                                                                                       |
| `CI_JOB_STAGE`                           | 9.0    | 0.5    | The name of the job’s stage.                                                                                                                                                                                                                                                                                                                                |
| `CI_JOB_STATUS`                          | all    | 13.5   | The status of the job as each runner stage is executed. Use with [`after_script`](https://docs.gitlab.com/ee/ci/yaml/README.html#after_script). Can be `success`, `failed`, or `canceled`.                                                                                                                                                                  |
| `CI_JOB_TOKEN`                           | 9.0    | 1.2    | A token to authenticate with [certain API endpoints](https://docs.gitlab.com/ee/api/README.html#gitlab-cicd-job-token). The token is valid as long as the job is running.                                                                                                                                                                                   |
| `CI_JOB_URL`                             | 11.1   | 0.5    | The job details URL.                                                                                                                                                                                                                                                                                                                                        |
| `CI_JOB_STARTED_AT`                      | 13.10  | all    | The UTC datetime when a job started, in [ISO 8601](https://tools.ietf.org/html/rfc3339#appendix-A) format.                                                                                                                                                                                                                                                  |
| `CI_KUBERNETES_ACTIVE`                   | 13.0   | all    | Only available if the pipeline has a Kubernetes cluster available for deployments. `true` when available.                                                                                                                                                                                                                                                   |
| `CI_NODE_INDEX`                          | 11.5   | all    | The index of the job in the job set. Only available if the job uses [`parallel`](https://docs.gitlab.com/ee/ci/yaml/README.html#parallel).                                                                                                                                                                                                                  |
| `CI_NODE_TOTAL`                          | 11.5   | all    | The total number of instances of this job running in parallel. Set to `1` if the job does not use [`parallel`](https://docs.gitlab.com/ee/ci/yaml/README.html#parallel).                                                                                                                                                                                    |
| `CI_OPEN_MERGE_REQUESTS`                 | 13.8   | all    | A comma-separated list of up to four merge requests that use the current branch and project as the merge request source. Only available in branch and merge request pipelines if the branch has an associated merge request. For example, `gitlab-org/gitlab!333,gitlab-org/gitlab-foss!11`.                                                                |
| `CI_PAGES_DOMAIN`                        | 11.8   | all    | The configured domain that hosts GitLab Pages.                                                                                                                                                                                                                                                                                                              |
| `CI_PAGES_URL`                           | 11.8   | all    | The URL for a GitLab Pages site. Always a subdomain of `CI_PAGES_DOMAIN`.                                                                                                                                                                                                                                                                                   |
| `CI_PIPELINE_ID`                         | 8.10   | all    | 流水线ID，gitlab实例内唯一                                                                                                                                                                                                                                                                                                                                           |
| `CI_PIPELINE_IID`                        | 11.0   | all    | 流水线ID，项目内唯一                                                                                                                                                                                                                                                                                                                                                 |
| `CI_PIPELINE_SOURCE`                     | 10.0   | all    | How the pipeline was triggered. Can be `push`, `web`, `schedule`, `api`, `external`, `chat`, `webide`, `merge_request_event`, `external_pull_request_event`, `parent_pipeline`, [`trigger`, or `pipeline`](https://docs.gitlab.com/ee/ci/triggers/README.html#authentication-tokens).                                                                       |
| `CI_PIPELINE_TRIGGERED`                  | all    | all    | `true` if the job was [triggered](https://docs.gitlab.com/ee/ci/triggers/README.html).                                                                                                                                                                                                                                                                      |
| `CI_PIPELINE_URL`                        | 11.1   | 0.5    | The URL for the pipeline details.                                                                                                                                                                                                                                                                                                                           |
| `CI_PIPELINE_CREATED_AT`                 | 13.10  | all    | The UTC datetime when the pipeline was created, in [ISO 8601](https://tools.ietf.org/html/rfc3339#appendix-A) format.                                                                                                                                                                                                                                       |
| `CI_PROJECT_CONFIG_PATH`                 | 13.8   | all    | (Deprecated) The CI configuration path for the project. [Deprecated](https://gitlab.com/gitlab-org/gitlab/-/issues/321334) in GitLab 13.10. [Removal planned](https://gitlab.com/gitlab-org/gitlab/-/issues/322807) for GitLab 14.0.                                                                                                                        |
| `CI_PROJECT_DIR`                         | all    | all    | The full path the repository is cloned to, and where the job runs from. If the GitLab Runner `builds_dir` parameter is set, this variable is set relative to the value of `builds_dir`. For more information, see the [Advanced GitLab Runner configuration](https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-section). |
| `CI_PROJECT_ID`                          | all    | all    | The ID of the current project. This ID is unique across all projects on the GitLab instance.                                                                                                                                                                                                                                                                |
| `CI_PROJECT_NAME`                        | 8.10   | 0.5    | The name of the directory for the project. For example if the project URL is `gitlab.example.com/group-name/project-1`, `CI_PROJECT_NAME` is `project-1`.                                                                                                                                                                                                   |
| `CI_PROJECT_NAMESPACE`                   | 8.10   | 0.5    | The project namespace (username or group name) of the job.                                                                                                                                                                                                                                                                                                  |
| `CI_PROJECT_PATH_SLUG`                   | 9.3    | all    | `$CI_PROJECT_PATH` in lowercase with characters that are not `a-z` or `0-9` replaced with `-`. Use in URLs and domain names.                                                                                                                                                                                                                                |
| `CI_PROJECT_PATH`                        | 8.10   | 0.5    | The project namespace with the project name included.                                                                                                                                                                                                                                                                                                       |
| `CI_PROJECT_REPOSITORY_LANGUAGES`        | 12.3   | all    | A comma-separated, lowercase list of the languages used in the repository. For example `ruby,javascript,html,css`.                                                                                                                                                                                                                                          |
| `CI_PROJECT_ROOT_NAMESPACE`              | 13.2   | 0.5    | The root project namespace (username or group name) of the job. For example, if `CI_PROJECT_NAMESPACE` is `root-group/child-group/grandchild-group`, `CI_PROJECT_ROOT_NAMESPACE` is `root-group`.                                                                                                                                                           |
| `CI_PROJECT_TITLE`                       | 12.4   | all    | The human-readable project name as displayed in the GitLab web interface.                                                                                                                                                                                                                                                                                   |
| `CI_PROJECT_URL`                         | 8.10   | 0.5    | The HTTP(S) address of the project.                                                                                                                                                                                                                                                                                                                         |
| `CI_PROJECT_VISIBILITY`                  | 10.3   | all    | The project visibility. Can be `internal`, `private`, or `public`.                                                                                                                                                                                                                                                                                          |
| `CI_REGISTRY_IMAGE`                      | 8.10   | 0.5    | The address of the project’s Container Registry. Only available if the Container Registry is enabled for the project.                                                                                                                                                                                                                                       |
| `CI_REGISTRY_PASSWORD`                   | 9.0    | all    | The password to push containers to the project’s GitLab Container Registry. Only available if the Container Registry is enabled for the project.                                                                                                                                                                                                            |
| `CI_REGISTRY_USER`                       | 9.0    | all    | The username to push containers to the project’s GitLab Container Registry. Only available if the Container Registry is enabled for the project.                                                                                                                                                                                                            |
| `CI_REGISTRY`                            | 8.10   | 0.5    | The address of the GitLab Container Registry. Only available if the Container Registry is enabled for the project. This variable includes a `:port` value if one is specified in the registry configuration.                                                                                                                                                |
| `CI_REPOSITORY_URL`                      | 9.0    | all    | The URL to clone the Git repository.                                                                                                                                                                                                                                                                                                                        |
| `CI_RUNNER_DESCRIPTION`                  | 8.10   | 0.5    | The description of the runner.                                                                                                                                                                                                                                                                                                                              |
| `CI_RUNNER_EXECUTABLE_ARCH`              | all    | 10.6   | The OS/architecture of the GitLab Runner executable. Might not be the same as the environment of the executor.                                                                                                                                                                                                                                              |
| `CI_RUNNER_ID`                           | 8.10   | 0.5    | The unique ID of the runner being used.                                                                                                                                                                                                                                                                                                                     |
| `CI_RUNNER_REVISION`                     | all    | 10.6   | The revision of the runner running the job.                                                                                                                                                                                                                                                                                                                 |
| `CI_RUNNER_SHORT_TOKEN`                  | all    | 12.3   | First eight characters of the runner’s token used to authenticate new job requests. Used as the runner’s unique ID.                                                                                                                                                                                                                                         |
| `CI_RUNNER_TAGS`                         | 8.10   | 0.5    | A comma-separated list of the runner tags.                                                                                                                                                                                                                                                                                                                  |
| `CI_RUNNER_VERSION`                      | all    | 10.6   | The version of the GitLab Runner running the job.                                                                                                                                                                                                                                                                                                           |
| `CI_SERVER_HOST`                         | 12.1   | all    | The host of the GitLab instance URL, without protocol or port. For example `gitlab.example.com`.                                                                                                                                                                                                                                                            |
| `CI_SERVER_NAME`                         | all    | all    | The name of CI/CD server that coordinates jobs.                                                                                                                                                                                                                                                                                                             |
| `CI_SERVER_PORT`                         | 12.8   | all    | The port of the GitLab instance URL, without host or protocol. For example `8080`.                                                                                                                                                                                                                                                                          |
| `CI_SERVER_PROTOCOL`                     | 12.8   | all    | The protocol of the GitLab instance URL, without host or port. For example `https`.                                                                                                                                                                                                                                                                         |
| `CI_SERVER_REVISION`                     | all    | all    | GitLab revision that schedules jobs.                                                                                                                                                                                                                                                                                                                        |
| `CI_SERVER_URL`                          | 12.7   | all    | The base URL of the GitLab instance, including protocol and port. For example `https://gitlab.example.com:8080`.                                                                                                                                                                                                                                            |
| `CI_SERVER_VERSION_MAJOR`                | 11.4   | all    | The major version of the GitLab instance. For example, if the GitLab version is `13.6.1`, the `CI_SERVER_VERSION_MAJOR` is `13`.                                                                                                                                                                                                                            |
| `CI_SERVER_VERSION_MINOR`                | 11.4   | all    | The minor version of the GitLab instance. For example, if the GitLab version is `13.6.1`, the `CI_SERVER_VERSION_MINOR` is `6`.                                                                                                                                                                                                                             |
| `CI_SERVER_VERSION_PATCH`                | 11.4   | all    | The patch version of the GitLab instance. For example, if the GitLab version is `13.6.1`, the `CI_SERVER_VERSION_PATCH` is `1`.                                                                                                                                                                                                                             |
| `CI_SERVER_VERSION`                      | all    | all    | The full version of the GitLab instance.                                                                                                                                                                                                                                                                                                                    |
| `CI_SERVER`                              | all    | all    | Available for all jobs executed in CI/CD. `yes` when available.                                                                                                                                                                                                                                                                                             |
| `CI_SHARED_ENVIRONMENT`                  | all    | 10.1   | Only available if the job is executed in a shared environment (something that is persisted across CI/CD invocations, like the `shell` or `ssh` executor). `true` when available.                                                                                                                                                                            |
| `GITLAB_CI`                              | all    | all    | Available for all jobs executed in CI/CD. `true` when available.                                                                                                                                                                                                                                                                                            |
| `GITLAB_FEATURES`                        | 10.6   | all    | The comma-separated list of licensed features available for the GitLab instance and license.                                                                                                                                                                                                                                                                |
| `GITLAB_USER_EMAIL`                      | 8.12   | all    | The email of the user who started the job.                                                                                                                                                                                                                                                                                                                  |
| `GITLAB_USER_ID`                         | 8.12   | all    | The ID of the user who started the job.                                                                                                                                                                                                                                                                                                                     |
| `GITLAB_USER_LOGIN`                      | 10.0   | all    | The username of the user who started the job.                                                                                                                                                                                                                                                                                                               |
| `GITLAB_USER_NAME`                       | 10.0   | all    | The name of the user who started the job.                                                                                                                                                                                                                                                                                                                   |
| `TRIGGER_PAYLOAD`                        | 13.9   | all    | The webhook payload. Only available when a pipeline is [triggered with a webhook](https://docs.gitlab.com/ee/ci/triggers/README.html#using-webhook-payload-in-the-triggered-pipeline).                                                                                                                                                                      |

## 学习链接

[实战：从 0 到 1 极狐GitLab CI/CD 前端持续部署_拿我格子衫来的博客-CSDN博客](https://fizzz.blog.csdn.net/article/details/119764533?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-119764533-blog-107552156.pc_relevant_multi_platform_whitelistv1&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-119764533-blog-107552156.pc_relevant_multi_platform_whitelistv1&utm_relevant_index=1)

[在CI/CD中演示前端三种部署方案，镜像部署，服务器部署，OSS部署_拿我格子衫来的博客-CSDN博客](https://fizzz.blog.csdn.net/article/details/108346647?spm=1001.2014.3001.5502)

[🛫 前端自动化部署：借助Gitlab CI/CD实现 - 掘金](https://juejin.cn/post/7037022688493338661)

[Docker安装Gitlab和Runner并实现CICD_包含如何创建自己的docker](https://blog.csdn.net/weixin_43835717/article/details/102073410)

[gitlab CI流水线配置文件.gitlab-ci.yml详解](https://www.cnblogs.com/wangshuyang/p/14110065.html)

[GitLab如何使CI/CD仅限指定用户使用_sandaawa的博客-CSDN博客](https://blog.csdn.net/sandaawa/article/details/112897733)

## Release命令测试

文档链接：[Releases API | GitLab](https://docs.gitlab.com/ee/api/releases/index.html)

创建release

```
curl -H "Content-Type: application/json" -H "PRIVATE-TOKEN: your token" -d '{ "name": "v2.1.8-alpha1-1656989338", "tag_name": "v2.1.8-alpha1-1656989338", "ref":"4b8756a1f27e3a8b0608ebe3a9ecde1255b6ba54" ,"description": "测试release"}' --request POST http://gitlab.com/api/v4/projects/projectId/releases
```

删除release

```
curl --request DELETE --header "PRIVATE-TOKEN: your token" "http://gitlab.com/api/v4/projects/projectId/releases/:tag_name"
```
