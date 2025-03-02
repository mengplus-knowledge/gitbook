---
title: check单元测试
slug: checkdan-yuan-ce-shi
cover: ""
categories:
  - git
tags:
  - 单元测试
halo:
  site: https://mengplus.top
  name: c1f90cdf-80ba-452f-8de3-ba56c7aa3f3a
  publish: true
---
## yaml文件编写示例
```yaml
name: build-test
on:
  push:
    branches:
      - master  # 更符合现代仓库规范的分支名
    filters:
      - '*.c,*.h'   # 只关注 C 文件的推送

jobs:
  build-test:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    - name: Install build tools
      run: |
        sudo apt-get update
        sudo apt-get install -y build-essential gcc make tree check

    - name: Test project
      run: |
        cd test
        make test

```

## makefile 示例模板
注意与普通的相比增加 `LIBS = -lcheck -lm -lpthread -lrt -lsubunit`


```makefile
# ./demo/Makefile

CC = gcc
CFLAGS = -Wall -Wextra -O2 -g -fPIC
INCLUDES = -I../
LIBS = -lcheck -lm -lpthread -lrt -lsubunit

# 目标文件路径
BUILD_DIR = ./build
SRCDIR = .
OBJDIR = $(BUILD_DIR)/obj

# 确保目标文件夹存在
$(shell mkdir -p $(OBJDIR))

# 源文件
SRCS = $(wildcard $(SRCDIR)/*.c) $(wildcard ../*.c)

# 目标文件
TARGET = $(BUILD_DIR)/test_mm_fifo.out
# 生成的可执行文件
OBJS = $(patsubst $(SRCDIR)/%.c, $(OBJDIR)/%.o, $(SRCS))

all: $(TARGET)

# 链接 demo_app 可执行文件
$(TARGET): $(OBJS)
	$(CC) $(OBJS) -o $@ $(LDFLAGS) $(LIBS)

# 编译目标文件
$(OBJDIR)/%.o: $(SRCDIR)/%.c
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

$(OBJDIR)/%.o: ../%.c
	$(CC) $(CFLAGS) $(INCLUDES) -c $< -o $@

clean:
	rm -rf $(BUILD_DIR) $(TARGET) $(OBJDIR)/*.o

test: $(TARGET)
	$(TARGET)

.PHONY: clean all test

# end of Makefile

```

关联知识
[[check 单元测试框架]]