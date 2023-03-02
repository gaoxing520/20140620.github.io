---
title: "linux环境计算物理CPU颗数"
date: 2023-03-02T12:44:31+08:00
draft: false
---

## linux下获取物理CPU的颗数
* golang 环境
* 如果是获取逻辑 cpu 数, 也就是 cores, 直接用`runtime.NumCPU()`就可以了
* golang没有标准库可以直接获取物理CPU颗数
* 我们可以分析`/proc/cpuinfo`的`physical id`来计算, 下面是 demo 程序

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"strconv"
	"strings"
)

func main() {
	var (
		maxID int
	)

	if runtime.GOOS != "linux" {
		fmt.Println("This program can only run on Linux.")
		os.Exit(1)
	}

	data, err := os.ReadFile("/proc/cpuinfo")
	if err != nil {
		panic(err)
	}

	lines := strings.Split(string(data), "\n")
	for _, line := range lines {
		if strings.Contains(line, "physical id") {
			fields := strings.Fields(line)
			if len(fields) >= 3 {
				id, err := strconv.Atoi(fields[3])
				if err != nil {
					continue
				}

				if id > maxID {
					maxID = id
				}
			}
		}
	}

	fmt.Printf("Physical CPUs: %d\n", maxID+1)
}

```

```rust
use std::fs;
use std::process;

fn main() {
    if std::env::consts::OS != "linux" {
        eprintln!("This program can only run on Linux.");
        process::exit(1);
    }

    let mut max_id: i32 = 0;

    let data = fs::read_to_string("/proc/cpuinfo").expect("Failed to read cpuinfo file");
    // println!("{}", data);
    
    for line in data.lines() {
        if line.contains("physical id") {
            let fields: Vec<&str> = line.split_whitespace().collect();
            if fields.len() >= 4 {
                if let Ok(id) = fields[3].parse::<i32>() {
                    if id > max_id {
                        max_id = id;
                    }
                }
            }
        }
    }

    println!("Physical CPUs: {}", max_id + 1);
}

```