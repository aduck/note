# docker基础

## 安装docker

```bash
brew cask install docker # 或者去官网下载安装包(mac)
# ubuntu 18.04
curl -fsSL get.docker.com -o get-docker.sh  # 下载脚本
sudo sh get-docker.sh --mirror Aliyun # 执行脚本
sudo systemctl enable docker # 开机自启动
sudo systemctl start docker # 启动
sudo groupadd docker # 添加docker组，脚本会自动创建可忽略
sudo usermod -aG docker $USER # 将当前用户加入docker组，退出重登（不用加sudo了）
```

## 修改源地址

修改或者创建文件/etc/docker/daemon.json，重启docker服务

```json
{ 
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

## 查看容器

```bash
docker ps
docker container ls [-la]
```

## 销毁容器

```bash
docker kill 容器id
```

## 销毁所有容器

```bash
docker container rm -f $(docker ps -aq)
```

## 查看镜像

```bash
docker images
docker image ls
```

## 下载镜像

```bash
docker pull imagename
```

## 删除所有镜像

```bash
docker rmi -f $(docker images -aq)
```

## 删除没有tag的镜像

```bash
docker rmi -f $(docker image ls -a | grep none | awk '{print $3}')
```

## 启动镜像(创建容器)

```bash
# -d后台方式 -v文件映射 -p端口映射
docker run --name xxx -d -v 工作目录:镜像目录 -p 工作端口:镜像端口 镜像名称
```

## 获取容器信息

```bash
docker inspect containerid
```

## 通过bash与容器交互

```bash
docker exec -it containerid bash
```

## 通过配置文件启动多个镜像

```bash
# 创建docker-compose.yml文件
touch docker-compose.yml
vi docker-compose.yml
# 添加
version: "3"
services:
  xware:
    image: yinheli/docker-thunder-xware:latest
    volumes:
      - ./download/:/app/TDDOWNLOAD
# 启动服务
docker-compose up -d
```

## 编写Dockerfile

* FROM 基础镜像
  > 可以是镜像库里的也可以是自定义的

* WORKDIR dist
  > 指定镜像里的工作目录，不存在会自动创建

* COPY ./src dist
  > 从本地复制文件到镜像

* RUN cmd
  > 执行一段脚本，这个命令每使用一次会加一层，要尽量把脚本合并编写

* ENV AAA=123 BBB=456
  > 设置环境变量，可以通过$AAA引用

* USER user
  > 指定当前用户

* CMD 启动命令
  > 一个dockerfile只会执行一次，多次只有最后一次执行

* EXPOSE port
  > 声明暴露端口

* HEALTHCHECK --interval=5s --timeout=5s --retries=3
  > 镜像健康状态和重试

* ONBUILD cmd arg
  > 如ONBUILD COPY . /app，写在基础镜像里，当其他镜像执行FROM命令时才会执行

* ENTRYPOINT cmd
  > 入口点，存在ENTRYPOINT后，CMD 的内容将会作为参数传给 ENTRYPOINT

### 一个demo

```bash
FROM node
WORKDIR /app
COPY ./package.json /app
RUN ["npm", "install"]
COPY . /app
CMD [ "npm", "run", "build" ]
EXPOSE 80
```

## 打包镜像

```bash
docker build -t 镜像名 .
```
