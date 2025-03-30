# entr命令



监控文件夹:

```
$ while sleep 0.1; do ls src/*.rb | entr -d make; done
```

```bash
 nohup while sleep 0.1; do ls uftp/*.* | entr -d mv uftp/*.* bak/; done &
```

