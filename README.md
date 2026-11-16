# STM32 HAL CMake Template

![MCU](https://img.shields.io/badge/MCU-STM32F103C8T6-blue)
![CMake](https://img.shields.io/badge/CMake-3.22%2B-064F8C)
![FreeRTOS](https://img.shields.io/badge/RTOS-FreeRTOS%20V10.3.1-green)
![GCC](https://img.shields.io/badge/arm--none--eabi--gcc-13.3.1-orange)
![License](https://img.shields.io/badge/license-MIT-green)

**开箱即用的 STM32 HAL + FreeRTOS + CMake 工程模板。**

基于 STM32CubeMX 生成的 CMake 标准工程整理，clone（或 GitHub 的 *Use this template*）之后改个名字就能开始开发，不再需要在 CubeMX、Makefile 和 IDE 之间反复折腾。

> A ready-to-use STM32 HAL + FreeRTOS + CMake template — clone it, rename it, build it, flash it.

- 目标芯片：**STM32F103C8T6**（Cortex-M3，64 KB Flash / 20 KB RAM）
- 示例程序：FreeRTOS 默认任务以 700 ms / 300 ms 的节奏翻转 PC13 板载 LED
- 构建方式：CMake + Ninja + `arm-none-eabi-gcc`，Debug / Release 双预设
- 已验证环境：Windows + CLion 2026.1.2（内置 CMake 4.2.2 + Ninja）+ GNU Tools for STM32 GCC 13.3.1

---

## 目录

- [特性](#特性)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [烧录与调试](#烧录与调试)
- [目录结构](#目录结构)
- [默认配置一览](#默认配置一览)
- [构建系统说明](#构建系统说明)
- [定制指南](#定制指南)
- [常见问题（FAQ）](#常见问题faq)
- [版本与许可证](#版本与许可证)
- [参考链接](#参考链接)

---

## 特性

- **CubeMX 原生 CMake 工程**：`.ioc` 中 Toolchain 已设为 `CMake`，用 CubeMX 重新生成时自动更新 `cmake/stm32cubemx/CMakeLists.txt` 和驱动文件，且不会覆盖用户维护的顶层 `CMakeLists.txt` 与 `USER CODE` 区域。
- **HAL + FreeRTOS 开箱即用**：STM32Cube FW_F1 V1.8.7（HAL 1.1.10）+ FreeRTOS V10.3.1 + CMSIS-RTOS V2，源码随仓库提供，**离线可构建**。
- **TIM4 作为 HAL 时基**：FreeRTOS 的调度 tick 使用 SysTick，TIM4 只负责 HAL 的 1 ms 时基，因此 `HAL_Delay()` / `HAL_GetTick()` 在 RTOS 下依然可用；两者独立、互不抢占。
- **Debug / Release 双预设**：Debug `-O0 -g3`，Release `-Os -g0`；链接时启用 `--gc-sections` 并打印内存占用。
- **构建后自动生成 `.hex` / `.bin` / `.map`**：一条命令得到可直接烧录的文件。
- **导出 `compile_commands.json`**：CLion / clangd / VS Code 拥有准确的补全、跳转与索引。
- **可选工具链**：默认 GCC，另提供 ST ArmClang 工具链文件 `cmake/starm-clang.cmake`。
- **CLion 开箱兼容**：仓库不包含 `.idea` 目录（已整体忽略），用 CLion 打开后按提示加载 `CMakePresets.json` 中的 `Debug` 预设即可；本机 IDE 状态不会进入仓库，也不会把某台机器的配置带给别人。

---

## 环境要求

| 依赖 | 版本 / 说明 |
|---|---|
| CMake | ≥ 3.22（`CMakeLists.txt` 要求；`CMakePresets.json` 为 v3 格式） |
| Ninja | 任意较新版本（预设固定使用 Ninja 生成器） |
| arm-none-eabi-gcc | 已验证 13.3.1；GNU Tools for STM32 / xPack / ARM GNU Toolchain 均可 |
| STM32CubeMX | 6.17.0（仅在修改 `.ioc` 时需要） |
| OpenOCD + ST-Link | 烧录 / 调试，建议 OpenOCD 0.12+ |
| IDE（可选） | CLion 2026.1.2（已验证）；VS Code + CMake Tools + Cortex-Debug 亦可 |
| 硬件 | STM32F103C8T6 最小系统板（Blue Pill）或自制板：8 MHz HSE 晶振、PC13 接 LED、SWD 接口 |

> **Windows 上最省事的方案**是安装 [STM32CubeCLT](https://www.st.com/en/development-tools/stm32cubeclt.html)，它通常已包含 GCC、CMake、Ninja、OpenOCD / ST-LINK GDB server 等工具。安装完成后确认以下命令在终端中可用：
>
> ```powershell
> arm-none-eabi-gcc --version
> cmake --version
> ninja --version
> ```
>
> 只装了 CLion 也可以：直接使用 CLion 内置的 CMake / Ninja 即可构建。

---

## 快速开始

```bash
# 1. 获取模板（二选一）
git clone https://github.com/Cossiant/STM32-HAL-Cmake-template.git MyProject
# 或者在 GitHub 页面点击 "Use this template" 创建自己的仓库

cd MyProject

# 2. 配置 Debug 预设（生成 build/Debug 构建目录）
cmake --preset Debug

# 3. 构建
cmake --build --preset Debug

# 4. Release 构建
cmake --preset Release
cmake --build --preset Release
```

构建产物位于 `build/<preset>/`：

| 文件 | 说明 |
|---|---|
| `xxx.elf` | 带调试符号的可执行文件（GDB 调试、OpenOCD 下载） |
| `xxx.hex` | Intel HEX，通用烧录格式 |
| `xxx.bin` | 原始二进制，烧录起始地址 `0x08000000` |
| `xxx.map` | 链接映射表，用于分析 Flash / RAM 占用 |
| `compile_commands.json` | 编译数据库，供 IDE / clangd 索引 |

其中 `xxx` 由 `CMakeLists.txt` 中的 `CMAKE_PROJECT_NAME` 决定（当前为 `STM32-HAL-Cmake-template`）。构建结束时会打印内存占用，例如：

```text
Memory region         Used Size  Region Size  %age Used
           FLASH:       ...
             RAM:       ...
```

常用命令：

| 目的 | 命令 |
|---|---|
| 删除缓存后重新配置 | `cmake --preset Debug --fresh`（需 CMake ≥ 3.24，否则手动删除 `build/`） |
| 直接构建（已配置） | `cmake --build build/Debug` |
| 查看固件大小 | `arm-none-eabi-size build/Debug/STM32-HAL-Cmake-template.elf` |
| 清理构建产物 | 删除 `build/` 目录即可 |

> **使用 CLion**：`File → Open` 选择仓库根目录，CLion 会自动读取 `CMakePresets.json`；在工具栏选择 `Debug` 预设后点击 Build 即可。

---

## 烧录与调试

### OpenOCD（命令行，推荐脚本化）

```bash
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg -c "program build/Debug/STM32-HAL-Cmake-template.elf verify reset exit"
```

- 较老版本的 OpenOCD 请把 `interface/stlink.cfg` 换成 `interface/stlink-v2.cfg`；
- 也可以使用板级配置：`-f board/stm32f1_stlink.cfg`（该文件同时指定接口与目标）。

### STM32CubeProgrammer（图形化）

- 使用 `build/Debug/xxx.hex` 烧录即可；
- 若使用 `.bin`，起始地址填 `0x08000000`。

### CLion + OpenOCD（图形化调试）

1. `File → Open` 打开仓库根目录，选择 `Debug` 预设并构建一次；
2. `Run → Edit Configurations… → + → OpenOCD Download & Run`；
3. Target 选择 `STM32-HAL-Cmake-template`；
4. Board config file 选择 OpenOCD 安装目录下的 `share/openocd/scripts/board/stm32f1_stlink.cfg`（不同安装方式路径不同）；
5. 点击 Debug，CLion 会自动下载固件、启动 GDB 并停在 `main()`。

> 仓库不包含 `.idea` 目录：每个新 clone 打开时都需要按 CLion 提示初始化一次 CMake 配置，OpenOCD 运行配置也需要首次手动创建。这样可以避免把本机的 CMake 状态、路径和窗口布局带进仓库（JetBrains 官方推荐做法）。

### GDB 手动连接

```bash
# 终端 1：启动 OpenOCD，GDB server 监听 localhost:3333
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg

# 终端 2：连接目标并下载固件
arm-none-eabi-gdb build/Debug/STM32-HAL-Cmake-template.elf
(gdb) target extended-remote localhost:3333
(gdb) monitor reset halt
(gdb) load
(gdb) monitor reset init
(gdb) continue
```

---

## 目录结构

```text
STM32-HAL-Cmake-template/
├── CMakeLists.txt                 # 顶层构建脚本（只生成一次，可自由修改）
├── CMakePresets.json              # Debug / Release 预设（Ninja + 工具链文件）
├── LICENSE                        # MIT 许可证（覆盖模板自身代码与文档）
├── .mxproject                     # CubeMX 元数据（记录生成过的文件）
├── STM32-HAL-Cmake-template.ioc   # CubeMX 工程：芯片、时钟、外设、FreeRTOS 配置
├── startup_stm32f103xb.s          # 启动文件
├── STM32F103XX_FLASH.ld           # 链接脚本：64 KB Flash / 20 KB RAM
├── cmake/
│   ├── gcc-arm-none-eabi.cmake    # 默认 GCC 工具链文件（编译 / 链接选项）
│   ├── starm-clang.cmake          # 可选：ST ArmClang 工具链文件
│   └── stm32cubemx/
│       └── CMakeLists.txt         # CubeMX 生成：HAL/FreeRTOS 源文件、宏、头文件路径
├── Core/
│   ├── Inc/
│   │   ├── main.h                 # 引脚 / 外设宏（System_LED = PC13）
│   │   ├── stm32f1xx_hal_conf.h   # HAL 模块使能、HSE_VALUE 等
│   │   ├── stm32f1xx_it.h
│   │   └── FreeRTOSConfig.h       # FreeRTOS 配置（堆、优先级、tick 等）
│   └── Src/
│       ├── main.c                 # 入口：时钟、GPIO、启动调度器、默认任务
│       ├── freertos.c             # RTOS 应用代码（USER CODE 区域）
│       ├── stm32f1xx_it.c         # 中断服务函数
│       ├── stm32f1xx_hal_msp.c    # HAL 底层初始化
│       ├── stm32f1xx_hal_timebase_tim.c  # 用 TIM4 作为 HAL 时基
│       ├── system_stm32f1xx.c
│       ├── syscalls.c             # newlib 系统调用桩（printf 重定向入口）
│       └── sysmem.c
├── Drivers/                       # CMSIS + STM32F1 HAL Driver
├── Middlewares/Third_Party/FreeRTOS/Source/  # FreeRTOS V10.3.1 + CMSIS-RTOS V2
├── .gitignore                     # 忽略 build/、.idea/ 等本地文件
└── build/                         # 构建输出目录（本地生成，勿提交）
```

---

## 默认配置一览

| 项目 | 值 |
|---|---|
| 芯片 | STM32F103C8T6（Cortex-M3，LQFP48） |
| 系统时钟 | 72 MHz（HSE 8 MHz × PLL×9，APB1 36 MHz / APB2 72 MHz） |
| Flash / RAM（链接脚本） | 64 KB / 20 KB |
| 调试接口 | SWD（PA13 / PA14） |
| 板载 LED | PC13，低电平点亮 |
| RTOS | FreeRTOS V10.3.1，CMSIS-RTOS V2，`heap_4`，3 KB 堆 |
| FreeRTOS tick | SysTick，1 kHz（72 MHz / 1000） |
| HAL 时基 | TIM4，1 kHz（仅 `HAL_GetTick()` / `HAL_Delay()`，独立于 SysTick） |
| 默认任务 | `defaultTask`，栈 512 B，优先级 Normal，每周期 700 ms 灭 / 300 ms 亮 |
| 已启用 HAL 模块 | GPIO、TIM、DMA、FLASH、EXTI、PWR、RCC、CORTEX |
| 链接库 | `m`（`--specs=nano.specs`） |
| 参考占用（Debug 构建） | Flash ≈ 17 KB（26%），RAM ≈ 8 KB（40%） |

---

## 构建系统说明

本工程采用 **CubeMX 生成 + 用户维护** 的双层结构：

| 文件 / 目录 | 维护者 | 说明 |
|---|---|---|
| `STM32-HAL-Cmake-template.ioc`、`.mxproject` | CubeMX | 芯片、时钟、外设、RTOS 配置，通过 CubeMX 修改 |
| `cmake/stm32cubemx/CMakeLists.txt` | CubeMX | 每次生成都会覆盖，**不要手改** |
| `Core/`、`startup_*.s`、`*.ld` | CubeMX | 只在 `USER CODE BEGIN/END` 之间写业务代码；链接脚本可按需手改 |
| `Drivers/`、`Middlewares/` | CubeMX | 随固件包复制，一般无需修改 |
| `CMakeLists.txt` | 用户 | 顶层构建入口，添加自己的源码 / 库 |
| `CMakePresets.json`、`cmake/*.cmake` | 用户 | 预设、工具链、编译与链接选项 |
| `.idea/` | CLion | 不随仓库提供，由 CLion 在本机生成并整体忽略 |

关键机制：

1. `CMakePresets.json` 定义 `Debug` / `Release` 两个预设，构建目录固定为 `build/<preset>`，并指定 `cmake/gcc-arm-none-eabi.cmake` 作为工具链文件；
2. 工具链文件设置 `-mcpu=cortex-m3`、链接脚本、`--specs=nano.specs`、`-Wl,--gc-sections`、`-Wl,--print-memory-usage`；
3. `cmake/stm32cubemx/CMakeLists.txt` 把 HAL 驱动与 FreeRTOS 分别编译成 `OBJECT` 库（`STM32_Drivers`、`FreeRTOS`），再链接到主程序；
4. 顶层 `CMakeLists.txt` 末尾的 `POST_BUILD` 命令调用 `objcopy` 生成 `.hex` / `.bin`；
5. `CMAKE_EXPORT_COMPILE_COMMANDS=TRUE` 生成 `compile_commands.json`，供 clangd / CLion 索引。

---

## 定制指南

### 1. 修改工程名 / 产物名

1. 修改 `CMakeLists.txt` 中的 `set(CMAKE_PROJECT_NAME MyProject)`（第 22 行），`.elf/.hex/.bin/.map` 名称会自动跟随；
2. 如需修改 CubeMX 工程名：用 CubeMX 打开 `.ioc`，在 `Project Manager` 中修改 `Project Name` 后重新生成（或另存为新的 `.ioc`）；
3. `.idea/` 中残留的旧名称配置可以删除，交给 CLion 重新生成（可选）。

> 想完全丢弃模板的提交历史：删除 `.git` 目录后重新 `git init`（Windows：`Remove-Item -Recurse -Force .git`），或直接使用 GitHub 的 *Use this template*。

### 2. 更换芯片 / 修改时钟

1. 用 CubeMX 打开 `.ioc`，切换 MCU、配置时钟树与外设，保持 `Project Manager → Toolchain/IDE = CMake`，点击 `GENERATE CODE`；
2. CubeMX 会自动更新 `cmake/stm32cubemx/CMakeLists.txt`、`Core/`、`Drivers/`、启动文件与链接脚本；
3. 仍需**手动同步** `cmake/gcc-arm-none-eabi.cmake`：
   - 把 `TARGET_FLAGS` 中的 `-mcpu=cortex-m3` 改为新内核（如 `cortex-m4`）；
   - M4F / M7 需要追加 `-mfpu=fpv4-sp-d16 -mfloat-abi=hard`（C、ASM、CXX 编译与链接参数都要加）；
   - 把 `-T "${CMAKE_SOURCE_DIR}/STM32F103XX_FLASH.ld"` 中的链接脚本文件名换成新芯片对应的 `.ld`；
4. 若外部晶振频率有变化，确认 `Core/Inc/stm32f1xx_hal_conf.h` 中的 `HSE_VALUE` 与 CubeMX 时钟树一致；
5. 换芯片后建议删除 `build/` 目录重新 configure。

### 3. 用 CubeMX 新增外设（UART / I2C / SPI / ADC…）

- 在 `.ioc` 中配置 → `GENERATE CODE`；
- 对应的 `MX_xxx_Init()` 会自动出现在 `main.c`，HAL 源文件也会加入 `cmake/stm32cubemx/CMakeLists.txt`；
- **自己的代码只写在 `USER CODE BEGIN/END` 之间**，重新生成不会丢失；
- 如果绕过 CubeMX 手动启用 HAL 模块，需要同时打开 `stm32f1xx_hal_conf.h` 中对应的 `HAL_xxx_MODULE_ENABLED`，并把 `Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_xxx.c` 加入构建（推荐仍然通过 CubeMX 勾选）。

### 4. 添加自己的源码 / 静态库

在**顶层** `CMakeLists.txt` 中添加（不要改 `cmake/stm32cubemx/CMakeLists.txt`，它每次生成都会被覆盖）：

```cmake
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    App/Src/app.c
)
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    App/Inc
)
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE mylib)
```

建议按 `App/`、`Bsp/`、`Lib/` 等目录组织业务代码，与 CubeMX 生成的 `Core/`、`Drivers/`、`Middlewares/` 分开，便于后续重新生成。

### 5. printf 重定向到串口

`Core/Src/syscalls.c` 中的 `_write()` 是 weak 的，最终调用 weak 的 `__io_putchar()`。在用户代码中实现它即可：

```c
int __io_putchar(int ch)
{
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, HAL_MAX_DELAY);
    return ch;
}
```

工具链文件已包含 `--specs=nano.specs`。nano printf 默认不支持 `%f`，需要打印浮点数时在链接参数中追加 `-u _printf_float`。

### 6. FreeRTOS 堆与任务

- `Core/Inc/FreeRTOSConfig.h` 中 `configTOTAL_HEAP_SIZE` 当前为 **3072 字节**（`heap_4`）。每创建一个任务、队列、信号量都会从该堆分配内存，`osThreadNew()` 返回 `NULL` 通常意味着堆不够；
- 默认任务栈为 `128 * 4 = 512` 字节（见 `main.c` 的 `defaultTask_attributes`）；
- 增大堆时注意总 RAM 只有 20 KB，且 FreeRTOS 堆与全局变量、主栈共享 RAM；必要时裁剪不用的 HAL 模块或减小任务栈。

### 7. 优化等级 / 编译选项

- Debug：`-O0 -g3`；Release：`-Os -g0`，定义在 `cmake/gcc-arm-none-eabi.cmake`；
- 默认开启 `-Wall`，但未开启 `-Werror`；
- 修改工具链文件后建议重新 configure（删除 `build/`，或 CMake ≥ 3.24 时使用 `cmake --preset Debug --fresh`），确保新选项生效。

### 8. 切换到 ST ArmClang 工具链（可选）

1. 安装 STM32CubeCLT（包含 ST ArmClang），确保 `starm-clang` 在 `PATH` 中；
2. 把 `CMakePresets.json` 中的 `toolchainFile` 改为 `${sourceDir}/cmake/starm-clang.cmake`；
3. 按需修改 `cmake/starm-clang.cmake` 中的 `STARM_TOOLCHAIN_CONFIG`（`STARM_PICOLIBC` / `STARM_NEWLIB` / `STARM_HYBRID`）；
4. 使用 `STARM_HYBRID` 时需要额外设置 `CLANG_GCC_CMSIS_COMPILER`、`GCC_TOOLCHAIN_ROOT` 环境变量；
5. 切换后删除 `build/` 重新 configure。

---

## 常见问题（FAQ）

**Q：`cmake: command not found` / `ninja: command not found`？**

安装 CMake 和 Ninja 并把 `bin` 目录加入 `PATH`（或安装 STM32CubeCLT），然后重开终端；CLion 用户可直接使用内置版本。

**Q：`arm-none-eabi-gcc: No such file or directory`？**

工具链未安装或不在 `PATH` 中。用 `where.exe arm-none-eabi-gcc`（Windows）确认；注意从 IDE 启动的终端与系统终端 `PATH` 可能不同。

**Q：配置时报 `CMake 3.22 or higher is required`？**

升级 CMake。`CMakePresets.json` 使用的 v3 格式也要求 CMake ≥ 3.21。

**Q：编译通过，但 LED 不闪？**

- PC13 的 LED 是**低电平点亮**，当前任务大部分时间输出高电平（灭），每周期有 300 ms 低电平（亮）；
- 板子没有 8 MHz HSE 晶振时，`SystemClock_Config()` 会进入 `Error_Handler()` 死循环。可在 CubeMX 中改用 HSI 时钟源后重新生成；
- 确认烧录的是最新固件，必要时复位或重新上电；
- 若你的板子 LED 是高电平点亮，交换 `main.c` 中 `StartDefaultTask()` 里的 `GPIO_PIN_SET` / `GPIO_PIN_RESET`。

**Q：无法下载 / 调试器连不上？**

- SWD 接线：`SWDIO(PA13)`、`SWCLK(PA14)`、`GND`、`3.3V`；`BOOT0` 接 GND；
- 检查目标板供电、ST-Link 驱动与固件；
- OpenOCD 配置选对（STM32F1 用 `target/stm32f1x.cfg` 或 `board/stm32f1_stlink.cfg`）；
- 若芯片被读保护或进入低功耗，先用 `openocd ... -c "init; reset halt"` 或 STM32CubeProgrammer 全片擦除。

**Q：修改 `.ioc` 重新生成后编译报错？**

- 确认 CubeMX 中 `Toolchain/IDE` 仍为 `CMake`；
- 删除 `build/` 后重新 `cmake --preset Debug`，避免旧缓存引用已删除的文件；
- 若提示找不到新增的 HAL 源文件，检查 `cmake/stm32cubemx/CMakeLists.txt` 是否已更新（CubeMX 6.x 会自动更新）。

**Q：链接报 `region RAM overflowed`？**

Release 构建通常更小；关闭不用的 HAL 模块、减小 `configTOTAL_HEAP_SIZE` 或任务栈、避免大缓冲区；把大型常量数据放到 Flash（加 `const`）。

**Q：FreeRTOS 下 `HAL_Delay()` 还能用吗？**

能。**FreeRTOS 的调度 tick 来自 SysTick**（`cmsis_os2.c` 中的 `SysTick_Handler` → `xPortSysTickHandler()`），**HAL 的时基才是 TIM4**（`stm32f1xx_hal_timebase_tim.c`，1 kHz），两者相互独立。因此 `HAL_Delay()` / `HAL_GetTick()` 不受 RTOS 影响；但 `HAL_Delay()` 是忙等，长延时请用 `osDelay()`。

**Q：如何从上游模板同步更新？**

```bash
# 用本模板创建新仓库后，origin 是模板仓库时先改名
git remote rename origin template
git remote add origin <你的仓库地址>

# 之后同步模板更新
git fetch template
git merge template/main      # 或 git cherry-pick 需要的提交
```

通过 GitHub *Use this template* 创建的仓库与原模板没有关联，需要按上面的方式手动添加 remote。

**Q：为什么不用 STM32CubeIDE / Keil 工程？**

CMake 工程不绑定 IDE：CLion、VS Code、命令行都能构建，也方便接入 CI；同时保留 CubeMX 重新生成能力。

---

## 版本与许可证

| 组件 | 版本 |
|---|---|
| STM32CubeMX | 6.17.0 |
| STM32Cube FW_F1 | V1.8.7 |
| STM32F1 HAL Driver | 1.1.10 |
| FreeRTOS | V10.3.1（`heap_4`） |
| CMSIS-RTOS | V2（`cmsis_os2`） |
| CMSIS Core | 随 FW_F1 V1.8.7 提供 |
| CMake | ≥ 3.22（验证时使用 4.2.2） |
| arm-none-eabi-gcc | 13.3.1（GNU Tools for STM32） |

许可证：

- **本模板自身**（顶层构建脚本、`cmake/` 工具链文件、`README.md`、`.gitignore` 等模板新增内容）：**MIT**，见根目录 [LICENSE](LICENSE)。
- `Drivers/STM32F1xx_HAL_Driver/`：BSD-3-Clause（ST）
- `Drivers/CMSIS/`：Apache-2.0（Arm）
- `Middlewares/Third_Party/FreeRTOS/`：MIT（Amazon）
- CubeMX 生成的 `Core/`、启动文件等保留文件头中的原有版权与许可；根目录的 MIT 协议**不会改变**第三方组件（HAL、CMSIS、FreeRTOS 等）的授权，重新分发时请保留各目录自带的 LICENSE 文件。
- 使用本模板创建自己的工程时，你可以自由选择该工程的许可证；建议同时保留上述第三方 LICENSE 文件。

---

## 参考链接

- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)
- [STM32CubeF1 固件包](https://github.com/STMicroelectronics/STM32CubeF1)
- [FreeRTOS](https://www.freertos.org/)
- [GNU Arm Embedded Toolchain](https://developer.arm.com/downloads/-/gnu-rm)
- [OpenOCD](https://openocd.org/)
- [CMake Presets 文档](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html)
- [CLion 嵌入式开发（OpenOCD）](https://www.jetbrains.com/help/clion/embedded-development.html)

---

欢迎提交 Issue / PR 来改进这个模板。如果它帮到了你，点个 Star ⭐ 就是最好的支持。
