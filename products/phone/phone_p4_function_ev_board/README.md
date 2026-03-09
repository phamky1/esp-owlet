# ESP32-P4-Function-EV-Board Running ESP-Brookesia Phone Example

[中文版本](./README_CN.md)

This example demonstrates how to run the ESP-Brookesia Phone on the [ESP32-P4-Function-EV-Board](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32p4/esp32-p4-function-ev-board/index.html) with a `1024 x 600` resolution UI stylesheet.

This project can be built as a **standalone repository**. The required components (`brookesia_core`, `brookesia_app_squareline_demo`) are automatically fetched from the [esp-owlet](https://github.com/phamky1/esp-owlet) repository via the ESP-IDF Component Manager.

## Getting Started

### Hardware Requirements

* An ESP32-P4-Function-EV-Board with a `1024 x 600` resolution LCD screen.

### ESP-IDF Required

- This example supports IDF release/v5.3 and later branches. By default, it runs on IDF release/v5.3.
- Please follow the [ESP-IDF Programming Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html) to set up the development environment. **We highly recommend** you [Build Your First Project](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/index.html#build-your-first-project) to get familiar with ESP-IDF and make sure the environment is set up correctly.

### Get the Repository

To build this project as a standalone repository, clone it to your local PC:

```
git clone https://github.com/phamky1/phone_p4_function_ev_board.git
cd phone_p4_function_ev_board
```

The ESP-IDF Component Manager will automatically download the required components from the [esp-owlet](https://github.com/phamky1/esp-owlet) repository during the build process.

### Configuration

Run `idf.py menuconfig` and modify the esp-brookesia configuration.

## How to Use the Example

### Build and Flash the Example

Build the project and flash it to the board, then run monitor tool to view serial output (replace `PORT` with your board's serial port name):

```c
idf.py -p PORT flash monitor
```

To exit the serial monitor, type `Ctrl-]`.

See the [ESP-IDF Getting Started Guide](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html) for full steps to configure and use ESP-IDF to build projects.

## Dependencies

This project depends on the following components from the [esp-owlet](https://github.com/phamky1/esp-owlet) repository:

| Component | Path in esp-owlet | Description |
|-----------|-------------------|-------------|
| `brookesia_core` | `core/brookesia_core` | Core framework component with GUI, systems, and services |
| `brookesia_app_squareline_demo` | `apps/brookesia_app_squareline_demo` | SquareLine Studio demo application |

These components are fetched automatically via the ESP-IDF Component Manager using the git references defined in `main/idf_component.yml`.

## Technical Support and Feedback

Please use the following feedback channels:

- For technical queries, go to the [esp32.com](https://esp32.com/viewforum.php?f=22) forum.
- For a feature request or bug report, create a [GitHub issue](https://github.com/phamky1/esp-owlet/issues).

We will get back to you as soon as possible.
