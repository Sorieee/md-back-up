# 安装

参考

http://doc.ruoyi.vip/ruoyi-cloud/cloud/dokcer.html#%E5%9F%BA%E6%9C%AC%E4%BB%8B%E7%BB%8D

```sh
yum install -y yum-utils
# 官方地址（比较慢）
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
	
# 阿里云地址（国内地址，相对更快）
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
yum install docker-ce docker-ce-cli containerd.io


docker version # 查看Docker版本信息

systemctl start docker		# 启动 docker 服务:
systemctl enable docker		# 开机启动 docker 服务:
systemctl status docker		# 查看 docker 服务状态
# 配置镜像
touch /etc/docker/daemon.json
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://mr63yffu.mirror.aliyuncs.com"]
}
EOF

systemctl daemon-reload
```

# 架构概念

​	通过下图可以得知，`Docker`在运行时分为`Docker引擎（服务端守护进程）`和`客户端工具`，我们日常使用各种`docker命令`，其实就是在使用`客户端工具`与`Docker`引擎进行交互。

![](https://oscimg.oschina.net/oscnet/up-216ccca6be8f28927f914b667ad9b2dad74.JPEG)

## 什么是容器

Simply put, a container is a sandboxed process on your machine that is isolated from all other processes on the host machine. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504), features that have been in Linux for a long time. Docker has worked to make these capabilities approachable and easy to use. To summarize, a container:

* is a runnable instance of an image. You can create, start, stop, move, or delete a container using the DockerAPI or CLI.
* can be run on local machines, virtual machines or deployed to the cloud.
* is portable (can be run on any OS)
* Containers are isolated from each other and run their own software, binaries, and configurations.

## 什么是容器镜像

When running a container, it uses an isolated filesystem. This custom filesystem is provided by a **container image**. Since the image contains the container’s filesystem, it must contain everything needed to run an application - all dependencies, configuration, scripts, binaries, etc. The image also contains other configuration for the container, such as environment variables, a default command to run, and other metadata.



# 常用命令

```sh
# 运行
docker run -dp 80:80 docker/getting-started
docker run -it ubuntu ls /
# 构建镜像
docker build -t getting-started .

# 推送镜像
docker push docker/getting-started

# 命名
docker tag getting-started YOUR-USER-NAME/getting-started
```

# 持久化

1. Create a volume by using the `docker volume create` command.

   ```
    $ docker volume create todo-db
   ```

2. Stop and remove the todo app container once again in the Dashboard (or with `docker rm -f <id>`), as it is still running without using the persistent volume.

3. Start the todo app container, but add the `-v` flag to specify a volume mount. We will use the named volume and mount it to `/etc/todos`, which will capture all files created at the path.

   ```
    $ docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
   ```

A lot of people frequently ask “Where is Docker *actually* storing my data when I use a named volume?” If you want to know, you can use the `docker volume inspect` command.

```
$ docker volume inspect todo-db
```

![](https://pic.imgdb.cn/item/62583d01239250f7c587cc23.jpg)

The `Mountpoint` is the actual location on the disk where the data is stored. Note that on most machines, you will need to have root access to access this directory from the host. But, that’s where it is!

# mount

| Named Volumes                                | Bind Mounts               |                               |
| :------------------------------------------- | :------------------------ | ----------------------------- |
| Host Location                                | Docker chooses            | You control                   |
| Mount Example (using `-v`)                   | my-volume:/usr/local/data | /path/to/data:/usr/local/data |
| Populates new volume with container contents | Yes                       | No                            |
| Supports Volume Drivers                      | Yes                       | No                            |

```
 docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
```

If you are using PowerShell then use this command:

```
 PS> docker run -dp 3000:3000 `
     -w /app -v "$(pwd):/app" `
     node:12-alpine `
     sh -c "yarn install && yarn run dev"
```

- `-dp 3000:3000` - same as before. Run in detached (background) mode and create a port mapping
- `-w /app` - sets the “working directory” or the current directory that the command will run from
- `-v "$(pwd):/app"` - bind mount the current directory from the host in the container into the `/app` directory
- `node:12-alpine` - the image to use. Note that this is the base image for our app from the Dockerfile
- `sh -c "yarn install && yarn run dev"` - the command. We’re starting a shell using `sh` (alpine doesn’t have `bash`) and running `yarn install` to install *all* dependencies and then running `yarn run dev`. If we look in the `package.json`, we’ll see that the `dev` script is starting `nodemon`.

# network

```
docker network create todo-app

docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
     
 docker run -it --network todo-app nicolaka/netshoot
 
 dig mysql
```

# Docker Compose

If you installed Docker Desktop/Toolbox for either Windows or Mac, you already have Docker Compose! Play-with-Docker instances already have Docker Compose installed as well. If you are on a Linux machine, you will need to [install Docker Compose](https://docs.docker.com/compose/install/).

1. At the root of the app project, create a file named `docker-compose.yml`.

2. In the compose file, we’ll start off by defining the schema version. In most cases, it’s best to use the latest supported version. You can look at the [Compose file reference](https://docs.docker.com/compose/compose-file/) for the current schema versions and the compatibility matrix.

   ```
    version: "3.7"
   ```

3. Next, we’ll define the list of services (or containers) we want to run as part of our application.

   ```
    version: "3.7"
   
    services:
   ```

1. First, let’s define the service entry and the image for the container. We can pick any name for the service. The name will automatically become a network alias, which will be useful when defining our MySQL service.

   ```
    version: "3.7"
   
    services:
      app:
        image: node:12-alpine
   ```

2. Typically, you will see the `command` close to the `image` definition, although there is no requirement on ordering. So, let’s go ahead and move that into our file.

   ```
    version: "3.7"
   
    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
   ```

3. Let’s migrate the `-p 3000:3000` part of the command by defining the `ports` for the service. We will use the [short syntax](https://docs.docker.com/compose/compose-file/compose-file-v3/#short-syntax-1) here, but there is also a more verbose [long syntax](https://docs.docker.com/compose/compose-file/compose-file-v3/#long-syntax-1) available as well.

   ```
    version: "3.7"
   
    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
   ```

4. Next, we’ll migrate both the working directory (`-w /app`) and the volume mapping (`-v "$(pwd):/app"`) by using the `working_dir` and `volumes` definitions. Volumes also has a [short](https://docs.docker.com/compose/compose-file/compose-file-v3/#short-syntax-3) and [long](https://docs.docker.com/compose/compose-file/compose-file-v3/#long-syntax-3) syntax.

   One advantage of Docker Compose volume definitions is we can use relative paths from the current directory.

   ```
    version: "3.7"
   
    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
   ```

5. Finally, we need to migrate the environment variable definitions using the `environment` key.

   ```yml
    version: "3.7"
   
    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
        environment:
          MYSQL_HOST: mysql
          MYSQL_USER: root
          MYSQL_PASSWORD: secret
          MYSQL_DB: todos
   ```

```yml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

# Image-building best practices

## Security scanning

For example, to scan the `getting-started` image you created earlier in the tutorial, you can just type

```
$ docker scan getting-started
```

## Image layering

Did you know that you can look at what makes up an image? Using the `docker image history` command, you can see the command that was used to create each layer within an image.

1. Use the `docker image history` command to see the layers in the `getting-started` image you created earlier in the tutorial.

   ```sh
    $ docker image history getting-started
   ```

## Layer caching

Now that you’ve seen the layering in action, there’s an important lesson to learn to help decrease build times for your container images.

> Once a layer changes, all downstream layers have to be recreated as well

Let’s look at the Dockerfile we were using one more time...

```
# syntax=docker/dockerfile:1
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

1. Update the Dockerfile to copy in the `package.json` first, install dependencies, and then copy everything else in.

   ```sh
    # syntax=docker/dockerfile:1
    FROM node:12-alpine
    WORKDIR /app
    COPY package.json yarn.lock ./
    RUN yarn install --production
    COPY . .
    CMD ["node", "src/index.js"]
   ```

2. Create a file named `.dockerignore` in the same folder as the Dockerfile with the following contents.

   ```ignore
    node_modules
   ```

   `.dockerignore` files are an easy way to selectively copy only image relevant files. You can read more about this [here](https://docs.docker.com/engine/reference/builder/#dockerignore-file). In this case, the `node_modules` folder should be omitted in the second `COPY` step because otherwise, it would possibly overwrite files which were created by the command in the `RUN` step. For further details on why this is recommended for Node.js applications and other best practices, have a look at their guide on [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/).

3. Build a new image using `docker build`.

   ```sh
    $ docker build -t getting-started .
   ```

## Multi-stage builds

While we’re not going to dive into it too much in this tutorial, multi-stage builds are an incredibly powerful tool to help use multiple stages to create an image. There are several advantages for them:

- Separate build-time dependencies from runtime dependencies
- Reduce overall image size by shipping *only* what your app needs to run

### Maven/Tomcat example

When building Java-based applications, a JDK is needed to compile the source code to Java bytecode. However, that JDK isn’t needed in production. Also, you might be using tools like Maven or Gradle to help build the app. Those also aren’t needed in our final image. Multi-stage builds help.

```dockerfile
# syntax=docker/dockerfile:1
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps 
```

### React example

When building React applications, we need a Node environment to compile the JS code (typically JSX), SASS stylesheets, and more into static HTML, JS, and CSS. If we aren’t doing server-side rendering, we don’t even need a Node environment for our production build. Why not ship the static resources in a static nginx container?

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

# Java

### Enable BuildKit

Before we start building images, ensure you have enabled BuildKit on your machine. BuildKit allows you to build Docker images efficiently. For more information, see [Building images with BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/).

To set the BuildKit environment variable when running the `docker build` command, run:

```sh
$ DOCKER_BUILDKIT=1 docker build .
```

To enable docker BuildKit by default, set daemon configuration in `/etc/docker/daemon.json` feature to `true` and restart the daemon. If the `daemon.json` file doesn’t exist, create new file called `daemon.json` and then add the following to the file.

```
{
  "features":{"buildkit" : true}
}
```

Restart the Docker daemon.

**Example**

```
 cd /path/to/working/directory
$ git clone https://github.com/spring-projects/spring-petclinic.git
$ cd spring-petclinic
```

```
 ./mvnw spring-boot:run
```

## Create a Dockerfile for Java

```sh
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline

COPY src ./src

CMD ["./mvnw", "spring-boot:run"]
```

### Create a `.dockerignore` file

To increase the performance of the build, and as a general best practice, we recommend that you create a `.dockerignore` file in the same directory as the Dockerfile. For this tutorial, your `.dockerignore` file should contain just one line:

```sh
target
```





```
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13 as base

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src

FROM base as test
CMD ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM openjdk:11-jre-slim as production
EXPOSE 8080

COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```
