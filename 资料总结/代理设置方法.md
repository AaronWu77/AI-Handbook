# Proxy Setting Tutorial

本教程将详细介绍如何查看和修改本地代理设置，尤其是 Git 的 SSH 代理设置。我们将分别针对 Linux、Mac 和 Windows 系统提供具体步骤。

---

## 1. 查看本地代理设置

### Linux 和 Mac

1. **检查环境变量中的代理设置**：
   打开终端，运行以下命令：
   ```bash
   echo $http_proxy
   echo $https_proxy
   echo $no_proxy
   ```
   如果有设置代理，输出将显示代理地址；如果未设置，输出为空。

2. **检查 Git 的代理设置**：
   ```bash
   git config --global --get http.proxy
   git config --global --get https.proxy
   ```

3. **检查 SSH 的代理设置**：
   查看 `~/.ssh/config` 文件是否包含以下内容：
   ```
   Host *
       ProxyCommand nc -X 5 -x <代理地址>:<端口> %h %p
   ```

### Windows

1. **检查环境变量中的代理设置**：
   打开命令提示符或 PowerShell，运行以下命令：
   ```cmd
   echo %http_proxy%
   echo %https_proxy%
   ```

2. **检查 Git 的代理设置**：
   ```cmd
   git config --global --get http.proxy
   git config --global --get https.proxy
   ```

3. **检查 SSH 的代理设置**：
   查看 `C:\Users\<用户名>\.ssh\config` 文件是否包含以下内容：
   ```
   Host *
       ProxyCommand plink -proxycmd "<代理地址>:<端口>" %h %p
   ```

---

## 2. 修改代理设置

### Linux 和 Mac

#### 设置系统代理

1. **修改环境变量**：
   编辑 `~/.bashrc` 或 `~/.zshrc` 文件，添加以下内容：
   ```bash
   export http_proxy="http://127.0.0.1:7897"
   export https_proxy="http://127.0.0.1:7897"
   export no_proxy="localhost,127.0.0.1,.example.com"
   ```
   保存后运行：
   ```bash
   source ~/.bashrc
   ```

2. **为 Git 设置代理**：
   ```bash
   git config --global http.proxy http://127.0.0.1:7897
   git config --global https.proxy http://127.0.0.1:7897
   ```

3. **为 SSH 设置代理**：
   编辑 `~/.ssh/config` 文件，添加以下内容：
   ```
   Host *
       ProxyCommand nc -X 5 -x 127.0.0.1:7897 %h %p
   ```

#### 取消代理设置

- **取消环境变量代理**：
  ```bash
  unset http_proxy
  unset https_proxy
  ```

- **取消 Git 代理**：
  ```bash
  git config --global --unset http.proxy
  git config --global --unset https.proxy
  ```

- **取消 SSH 代理**：
  删除 `~/.ssh/config` 文件中的 `ProxyCommand` 配置。

### Windows

#### 设置系统代理

1. **通过环境变量设置代理**：
   打开命令提示符或 PowerShell，运行以下命令：
   ```cmd
   setx http_proxy http://<代理地址>:<端口>
   setx https_proxy http://<代理地址>:<端口>
   ```

2. **为 Git 设置代理**：
   ```cmd
   git config --global http.proxy http://<代理地址>:<端口>
   git config --global https.proxy http://<代理地址>:<端口>
   ```

3. **为 SSH 设置代理**：
   编辑 `C:\Users\<用户名>\.ssh\config` 文件，添加以下内容：
   ```
   Host *
       ProxyCommand plink -proxycmd "<代理地址>:<端口>" %h %p
   ```

#### 取消代理设置

- **取消环境变量代理**：
  ```cmd
  setx http_proxy ""
  setx https_proxy ""
  ```

- **取消 Git 代理**：
  ```cmd
  git config --global --unset http.proxy
  git config --global --unset https.proxy
  ```

- **取消 SSH 代理**：
  删除 `C:\Users\<用户名>\.ssh\config` 文件中的 `ProxyCommand` 配置。

---

## 3. 验证代理设置

1. **验证系统代理**：
   ```bash
   curl -I http://example.com
   ```
   如果返回正常的 HTTP 响应，则代理设置成功。

2. **验证 Git 代理**：
   ```bash
   git clone https://github.com/example/repo.git
   ```
   如果克隆成功，则代理设置成功。

3. **验证 SSH 代理**：
   ```bash
   ssh -T git@github.com
   ```
   如果连接成功，则代理设置成功。

---

## 4. 代理的种类和使用方法

### 代理的种类

1. **HTTP/HTTPS 代理**：
   - 用于浏览器和 HTTP 请求。
   - 示例：Squid、Nginx。

2. **SOCKS 代理**：
   - 支持 TCP 和 UDP 流量。
   - 示例：Shadowsocks。

3. **透明代理**：
   - 用户无需配置，流量自动通过代理。
   - 示例：公司网络代理。

4. **反向代理**：
   - 位于服务器端，用于负载均衡和缓存。
   - 示例：Nginx、HAProxy。

5. **VPN**：
   - 加密所有流量并通过远程服务器转发。
   - 示例：OpenVPN、WireGuard。

### 使用方法

1. **浏览器代理**：
   - 配置浏览器插件（如 SwitchyOmega）。

2. **系统代理**：
   - 配置环境变量，全局生效。

3. **应用程序代理**：
   - 单独为应用程序（如 Git、npm）配置代理。

4. **SOCKS 代理**：
   - 使用 `ssh` 创建 SOCKS 代理：
     ```bash
     ssh -D 1080 user@remote-server
     ```

5. **VPN**：
   - 安装 VPN 客户端，连接到远程服务器。

---

通过以上步骤，您可以轻松配置和管理代理设置。如果有任何问题，请随时联系！