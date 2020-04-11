# Gitlab CI持续集成

## 什么是持续集成

![GitLab workflow example](https://docs.gitlab.com/ee/ci/introduction/img/gitlab_workflow_example_11_9.png)

持续集成可以让项目每一次代码的commit都经过检测，持续集成过程重点是自动检测，可以及早发现代码上的问题，保证项目的质量。

## gitlab-CI核心概念

gitlab-CI是gitlab这个平台提出的持续集成的一整套实现，jenkins也是持续集成工具，本质上这两个东西完成的功能是类似的，但是实现方式是不同的，jenkins当中也有Jenkinsfile这种东西来定义持续集成工作流，语法和gitlab的.gitlab-ci.yml不同而已。

```html
+--------------------------------------------------------+
|                                                        |
|  Pipeline                                              |
|                                                        |
|  +-----------+     +------------+      +------------+  |
|  |  Stage 1  |---->|   Stage 2  |----->|   Stage 3  |  |
|  +-----------+     +------------+      +------------+  |
|                                                        |
+--------------------------------------------------------+

+------------------------------------------+
|                                          |
|  Stage 1                                 |
|                                          |
|  +---------+  +---------+  +---------+   |
|  |  Job 1  |  |  Job 2  |  |  Job 3  |   |
|  +---------+  +---------+  +---------+   |
|                                          |
+------------------------------------------+
```



git 仓库当中定义的**.gitlab-ci.yml就是在定义一个pipeline**，pipeline就是一个项目的持续集成工作流，也就是在持续集成过程中需要执行的任务流。

一个pipeline可以包含多个stages，不同stage按照顺序执行，一个stages可以包含多个jobs，一个stage中的多个jobs并行执行（只要有足够的并行runner）。

gitlab-runner执行的是jobs，gitlab-runner可以设置并行执行的数量，可以同时执行多个job，

## gitlab-ci.yml

###stages: 配置（top level）

stages用来配置当前pipeline具体包括哪些阶段，这些阶段的执行是按照顺序执行的，job

```yaml
stages:
  - build
  - test
  - deploy
```



###job: 配置（top level）

| Keyword                                                      | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`script`](https://docs.gitlab.com/ee/ci/yaml/#script)       | Shell script which is executed by Runner.                    |
| [`image`](https://docs.gitlab.com/ee/ci/yaml/#image)         | Use docker images. Also available: `image:name` and `image:entrypoint`. |
| [`services`](https://docs.gitlab.com/ee/ci/yaml/#services)   | Use docker services images. Also available: `services:name`, `services:alias`, `services:entrypoint`, and `services:command`. |
| [`before_script`](https://docs.gitlab.com/ee/ci/yaml/#before_script-and-after_script) | Override a set of commands that are executed before job.     |
| [`after_script`](https://docs.gitlab.com/ee/ci/yaml/#before_script-and-after_script) | Override a set of commands that are executed after job.      |
| [`stages`](https://docs.gitlab.com/ee/ci/yaml/#stages)       | Define stages in a pipeline.                                 |
| [`stage`](https://docs.gitlab.com/ee/ci/yaml/#stage)         | Defines a job stage (default: `test`).                       |
| [`only`](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic) | Limit when jobs are created. Also available: [`only:refs`, `only:kubernetes`, `only:variables`, and `only:changes`](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-advanced). |
| [`except`](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-basic) | Limit when jobs are not created. Also available: [`except:refs`, `except:kubernetes`, `except:variables`, and `except:changes`](https://docs.gitlab.com/ee/ci/yaml/#onlyexcept-advanced). |
| [`tags`](https://docs.gitlab.com/ee/ci/yaml/#tags)           | List of tags which are used to select Runner.                |
| [`allow_failure`](https://docs.gitlab.com/ee/ci/yaml/#allow_failure) | Allow job to fail. Failed job doesn’t contribute to commit status. |
| [`when`](https://docs.gitlab.com/ee/ci/yaml/#when)           | When to run job. Also available: `when:manual` and `when:delayed`. |
| [`environment`](https://docs.gitlab.com/ee/ci/yaml/#environment) | Name of an environment to which the job deploys. Also available: `environment:name`, `environment:url`, `environment:on_stop`, and `environment:action`. |
| [`cache`](https://docs.gitlab.com/ee/ci/yaml/#cache)         | List of files that should be cached between subsequent runs. Also available: `cache:paths`, `cache:key`, `cache:untracked`, and `cache:policy`. |
| [`artifacts`](https://docs.gitlab.com/ee/ci/yaml/#artifacts) | List of files and directories to attach to a job on success. Also available: `artifacts:paths`, `artifacts:name`, `artifacts:untracked`, `artifacts:when`, `artifacts:expire_in`, `artifacts:reports`, and `artifacts:reports:junit`.  In GitLab [Enterprise Edition](https://about.gitlab.com/pricing/), these are available: `artifacts:reports:codequality`, `artifacts:reports:sast`, `artifacts:reports:dependency_scanning`, `artifacts:reports:container_scanning`, `artifacts:reports:dast`, `artifacts:reports:license_management`, `artifacts:reports:performance` and `artifacts:reports:metrics`. |
| [`dependencies`](https://docs.gitlab.com/ee/ci/yaml/#dependencies) | Other jobs that a job depends on so that you can pass artifacts between them. |
| [`coverage`](https://docs.gitlab.com/ee/ci/yaml/#coverage)   | Code coverage settings for a given job.                      |
| [`retry`](https://docs.gitlab.com/ee/ci/yaml/#retry)         | When and how many times a job can be auto-retried in case of a failure. |
| [`parallel`](https://docs.gitlab.com/ee/ci/yaml/#parallel)   | How many instances of a job should be run in parallel.       |
| [`trigger`](https://docs.gitlab.com/ee/ci/yaml/#trigger-premium) | Defines a downstream pipeline trigger.                       |
| [`include`](https://docs.gitlab.com/ee/ci/yaml/#include)     | Allows this job to include external YAML files. Also available: `include:local`, `include:file`, `include:template`, and `include:remote`. |
| [`extends`](https://docs.gitlab.com/ee/ci/yaml/#extends)     | Configuration entries that this job is going to inherit from. |
| [`pages`](https://docs.gitlab.com/ee/ci/yaml/#pages)         | Upload the result of a job to use with GitLab Pages.         |
| [`variables`](https://docs.gitlab.com/ee/ci/yaml/#variables) | Define job variables on a job level.                         |

####script

配置runner在一个job当中执行的shell脚本，可以有多条shell命令

```yaml
job:
  script:
    - uname -a
    - bundle exec rspec
```

####image





##config.toml

此文件用来配置gitlab-runner，但是这个文件是首先需要使用gitlab-runner register命令来生成的，





详细参考：https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-global-section



##备注：

###Grouping jobs

Grouping jobs就是为了把多个类似的job编组成一个job组，要想要把多个job组成一个group，这些job的name需要符合一定规则，比如下面这样:

```yaml
test_me 1 2:
  script:
  - make

test_me 2 2:
  script:
  - make
```



gitlab-runner使用go语言进行编写，所以可以生成跨平台执行的二进制程序，如果要使用docker作为执行器，那么docker版本必须是>=v1.13.0



##参考：

* gitlab-ci.yml模板：https://gitlab.com/gitlab-org/gitlab-ce/tree/master/lib/gitlab/ci/templates

* gitlab-ci简介：https://docs.gitlab.com/ee/ci/introduction/

* gitlab-ci.yml配置：https://docs.gitlab.com/ee/ci/yaml/README.html#introduction

* gitlab-ci使用docker runner：https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#register-docker-runner
* 在gitlab上配置runner：https://docs.gitlab.com/ee/ci/runners/README.html
* 

