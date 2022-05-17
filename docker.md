

## docker

##### 安装curl报错

https://blog.csdn.net/PY0312/article/details/95265761

##### 登录报错

Error response from daemon: Get "https://192.168.0.7/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

修改配置

- sudo vim /etc/docker/daemon.json
- 把要登录的服务器地址加入到`insecure-registries`中

```json
{
    "registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],
    "insecure-registries": ["192.168.0.7"]
}
```

- 重启docker

    systemctl daemon-reload 
    systemctl restart docker

docker build -t calculator_v .

docker run -p 3000:8000 -d calculator_v1  把容器中的8000端口（server.js中监听的端口）映射到本机上的3000端口



##### npm ci与npm i主要有以下的区别：

- npm i依赖package.json，而npm ci依赖package-lock.json。
- 当package-lock.json中的依赖于package.json不一致时，npm ci退出但不会修改package-lock.json。
- npm ci只可以一次性的安装整个项目依赖，但无法添加单个依赖项。
- npm ci安装包之前，会删除掉node_modules文件夹，因此他不需要去校验已下载文件版本与控制版本的关系，也不用校验是否存在最新版本的库，所以下载的速度更快。
- npm安装时，不会修改package.json与package-lock.json。



##### .gitlab-ci.yml示例

```yaml
image: node:lts-alpine

stages:
  - make
  - build
  - push

cache:
  paths:
    - node_modules/

make:
  stage: make
  script:
    - npm install --registry=https://registry.npm.taobao.org --force
    - npm run build
  tags:
    - shell
  only:
    - tags
  artifacts:
    paths:
      - dist/

dockerimage:
  stage: build
  script:
    - docker build -t 192.168.0.7/node/calculator_vue:$CI_COMMIT_REF_NAME .
  tags:
    - shell
  only:
    - tags
  dependencies:
    - make

dockerpush:
  stage: push
  script:
    - docker login 192.168.0.7 -u forthink -p Ehigh2019
    - docker push 192.168.0.7/node/calculator_vue:$CI_COMMIT_REF_NAME
    - docker rmi 192.168.0.7/node/calculator_vue:$CI_COMMIT_REF_NAME
  only:
    - tags
  tags:
    - shell
```

##### 容器启动后立即自动退出

- node版本问题
- 没有加入需要的依赖

docker logs --tail 20 <container_id>查看最近20行的容器日志

要编译ts文件，需要把typescript依赖添加到dependencies中

```
sh: tsc: not found
```

dependencies中加了的依赖，devDependencies中需要删除

```
error TS2688: Cannot find type definition file for 'koa'.
  The file is in the program because:
    Entry point of type library 'koa' specified in compilerOptions
```



##### Dockerfile中`CMD`和`RUN`的区别：

- CMD用于指定在容器启动时所要执行的命令

- RUN用于在镜像容器中执行命令

也就是RUN在创建容器时会立即执行，CMD是在容器启动时执行

在CMD中执行npm命令：

```
CMD ["npm", "run", "start"]
```

##### 命令

新建文件 touch filename

清空 ctrl + l

vim 命令：https://blog.csdn.net/feosun/article/details/73196299
