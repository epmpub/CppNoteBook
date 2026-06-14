写入代码



```powershell
@'
ls 'name' -ea 0 | Out-Null && echo "OK" || echo "failed"
'@ | Out-File pwsh7Test.ps1 -Encoding utf8

```

