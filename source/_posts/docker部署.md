## Docker部署
1. [下载地址](https://docs.docker.com/engine/install/)

2. 终端输入docker run hello-word，查看是否成功，如果出现一下报错，说明需要登录
```bash
Unable to find image 'hello:latest' locally
docker: Error response from daemon: pull access denied for hello, repository does not exist or may require 'docker login': denied: requested access to the resource is denied.
See 'docker run --help'.
```
3. 安装[docker-compose](https://docs.docker.com/compose/install/linux/#install-the-plugin-manually),没有linux环境，选择第二中手动下载的方式

4. docker login，这个时候你可以使用[token](https://docs.docker.com/security/for-developers/access-tokens/)的形式来进行登录
```bash
docker login -u xxx
password: tokenxxx
```
