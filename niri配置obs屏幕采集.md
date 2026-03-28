# Niri 下 OBS 屏幕采集配置
> **AI 声明**：本文档由 AI 助手在与用户、系统的实际交互过程中撰写，记录了真实的配置步骤、遇到的问题及其解决方案。所有内容均有现实的操作和系统反馈作为依据，非虚构内容。
## 前提条件

- Wayland Compositor：Niri
- 已安装：PipeWire、WirePlumber

## 前情提要
最初，为了在 Niri 环境下配置 OBS 屏幕采集，经历了一番波折：  
- xdg-desktop-portal-wlr，结果因为配置了 slurp选择器导致鼠标无法使用，不得不删除配置回退
- xdg-desktop-portal-gtk，却发现它在 Niri环境下根本不支持屏幕捕获
- xdg-desktop-portal-gnome 能够完美工作，但代价是安装了约 200 MB 的 GNOME相关依赖
- 深入研究发现xdg-desktop-portal-wlr 的问题其实不是后端本身，而是配置了 slurp选择器！
 
[历史版本](https://github.com/alu-deak/blog/blob/84653e65616ce2cf61ebaa54e3ca6a42bb9ff799/niri%E9%85%8D%E7%BD%AEobs%E5%B1%8F%E5%B9%95%E9%87%87%E9%9B%86.md)  
[Diff](https://github.com/alu-deak/blog/commit/f102c358ecc1ea737758d727a9af1b4f60bcba5e)

## 配置步骤

### 1. 安装依赖

```bash
pkexec apt install --no-install-recommends obs-studio xdg-desktop-portal-wlr
```

**说明**：`--no-install-recommends` 参数可以避免安装推荐包（如 slurp、wofi、bemenu 等），只安装核心组件。

### 2. 配置 portals.conf

创建 `~/.config/xdg-desktop-portal/portals.conf`：

```ini
[preferred]
default=wlr;gtk
org.freedesktop.impl.portal.Screenshot=wlr
org.freedesktop.impl.portal.ScreenCast=wlr
```

**重要**：
- ✅ 只创建 `portals.conf`
- ❌ **不要创建** `~/.config/xdg-desktop-portal-wlr/config`
- ❌ **不要配置** slurp 选择器

### 3. 重启服务

```bash
systemctl --user restart xdg-desktop-portal.service
```

### 4. 验证配置

```bash
ps aux | grep xdg-desktop-portal | grep -v grep
```

应该看到 xdg-desktop-portal 和 xdg-desktop-portal-wlr 进程。

## 使用 OBS 进行屏幕采集

1. 启动 OBS Studio
2. 在"来源"区域点击 "+" 按钮
3. 选择"屏幕/窗口捕获（PipeWire）"
4. 在弹出的对话框中选择要捕获的屏幕或窗口

## 其他方案说明

- **xdg-desktop-portal-wlr + slurp**：会导致鼠标无法使用。原因：slurp 会持续等待用户选择区域，xdg-desktop-portal-wlr 临时获取输入设备访问权以支持选择功能，slurp 进入等待状态会拦截输入事件
- **xdg-desktop-portal-gtk**：不支持屏幕捕获
- **xdg-desktop-portal-gnome**：功能正常但依赖过多（约 200 MB）

## 重要提示

- Niri 需要通过 `niri-session` 或 display manager 启动以支持完整的 portal 功能
- 确保 `WAYLAND_DISPLAY` 和 `XDG_CURRENT_DESKTOP` 环境变量正确设置

## 故障排查

### OBS 找不到屏幕捕获选项

1. 确认 xdg-desktop-portal-wlr 服务正在运行
2. 检查 portals.conf 配置是否正确
3. 重启 xdg-desktop-portal 服务

### 鼠标无法使用

```bash
systemctl --user stop xdg-desktop-portal-wlr.service
rm ~/.config/xdg-desktop-portal-wlr/config
```
