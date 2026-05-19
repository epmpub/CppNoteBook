 

Step 1: Generate SSH Keys (Windows 11 Client)

1. Open PowerShell or Command Prompt on your Windows 11 machine.

2. Generate a key pair by running:

   powershell

   ```
   ssh-keygen -t ed25519
   ```

   请谨慎使用此类代码。

3. Press **Enter** to accept the default location (`C:\Users\YourUsername\.ssh\id_ed25519`).

Step 2: Configure the Server (Windows Server 2022)

1. Open PowerShell as an **Administrator** on Windows Server 2022.

2. Navigate to the OpenSSH directory:

   powershell

   ```
   cd C:\ProgramData\ssh
   ```

   请谨慎使用此类代码。

   

3. Depending on the user account you are logging into:

   - **For Standard Users:** Create a file named `authorized_keys` in `C:\Users\TargetUsername\.ssh\authorized_keys`.
   - **For Administrators:** Paste your public key into `C:\ProgramData\ssh\administrators_authorized_keys`.

4. Restrict permissions so only the correct users can access the key files. This is a strict security requirement for Windows OpenSSH:

   powershell

   ```
   # For administrators_authorized_keys:
   icacls.exe "C:\ProgramData\ssh\administrators_authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
   ```

   请谨慎使用此类代码。

   

   

   ===========================OK===========================================================================================

    

Step 3: Verify Settings and Connect

1. Ensure `sshd_config` allows key-based authentication. Open `C:\ProgramData\ssh\sshd_config` and verify these lines are uncommented and set correctly:

   text

   ```
   PubkeyAuthentication yes
   ```

   请谨慎使用此类代码。

   

2. Restart the OpenSSH SSH Server service:

   powershell

   ```
   Restart-Service sshd
   ```

   请谨慎使用此类代码。

   

3. Connect from your Windows 11 machine using the private key:

   powershell

   ```
   ssh username@ServerIP -i "C:\Users\YourUsername\.ssh\id_ed25519"
   ```

   请谨慎使用此类代码。