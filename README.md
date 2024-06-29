# 📡 TCP over HTTP

## 🥦 The Questions

- #### 🪃 What does it do?

  You can proxy TCP traffic over HTTP.

  A basic setup would be:

  ```
  [Your TCP target] <--TCP-- [Exit Node]
                                  ^
                                  |
                                HTTP
                                  |
  [Your TCP client] --TCP--> [Entry Node]
  ```

- #### 🍩 Why?

  ~~I was bored.~~

  This allows you to reach servers behind a HTTP reverse proxy.  
  Suddenly you can do SSH to a server which is behind a NGINX proxy.

  If you have for example a HTTP gateway, you can now also have
  a TCP gateway.

- #### 🍾 Why not?

  If a server only opens port 80, nobody expects you
  to tunnel through and reach the SSH server.  
  Security wise, no admin would want this tool on their
  server without them knowing.

## 🌲 Installation - host

- get yourself a rust toolchain via rustup https://www.rust-lang.org/tools/install
- go nightly: `rustup default nightly`
- ```bash
  cargo install --locked --git https://github.com/julianbuettner/tcp-over-http
  ```

_Note_:  
If you want to use the stable version, checkout tag `rocket-stable`,
which uses rocket and can be build with stable.

- ```bash
  cargo install --locked --git https://github.com/julianbuettner/tcp-over-http --tag rocket-stable
  ```

## 🎺 Usage - host

Replace `tcp-over-http` by `cargo run --release --`
if you have not installed the binary.

```bash
tcp-over-http --help

# Start our exit node to reach our SSH server (default listen localhost:8080)
tcp-over-http exit --help
tcp-over-http exit --target-addr localhost:22

# Start our entry node (default listen localhost:1415)
tcp-over-http entry --help
tcp-over-http entry --target-url http://localhost:8080/

# Test it
ssh localhost -p 1415
```

## 🛥️ Installation - image
_Note_:  
This is applied to the stable version, checkout tag `rocket-stable`.
If you want to modify it, modify the Dockerfie.

- ```bash
  docker build . -t <image_name>
  ```

## 🐋 Usage - image

Replace `listen_http_port`, `target_host`, `target_port` with your on values.
```bash
# Start exit node to reach our SSH server, on 192.168.1.100:22.
docker run -it -d -e listen_http_port=8080 -p 8080:8080 -e target_host=192.168.1.100 -e target_port=22 the_exit

# Start our entry node (Reaching exit in http://192.168.1.44:8080/)
todo:

# Test it
ssh localhost -p 1415
```


## ⌚️ Performance

This package is not optimized for stability ~~or speed~~.

_Setup_

```bash
# Terminal 0 - Netcat listening
nc -l 1234 > /dev/null

# Terminal 1 - Exit Node
tcp-over-http exit --target-addr localhost:1234

# Terminal 2 - Entry Node
tcp-over-http entry --target-url http://localhost:8080/

# Terminal 3 - Sending \0 data
# Using pipeviewer (pv) to see current data rate
time cat /dev/zero | pv | nc localhost 1415
```

### 🏅 Result: 900MiB/s vs 1.3GiB/s (nc | pv > nc)
