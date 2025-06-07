

# MAC过滤



```C++
for /f "tokens=1 delims=, skip=1" %a in ('getmac /fo csv ^| findstr /v "18-56-80-D8*" ^| findstr /v "00-50*" ') do @echo %a
    
```



```powershell
Get-NetAdapter 
| Where-Object { $_.MacAddress -notlike "00-50-56-C0*" -and $_.MacAddress -notlike "00-15-5D*" } 
| Select-Object Name, MacAddress | Format-Table -AutoSize
```

 

```powershell
 Get-NetAdapter | Where-Object { $_.MacAddress -notlike "00-50-56-C0*" -and $_.MacAddress -notlike "00-15-5D*" }

 | Select-Object Name, @{Name='MacAddress';Expression={$_.MacAddress -replace '-', ':'}}

 | Format-Table -AutoSize
```



@{Name='MacAddress';Expression={$_.MacAddress -replace '-', ':'}}：

- 这部分定义了一个**计算属性**，用于动态创建或修改属性。
- 组成部分：
  - Name='MacAddress'：指定输出属性的名称为 MacAddress，这将是输出中的列名。
  - Expression={...}：定义一个脚本块，用于计算属性的值。
  - $_：表示管道中当前的对象（例如，Get-NetAdapter 返回的一个网络适配器对象）。
  - $_.MacAddress：访问当前对象的 MacAddress 属性，包含格式为 XX-XX-XX-XX-XX-XX 的 MAC 地址（例如，18-56-80-D8-5E-C8）。
  - -replace '-', ':'：使用 -replace 操作符，将 MacAddress 中的所有连字符 (-) 替换为冒号 (:)。例如，18-56-80-D8-5E-C8 变成 18:56:80:D8:5E:C8。



findstr的语法如下：

```notranslate
findstr [/b] [/e] [/l] [/r] [/s] [/i] [/x] [/v] [/m] [/n] [/o] [/g:file] [/f:file] [/c:string] [/p] [/offline] [/d:dirlist] [/a:ColorAttribute] [/nologo] [strings] [[drive:][path]filename[ ...]]
```

其中，常用的参数包括：

- /b：匹配字符串的开头。
- /e：匹配字符串的结尾。
- /l：匹配精确字符串。
- /r：使用正则表达式进行匹配。
- /s：在子目录中搜索。
- /i：忽略大小写。
- /x：匹配整行。
- /v：反转匹配，即返回不包含字符串的行。
- /m：只返回文件名，而不是匹配的行。
- /n：在匹配的行前面显示行号。
- /o：在匹配的行中显示偏移量。
- /g:file：从指定的文件中获取搜索字符串。
- /f:file：从指定的文件中获取搜索模式。
- /c:string：指定要搜索的字符串。
- /p：将匹配的行输出到打印机。
- /offline：搜索离线文件。
- /d:dirlist：搜索指定的目录列表。
- /a:ColorAttribute：指定输出文本的颜色属性。
- /nologo：不显示版权信息。

