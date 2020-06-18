---
title: Github Actions的简单使用
date: 2020-04-13 21:51:49
updated: 2020-04-14 00:00:00
tags: [CI/CD]
categories: [使用教程]
---

### CI/CD

要使用Github Actions，首先要了解CI/CD的概念，在软件工程中，CI / CD或CICD通常是指持续集成以及持续交付或持续部署的组合实践(from [wiki](https://en.wikipedia.org/wiki/CI/CD))。具体而言，CI/CD 可让持续自动化和持续监控贯穿于应用的整个生命周期（从集成和测试阶段，到交付和部署）。这些关联的事务通常被统称为“CI/CD 管道”，由开发和运维团队以敏捷方式协同支持。

### GitHub Actions

GitHub Actions 就是 GitHub 的持续集成服务，于2018年10月推出，类似于Travis-ci可以帮助我们做持续集成。

### 基本概念

先来看一个基础的actions文件：

```yaml
name: CI
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run a one-line script
      run: echo Hello, world!
    - name: Run a multi-line script
      run: |
        echo Add other actions to build,
        echo test, and deploy your project.
```

GitHub Actions 有一些自己的术语。

（1）**workflow** （工作流程）：持续集成一次运行的过程，就是一个 workflow。

（2）**job** （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。

（3）**step**（步骤）：每个 job 由多个 step 构成，一步步完成。

（4）**action** （动作）：每个 step 可以依次执行一个或多个命令（action）。

### 文件详解

#### name

首先就是name，name就是这个actions的名字；

#### on

on：是指什么时候会触发这个action，常用的是`push`；

也可指定分支或路径：

```yaml
on:
	push:
		branches: [master]
		paths: [src/*]
```

或者指定时间除出发(每15分钟执行一次)

```yaml
on:
  schedule:
    # * is a special character in YAML so you have to quote this string
    - cron:  '*/15 * * * *'
```

#### jobs

```yaml
jobs:
	job_name:
		runs-on: 运行actions的环境，如	ubuntu-latest
		container: #在容器中运行
          image: node:10.16-jessie
          env:
            NODE_ENV: development
          ports:
            - 80
          volumes:
            - my_docker_volume:/volume_mount
          options: --cpus 1
        services: #用于为工作流程中的作业托管服务容器
          nginx:
            image: nginx
            # Map port 8080 on the Docker host to port 80 on the nginx container
            ports:
              - 8080:80
		needs: 在这之前需要执行的job，如pre_job
		strategy: 多策略
			matrix:
    		node: [6, 8, 10]
		timeout-minutes: 360 #让作业运行的最大分钟数。 默认值：360
		steps:
		- uses: actions/checkout@v2 #下载git库中的文件,uses复用流程
		  with: #传入环境变量
		  	env: env_val
		  	node-version: ${{ matrix.node }}
		- name: step_name
		  run: run_scrpits
		  env: #环境变量
		  	name: x
		  	val: h
	pre_job:
		runs-on: macos-latest
		outputs: #作业的输出 map。 作业输出可用于所有依赖此作业的下游作业。
			output1: ${{ steps.step1.outputs.test }}
      		output2: ${{ steps.step2.outputs.test }}
```

可用的runs-on：

| 虚拟环境             | YAML 工作流程标签                  |
| :------------------- | :--------------------------------- |
| Windows Server 2019  | `windows-latest` 或 `windows-2019` |
| Ubuntu 18.04         | `ubuntu-latest` 或 `ubuntu-18.04`  |
| Ubuntu 16.04         | `ubuntu-16.04`                     |
| macOS Catalina 10.15 | `macos-latest` 或 `macos-10.15`    |

### 添加密钥

`repo->settings->secrets`:

如{name: MYSECRET, value: my_val}

就可以用 `${{ secrets.MYSECRETS }}`取出。

### 上传build完成的文件

#### Upload an Individual File

```
steps:
- uses: actions/checkout@v2

- run: mkdir -p path/to/artifact

- run: echo hello > path/to/artifact/world.txt

- uses: actions/upload-artifact@v2
  with:
    name: my-artifact
    path: path/to/artifact/world.txt
```

#### Upload an Entire Directory

```
- uses: actions/upload-artifact@v2
  with:
    name: my-artifact
    path: path/to/artifact/ # or path/to/artifact
```

#### Upload using a Wildcard Pattern:

```
- uses: actions/upload-artifact@v2
  with:
    name: my-artifact
    path: path/**/[abc]rtifac?/*
```

### 参考

[Awesome Actions](https://github.com/sdras/awesome-actions)

[官方帮助文档](https://help.github.com/cn/actions)

[Github Actions入门](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)