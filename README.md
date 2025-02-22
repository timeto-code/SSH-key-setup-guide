# SSH 密钥生成与管理

## Windows

参考：微软官网 [文档](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_keymanagement)。

1. 使用 PowerShell 生成 SSH 密钥。

    ```bash
    # 生成 SSH 密钥
    ssh-keygen -t ed25519 -C "your_email@example.com"

    # 设置 SSH 密钥保存路径和名称，直接回车使用默认路径和名称。
    # 默认路径一般为用户目录下的 .ssh 文件夹，如：C:\Users\you\.ssh\id_ed25519
    > Enter file in which to save the key (C:\Users\you/.ssh/id_ed25519): 

    # 输入密钥密码，如果不需要密码，直接回车。
    # 推荐设置密码，以增加密钥的安全性。并且可以使用 ssh-agent 来管理密钥以及密码，避免重复输入。
    > Enter passphrase (empty for no passphrase): 
    ```

    或者

    ```bash
    # 如果您要连接的主机系统不支持新算法（如：Ed25519 ）的密钥，可以使用兼容性更好的算法，如 RSA。
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

1. 启动 ssh-agent 服务。

    1. 检查 ssh-agent 服务是否正在运行。

        ```bash
        # PowerShell 执行以下命令
        Get-Service ssh-agent

        # 打印结果如下。Status 为 Running 表示服务正在运行。应该不是首次配置 ssh-agent。可以跳过第 2 步。
        # 如果是第一次配置 ssh-agent，那么 Status 应该为 Stopped。此时安装步骤 2 执行即可。
        Status   Name               DisplayName
        ------   ----               -----------
        Running  ssh-agent          OpenSSH Authentication Agent
        ```

    1. 配置 ssh-agent 服务开机自动启动。

        ```bash
        # 默认情况下是当执行第 1 步操作时，Status 应该为 Stopped。
        # 所以需要手动启动，并且设置开机时自动启动。

        # 以下步骤需要以管理员身份运行 PowerShell。
        # 以下步骤需要以管理员身份运行 PowerShell。
        # 以下步骤需要以管理员身份运行 PowerShell。

        # 设置 ssh-agent 服务开机时自动启动
        Get-Service ssh-agent | Set-Service -StartupType Automatic

        # 启动 ssh-agent 服务
        Start-Service ssh-agent

        # 检查 ssh-agent 状态，此时应该为 Running。
        Get-Service ssh-agent
        ```

    1. 添加密钥到 ssh-agent。

        ```bash
        # 此步骤在普通用户模式下的 PowerShell 中执行。
        # 因为在管理员模式下的 PowerShell 可能不支持密钥密码中的特殊字符，会一直提示密码错误。
        # 如果没有特殊字符，那么继续在管理员模式下的 PowerShell 中执行也可以。
        ssh-add 'C:\Users\you\.ssh\id_ed25519'
        ```

1. 获取并分发公钥。

    - 在私钥文件的同一目录下，会生成一个同名的 `.pub` 结尾的文件，这个文件就是公钥文件。
    - 使用文本编辑器打开公钥文件，即可获取到公钥内容。

1. （可选）配置 SSH config 文件。

    ```bash
    # 在 .ssh 文件夹下创建 config 文件，如果没有的话。

    # 编辑 config 文件，添加如下内容：
    Host github.com
        HostName github.com
        User root
        IdentityFile C:\Users\you\.ssh\id_ed25519

    Host my-server
        HostName 20.47.432.123
        User root
        IdentityFile C:\Users\you\.ssh\id_ed25519

    # Host：主机别名，可以自定义。
    # HostName：主机 IP 或者域名。
    # User：登录用户名。
    # IdentityFile：私钥文件路径。如果只有一个密钥，可以省略 IdentityFile。多个密钥时，需要把目标主机所持公钥对应的私钥路径写在这里。

    # 保存后可以使用 ssh -T my-server 命令来测试连接。
    ```

## Mac