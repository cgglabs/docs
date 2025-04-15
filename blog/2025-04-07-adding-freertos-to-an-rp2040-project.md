---
slug: rp2040-freertos
title: Adding FreeRTOS to an RP2040 Project
authors: carlos
image: /img/freertos_blog_post.png
tags: [RP2040, FreeRTOS]
---

We at CGG Labs love the RP2040 :heart: 

Here's how to use it with FreeRTOS to unlock the power of full preemptive scheduling.

<!-- truncate -->
Or you can use our [template](https://github.com/cgglabs/Pico-FreeRTOS) and get on with building. 

# Adding FreeRTOS to an RP2040 Project in VSCode

## Prerequisites

- **Pico SDK**: Installed and configured.
- **VSCode**: Equipped with [CMake Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) and [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) extensions.
- **RP2040 Project**: A working project set up in VSCode with the Pico SDK.
- **Git**: Installed for cloning FreeRTOS.

## Step-by-Step Instructions

### 1. Download FreeRTOS

Clone the FreeRTOS repository or download a release:

```bash
git clone https://github.com/FreeRTOS/FreeRTOS.git --recurse-submodules
```

Alternatively, grab a zip from [FreeRTOS GitHub releases](https://github.com/FreeRTOS/FreeRTOS/releases) and extract it.

### 2. Copy FreeRTOS Files to Your Project

Organize FreeRTOS files in your project:

- Create a `FreeRTOS` directory in your project root.
- Copy the following from the FreeRTOS repository to `FreeRTOS`:
  - `FreeRTOS/Source` (core files).
  - `FreeRTOS/Source/portable/GCC/ARM_CM0` (Cortex-M0 port for RP2040).
  - `FreeRTOS/Source/portable/MemMang/heap_4.c` (heap_4 is a common memory manager; others like `heap_1.c` can also be used).
- Copy `FreeRTOS/Source/include` to `FreeRTOS/include`.

### 3. Add FreeRTOS Configuration File

Create `FreeRTOSConfig.h` in the `FreeRTOS/include` directory with a minimal configuration:

```c
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

#define configUSE_PREEMPTION 1
#define configUSE_IDLE_HOOK 0
#define configUSE_TICK_HOOK 0
#define configCPU_CLOCK_HZ 125000000 // 125MHz
#define configTICK_RATE_HZ ((TickType_t)1000) // 1ms tick
#define configMAX_PRIORITIES (5)
#define configMINIMAL_STACK_SIZE ((uint16_t)128)
#define configTOTAL_HEAP_SIZE ((size_t)(10 * 1024)) // 10KB heap
#define configMAX_TASK_NAME_LEN (16)
#define configUSE_16_BIT_TICKS 0
#define configIDLE_SHOULD_YIELD 1
#define configUSE_MUTEXES 1
#define configQUEUE_REGISTRY_SIZE 8

/* Memory allocation definitions */
#define configSUPPORT_DYNAMIC_ALLOCATION 1
#define configSUPPORT_STATIC_ALLOCATION 0

/* Interrupt priorities */
#define configKERNEL_INTERRUPT_PRIORITY 255
#define configMAX_SYSCALL_INTERRUPT_PRIORITY 191

/* Map to Pico SDK */
#define configUSE_TIMERS 1
#define configTIMER_TASK_PRIORITY (configMAX_PRIORITIES - 1)
#define configTIMER_QUEUE_LENGTH 10
#define configTIMER_TASK_STACK_DEPTH configMINIMAL_STACK_SIZE

#define configENABLE_MPU 0

/* Set the following INCLUDE_* constants to 1 to include the named API function,
 * or 0 to exclude the named API function.  Most linkers will remove unused
 * functions even when the constant is 1. */
#define INCLUDE_vTaskPrioritySet               1
#define INCLUDE_uxTaskPriorityGet              1
#define INCLUDE_vTaskDelete                    1
#define INCLUDE_vTaskSuspend                   1
#define INCLUDE_vTaskDelayUntil                1
#define INCLUDE_vTaskDelay                     1
#define INCLUDE_xTaskGetSchedulerState         1
#define INCLUDE_xTaskGetCurrentTaskHandle      1
#define INCLUDE_uxTaskGetStackHighWaterMark    0
#define INCLUDE_xTaskGetIdleTaskHandle         0
#define INCLUDE_eTaskGetState                  0
#define INCLUDE_xTimerPendFunctionCall         0
#define INCLUDE_xTaskAbortDelay                0
#define INCLUDE_xTaskGetHandle                 0
#define INCLUDE_xTaskResumeFromISR             1

#endif /* FREERTOS_CONFIG_H */
```

Customize `configTOTAL_HEAP_SIZE`, `configMAX_PRIORITIES`, etc., based on your project needs.

### 4. Update CMakeLists.txt

Modify your `CMakeLists.txt` to include FreeRTOS:

```cmake
# Add FreeRTOS as a library
add_library(freertos
    FreeRTOS/Source/croutine.c
    FreeRTOS/Source/event_groups.c
    FreeRTOS/Source/list.c
    FreeRTOS/Source/queue.c
    FreeRTOS/Source/stream_buffer.c
    FreeRTOS/Source/tasks.c
    FreeRTOS/Source/timers.c
    FreeRTOS/Source/portable/GCC/ARM_CM0/port.c
    FreeRTOS/Source/portable/GCC/ARM_CM0/portasm.c
    FreeRTOS/Source/portable/GCC/ARM_CM0/mpu_wrappers_v2_asm.c
    FreeRTOS/Source/portable/MemMang/heap_4.c
)

# Include FreeRTOS headers
target_include_directories(freertos PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/FreeRTOS/include
    ${CMAKE_CURRENT_LIST_DIR}/FreeRTOS/Source/portable/GCC/ARM_CM0
    ${CMAKE_CURRENT_LIST_DIR}/FreeRTOS
)

# Link FreeRTOS to your executable
target_link_libraries(your_executable_name PRIVATE
    pico_stdlib
    freertos
)
```

- Replace `your_executable_name` with your target's name (from `add_executable`).
- Ensure `pico_stdlib` is included for Pico SDK functionality.

### 5. (optional) Implement Stack Overflow Hook

If `configCHECK_FOR_STACK_OVERFLOW` is enabled, add the hook in your main C file or a utilities file:

```c
void vApplicationStackOverflowHook(TaskHandle_t pxTask, char *pcTaskName) {
    (void)pxTask;
    (void)pcTaskName;
    while (1) {
        // Handle stack overflow (e.g., log error, blink LED, or halt)
    }
}
```

### 6. Write FreeRTOS Code

Include FreeRTOS in your main file and create a sample task:

```c
#include "pico/stdlib.h"
#include <FreeRTOS.h>
#include <task.h>

#define LED_PIN 25

void vTaskExample(void *pvParameters) {
    // Initialize LED pin
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);

    while (1) {
        gpio_put(LED_PIN, 1);           // Turn LED on
        vTaskDelay(pdMS_TO_TICKS(500)); // On for 500ms
        gpio_put(LED_PIN, 0);           // Turn LED off
        vTaskDelay(pdMS_TO_TICKS(500)); // Off for 500ms
    }
}

int main() {
    // Initialize Pico SDK
    stdio_init_all();

    // Create task
    xTaskCreate(vTaskExample, "ExampleTask", 256, NULL, 1, NULL);

    // Start scheduler
    vTaskStartScheduler();

    // Should never reach here
    while (1);
}

```

### 7. Configure VSCode Build

Set up VSCode to build with the Pico SDK toolchain:

- Use the CMake Tools extension.
- Open the Command Palette (`Ctrl+Shift+P`), select **CMake: Configure**.
- Choose your kit (e.g., Pico SDK toolchain).
- Build the project (**CMake: Build** or `F7`).

### 8. Build and Flash

- Build the project in VSCode.
- Flash the `.uf2` file to your RP2040 (drag to the USB drive in BOOTSEL mode).

### 9. Test the Project

- Connect a terminal (e.g., PuTTY or `minicom`) to the RP2040's UART (default: USB CDC).
- Confirm the task prints "Task running" every second.

## Notes

- **Heap Size**: Tune `configTOTAL_HEAP_SIZE` (RP2040 has 264KB SRAM).
- **Stack Size**: Set task stack sizes (e.g., 256 in `xTaskCreate`) to avoid overflows.

## Troubleshooting

If you encounter linker errors or crashes:
- Verify `CMakeLists.txt` paths.
- Check `FreeRTOSConfig.h` settings.
- Adjust stack and heap sizes.
