## Git Proxy 

- `git config --global http.proxy` → **未设置**
- `git config --global https.proxy` → **未设置**
- 本地仓库配置 → **未设置**
- 所有配置（包括 system/global/local）中包含 `proxy` 的项 → **无**

### 如何查看/设置 Git Proxy

1. **查看所有代理相关配置**（推荐）：
   ```bash
   git config --list | grep -i proxy
   ```

2. **设置 HTTP/HTTPS 代理**（例如使用 Clash 或其他代理）：
   ```bash
   # 设置代理（把 127.0.0.1:7890 换成你的实际代理地址）
   git config --global http.proxy http://127.0.0.1:7890
   git config --global https.proxy http://127.0.0.1:7890
   ```

3. **取消代理**：
   ```bash
   git config --global --unset http.proxy
   git config --global --unset https.proxy
   ```

需要我帮你设置某个具体的代理吗？告诉我代理地址即可。