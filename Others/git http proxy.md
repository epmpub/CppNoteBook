## Git HTTP/HTTPS proxy 

on Windows 11, use these steps:1. Basic Proxy Configuration (Most Common)Open Command Prompt, PowerShell, or Git Bash as a normal user and run:

bash

```bash
# For HTTP and HTTPS (replace with your actual proxy details)
git config --global http.proxy http://proxy.example.com:8080
git config --global https.proxy http://proxy.example.com:8080
```

- If your proxy requires authentication:

  bash

  ```bash
  git config --global http.proxy http://username:password@proxy.example.com:8080
  git config --global https.proxy http://username:password@proxy.example.com:8080
  ```

- Recommended for Windows (Avoid Plain-Text Password)For corporate proxies (especially NTLM/Basic auth), use the Windows Credential Manager:

bash

```bash
git config --global http.proxy http://username@proxy.example.com:8080
git config --global https.proxy http://username@proxy.example.com:8080

# Use Windows Credential Manager
git config --global credential.helper wincred
```

Git will prompt for the password the first time and store it securely.3. Verify the Settings

bash

```bash
git config --global --get http.proxy
git config --global --get https.proxy
git config --global --list | grep proxy
```

4. Other Useful CommandsUnset/remove proxy (if you no longer need it):

bash

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

Bypass proxy for specific domains (e.g., localhost or internal sites):

bash

```bash
git config --global http.noProxy "localhost,127.0.0.1,example.com,.internal"
```

Disable SSL verification (only if you have certificate issues — not recommended for security):

bash

```bash
git config --global http.sslVerify false
```

5. Alternative: Environment Variables (System-wide)You can also set them via Windows Environment Variables:

1. Search for "Environment Variables" in Windows Settings.
2. Under User variables, add:
   - HTTP_PROXY = http://proxy.example.com:8080
   - HTTPS_PROXY = http://proxy.example.com:8080

Common Proxy Formats

- http://proxy.company.com:8080
- http://username:password@proxy.company.com:8080
- For SOCKS5: socks5://proxy.company.com:1080

Tip: On many corporate networks, the proxy address is the same as your browser’s proxy (check Internet Options → Connections → LAN settings).Let me know your proxy type (e.g., with username/password, NTLM, or specific port) if you run into errors!