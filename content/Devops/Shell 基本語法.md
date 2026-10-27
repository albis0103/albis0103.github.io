

## 1. 變數

```bash
name="Jason"          # 等號兩側不可有空格
echo "$name"          # 引用用 $name 或 ${name}
readonly PI=3.14      # 唯讀
unset name            # 刪除變數

# 指令替換
today=$(date +%F)

# 算術
a=$((3 + 4))
```

**引號差異**

- `"..."`：展開變數與指令替換
- `'...'`：完全字面，不展開

**特殊變數**

|變數|意義|
|---|---|
|`$0`|腳本名稱|
|`$1`…`$9`|位置參數|
|`$#`|參數個數|
|`$@`|所有參數（各自獨立）|
|`$*`|所有參數（合併為一字串）|
|`$?`|上一個指令的 exit status（0 = 成功）|
|`$$`|當前 shell 的 PID|

## 2. 條件判斷

```bash
if [[ "$x" -gt 10 ]]; then
    echo "big"
elif [[ "$x" -eq 10 ]]; then
    echo "equal"
else
    echo "small"
fi
```

**數值比較**：`-eq` `-ne` `-gt` `-ge` `-lt` `-le`  
**字串比較**：`==` `!=` `-z`（空字串）`-n`（非空）  
**檔案測試**：`-e`（存在）`-f`（一般檔案）`-d`（目錄）`-r` `-w` `-x`（權限）

`[[ ]]` 是 Bash 擴充，支援 `&&`、`||`、`=~`（regex），且不需對變數額外防 word splitting；`[ ]` 是 POSIX 版本。

```bash
case "$1" in
    start)  echo "starting" ;;
    stop)   echo "stopping" ;;
    *)      echo "usage: $0 {start|stop}" ;;
esac
```

## 3. 迴圈

```bash
# for
for i in 1 2 3; do echo "$i"; done
for f in *.txt; do echo "$f"; done
for ((i=0; i<5; i++)); do echo "$i"; done

# while
while [[ $n -lt 5 ]]; do
    ((n++))
done

# 逐行讀檔
while IFS= read -r line; do
    echo "$line"
done < input.txt
```

`break` 跳出迴圈，`continue` 跳到下一輪。

## 4. 函式

```bash
greet() {
    local who="$1"        # local 限制作用域
    echo "Hello, $who"
    return 0              # return 只能回傳 0–255 的 exit status
}

greet "Jason"
result=$(greet "Jason")   # 取得 stdout 作為「回傳值」
```

## 5. 輸入輸出重導向與管線

```bash
cmd > file        # stdout 覆寫
cmd >> file       # stdout 附加
cmd 2> err.log    # stderr
cmd > out 2>&1    # stderr 併入 stdout
cmd &> all.log    # Bash 簡寫
cmd < file        # stdin
cmd1 | cmd2       # 管線：cmd1 的 stdout → cmd2 的 stdin
cmd > /dev/null   # 丟棄輸出
```

Here document：

```bash
cat <<EOF
name: $name
EOF
```

## 6. 邏輯運算與指令串接

```bash
cmd1 && cmd2    # cmd1 成功才執行 cmd2
cmd1 || cmd2    # cmd1 失敗才執行 cmd2
cmd1 ; cmd2     # 無條件依序執行
cmd &           # 背景執行
```

## 7. 陣列

```bash
arr=(a b c)
echo "${arr[0]}"        # a
echo "${arr[@]}"        # 全部元素
echo "${#arr[@]}"       # 長度
arr+=(d)                # 附加

declare -A map          # 關聯陣列（Bash 4+）
map[key]="value"
echo "${map[key]}"
```

## 8. 參數展開

```bash
${var:-default}    # var 未設定或為空時用 default
${var:=default}    # 同上，並賦值
${var#prefix}      # 刪除最短前綴匹配
${var##*/}         # 刪除最長前綴匹配（取檔名）
${var%suffix}      # 刪除最短後綴匹配
${var%%.*}         # 刪除最長後綴匹配
${var/old/new}     # 取代第一個
${var//old/new}    # 取代全部
${#var}            # 字串長度
```

## 9. 腳本結構與安全設定

```bash
#!/usr/bin/env bash
set -euo pipefail
# -e : 指令失敗即退出
# -u : 使用未定義變數即報錯
# -o pipefail : 管線中任一指令失敗，整條管線視為失敗

main() {
    [[ $# -ge 1 ]] || { echo "usage: $0 <arg>" >&2; exit 1; }
    echo "arg = $1"
}

main "$@"
```

執行方式：

```bash
chmod +x script.sh
./script.sh arg1
bash script.sh arg1
```

## 10. 常用文字處理指令

|指令|用途|範例|
|---|---|---|
|`grep`|搜尋|`grep -rn "ERROR" logs/`|
|`sed`|串流編輯|`sed 's/old/new/g' f`|
|`awk`|欄位處理|`awk -F, '{print $2}' f.csv`|
|`cut`|切欄|`cut -d: -f1 /etc/passwd`|
|`sort` / `uniq`|排序 / 去重|`sort f \| uniq -c`|
|`xargs`|將 stdin 轉為參數|`find . -name "*.log" \| xargs rm`|
|`find`|搜尋檔案|`find . -type f -mtime -1`|

## 常見陷阱

1. **變數未加引號**：`rm $file` 在檔名含空白時會被拆成多個參數，應寫 `rm "$file"`。
2. **賦值時加空格**：`x = 1` 會被解析為執行指令 `x`。
3. **`[ ]` 內用 `<` `>`**：會被當作重導向；字串比較用 `[[ ]]`。
4. **管線中的子 shell**：`cmd | while read ...` 內修改的變數，離開迴圈後不保留；改用 `while ... done < <(cmd)`。
5. **`$?` 會被覆蓋**：需要時立刻存起來。

需要的話，我可以針對某一部分（例如 `awk`/`sed`、陣列、或 log 分析腳本實作）再深入。