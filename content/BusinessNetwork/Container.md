#### Namespace
- 對全域系統資源抽象分割，讓 **container以為獨佔電腦**（以為自己是 `root` 可以看到獨立的File System, IP, pid）
- 只隔離 資源View，沒限制資源用量
- 同一 namespace的process 可以看到彼此
- 不同種類 namespace表示：`/proc/<pid>/ns/*`

| 類型     | 隔離對象                                                           | clone flag        |
| ------ | -------------------------------------------------------------- | ----------------- |
| mnt    | **掛載點 Mount**<br>每個 container有獨立掛載目錄                           | `CLONE_NEWNS`     |
| pid    | **PID**<br>看不到container外其他process                              | `CLONE_NEWPID`    |
| net    | **網路介面、路由表、iptables（防火牆規則）、socket**<br>每個Container獨立           | `CLONE_NEWNET`    |
| uts    | **hostname、domainname**<br>每隔容器獨立主機名稱與Domain Name              | `CLONE_NEWUTS`    |
| ipc    | **SysV IPC、POSIX message queue**<br>隔離 Memory與 Message Queue機制 | `CLONE_NEWIPC`    |
| user   | UID/GID 對應                                                     | `CLONE_NEWUSER`   |
| cgroup | cgroup 根目錄的視圖                                                  | `CLONE_NEWCGROUP` |
| time   | `CLOCK_MONOTONIC`、`CLOCK_BOOTTIME` offset                      | `CLONE_NEWTIME`   |
相關 syscall：

- `clone()` / `clone3()`：建立 process 時指定新 namespace。
- `unshare()`：讓當前 process 脫離現有 namespace 並建立新的。
- `setns()`：加入既有 namespace，`docker exec` 即用此機制。


#### cgroup
- 限制、紀錄、隔離 Container（或 一組 process）能使用的實體資源
- 由 Controller 對群組做管理
	- v1：每個 controller 獨立階層，process 在不同階層可屬於不同群組。
	- v2：單一統一階層（`/sys/fs/cgroup`），process 只屬於一個 cgroup；controller 透過 `cgroup.subtree_control` 啟用；有 “no internal process” 規則（非葉節點啟用 controller 時不能直接含 process）。

**常用 controller（v2 檔案）：**
- cpu：`cpu.max`（quota/period）、`cpu.weight`
- memory：`memory.max`（超過則觸發 cgroup 內 OOM killer）、`memory.high`（軟限制，觸發 reclaim throttling）、`memory.swap.max`
- io：`io.max`、`io.weight`
- pids：`pids.max`（限制 process 數量，防 fork bomb）
- cpuset：`cpuset.cpus`、`cpuset.mems`

Kubernetes 的 `resources.limits.cpu` / `memory` 最終對應到 `cpu.max` / `memory.max`；`requests` 對應 `cpu.weight` 等（調度權重）。



##### Docker 中執行一個容器時，Linux 核心在幕後做了以下事：

1. **建立 Namespace：** 為該容器建立獨立的 PID、NET、MNT 等 Namespace。這使得容器內看到的是全新的檔案系統、獨立的 IP 以及編號為 1 的 PID。
    
2. **分配 cgroup：** 將容器對應的進程加入特定的 cgroup 群組中，並設定上限（例如限制最多使用 2核 CPU、4GB 記憶體）。