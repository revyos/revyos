C910V强制cpu指定补丁

```patch
From 5164bca5a4bcde4534dc1a9aa3a7f619719874cf Mon Sep 17 00:00:00 2001
From: Han Gao <gaohan@iscas.ac.cn>
Date: Sun, 23 Apr 2023 22:11:35 +0800
Subject: [PATCH] qemu-user-riscv64 default cpu is c910v

Signed-off-by: Han Gao <gaohan@iscas.ac.cn>
---
 linux-user/riscv/target_elf.h | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

diff --git a/linux-user/riscv/target_elf.h b/linux-user/riscv/target_elf.h
index 9dd65652ee45..3195cfa71408 100644
--- a/linux-user/riscv/target_elf.h
+++ b/linux-user/riscv/target_elf.h
@@ -9,7 +9,7 @@
 #define RISCV_TARGET_ELF_H
 static inline const char *cpu_get_model(uint32_t eflags)
 {
-    /* TYPE_RISCV_CPU_ANY */
-    return "any";
+    /* TYPE_RISCV_CPU_C910V */
+    return "c910v";
 }
 #endif
```

编译流程

```bash
./configure \
  --prefix=$HOME/qemu-install \
  --static \
  --target-list=riscv64-linux-user \
  --disable-system \
  --disable-pie \
  --interp-prefix=/etc/qemu-binfmt/%M

make -j20
```