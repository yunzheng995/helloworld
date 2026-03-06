#### OpenClaw 如何配置连接 GitHub

这篇文档记录的是在 `OpenClaw` 所在服务器上，为 `GitHub CLI (gh)` 完成安装、登录与验证的实际流程。配好之后，服务器就可以直接用你的 `GitHub` 账号去访问仓库、查看 `Issue/PR`、拉代码、提交改动等。

---

#### 一、先确认是否安装了 `gh`

在服务器里执行：

```bash
gh --version
```

如果能看到版本号，说明已经安装。
如果提示 `command not found`，说明还没装，需要先安装 `GitHub CLI`。

这次实际采用的是直接下载安装官方二进制的方式，因为系统仓库里没有现成的 `gh` 包。

---

#### 二、安装 `GitHub CLI`

如果系统软件源里装不到，可以直接下载官方发布包并放到 `/usr/local/bin/gh`。

参考命令：

```bash
arch=$(uname -m)
case "$arch" in
  x86_64) arch=amd64 ;;
  aarch64) arch=arm64 ;;
  *) echo "Unsupported arch: $arch" >&2; exit 1 ;;
esac

version=$(curl -fsSL https://api.github.com/repos/cli/cli/releases/latest | sed -n 's/.*"tag_name": "v\([^"]*\)".*/\1/p' | head -n1)
url="https://github.com/cli/cli/releases/download/v${version}/gh_${version}_linux_${arch}.tar.gz"

cd /tmp
curl -fL "$url" -o gh.tgz
tar -xzf gh.tgz
install -m 0755 gh_${version}_linux_${arch}/bin/gh /usr/local/bin/gh
```

安装完成后再执行：

```bash
gh --version
```

确认命令已经可用。

---

#### 三、登录 GitHub 账号

在 **安装了 `gh` 的那台服务器上** 执行：

```bash
gh auth login
```

按下面选项走：

1. `Where do you use GitHub?` → 选 `GitHub.com`
2. `What is your preferred protocol for Git operations?` → 选 `HTTPS`
3. `Authenticate Git with your GitHub credentials?` → 选 `Yes`
4. 登录方式 → 选 `Login with a web browser`

然后 `gh` 会输出两样东西：

- 一个一次性验证码
- 一个登录网址，通常是：`https://github.com/login/device`

接下来在浏览器里：

1. 打开这个网址
2. 登录你的 `GitHub` 账号
3. 输入验证码
4. 确认授权

授权完成后，服务器上的 `gh` 就会获得你的账号访问能力。

---

#### 四、验证是否登录成功

执行：

```bash
gh auth status
```

如果登录成功，会看到类似信息：

```bash
github.com
  ✓ Logged in to github.com account <你的用户名>
  - Active account: true
  - Git operations protocol: https
```

这次实际验证结果是：

- 已登录账号：`yunzheng995`
- Host：`github.com`
- Git 协议：`https`
- Token scopes：`gist`, `read:org`, `repo`, `workflow`

说明已经可以正常访问 GitHub，并且具备常见仓库操作能力。

---

#### 五、登录成功后可以做什么

配置完成后，可以直接在服务器上做这些事：

- 查看仓库：`gh repo view owner/repo`
- 克隆仓库：`gh repo clone owner/repo`
- 查看 Issue：`gh issue list --repo owner/repo`
- 查看 PR：`gh pr list --repo owner/repo`
- 查看 Actions：`gh run list --repo owner/repo`
- 创建仓库：`gh repo create owner/repo`

如果是私有仓库，只要当前账号有权限，也可以直接操作。

---

#### 六、这次的本地约定

对于 `yunzheng995` 名下的 GitHub 项目，统一放在：

```bash
/root/github.com/yunzheng995
```

例如：

```bash
/root/github.com/yunzheng995/helloworld
```

这样目录结构会比较整齐，后面找项目不会乱。

---

#### 七、常见问题

**1. 为什么要在服务器上执行 `gh auth login`？**
因为授权信息保存在当前机器上。哪台机器要用 `gh` 操作 GitHub，就在哪台机器上登录。

**2. 登录后是不是就能访问所有仓库？**
不是。只能访问当前 GitHub 账号本来就有权限访问的仓库。

**3. 如果换账号怎么办？**
可以重新登录，或者先退出当前登录后再切换账号。

---

#### 八、结论

如果目标是在 `OpenClaw` 所在服务器上连接 GitHub，最直接的路径就是：

- 安装 `gh`
- 执行 `gh auth login`
- 用浏览器完成授权
- 用 `gh auth status` 验证

走完这几步，服务器就具备了用 GitHub 账号进行自动化操作的能力。