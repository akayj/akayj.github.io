---
title: "博客中的语法高亮"
description: "语法高亮测试"
date: 2019-12-03T11:42:18+08:00
tags: ["syntax", "highlight"]
---

<!--more-->

## 代码样例

### Golang

```go
// waitSignal regist quit signals, a done channel is returned
// as signal receiver
func waitSignal() <-chan os.Signal {
    sigs := make(chan os.Signal, 1)
    done := make(chan os.Signal)

    signal.Notify(sigs, syscall.SIGINT, syscal.SIGTERM)

    go func() {
        sig := <- sigs
        done <- sig
    }()

    return done
}
```

### Rust

```rust
fn main() {
    println!("Hello Rust!");
}
```
