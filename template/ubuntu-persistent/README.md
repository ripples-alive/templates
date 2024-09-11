# Ubuntu Persistent

## 应用说明

本应用基于 `phusion/baseimage` 构建，所有初始系统数据放在 `/template`，首次启动时会自动同步到挂载的存储卷 `/data` 中，`/template` 中的各个子目录同时也会分别被挂载到根目录下，从而实现系统目录的数据持久化。

## SSH

应用安装后，在应用商店的应用资源管理页面可以看到 SSH 对外映射的端口。

应用默认关闭密码登录，没有配置公钥，需要在网页上开启终端进入后自行添加公钥。

## 变更配置

目前的网页不支持同一存储卷的多次挂载，所以不能在网页上直接进行任何更新，否则会导致系统目录的持久化失效。

如果要更改配置，需要在网页终端中（不要进入容器内）手动操作。

```bash
# 列出所有 pod 并找到对应 ubuntu-persisten 的名字
kubectl get pod | grep ubuntu-persistent

# 手动更改配置项
kubectl edit pod ubuntu-persistent-xxx
```

如果在网页上直接操作更新导致持久化失效后，可以手动修改配置项中的 `volumeMounts` 以恢复：

```yaml
    volumeMounts:
    - mountPath: /data
      name: vn-appvn-data
    - mountPath: /etc
      name: vn-appvn-data
      subPath: etc
    - mountPath: /home
      name: vn-appvn-data
      subPath: home
    - mountPath: /opt
      name: vn-appvn-data
      subPath: opt
    - mountPath: /root
      name: vn-appvn-data
      subPath: root
    - mountPath: /usr
      name: vn-appvn-data
      subPath: usr
    - mountPath: /var
      name: vn-appvn-data
      subPath: var
```

## 重启

`reboot/poweroff/shutdown` 命令在容器内都会失效，命令行重启容器需要通过 `kill 1` 来实现。

## 系统升级

如果需要升级或者重置系统，直接删除 `/data/usr` 目录，然后重启容器即可重新同步 `/template`。
同步时，`/root` `/home` 中已有的文件会被保留，其他文件都会被删除。
