## 附录B 内核编译文件Makefile



```makefile,editable
# General options
ARCH ?= riscv64
TARGET := riscv64gc-unknown-none-elf
SMP ?= 1
LOG ?= warn
FEATURES ?=

# Utility definitions and functions
GREEN_C := \033[92;1m
CYAN_C := \033[96;1m
YELLOW_C := \033[93;1m
GRAY_C := \033[90m
WHITE_C := \033[37m
END_C := \033[0m

QEMU := qemu-system-$(ARCH)
OBJDUMP ?= rust-objdump -d --print-imm-hex --x86-asm-syntax=intel
OBJCOPY ?= rust-objcopy --binary-architecture=$(ARCH)

# App options
A ?= axorigin
APP ?= $(A)

APP_NAME := $(shell basename $(APP))
LD_SCRIPT := $(CURDIR)/linker.lds

OUT_DIR ?= target/$(TARGET)/release

OUT_ELF := $(OUT_DIR)/$(APP_NAME)
OUT_BIN := $(OUT_DIR)/$(APP_NAME).bin

ifeq ($(filter $(MAKECMDGOALS),test),)  # not run `cargo test`
RUSTFLAGS := -C link-arg=-T$(LD_SCRIPT) -C link-arg=-no-pie
export RUSTFLAGS
endif
export LOG

all: build

build: $(OUT_BIN)

disasm: build
    $(OBJDUMP) $(OUT_ELF) | less

run: build justrun

justrun:
    @printf "    $(CYAN_C)Running$(END_C) on qemu...\n"
    $(QEMU) -m 128M -smp $(SMP) -machine virt \
        -bios default -kernel $(OUT_BIN) -nographic \
        -D qemu.log -d in_asm

$(OUT_BIN): $(OUT_ELF)
    $(OBJCOPY) $(OUT_ELF) --strip-all -O binary $@

$(OUT_ELF): FORCE
    @printf "    $(GREEN_C)Building$(END_C) App: $(APP_NAME), Arch: riscv64, Platform: qemu-virt, App type: rust\n"
    cargo build --manifest-path $(APP)/Cargo.toml --release \
        --target $(TARGET) --target-dir $(CURDIR)/target $(FEATURES)

clean:
    @rm -rf ./target
    @rm -f ./qemu.log

test:
    cargo test --workspace --exclude "axorigin" -- --nocapture

FORCE:
    @:

.PHONY: all build disasm run justrun test clean FORCE
```

注：建立axhal组件后，把 linker.lds 挪到该组件目录之下，`LD_SCRIPT := $(CURDIR)/axhal/linker.lds`。





<script src="https://utteranc.es/client.js"
        repo="OSLearning365/blog-issues"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
