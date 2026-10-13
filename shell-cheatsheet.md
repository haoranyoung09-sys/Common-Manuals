# Shell 高频代码速查

适用范围：Linux / Ubuntu / Git Bash 偏 POSIX 的 bash 脚本。核心原则：脚本要可重复运行、失败要可见、路径变量要加引号、输入目录只读、输出目录可写。

---

## 0. 通用脚本开头

```bash
#!/bin/bash
set -eu

input=${1:?需要输入目录}
output=${2:?需要输出目录}
```

- `set -e`：命令失败立刻退出。
- `set -u`：使用未定义变量立刻报错。
- 引用变量默认加双引号：`"$input"`、`"$output"`。
- 需要可选参数时给默认值：

```bash
action=${3:-repair}
```

---

## 1. 目录与文件

```bash
mkdir -p "$output/a" "$output/b"     # 递归创建，已存在不报错
: > "$output/empty.txt"              # 创建或清空文件
cp "$input/a.conf" "$output/a.conf"  # 复制
mv "$output/a.conf" "$output/b.conf" # 移动或改名
rm -rf "$output/tmp"                 # 删除目录，谨慎使用
```

只清理自己负责的输出目录，不要删 `$input`：

```bash
rm -rf "$output/work"
mkdir -p "$output/work"
```

---

## 2. 写文件：覆盖、追加、heredoc

```bash
printf '%s\n' "content" > "$output/a.txt"   # 覆盖写一行
printf '%s\n' "content" >> "$output/a.txt"  # 追加写一行
```

固定内容用带引号的 heredoc，变量不展开：

```bash
cat > "$output/config.txt" <<'EOF'
robot=sentry
mode=training
log_level=INFO
EOF
```

需要变量展开时用不带引号的 heredoc：

```bash
cat > "$output/ssh_config" <<EOF
Host aim-sentry
    HostName $HOST
    Port $PORT
    User $REMOTE_USER
EOF
```

判断标准：

- 内容必须原样写：`<<'EOF'`
- 内容要替换变量：`<<EOF`

---

## 3. 读取与拼接文本

```bash
cat "$input/a.log"                          # 输出整个文件
cat "$input/boot.log" "$input/run.log" > "$output/all.log"
head -n 3 "$output/errors.log" > "$output/first-errors.log"
tail -n 5 "$output/all.log" > "$output/recent.log"
```

查看不可见字符：

```bash
cat -A "$output/robot-id.txt"   # 行尾 $ 表示 LF，^I 表示 Tab
```

---

## 4. grep：筛选、计数、精确匹配

```bash
grep "^ERROR " "$output/all.log" > "$output/errors.log" || true
grep -c "^ERROR " "$output/all.log" > "$output/error-count.txt" || true
grep -E "^(WARN |ERROR )" "$output/all.log" > "$output/attention.log" || true
grep -F "TICK $TARGET_TOKEN " "$WORKER_LOG" > "$output/sample.log" || true
```

要点：

- `^` 匹配行首，避免误匹配行中间的关键字。
- `-E` 使用扩展正则，支持 `(A|B)`。
- `-F` 把模式当普通字符串。
- `-c` 只输出匹配行数。
- `|| true` 允许“没有匹配行”这种正常结果，避免触发 `set -e`。

---

## 5. 捕获外部命令退出码

适用于“外部程序可能失败，但脚本要记录结果并正常结束”。

```bash
set +e
bash "$LAUNCH_TOOL" "$MODEL_PATH" "$CAMERA_PATH" > "$output/stdout.log" 2> "$output/stderr.log"
code=$?
set -e

printf '%s\n' "$code" > "$output/exit-code.txt"
```

根据退出码写报告：

```bash
case "$code" in
    0)  result=SUCCESS; reason=NONE ;;
    10) result=FAILED;  reason=MODEL_MISSING ;;
    20) result=FAILED;  reason=CAMERA_DENIED ;;
    *)  result=FAILED;  reason=NONE ;;
esac

printf 'result=%s\nexit_code=%s\nreason=%s\n' "$result" "$code" "$reason" > "$output/report.txt"
```

注意：`$?` 必须立刻接住，中间不能插命令。

---

## 6. 从 env 文件注入参数

```bash
source "$input/process.env"
```

之后直接使用文件里的变量：

```bash
"$TARGET_PID"
"$TARGET_PORT"
"$START_TOOL"
```

不要硬编码 PID、端口、路径；都从 env 读。

---

## 7. 权限

```bash
chmod 750 "$output/scripts/hello.sh"   # rwxr-x---，脚本/目录常用
chmod 640 "$output/config/robot.conf"  # rw-r-----，配置常用
chmod 600 "$output/ssh_config"         # rw-------，私钥/敏感配置
```

优先用精确数字权限。少用 `chmod +x`，因为它会在原有权限上加执行位，结果可能不是题目/环境要求的精确权限。

---

## 8. 进程：查看、停止、等待

判断进程是否活着，排除 zombie：

```bash
alive() {
    ps -o stat= -p "$1" 2>/dev/null | grep -qv '^Z'
}
```

正常停止一个 PID，并最多等 5 秒：

```bash
stop_pid() {
    local pid=$1 attempt
    kill "$pid" 2>/dev/null || true
    for attempt in $(seq 1 50); do
        if ! alive "$pid"; then
            return 0
        fi
        sleep 0.1
    done
    printf '进程 %s 未在预期时间内退出\n' "$pid" >&2
    return 1
}
```

查看进程信息：

```bash
ps -o user= -p "$pid"   # 属主
ps -o stat= -p "$pid"   # 状态
ps -o args= -p "$pid"   # 完整命令行
```

原则：

- 按 PID 操作，不按名字 `pkill`/`killall`。
- 先用 SIGTERM（默认 `kill`），少用 `kill -9`。
- kill 后要有等待/确认逻辑。

---

## 9. HTTP 服务：等待就绪与验证

```bash
for _ in $(seq 1 80); do
    if body=$(curl -s --noproxy '*' --max-time 2 "http://127.0.0.1:$TARGET_PORT/status" 2>/dev/null) \
        && [ "$body" = "SENTRY READY" ]; then
        break
    fi
    sleep 0.1
done
```

保存状态码和响应体：

```bash
status=$(curl -s --noproxy '*' -o "$output/http-body.txt" -w '%{http_code}' "http://127.0.0.1:$TARGET_PORT/status")
printf '%s\n' "$status" > "$output/http-status.txt"
```

常用参数：

- `-s`：静默。
- `--noproxy '*'`：本机地址不走代理。
- `--max-time N`：单次请求超时。

---

## 10. tmux 后台分离会话

```bash
tmux -L "$TMUX_SOCKET" new-session -d -s "$SESSION" \
    "python3 '$WORKER_TOOL' --token '$TOKEN' --log '$output/worker.log'"
```

停止会话：

```bash
tmux -L "$TMUX_SOCKET" kill-session -t "$SESSION" 2>/dev/null || true
```

检查会话是否分离，`0` 表示 detached：

```bash
tmux -L "$TMUX_SOCKET" display-message -p -t "$SESSION" '#{session_attached}'
```

其他常用：

```bash
tmux -L "$TMUX_SOCKET" ls
tmux -L "$TMUX_SOCKET" capture-pane -p -t "$SESSION" | tail -n 50
tmux -L "$TMUX_SOCKET" send-keys -t "$SESSION" 'echo hi' Enter
```

---

## 11. SSH / SCP

写最小 SSH 配置：

```bash
cat > "$output/ssh_config" <<EOF
Host aim-sentry
    HostName $HOST
    Port $PORT
    User $REMOTE_USER
    IdentityFile "$KEY_PATH"
    UserKnownHostsFile "$KNOWN_HOSTS"
    IdentitiesOnly yes
    StrictHostKeyChecking yes
    BatchMode yes
EOF
chmod 600 "$output/ssh_config"
```

远程执行命令：

```bash
ssh -F "$output/ssh_config" aim-sentry whoami > "$output/remote-user.txt"
```

拉取远程文件：

```bash
scp -F "$output/ssh_config" "aim-sentry:$REMOTE_LOG" "$output/received/errors.log"
```

要点：

- `BatchMode yes`：禁止交互式询问，脚本才不会卡住。
- `IdentitiesOnly yes`：只用指定私钥。
- 配置文件权限通常要 `600`。

---

## 12. 打包与压缩

```bash
tar -czf "$output/archive.tar.gz" -C "$output/work" .
tar -tzf "$output/archive.tar.gz" | head
tar -xzf "$output/archive.tar.gz" -C "$output/unpacked"
```

zip/unzip：

```bash
zip -r "$output/pack.zip" "$output/work"
unzip -l "$output/pack.zip"
unzip "$output/pack.zip" -d "$output/unzip"
```

---

## 13. find / xargs / sort / uniq

```bash
find "$input" -type f -name '*.log' | sort
find "$input" -type f -name '*.tmp' -delete
find "$input" -type f -print0 | xargs -0 grep -n "ERROR"
```

统计 Top N：

```bash
sort "$output/names.txt" | uniq -c | sort -nr | head -n 10
```

注意：处理可能含空格的文件名时，用 `-print0` + `xargs -0`。

---

## 14. sed / awk 小补丁

替换文本：

```bash
sed 's/port=8080/port=9090/' "$input/server.conf" > "$output/server.conf"
sed -i.bak 's/debug=true/debug=false/' "$output/app.conf"
```

取列：

```bash
awk '{print $1}' "$output/access.log" | sort | uniq -c | sort -nr | head
```

精确匹配字段并解析 `pid=12345`：

```bash
awk -v token="$TOKEN" '$1 == "START" && $2 == token {split($3, part, "="); print part[2]}' "$output/worker.log" | head -n 1
```

---

## 15. 时间、超时、重试

```bash
date +%F
date '+%Y-%m-%d %H:%M:%S'
timeout 5 bash "$TOOL" arg1 arg2
```

通用重试：

```bash
for attempt in 1 2 3 4 5; do
    if curl -fsS --max-time 2 "http://127.0.0.1:$PORT/health"; then
        break
    fi
    sleep 1
done
```

---

## 16. 调试脚本

```bash
bash -n script.sh    # 只检查语法，不执行
bash -x script.sh    # 跟踪执行，适合调试
shellcheck script.sh # 静态检查，强烈推荐
```

打印错误到 stderr：

```bash
printf '错误：缺少参数\n' >&2
exit 1
```

---

## 17. 常见红线

1. 不修改 `$input`；所有结果写到 `$output`。
2. 不硬编码输入内容；用 `cat`、`grep`、`source` 现读。
3. 默认脚本会跑两次；写文件优先 `>`，建目录用 `mkdir -p`。
4. `grep` 没有匹配时是正常结果，常用 `|| true`。
5. 杀进程按 PID，优先 SIGTERM，不滥用 `kill -9`。
6. SSH/tmux/服务启动都要避免交互式等待。
7. 路径变量永远加引号：`"$input/file"`。
8. 生成文本注意 LF 换行；需要检查时看 `cat -A`。

---

## 18. 最小排障清单

脚本挂了先看：

```bash
bash -n src/main/q4.sh
bash -x src/main/q4.sh input output
```

然后确认：

- 传入路径是否存在：`ls -la "$input" "$output"`
- 变量是否拼错：脚本是否开了 `set -u`
- 权限是否正确：`stat -c '%a %n' file`
- 进程是否存在：`ps -o pid=,user=,stat=,args= -p "$pid"`
- 端口是否监听：`ss -ltnp | grep ":$PORT"` 或 `curl -v "http://127.0.0.1:$PORT/"`
- tmux 会话是否在：`tmux -L "$SOCKET" ls`
