# ホットリロード環境を構築する

ではホットリロード環境を構築していきたいと思います。
今回はairを採用します。

Dockerfile, docker-compose.yml, .air.tomlを作成しましょう。

Dockerfile
```
FROM golang:1.20.6

WORKDIR /app

RUN go install github.com/cosmtrek/air@latest

CMD ["air", "-c", ".air.toml"]
```

docker-compose.yml
```
version: "3.8"

services:
  api:
    container_name: login-go-api
    build:
      dockerfile: Dockerfile
      context: .
    volumes:
      - ".:/app"
    ports:
      - "8000:8000"
```

.air.toml
```
# Config file for [Air](https://github.com/cosmtrek/air) in TOML format

# Working directory
# . or absolute path, please note that the directories following must be under root.
root = "."
tmp_dir = "tmp"

[build]
# Just plain old shell command. You could use `make` as well.
cmd = "go build -o ./tmp/main ."
# Binary file yields from `cmd`.
bin = "tmp/main"
# Customize binary, can setup environment variables when run your app.
full_bin = "APP_ENV=dev APP_USER=air ./tmp/main"
# Watch these filename extensions.
include_ext = ["go", "tpl", "tmpl", "html"]
# Ignore these filename extensions or directories.
exclude_dir = ["assets", "tmp", "vendor", "frontend/node_modules"]
# Watch these directories if you specified.
include_dir = []
# Watch these files.
include_file = []
# Exclude files.
exclude_file = []
# Exclude specific regular expressions.
exclude_regex = ["_test\\.go"]
# Exclude unchanged files.
exclude_unchanged = true
# Follow symlink for directories
follow_symlink = true
# This log file places in your tmp_dir.
log = "air.log"
# Poll files for changes instead of using fsnotify.
poll = false
# Poll interval (defaults to the minimum interval of 500ms).
poll_interval = 500 # ms
# It's not necessary to trigger build each time file changes if it's too frequent.
delay = 0 # ms
# Stop running old binary when build errors occur.
stop_on_error = true
# Send Interrupt signal before killing process (windows does not support this feature)
send_interrupt = false
# Delay after sending Interrupt signal
kill_delay = 500 # ms
# Rerun binary or not
rerun = false
# Delay after each executions
rerun_delay = 500
# Add additional arguments when running binary (bin/full_bin). Will run './tmp/main hello world'.
args_bin = ["hello", "world"]

[log]
# Show log time
time = false
# Only show main log (silences watcher, build, runner)
main_only = false

[color]
# Customize each part's color. If no color found, use the raw app log.
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"

[misc]
# Delete tmp directory on exit
clean_on_exit = true

[screen]
clear_on_rebuild = true
keep_scroll = true
```

これでホットリロード環境を構築しました。

# 確認方法

実際にホットリロード環境ができているか確認していきます。

コンテナを起動します。
```
docker compose up
```

するとちょっと待った後に次のように出力されるはずです。
```
 ✔ Network go_default           Created                                    0.1s 
 ✔ Container login-example-api  Created                                    0.1s 
Attaching to login-example-api
login-example-api  | 
login-example-api  |   __    _   ___  
login-example-api  |  / /\  | | | |_) 
login-example-api  | /_/--\ |_| |_| \_ , built with Go 
login-example-api  | 
login-example-api  | mkdir /app/tmp
login-example-api  | watching .
login-example-api  | !exclude tmp
login-example-api  | building...
login-example-api  | go: downloading github.com/labstack/echo/v4 v4.11.1
login-example-api  | go: downloading github.com/labstack/gommon v0.4.0
login-example-api  | go: downloading golang.org/x/crypto v0.11.0
login-example-api  | go: downloading golang.org/x/net v0.12.0
login-example-api  | go: downloading github.com/valyala/fasttemplate v1.2.2
login-example-api  | go: downloading github.com/mattn/go-isatty v0.0.19
login-example-api  | go: downloading github.com/valyala/bytebufferpool v1.0.0
login-example-api  | go: downloading golang.org/x/sys v0.10.0
login-example-api  | go: downloading golang.org/x/text v0.11.0
login-example-api  | running...
login-example-api  | 
login-example-api  |    ____    __
login-example-api  |   / __/___/ /  ___
login-example-api  |  / _// __/ _ \/ _ \
login-example-api  | /___/\__/_//_/\___/ v4.11.1
login-example-api  | High performance, minimalist Go web framework
login-example-api  | https://echo.labstack.com
login-example-api  | ____________________________________O/_______
login-example-api  |                                     O\
login-example-api  | ⇨ http server started on [::]:8000
```

では、本当にコードを修正したら、すぐに反映されるか確認していきましょう。
新しいターミナルを起動し、次のコマンドを実行してください。

```
curl localhost:8000
Hello, world! # と出力されればOK
```

これでホットリロード環境の構築は完了です。

# まとめ

現在のディレクトリはこんな感じです
```
.
├── .air.toml
├── Dockerfile
├── docker-compose.yml
├── go.mod
├── go.sum
└── main.go
```