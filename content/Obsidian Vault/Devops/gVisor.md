gVisor 是 Google 開發的開源應用程式核心（application kernel）沙盒，以使用者空間（user space）重新實作 Linux 核心的系統呼叫介面，在容器與宿主核心之間插入一道隔離層。

## 設計動機

傳統容器（runc）容器內程式發 System call 直接由 Kernel 處理。VM 隔離較強，但啟動慢、資源開銷高。gVisor 位於兩者之間：不讓不受信任的程式直接接觸宿主核心。

## 核心元件

- **Sentry**：以 Go 撰寫的User-Space Kernel，攔截並實作應用程式的 System call 以 seccomp 限制。
- **Gofer**：負責檔案系統存取的process。Sentry 向 Gofer 請求，Gofer 再以較低權限存取宿主檔案系統。
- **runsc**：符合 OCI 規範的 runtime，可接入 Docker、containerd、Kubernetes（以 RuntimeClass 指定）。
- **Netstack**：Sentry 內建的使用者空間 TCP/IP 堆疊（Go 實作），網路封包不經宿主核心的網路堆疊處理（亦可選 host network 模式）。

## 攔截 syscall 的平台（Platform）

|Platform|機制|特性|
|---|---|---|
|systrap（目前預設）|seccomp `SECCOMP_RET_TRAP` + 訊號處理攔截 syscall|不需虛擬化支援，可在 VM 內執行|
|KVM|使用 KVM，Sentry 同時扮演 guest OS 與 VMM|巢狀虛擬化外效能較佳，需 KVM 支援|
|ptrace（舊版）|`PTRACE_SYSEMU`|相容性高，開銷大，已淘汰|

