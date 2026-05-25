# GitHub Copilot CLI API配置教程（Windows / Mac / Linux）
---

这份教程，目标是在本地利用 Github 官方提供的 `BYOK` 功能把自己的API模型配置到 Copilot CLI 中：

本教程以下面两个公司的产品为例
- DeepSeek
- Qwen

> 安全提示：如果你曾经把 API Key 明文写到终端或对话里，请先去对应平台重新生成新的 Key，再把旧 Key 作废。

---

## 第 1 步：准备目录

### Windows（PowerShell）
```powershell
New-Item -ItemType Directory -Force "$HOME\.copilot" | Out-Null
New-Item -ItemType Directory -Force "$HOME\bin" | Out-Null
```

### Mac（zsh / bash）
```bash
mkdir -p ~/.copilot ~/bin
```

### Linux（bash / zsh）
```bash
mkdir -p ~/.copilot ~/bin
```

---

## 第 2 步：创建 DeepSeek/Qwen 配置文件

### Windows（PowerShell）

```powershell
@'
COPILOT_PROVIDER_TYPE=openai
COPILOT_PROVIDER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
COPILOT_PROVIDER_API_KEY=你的QwenKey
COPILOT_MODEL=qwen3.6-plus
COPILOT_PROVIDER_MAX_PROMPT_TOKENS=840000
COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000
'@ | Set-Content -Encoding UTF8 "$HOME\.copilot\deepseek.env (或者qwen.env)"
```

```powershell
@'
COPILOT_PROVIDER_TYPE=anthropic
COPILOT_PROVIDER_BASE_URL=https://api.deepseek.com/anthropic
COPILOT_PROVIDER_API_KEY=你的DeepSeekKey
COPILOT_MODEL=deepseek-v4-pro
COPILOT_PROVIDER_MAX_PROMPT_TOKENS=840000
COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000
'@ | Set-Content -Encoding UTF8 "$HOME\.copilot\deepseek.env (或者qwen.env)"
```

### Mac/Linux（zsh / bash）

```bash
cat > ~/.copilot/qwen.env <<'EOF'
COPILOT_PROVIDER_TYPE=openai
COPILOT_PROVIDER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
COPILOT_PROVIDER_API_KEY=你的QwenKey
COPILOT_MODEL=qwen3.6-plus
COPILOT_PROVIDER_MAX_PROMPT_TOKENS=840000
COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000
EOF
```

```bash
cat > ~/.copilot/deepseek.env <<'EOF'
COPILOT_PROVIDER_TYPE=anthropic
COPILOT_PROVIDER_BASE_URL=https://api.deepseek.com/anthropic
COPILOT_PROVIDER_API_KEY=你的DeepSeekKey
COPILOT_MODEL=deepseek-v4-pro
COPILOT_PROVIDER_MAX_PROMPT_TOKENS=840000
COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000
EOF
```

### 说明

- `COPILOT_PROVIDER_TYPE=openai`：Qwen 使用 OpenAI-compatible 接口
- `COPILOT_PROVIDER_TYPE=anthropic`：DeepSeek 使用 Anthropic 接口（在deepseek-v4之后，openai的结构会发生400错误，所以需要使用 Anthropic 接口）
- `COPILOT_PROVIDER_BASE_URL`：DeepSeek 和 Qwen 的接口地址
- `COPILOT_PROVIDER_API_KEY`：你的 DeepSeek 或 Qwen Key
- `COPILOT_MODEL=deepseek-v4-pro/qwen3.6-plus`：模型选择
- `COPILOT_PROVIDER_MAX_PROMPT_TOKENS=840000`：上下文最大长度
- `COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000`：输出最大长度

> 如果 Copilot 提示 `Model ... is not in the built-in catalog`，通常不是错误，只是 Copilot 不认识这个模型名，无法自动推断 token 上限；手动配置即可。

---

## 第 3 步：创建启动脚本

### Windows（PowerShell）

```powershell
@'
Get-Content "$HOME\.copilot\deepseek.env (或者 qwen.env)" | ForEach-Object {
  if ($_ -match '^\s*([^#=]+?)=(.*)$') {
    [Environment]::SetEnvironmentVariable($matches[1], $matches[2], "Process")
  }
}
copilot
'@ | Set-Content -Encoding UTF8 "$HOME\bin\copilot-deepseek.ps1 (copilot-qwen.ps1)"
```

### Mac（zsh / bash）已验证

```bash
cat > ~/bin/copilot-deepseek (或者 copilot-qwen) <<'EOF'
#!/bin/zsh
set -a
source ~/.copilot/deepseek.env (或者 qwen.env)
set +a
exec copilot
EOF
chmod +x ~/bin/copilot-deepseek (或者~/bin/copilot-qwen)
```

### Linux（bash / zsh）

```bash
cat > ~/bin/copilot-deepseek (或者 copilot-qwen) <<'EOF'
#!/bin/bash
set -a
source ~/.copilot/deepseek.env (或者 qwen.env)
set +a
exec copilot
EOF
chmod +x ~/bin/copilot-deepseek (或者 ~/bin/copilot-qwen)
```

---

## 第 4 步：把脚本目录加入 PATH

### Windows（PowerShell）

```powershell
$old = [Environment]::GetEnvironmentVariable("Path", "User")
if ($old -notlike "*$HOME\bin*") {
  [Environment]::SetEnvironmentVariable("Path", "$old;$HOME\bin", "User")
}
$env:Path = "$HOME\bin;$env:Path"
```

### Mac（zsh）

```bash
grep -q 'export PATH="$HOME/bin:$PATH"' ~/.zshrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Linux（bash / zsh）

```bash
grep -q 'export PATH="$HOME/bin:$PATH"' ~/.bashrc || echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

> 如果你在 Linux 上用的是 zsh，把 `.bashrc` 改成 `.zshrc`。

---

## 第 7 步：运行 Copilot

### Windows（PowerShell）

使用 DeepSeek：

```powershell
& "$HOME\bin\copilot-deepseek.ps1"
& "$HOME\bin\copilot-qwen.ps1"
```

### Mac/Linux（zsh / bash）

```bash
copilot-deepseek
copilot-qwen
```