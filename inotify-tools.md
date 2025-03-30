# inotify-tools



整体：

```bash
entr
```

个别：

```bash
sudo apt install inotify-tools

inotifywait -m -e modify . | while read -r dir action file;do echo "The file '$file' appeared in directory '$dir' via '$action'";done
```

