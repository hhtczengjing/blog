---
layout: post
title: "Jenkins Pipeline 插件自定义环境变量丢失问题"
date: 2023-10-10 19:08:06 +0800
comments: true
tags: Note
---

最近将一些构建任务由 Freestyle 迁移为 Pipeline 的实现 （主要是考虑后续可以通过任务模板的方式动态创建和维护），整体的实现过程较为简单，基本上就是把原来的执行步骤拆分成一个个的 Stage，将里面用到的一些配置抽取成环境变量的方式，统一设置到一起。

最终的配置脚本如下（伪代码）：

```
node("mac01") {
    stage('Environment Setting') {
        env.ENV_BUILD_SCRIPT_URL = "http://127.0.0.1:8080/test/script.zip"
        env.ENV_BUILD_CONFIG_NAME = "xxxxx"
        env.ENV_BUILD_XCODE_PATH = "/Applications/Xcode-14.3.1.app"
    }

    try {
        stage('Download Source Code') {
            // ...
        }

        stage('Build Project') {
            executeScript ext: '', path: '${WORKSPACE}/script.zip', url: "${ENV_BUILD_SCRIPT_URL}", commandLine: '''
                export LANG=en_US.UTF-8
                [ -d "${ENV_BUILD_XCODE_PATH}" ] && export DEVELOPER_DIR="${ENV_BUILD_XCODE_PATH}/Contents/Developer"
                cd ${WORKSPACE}/script
                ./build --config "${ENV_BUILD_CONFIG_NAME}" --job_name "${JOB_BASE_NAME}" --job_id "${BUILD_ID}" --branch "${BRANCH}" --workspace "${WORKSPACE}" --verbose
            '''
        }
        
        stage('Upload Artifactory') {
            //....
        }

        stage('Clean Workspace') {
            cleanWs()
        }
    } catch(e) {
        // ...
        cleanWs()
        throw e
    }
}
```

运行过程中发现执行命令行的时候，环境变量 `ENV_BUILD_XCODE_PATH` 和 `ENV_BUILD_CONFIG_NAME` 获取不到，但是写一个单独的脚本执行却能正常打印出来：

```
sh label: '', script: '''
    export LANG=en_US.UTF-8
    echo $ENV_BUILD_CONFIG_NAME
    echo $ENV_BUILD_XCODE_PATH
'''
```

`executeScript` 是一个自定义的插件，用于下载脚本工具（下载完成后会自动注入一些通用的上下文数据），并执行指定的命令。

![plugin_settings](/images/pipeline_plugin_env_missing/plugin_settings.png)

怀疑是不是插件实现的代码有问题，可能忘了对命令行字符串里面的环境变量进行替换。看了一下代码，在插件内部的主要实现是获取当前的执行环境变量，然后对命令脚本字符串的变量进行替换，核心代码如下：

```
EnvVars env = getPipelineEnvironment(run, filePath, listener);
commandLine = env.expand(commandLine);
```

获取 Pipeline 的环境变量的方法：

```
public static EnvVars getPipelineEnvironment(Run<?, ?> run, FilePath filePath, TaskListener listener) throws IOException, InterruptedException {
    EnvVars env = run.getEnvironment(listener);
    if (!env.containsKey("WORKSPACE") && filePath != null) {
        env.put("WORKSPACE", filePath.getRemote());
    }
    return env;
}
```

咋一看没啥毛病，在 `env.expand` 之前下了一个断点，查看了一下 `env` (其实内容是一个字典)，确实没有找到前面定义的 `ENV_` 开头的自定义环境变量。查了一些资料发现获取环境变量的代码都是上面这段代码（其实这段代码也是从网上Copy来的）。通过对 `getPipelineEnvironment` 下断点发现通过获取action里面的EnvActionImpl的数据可以拿到想要的环境变量的数据：

![stack_value](/images/pipeline_plugin_env_missing/stack_value.png)

那就好办了，只要拿到 Action 列表遍历判断是不是 EnvActionImpl(`org.jenkinsci.plugins.workflow.cps.EnvActionImpl`) 这个Action，然后把 env 属性对应的数据拿出来就完成了。本来打算通过动态Class获取，搜索了一番发现可以直接通过添加依赖集成。在 pom.xml 中添加依赖

```
<dependency>
    <groupId>org.jenkins-ci.plugins.workflow</groupId>
    <artifactId>workflow-cps</artifactId>
    <version>2.41</version>
</dependency>
```

修改上面的 getPipelineEnvironment 方法增加如下代码：

```
Iterator envActionIterator = run.getActions(EnvActionImpl.class).iterator();
while(envActionIterator.hasNext()) {
    EnvActionImpl a = (EnvActionImpl)envActionIterator.next();
    env.putAll(a.getEnvironment());
}
```

重新运行，一切正常，重新打包，发布。

### 参考资料

- 1、[EnvActionImpl.java](https://www.javatips.net/api/Pipeline-master/workflow-cps-plugin-master/src/main/java/org/jenkinsci/plugins/workflow/cps/EnvActionImpl.java)

- 2、[Using environment variables](https://www.jenkins.io/doc/pipeline/tour/environment/)