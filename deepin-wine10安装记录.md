# deepin-wine10 安装记录

> [!NOTE]
> 此为我命令iflow安装wine-debian后，让iflow总结安装过程而得出的文章

## 安装时间
2026年2月11日

## 系统环境
- 操作系统：`Debian GNU/Linux forky/sid (forky) x86_64  Linux 6.18.5+deb13-amd64`

## 安装目的
在 Debian 系统上安装 deepin-wine10，用于运行 Windows 应用程序。

## 安装步骤

### 1. 下载安装包
从华中科技大学镜像站下载 deepin-wine10 安装包：
```bash
wget https://mirrors.sdu.edu.cn/spark-store-repository/store/depends/deepin-wine10/deepin-wine10-stable_10.14deepin5-spark1_amd64.deb
```

文件大小：51M

### 2. 尝试直接安装
```bash
sudo dpkg -i deepin-wine10-stable_10.14deepin5-spark1_amd64.deb
```

### 3. 遇到的问题

#### 问题一：缺少依赖 `deepin-elf-verify`
**错误信息：**
```
deepin-wine10-stable 依赖于 deepin-elf-verify (>= 1.1.10-1)；然而：未安装软件包 deepin-elf-verify。
```

**解决方案：**
从华为云镜像站下载 deepin-elf-verify：
```bash
wget https://mirrors.huaweicloud.com/deepin/pool/main/d/deepin-elf-verify/deepin-elf-verify_1.2.0.6-1_amd64.deb
```

#### 问题二：缺少依赖 `libssl1.1`
**错误信息：**
```
deepin-elf-verify 依赖于 libssl1.1 (>= 1.1.1)；然而：未安装软件包 libssl1.1。
```

**解决方案：**
从华为云镜像站下载 libssl1.1：
```bash
wget https://mirrors.huaweicloud.com/debian/pool/main/o/openssl/libssl1.1_1.1.1w-0+deb11u1_amd64.deb
```

### 4. 安装依赖
```bash
sudo dpkg -i libssl1.1_1.1.1w-0+deb11u1_amd64.deb deepin-elf-verify_1.2.0.6-1_amd64.deb
```

### 5. 重新安装 deepin-wine10
```bash
sudo dpkg -i deepin-wine10-stable_10.14deepin5-spark1_amd64.deb
```

## 安装结果

### 成功安装的软件包
- deepin-wine10-stable (10.14deepin5-spark1)
- deepin-elf-verify (1.2.0.6-1)
- libssl1.1 (1.1.1w-0+deb11u1)

### 版本验证
```bash
deepin-wine10-stable --version
```
输出：`wine-10.14`

## 使用方法

### 运行 Windows 程序
```bash
deepin-wine10-stable program.exe
```

## 下载文件清单
以下文件可以删除（安装后不再需要）：
- deepin-wine10-stable_10.14deepin5-spark1_amd64.deb
- deepin-elf-verify_1.2.0.6-1_amd64.deb
- libssl1.1_1.1.1w-0+deb11u1_amd64.deb

## 参考资源
- CSDN 博客：https://blog.csdn.net/weixin_46244623/article/details/157652851
- 华中科技大学镜像站：https://mirrors.sdu.edu.cn/
- 华为云镜像站：https://mirrors.huaweicloud.com/
