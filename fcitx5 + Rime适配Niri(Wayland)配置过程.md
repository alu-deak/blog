# fcitx5 + Rime适配Niri(Wayland)配置过程
> [!NOTE]
> 此为我命令iflow配置输入法后，让iflow总结安装过程而得出的文章 
## 安装

```bash
pkexec apt update
pkexec apt install -y fcitx5 fcitx5-rime fcitx5-chinese-addons fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 fcitx5-frontend-qt5
```

## 配置环境变量

在 `~/.bashrc` 末尾添加：

```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export INPUT_METHOD=fcitx
export SDL_IM_MODULE=fcitx
export GLFW_IM_MODULE=ibus
```

执行 `source ~/.bashrc` 生效。

## Rime 配置

```bash
mkdir -p ~/.local/share/fcitx5/rime
```

创建 `~/.local/share/fcitx5/rime/default.custom.yaml`：

```yaml
patch:
  schema_list:
    - schema: luna_pinyin
    - schema: terra_pinyin
```

创建 `~/.local/share/fcitx5/rime/ibus_rime.custom.yaml`：

```yaml
patch:
  "style/horizontal": true
  "style/inline_preedit": true
```

## Niri + Wayland 适配

编辑 `~/.config/niri/config.kdl`，在启动配置中添加：

```kdl
// fcitx5 输入法
spawn-at-startup "fcitx5" "-d"
```

**Wayland 特别说明：**
- fcitx5 原生支持 Wayland
- GTK4/Qt6 应用可直接使用
- GTK3 应用通过 `fcitx5-frontend-gtk3` 支持
- 部分应用可能需要通过 XWayland 运行

## 启动

### 重启 Niri 或执行：

```bash
fcitx5 -d
```

## 快捷键

- `Ctrl + Space` - 切换中英文
- `Ctrl + `` - 切换输入方案
- `F4` - 打开配置界面

## 配置工具

```bash
fcitx5-config-qt
```

## 故障排除

检查 fcitx5 是否运行：
```bash
ps aux | grep fcitx5
```

查看日志：
```bash
fcitx5 -d --verbose
```

重启 fcitx5：
```bash
pkill fcitx5
fcitx5 -d
```

---

配置完成后，按 `Ctrl + Space` 即可使用中文输入法！
