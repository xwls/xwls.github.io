+++
title = '使用 Shell 脚本批量反编译 JAR 包：老旧项目逆向工程实战'
date = 2025-02-10T11:25:00+08:00
categories = ['Java']
tags = ['shell', 'java', 'decompile', 'fernflower']
summary = '接手一个只有 JAR 包没有源码的老旧 Java 项目，面对数百个 JAR 文件，手动反编译不现实。'
+++

## 背景

最近接手了一个通过 JAR 包运行的老旧 Java 项目（Epoint 架构），任务是对其进行升级改造。

棘手的问题在于：**只有部署环境的 JAR 包，没有源代码。**

为了搞清楚其中的业务逻辑，方便在重构时进行全局搜索和参考，我需要将这些 JAR 包反编译回 Java 源码。面对目录下几百个混杂的 JAR 文件，手动一个一个反编译显然是不现实的，于是我编写了一套 Shell 脚本来自动化完成这个过程。

## 难点分析

拿到 `WEB-INF/lib` 目录下的文件列表后，我发现里面有数百个 JAR 包。经过分析，这些包主要分为两类：

1.  **第三方开源库/中间件**：如 `spring-*`, `hadoop-*`, `commons-*`, `cxf-*` 等。这些不需要反编译，网上都能找到源码，且不包含业务逻辑。
2.  **业务模块 JAR**：如 `Epoint*.jar`, `datacenter*.jar`, `zwbg*.jar` 以及一些带有特定后缀（如 `-own`, `-epoint`）的魔改包。

**策略**：采用"白名单"模式，编写脚本自动过滤掉开源库，只针对业务相关的 JAR 包调用反编译工具。

## 工具选择

使用的反编译核心工具是 IntelliJ IDEA 自带的反编译插件核心：`java-decompiler.jar` (FernFlower)。
命令格式如下：
```bash
java -cp "java-decompiler.jar" org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dgs=true <source.jar> <dest_dir>
```

## 步骤一：智能批量反编译

为了避免在 `spring-core` 这种大包上浪费时间，我编写了如下脚本。它会遍历目录，根据文件名特征匹配出业务包，然后进行反编译。

**脚本：`decompile_filter.sh`**

```bash
#!/bin/bash

# --- 配置区域 ---
SOURCE_DIR="/Users/wzw/Downloads/epoint"
# 反编译工具路径
DECOMPILER_JAR="/Users/wzw/Tools/decompiler/java-decompiler.jar"
# 输出目录
OUTPUT_DIR="${SOURCE_DIR}/java_source"

mkdir -p "$OUTPUT_DIR"
cd "$SOURCE_DIR" || exit

echo "开始智能反编译 (仅处理业务相关 JAR 包)..."

count=0

for jar_file in *.jar; do
    if [ ! -e "$jar_file" ]; then continue; fi
    
    # --- 白名单过滤逻辑 ---
    # 仅反编译 Epoint, datacenter, lyedc, zwbg 等业务相关前缀，以及包含特殊标识的包
    if [[ "$jar_file" == Epoint* ]] || \
       [[ "$jar_file" == epoint* ]] || \
       [[ "$jar_file" == datacenter* ]] || \
       [[ "$jar_file" == lyedc* ]] || \
       [[ "$jar_file" == zwbg* ]] || \
       [[ "$jar_file" == F922* ]] || \
       [[ "$jar_file" == PushJavaSDK* ]] || \
       [[ "$jar_file" == *"-epoint.jar" ]] || \
       [[ "$jar_file" == *"own"* ]] || \
       [[ "$jar_file" == *"ueditor"* ]]; then
       
        ((count++))
        echo "[$count] 正在反编译业务包: $jar_file"
        
        /usr/bin/java -cp "$DECOMPILER_JAR" \
            org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler \
            -dgs=true \
            "$jar_file" \
            "$OUTPUT_DIR"
    else
        # 这是一个开源或第三方包，静默跳过
        : 
    fi
done

echo "任务完成！共反编译了 $count 个业务 JAR 包。"
```

## 步骤二：批量解压源码

FernFlower 反编译的结果是一个包含了 `.java` 文件的 JAR 包（例如 `original-src.jar`）。为了能在编辑器（VS Code / IDEA）中直接进行全局文本搜索，我需要把它们全部解压成文件夹。

**脚本：`unzip_all.sh`**

```bash
#!/bin/bash

TARGET_DIR="/Users/wzw/Downloads/epoint/java_source"

cd "$TARGET_DIR" || exit

count=0
for jar_file in *.jar; do
    if [ -e "$jar_file" ]; then
        ((count++))
        
        # 获取不带后缀的文件夹名
        dir_name="${jar_file%.jar}"
        
        echo "[$count] 正在解压: $jar_file -> $dir_name/"
        
        mkdir -p "$dir_name"
        # -q: 安静模式, -o: 覆盖, -d: 指定目录
        unzip -q -o "$jar_file" -d "$dir_name"
    fi
done

echo "全部任务结束，共解压 $count 个文件。"
```

## 结果

执行完上述两个脚本后，目录结构变得清晰可见：

```text
java_source/
├── datacenter-base-action/
│   ├── com/epoint/datacenter/... (可以直接查看和搜索的 .java 文件)
├── Epoint.Frame.Utils/
│   └── ...
└── ...
```

现在，只需要将 `java_source` 文件夹拖入 IDE，即可利用 IDE 强大的索引功能快速定位老项目的业务逻辑。

## 小知识点：chmod +x 与 u+x

在给脚本赋予执行权限时，我顺便复习了一下 `chmod` 的区别：

* `chmod +x script.sh`：简单粗暴，等同于 `a+x`，意味着**所有人**（包括同组用户和其他用户）都有权限执行该脚本。
* `chmod u+x script.sh`：**推荐做法**。仅给**当前用户（Owner）**赋予执行权限，更加安全，符合最小权限原则。

虽然在个人 Mac 电脑上区别不大，但在服务器操作时，习惯使用 `u+x` 是个好习惯。