# Niri 下 OBS 屏幕采集配置记录

> **AI 声明**：本文档由 AI 助手在与用户、系统的实际交互过程中撰写，记录了真实的配置步骤、遇到的问题及其解决方案。所有内容均有现实的操作和系统反馈作为依据，非虚构内容。

---

## 前提条件

- 操作系统：Debian Testing (Linux 6.18.5+deb14-amd64)
- Wayland Compositor：Niri
- 已安装：PipeWire、WirePlumber

## 目标

在 Niri 环境下配置 OBS Studio，使其能够进行屏幕采集。

---

## 配置建议

### 设置 apt 不安装推荐包

**强烈建议**在开始安装前，先配置 apt 只安装必要依赖，避免安装大量不必要的软件包。

创建配置文件：

```bash
pkexec bash -c 'echo "APT::Install-Recommends \"0\";" > /etc/apt/apt.conf.d/00no-recommends && echo "APT::Install-Suggests \"0\";" >> /etc/apt/apt.conf.d/00no-recommends'
```

验证配置：

```bash
apt-config dump | grep -E "Install-Recommends|Install-Suggests"
```

**说明**：
- 此配置将使 `apt install` 默认只安装必要依赖
- 如需安装推荐包，可以使用 `--install-recommends` 参数：
  ```bash
  apt install --install-recommends <package>
  ```

**注意事项**：
如果在设置此配置前已经安装了包，后续的安装步骤可能仍然会安装推荐依赖。配置完成后，可以使用 `apt autoremove` 清理不必要的依赖包。

---

## 配置过程

### 第一步：安装 OBS 和基础依赖

```bash
pkexec apt install xdg-desktop-portal-wlr obs-studio
```

已安装的组件：
- OBS Studio 30.2.3
- xdg-desktop-portal-wlr 0.8.1
- PipeWire 1.4.10（运行中）
- WirePlumber 0.5.13（运行中）

### 第二步：初次配置尝试（失败）

创建配置文件：

**`~/.config/xdg-desktop-portal/niri-portals.conf`**：
```ini
[preferred]
default=wlr;gtk
org.freedesktop.impl.portal.Screenshot=wlr
org.freedesktop.impl.portal.ScreenCast=wlr
```

**`~/.config/xdg-desktop-portal-wlr/config`**：
```ini
[screencast]
chooser_type=simple
chooser_cmd=slurp -f "%o"
```

**问题**：配置完成后鼠标无法使用。

#### 故障原理分析

`xdg-desktop-portal-wlr` 在处理屏幕共享时会临时获取对输入设备的访问权来支持选择功能。配置中的 `slurp` 选择器可能进入了持续等待状态，导致输入事件被拦截。

### 第三步：恢复鼠标功能

删除配置文件并停止服务：

```bash
rm ~/.config/xdg-desktop-portal/niri-portals.conf
rm ~/.config/xdg-desktop-portal-wlr/config
systemctl --user stop xdg-desktop-portal-wlr.service
```

### 第四步：尝试使用 xdg-desktop-portal-gtk（失败）

创建配置文件：

**`~/.config/xdg-desktop-portal/portals.conf`**：
```ini
[preferred]
default=gtk
org.freedesktop.impl.portal.Screenshot=gtk
org.freedesktop.impl.portal.ScreenCast=gtk
```

**问题**：OBS 仍然无法使用屏幕捕获。gtk 后端主要针对 GNOME 设计，在 niri (wlroots) 环境下不支持屏幕捕获。

### 第五步：最终解决方案 - 使用 xdg-desktop-portal-gnome

根据 Niri 官方文档和研究，Niri 推荐使用 `xdg-desktop-portal-gnome` 进行屏幕捕获。

#### 1. 安装 xdg-desktop-portal-gnome

```bash
pkexec apt install xdg-desktop-portal-gnome
```

此操作会安装大量 GNOME 相关依赖，包括 gnome-shell、gdm3 等。

#### 2. 修改 portals.conf 配置

**`~/.config/xdg-desktop-portal/portals.conf`**：
```ini
[preferred]
default=gnome;gtk;
org.freedesktop.impl.portal.Screenshot=gnome
org.freedesktop.impl.portal.ScreenCast=gnome
```

#### 3. 重启服务

```bash
systemctl --user restart xdg-desktop-portal.service
```

### 第六步：验证配置

运行以下命令检查服务状态：

```bash
ps aux | grep xdg-desktop-portal | grep -v grep
```

应该看到以下进程正在运行：
- `/usr/libexec/xdg-desktop-portal`（主服务）
- `/usr/libexec/xdg-desktop-portal-gnome`（屏幕捕获后端）
- `/usr/libexec/xdg-desktop-portal-gtk`（备用后端）

---

## 使用 OBS 进行屏幕采集

1. 启动 OBS Studio
2. 在"来源"区域点击 "+" 按钮
3. 选择"屏幕/窗口捕获（PipeWire）"
4. 在弹出的对话框中选择要捕获的屏幕或窗口

**Niri 支持的屏幕捕获方式：**
- 整个显示器
- 特定窗口
- 动态投屏（Dynamic Cast）- Niri 特有功能
- 显示器的特定区域（需要 OBS 支持）

---

## 重要提示

### Niri 启动方式要求

Niri 官方要求通过 `niri-session` 或 display manager 启动以支持完整的 portal 功能。如果您是通过其他方式启动的，可能需要调整启动方式。

### 环境变量

确保以下环境变量正确设置：
- `WAYLAND_DISPLAY` - 通常由 Wayland 会话自动设置
- `XDG_CURRENT_DESKTOP` - 某些 portal 后端需要此变量

### 配置开机启动（可选）

如果需要 xdg-desktop-portal-gnome 开机启动，可以配置 systemd：

```bash
systemctl --user enable xdg-desktop-portal-gnome.service
```

---

## 故障排查

### OBS 找不到屏幕捕获选项

1. 确认 xdg-desktop-portal-gnome 服务正在运行
2. 检查 portals.conf 配置是否正确
3. 重启 xdg-desktop-portal 服务

### 鼠标无法使用

如果再次出现鼠标问题，停止 xdg-desktop-portal-wlr 服务：

```bash
systemctl --user stop xdg-desktop-portal-wlr.service
```

### 验证 ScreenCast 功能

```bash
busctl --user introspect org.freedesktop.portal.Desktop /org/freedesktop/portal/desktop | grep ScreenCast
```

---

## 参考资源

- [Niri 官方文档 - Important Software](https://yalter.github.io/niri/Important-Software.html)
- [Screen Sharing with Niri, Including Specific Regions with OBS](https://nickjanetakis.com/blog/screen-sharing-with-niri-including-specific-regions-with-obs)
- [OBS cannot capture screen · GitHub Discussion](https://github.com/YaLTeR/niri/discussions/309)
- [XDG Desktop Portal - ArchWiki](https://wiki.archlinux.org/title/XDG_Desktop_Portal)

---

## 清理不必要的依赖

### 删除失败的方案

删除 `xdg-desktop-portal-wlr` 及其相关依赖：

```bash
pkexec apt autoremove -y xdg-desktop-portal-wlr
```

此操作会删除约 149 个软件包，释放约 279 MB 空间，包括：
- xdg-desktop-portal-wlr
- 大量 KDE Framework 6 相关包
- XFCE 组件（thunar、xfce4-panel 等）
- 其他不再需要的依赖

### 清理 xdg-desktop-portal-gnome 的推荐依赖

如果在设置 apt 不安装推荐之前已经安装了 `xdg-desktop-portal-gnome`，可能已经安装了大量 GNOME 相关依赖，包括：
- gnome-shell
- gdm3
- gnome-settings-daemon
- gnome-control-center
- papers
- gnome-browser-connector

这些包可以作为"孤儿包"自动删除：

```bash
pkexec apt-get autoremove
```

### 验证清理结果

检查保留的关键包：

```bash
dpkg -l | grep -E "xdg-desktop-portal|obs" | grep ^ii
```

应该看到以下包仍已安装：
- obs-studio、obs-plugins、libobs0t64
- xdg-desktop-portal-gnome（屏幕捕获后端）
- xdg-desktop-portal-gtk（备用后端）
- xdg-desktop-portal（主服务）

检查服务运行状态：

```bash
ps aux | grep xdg-desktop-portal | grep -v grep
```

应该看到以下进程正在运行：
- `/usr/libexec/xdg-desktop-portal`
- `/usr/libexec/xdg-desktop-portal-gnome`
- `/usr/libexec/xdg-desktop-portal-gtk`

### 注意事项

以下 GNOME 相关包被其他程序依赖，无法删除：
- nautilus（被 xdg-desktop-portal-gnome 依赖）
- gnome-keyring（密码管理）
- libgnome-desktop-*（库文件）
- gnome-themes-extra（被 lightdm-gtk-greeter 依赖）

---

## 总结

在 Niri 下配置 OBS 屏幕采集的关键是使用正确的 portal 后端。经过多次尝试，发现：

1. ❌ `xdg-desktop-portal-wlr` - 导致鼠标问题
2. ❌ `xdg-desktop-portal-gtk` - 不支持屏幕捕获
3. ✅ `xdg-desktop-portal-gnome` - 完美支持

最终配置仅需要：
- 安装 `obs-studio` 和 `xdg-desktop-portal-gnome`
- 配置 `~/.config/xdg-desktop-portal/portals.conf` 使用 gnome 后端
- 确保 PipeWire 服务正常运行

---

**文档创建日期**：2026年2月13日
**最后更新**：2026年2月13日