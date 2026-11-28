# 纸鸢 Web

[English](README.md) | 简体中文

**编程 Agent 在你的电脑上运行。你可以随时跟进，需要时接手。**

纸鸢的英文名是 Shayan（/ˈʃɑːjɛn/，读作 SHAH-yen），取自北京的沙燕风筝。

Web 提供纸鸢的浏览器界面：一组静态文件，由服务器上的 Caddy 在 Web 自己的域名下提供；网页通过 Relay 连接电脑上的 `shayand`。Web 和 Relay 分开安装（[Relay 说明](../relay/README.zh-CN.md)），可以在同一台服务器上，也可以在不同的服务器上。Web 使用自己的安装包运行：`shayan-web-<版本>-any.tar.gz`，经过签名，发布在 [`kotaoching/shayan-releases`](https://github.com/kotaoching/shayan-releases) 的发行版中。

## 系统要求

- 一个域名，A 记录（有 IPv6 时再加 AAAA 记录）指向这台服务器；开放入站 TCP 80 和 443 端口。
- Debian 12 或 13，或 Ubuntu 24.04 LTS 或 26.04 LTS，x86_64 或 arm64；root 或可以使用 `sudo` 的用户。
- `sudo apt install caddy curl`。如果同时安装了 `minisign`，每次安装和升级都会用发布公钥验证安装包的签名；没有安装时只依赖 HTTPS。Caddy 作为系统服务运行，配置文件是 `/etc/caddy/Caddyfile`。
- `/etc/caddy/Caddyfile` 中有下面这一行（安装前加一次；即使还没有任何站点文件也不会出错）：

  ```
  import sites/*.caddy
  ```

  安装程序不会修改 `/etc/caddy/Caddyfile`，只写入 `/etc/caddy/sites/shayan-web.caddy`。缺少这一行时，安装程序会停止并给出提示。

## 安装

```sh
curl -fsSL https://github.com/kotaoching/shayan-releases/releases/download/web/install.sh | sudo sh -s -- --domain <Web 域名> [--dir <目录>]
```

`<目录>` 默认为 `/srv/shayan-web`。安装指定版本（包括尚未设为最新版本的）：

```sh
curl -fsSL https://github.com/kotaoching/shayan-releases/releases/download/web-v<版本>/install.sh | sudo sh -s -- --version <版本> --domain <Web 域名>
```

这个脚本会下载该版本的安装包，在装有 `minisign` 时用脚本内置的发布公钥验证签名，解压后运行安装包里的 `install.sh`。安装完成后会输出：安装的版本、配置文件、Caddy 是否已在提供站点、查看日志的命令和回滚的命令，以及从这台服务器能否访问 `https://<域名>/`（DNS 或证书尚未就绪时可能无法访问，这不算安装失败）。

## 升级

```sh
sudo /srv/shayan-web/current/fetch.sh                      # 升级到最新版本（比已安装的新时才安装）
sudo /srv/shayan-web/current/fetch.sh --version <版本>      # 安装指定版本，也可以是更旧的版本
```

正常升级只切换 `current`，站点配置不变。安装程序仍会把配置交给 Caddy（`caddy reload`，不带 `--force`）；配置没有变化时 Caddy 不会重新加载，同一台 Caddy 后面的 Relay 的 WebSocket 连接也不会断开。下一个请求拿到的就是新版本。已经打开的页面继续使用已加载的内容，页面上会显示"纸鸢新版本已就绪。"，点击"重新加载"即可切换。

## 从 Patchbay 0.6.1 换过来

纸鸢到 0.6.1 为止叫 Patchbay。Patchbay 不会自己升级成新名字，纸鸢也不读 Patchbay 的数据，所以要手工换一次：

1. 照下面的「卸载」一节卸掉 Patchbay 的 Web，名字里的 `shayan` 换成 `patchbay`（`/etc/caddy/sites/patchbay-web.caddy`、`/srv/patchbay-web`、`/etc/patchbay-web`）。
2. 如果 Web 换到新域名，先把域名指向这台服务器。
3. 照上面的「安装」一节装上纸鸢的 Web。手机打开新页面后重新配对；新页面不读旧页面存下的东西。

## 配置

配置文件是 `/etc/shayan-web/web.env`，每行一个 `KEY=值`。首次安装根据选项生成，之后由你自己修改。修改后再运行一次 `sudo <DIR>/current/install.sh`。配置文件存在后，传给安装程序的选项不会生效，安装程序会给出提示。缺少 `DIR` 或 `DOMAIN` 时，安装程序会停止并说明需要添加什么。

| 键 | 含义 |
| --- | --- |
| `DIR` | 安装目录，绝对路径；默认为 `/srv/shayan-web` |
| `DOMAIN` | Web 的域名，手机打开 `https://<DOMAIN>` |
| `UPDATE_SOURCE` | 升级时获取版本的位置：`https://github.com/kotaoching/shayan-releases/releases/download/web`，或本机上布局相同的目录 |

- 更换域名：修改 `DOMAIN` 后重新运行安装程序。站点会按新域名重写，先在 `/etc/caddy` 的副本上验证，再替换并重新加载。手机上保存的数据都属于原来的地址，所以每部手机都要打开新地址、重新添加到主屏幕并重新配对。
- 更换目录：`sudo cp -a <原目录> <新目录>`，修改 `DIR`，运行 `sudo <新目录>/current/install.sh`，再执行 `sudo rm -r <原目录>`。

## 回滚

服务器上会保留被替换的上一个版本，回滚不需要重新下载：

```sh
sudo /srv/shayan-web/versions/<上一个版本>/install.sh
```

更早的版本可以按版本号安装（见"升级"中的第二条命令）。每个版本都会一直保留在发行版中。

## 卸载

```sh
sudo rm /etc/caddy/sites/shayan-web.caddy && sudo systemctl reload caddy
sudo rm -r /srv/shayan-web /etc/shayan-web
```

## 故障排查

- 日志：`journalctl -u caddy`。查看 Caddy 是否在运行：`systemctl status caddy`。
- 安装程序停止并提示 "needs the line 'import sites/*.caddy'"：在 `/etc/caddy/Caddyfile` 中加上这一行后重新运行。
- 安装失败（Caddy 没有运行，或重新加载失败）：安装程序以状态 1 退出，并输出失败的步骤、查看日志的命令和回滚的命令。它不会自动切回旧版本。
- HTTPS 无法访问：检查域名的 A 记录、80 和 443 端口，以及 Caddy 的证书（见日志）。
- 手机提示 `protocol`：Web 和 Relay 使用的 Relay 协议版本不同，把两者都升级到最新版本。

## 在手机上

打开 `https://<Web 域名>`，添加到主屏幕，然后在其中扫描电脑上 `shayan devices pair` 显示的配对码。
