# Windows

```sh
netstat -aon|findstr "9996"
tasklist|findstr "28944"
taskkill -pid 28944 -f 
```

