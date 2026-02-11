# Deepin Wine10 图标显示问题解决方案
> [!NOTE]
> ai在解决问题后，总结生成

## 问题描述

在 Linux 系统上使用 deepin-wine10 运行 Windows 应用程序时，应用中的图标被替换成了无意义的汉字或者占位符，无法正常显示。

## 问题原因

这个问题通常由以下原因造成：
1. Wine 环境中缺少微软核心字体
2. 字体渲染配置不正确
3. DirectDraw 渲染模式设置不当

## 解决方案

### 1. 安装 winetricks 工具

winetricks 是一个用于下载和安装 Windows 应用程序库的辅助工具。

```bash
# 下载 winetricks
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks -O /tmp/winetricks

# 安装到系统
pkexec install -m 755 /tmp/winetricks /usr/local/bin/winetricks

# 验证安装
winetricks --version
```

### 2. 安装 cabextract 工具

cabextract 用于解压 .cab 格式的微软字体安装包。

```bash
pkexec apt install -y cabextract
```

### 3. 安装微软核心字体

使用 winetricks 安装微软核心字体包，包括 Arial、Times New Roman、Courier New、Georgia、Comic Sans 等常用字体。

```bash
WINEPREFIX=~/.deepinwine winetricks corefonts
```

安装的字体包括：
- Arial 系列（常规、粗体、斜体、粗斜体、Black）
- Times New Roman 系列
- Courier New 系列
- Georgia 系列
- Comic Sans 系列
- Impact
- Trebuchet MS 系列
- Verdana 系列
- Andale Mono

### 4. 配置字体平滑

启用 RGB 子像素抗锯齿，使字体显示更清晰。

```bash
WINEPREFIX=~/.deepinwine winetricks fontsmooth=rgb
```

### 5. 配置渲染模式

使用 GDI 渲染器，能更好地处理字体和图标显示。

```bash
WINEPREFIX=~/.deepinwine winetricks renderer=gdi
```

或者使用旧命令（已弃用但仍然有效）：

```bash
WINEPREFIX=~/.deepinwine winetricks ddr=gdi
```

## 验证安装

查看已安装的字体：

```bash
ls -la ~/.deepinwine/drive_c/windows/Fonts/
```

## 使用方法

使用 deepin-wine10 运行 Windows 程序：

```bash
deepin-wine10-stable program.exe
```

## 故障排除

如果问题仍然存在，可以尝试重新安装（不卸载）对应的应用


## 环境信息

- 系统：Linux 6.18.5+deb14-amd64
- Wine 版本：deepin-wine10-stable 10.14
- winetricks 版本：20260125-next

## 参考资料

- [Winetricks 官方文档](https://github.com/Winetricks/winetricks)
- [Deepin Wine10 安装记录](deepin-wine10安装记录.md)
