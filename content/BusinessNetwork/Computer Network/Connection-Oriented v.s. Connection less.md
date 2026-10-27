
|            | Connection-Oriented    | Connectionless     |
| ---------- | ---------------------- | ------------------ |
| Definition | 雙方須先建立連線，傳輸時維護連線，結束時釋放 | 無需建立連線，傳輸速度快但不保證可靠 |
| 代表         | TCP                    | UDP, IP            |
| Reliable   | Ack, Retransmission機制  | 不保證                |
| 資料單位       | Byte - Stream          | Datagram           |
| Header     | 20 bytes               | 8 bytes            |
| 傳輸速度       | 慢                      | 快                  |
