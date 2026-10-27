#### Find Request
- `ftp.request.command == "USER"`
- `ftp.request.command == "PASS"`
- `ftp.request.command == "RETR"`: 篩選下載檔案(retrieve)指令
- `ftp.request.commamd == "STOR"`: 篩選上傳檔案(store)指令
- ftp.request.command == "LIST"：篩選列出檔案清單指令


#### FInd Server Response
- `ftp.response.code == 230`：登入成功response
- `ftp.response.code == 530`：登入失敗（拒絕存取）response
- `ftp.response.code >= 400`：所有錯誤或失敗的Server response


#### Other
- `ftp contains "txts"`
