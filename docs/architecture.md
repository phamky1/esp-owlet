# ESP-Owlet (ESP-Brookesia) Architecture Document

[中文版本](#中文版本-architecture-document)

## Table of Contents

- [1. Overview](#1-overview)
- [2. High-Level Architecture](#2-high-level-architecture)
- [3. Layer Diagram](#3-layer-diagram)
- [4. Component Dependency Graph](#4-component-dependency-graph)
- [5. Core Component Internal Structure](#5-core-component-internal-structure)
- [6. Service Layer Architecture](#6-service-layer-architecture)
- [7. Agent Layer Architecture](#7-agent-layer-architecture)
- [8. Product Composition](#8-product-composition)
- [9. Build System & Configuration Flow](#9-build-system--configuration-flow)
- [10. Data Flow](#10-data-flow)

---

## 1. Overview

ESP-Owlet (based on ESP-Brookesia) is a monorepo framework for building AIoT products on Espressif chips (ESP32-P4, ESP32-S3, etc.). It provides a modular, plugin-based architecture with:

- **GUI Framework** built on LVGL 9.2
- **AI Framework** with multi-provider LLM agent support (OpenAI, Coze, Xiaozhi)
- **Service Layer** with RPC, event pub/sub, and plugin lifecycle management
- **System Shells** for Phone and Speaker form factors
- **Application Plugin System** for modular UI apps

---

## 2. High-Level Architecture

```mermaid
graph TB
    subgraph Products["📱 Products Layer"]
        P1[Phone P4 EV Board]
        P2[Phone S3 LCD EV Board]
        P3[Phone M5Stack Core S3]
        P4[Phone S3 Box 3]
        P5[Speaker]
    end

    subgraph Apps["📦 Application Layer"]
        A1[Settings]
        A2[Calculator]
        A3[Game 2048]
        A4[Timer]
        A5[POS]
        A6[AI Profile]
        A7[SquareLine Demo]
        A8[USB NCM]
    end

    subgraph Agents["🤖 Agent Layer"]
        AG1[Agent Manager]
        AG2[OpenAI Agent]
        AG3[Coze Agent]
        AG4[Xiaozhi Agent]
        AG5[Agent Helper]
    end

    subgraph Expression["🎭 Expression Layer"]
        EX1[Expression Emote]
    end

    subgraph Core["⚙️ Core Layer - brookesia_core"]
        C1[GUI Module]
        C2[AI Framework Module]
        C3[Systems Module]
        C4[Services Module]
    end

    subgraph Services["🔧 Service Layer"]
        S1[Service Manager]
        S2[Audio Service]
        S3[WiFi Service]
        S4[SNTP Service]
        S5[NVS Service]
        S6[Service Helper]
    end

    subgraph Utils["🛠️ Utilities Layer"]
        U1[brookesia_lib_utils]
    end

    subgraph External["📚 External Dependencies"]
        E1[LVGL 9.2]
        E2[ESP-IDF 5.3+]
        E3[Boost ASIO/JSON/Thread]
        E4[GMF Audio Framework]
        E5[Hardware BSPs]
    end

    Products --> Apps
    Products --> Core
    Apps --> Core
    Agents --> Services
    Expression --> Services
    Core --> External
    Services --> Utils
    Utils --> External
```

---

## 3. Layer Diagram

```mermaid
block-beta
    columns 1

    block:ProductLayer["Products Layer (Hardware-Specific Builds)"]
        columns 5
        phone_p4["Phone P4\nEV Board"]
        phone_s3_lcd["Phone S3\nLCD EV Board"]
        phone_m5stack["Phone\nM5Stack S3"]
        phone_box3["Phone\nS3 Box 3"]
        speaker["Speaker\nProduct"]
    end

    space

    block:AppLayer["Application Layer (Pluggable UI Apps)"]
        columns 4
        settings["Settings"]
        calculator["Calculator"]
        game["Game 2048"]
        timer["Timer"]
        pos["POS"]
        ai_profile["AI Profile"]
        sq_demo["SquareLine Demo"]
        usb_ncm["USB NCM"]
    end

    space

    block:MiddleLayer["Middleware Layer"]
        columns 3
        block:AgentBlock["Agent Layer"]
            agent_mgr["Agent Manager"]
            openai["OpenAI"]
            coze["Coze"]
            xiaozhi["Xiaozhi"]
        end
        block:ExprBlock["Expression Layer"]
            expr_emote["Expression\nEmote"]
        end
        block:CoreBlock["Core (brookesia_core)"]
            gui["GUI\n(LVGL Wrapper)"]
            ai_fw["AI Framework"]
            systems["Systems\n(Phone/Speaker)"]
            services_core["Services\n(Storage NVS)"]
        end
    end

    space

    block:ServiceLayer["Service Layer (Plugin-based Microservices)"]
        columns 6
        svc_mgr["Service\nManager"]
        svc_audio["Audio\nService"]
        svc_wifi["WiFi\nService"]
        svc_sntp["SNTP\nService"]
        svc_nvs["NVS\nService"]
        svc_helper["Service\nHelper"]
    end

    space

    block:UtilsLayer["Utilities Layer"]
        columns 1
        utils["brookesia_lib_utils\n(Task Scheduler, State Machine, Plugin System, Profilers, Logging)"]
    end

    space

    block:ExtLayer["External Dependencies"]
        columns 5
        lvgl["LVGL 9.2"]
        idf["ESP-IDF\n5.3+"]
        boost["Boost\nASIO/JSON"]
        gmf["GMF Audio\nFramework"]
        bsp["Hardware\nBSPs"]
    end

    ProductLayer --> AppLayer
    AppLayer --> CoreBlock
    AgentBlock --> ServiceLayer
    ExprBlock --> ServiceLayer
    CoreBlock --> ExtLayer
    ServiceLayer --> UtilsLayer
    UtilsLayer --> ExtLayer
```

---

## 4. Component Dependency Graph

### 4.1 Service Layer Dependencies

```mermaid
graph LR
    subgraph "Service Layer"
        SM[brookesia_service_manager\nv0.7.4]
        SH[brookesia_service_helper\nv0.7.5]
        SA[brookesia_service_audio\nv0.7.2]
        SW[brookesia_service_wifi\nv0.7.4]
        SS[brookesia_service_sntp\nv0.7.1]
        SN[brookesia_service_nvs\nv0.7.2]
    end

    subgraph "Utils"
        LU[brookesia_lib_utils\nv0.7.5]
    end

    subgraph "External"
        AVP[av_processor]
        EWR[esp_wifi_remote\n ESP32-P4 only]
        EH[esp_hosted\n ESP32-P4 only]
        CMU[cmake_utilities]
        BOOST[esp-boost 0.4.*]
    end

    SH -->|public| SM
    SA -->|public| SH
    SA -.->|private| AVP
    SW -->|public| SH
    SW -.->|private, P4 only| EWR
    SW -.->|private, P4 only| EH
    SS -->|public| SH
    SN -->|public| SH
    SM -->|public| LU
    LU -->|public| CMU
    LU -->|public| BOOST
```

### 4.2 Agent Layer Dependencies

```mermaid
graph LR
    subgraph "Agent Layer"
        AM[brookesia_agent_manager\nv0.7.3]
        AH[brookesia_agent_helper\nv0.7.0]
        AC[brookesia_agent_coze\nv0.7.2]
        AO[brookesia_agent_openai\nv0.7.3]
        AX[brookesia_agent_xiaozhi\nv0.7.1]
    end

    subgraph "Service Layer"
        SS[brookesia_service_sntp]
        SA[brookesia_service_audio]
        SH[brookesia_service_helper]
    end

    subgraph "External"
        WS[esp_websocket_client]
        EP[esp_peer]
    end

    AM -->|public| AH
    AM -.->|private| SS
    AM -.->|private| SA
    AH -->|public| SH
    AC -->|public| AM
    AC -.->|private| WS
    AO -->|public| AM
    AO -.->|private| EP
    AX -->|public| AM
```

### 4.3 Full Dependency Graph

```mermaid
graph TB
    subgraph "Products"
        PP4[phone_p4_function_ev_board]
        PS3[phone_s3_lcd_ev_board]
        PM5[phone_m5stack_core_s3]
        PB3[phone_s3_box_3]
        SPK[speaker]
    end

    subgraph "Apps"
        APP_SQ[app_squareline_demo]
        APP_SET[app_settings]
        APP_AI[app_ai_profile]
        APP_CALC[app_calculator]
        APP_GAME[app_game_2048]
        APP_TMR[app_timer]
        APP_POS[app_pos]
        APP_USB[app_usbd_ncm]
    end

    subgraph "Core"
        BC[brookesia_core\nv0.6.0-beta2]
    end

    subgraph "Agents"
        AM[agent_manager]
        AH[agent_helper]
        AC[agent_coze]
        AO[agent_openai]
        AX[agent_xiaozhi]
    end

    subgraph "Expression"
        EE[expression_emote]
    end

    subgraph "Services"
        SM[service_manager]
        SH[service_helper]
        SA[service_audio]
        SW[service_wifi]
        SS[service_sntp]
        SN[service_nvs]
    end

    subgraph "Utils"
        LU[lib_utils]
    end

    %% Products -> Apps
    PP4 --> APP_SQ
    PP4 --> BC
    PS3 --> APP_SQ
    PS3 --> BC
    PM5 --> APP_SQ
    PM5 --> BC
    PB3 --> APP_SQ
    PB3 --> BC
    SPK --> BC
    SPK --> APP_SET
    SPK --> APP_AI
    SPK --> APP_CALC
    SPK --> APP_GAME
    SPK --> APP_TMR
    SPK --> APP_POS
    SPK --> APP_USB

    %% Apps -> Core
    APP_SQ --> BC
    APP_SET --> BC
    APP_AI --> BC
    APP_CALC --> BC
    APP_GAME --> BC
    APP_TMR --> BC
    APP_POS --> BC
    APP_USB --> BC

    %% Agent chain
    AC --> AM
    AO --> AM
    AX --> AM
    AM --> AH
    AM --> SS
    AM --> SA
    AH --> SH

    %% Expression
    EE --> SH

    %% Service chain
    SH --> SM
    SA --> SH
    SW --> SH
    SS --> SH
    SN --> SH
    SM --> LU
```

---

## 5. Core Component Internal Structure

```mermaid
graph TB
    subgraph brookesia_core["brookesia_core (v0.6.0-beta2)"]
        direction TB

        subgraph AI["AI Framework Module"]
            AI_AGENT[Agent Framework]
            AI_EXPR[Expression Framework]
        end

        subgraph GUI["GUI Module"]
            LVGL_W[LVGL Wrapper\n- Animation\n- Canvas\n- Container\n- Display\n- Lock\n- Object\n- Screen\n- Timer]
            ANIM[Animation Player]
            SQ[SquareLine Integration\n- UI Components\n- UI Helpers]
            STYLE[Style Management]
        end

        subgraph SYSTEMS["Systems Module"]
            BASE[Base System\n- App\n- Display\n- Event\n- Manager\n- Core]
            PHONE[Phone System\n- App Launcher\n- Status Bar\n- Navigation Bar\n- Gesture\n- Recents Screen\n- Stylesheets]
            SPEAKER[Speaker System\n- App Launcher\n- Quick Settings\n- Keyboard\n- Gesture\n- AI Buddy\n- Animations\n- Stylesheets]
        end

        subgraph SVC["Services Module"]
            NVS_S[Storage NVS]
        end
    end

    subgraph ext_deps["External Dependencies"]
        LVGL_EXT[lvgl/lvgl 9.2.*]
        IMG_PLAYER[image_player 1.1.*]
        MMAP[esp_mmap_assets 1.3.*]
        COZE_LIB[esp_coze ^0.6]
        GMF[GMF Framework\n- gmf_core\n- gmf_ai_audio\n- gmf_io\n- gmf_misc\n- gmf_audio]
        AUDIO_P[esp_audio_simple_player]
        WS_CLIENT[esp_websocket_client]
        CMAKE_U[cmake_utilities]
        LIB_UTILS[esp-lib-utils 0.3.*]
        ESP_BOOST[esp-boost 0.3.*]
    end

    GUI --> LVGL_EXT
    GUI --> IMG_PLAYER
    GUI --> MMAP
    AI --> COZE_LIB
    AI --> GMF
    AI --> AUDIO_P
    AI --> WS_CLIENT
    brookesia_core --> CMAKE_U
    brookesia_core --> LIB_UTILS
    brookesia_core --> ESP_BOOST
```

### Kconfig-Controlled Conditional Compilation

```mermaid
graph TD
    ROOT[ESP Brookesia Core Config]

    ROOT --> AI_FW[CONFIG_ESP_BROOKESIA_ENABLE_AI_FRAMEWORK\n default: y]
    ROOT --> GUI_EN[CONFIG_ESP_BROOKESIA_ENABLE_GUI\n default: y]
    ROOT --> SVC_EN[CONFIG_ESP_BROOKESIA_ENABLE_SERVICES\n default: y]
    ROOT --> SYS_EN[CONFIG_ESP_BROOKESIA_ENABLE_SYSTEMS\n default: y]

    AI_FW --> AGENT_EN[ENABLE_AGENT]
    AI_FW --> EXPR_EN[ENABLE_EXPRESSION]

    GUI_EN --> ANIM_EN[ENABLE_ANIM_PLAYER]
    GUI_EN --> LVGL_EN[LVGL Debug Logs]
    GUI_EN --> SQ_EN[ENABLE_SQUARELINE\n- UI_COMP\n- UI_HELPERS]
    GUI_EN --> STYLE_EN[Style Debug Logs]

    SYS_EN --> PHONE_EN[ENABLE_PHONE]
    SYS_EN --> SPEAKER_EN[ENABLE_SPEAKER]

    SVC_EN --> NVS_EN[ENABLE_STORAGE_NVS]
```

---

## 6. Service Layer Architecture

```mermaid
classDiagram
    direction TB

    class ServiceManager {
        <<brookesia_service_manager v0.7.4>>
        +Plugin Lifecycle Management
        +RPC Server/Client (TCP)
        +Event Pub/Sub System
        +Function Registration
        +Dual-Worker Thread Model
        ---
        REQUIRES: brookesia_lib_utils
        IDF_REQUIRES: esp_netif
    }

    class ServiceHelper {
        <<brookesia_service_helper v0.7.5>>
        +CRTP-based Type Definitions
        +Service Schemas & Interfaces
        +Service Binding (RAII)
        ---
        REQUIRES: brookesia_service_manager
    }

    class AudioService {
        <<brookesia_service_audio v0.7.2>>
        +Audio Playback
        +Audio Encoding/Decoding
        +Volume Control
        +Playback State Machine
        ---
        REQUIRES: brookesia_service_helper
        PRIVATE: av_processor
    }

    class WiFiService {
        <<brookesia_service_wifi v0.7.4>>
        +WiFi State Machine
        +AP Scanning & Connection
        +SoftAP Mode
        +WiFi Provisioning + DNS
        ---
        REQUIRES: brookesia_service_helper
        PRIVATE: esp_wifi_remote (P4)
    }

    class SNTPService {
        <<brookesia_service_sntp v0.7.1>>
        +NTP Server Management
        +Timezone Configuration
        +Auto-Sync
        ---
        REQUIRES: brookesia_service_helper
    }

    class NVSService {
        <<brookesia_service_nvs v0.7.2>>
        +Namespace-based KV Store
        +NVS Partition Management
        ---
        REQUIRES: brookesia_service_helper
    }

    class LibUtils {
        <<brookesia_lib_utils v0.7.5>>
        +Task Scheduler
        +State Machine
        +Plugin System
        +Memory/Time/Thread Profilers
        +Configurable Logging
    }

    ServiceHelper --> ServiceManager : depends on
    AudioService --> ServiceHelper : depends on
    WiFiService --> ServiceHelper : depends on
    SNTPService --> ServiceHelper : depends on
    NVSService --> ServiceHelper : depends on
    ServiceManager --> LibUtils : depends on
```

---

## 7. Agent Layer Architecture

```mermaid
classDiagram
    direction TB

    class AgentManager {
        <<brookesia_agent_manager v0.7.3>>
        +Agent Lifecycle Management
        +State Machine (idle/listening/thinking/speaking)
        +AFE Event Processing
        +Optional Worker Thread
        +Auto-Register Plugin
        ---
        REQUIRES: brookesia_agent_helper
        PRIVATE: service_sntp, service_audio
    }

    class AgentHelper {
        <<brookesia_agent_helper v0.7.0>>
        +Type-safe Definitions
        +Agent Schemas
        +Calling Interfaces
        ---
        REQUIRES: brookesia_service_helper
    }

    class CozeAgent {
        <<brookesia_agent_coze v0.7.2>>
        +Coze API Integration
        +WebSocket Communication
        +Auto-Register Plugin
        ---
        REQUIRES: brookesia_agent_manager
        PRIVATE: esp_websocket_client
    }

    class OpenAIAgent {
        <<brookesia_agent_openai v0.7.3>>
        +OpenAI API Integration
        +P2P Communication
        +TLS/SSL Support
        +Auto-Register Plugin
        ---
        REQUIRES: brookesia_agent_manager
        PRIVATE: esp_peer
    }

    class XiaozhiAgent {
        <<brookesia_agent_xiaozhi v0.7.1>>
        +Xiaozhi API Integration
        +OTA Support
        +Wake Word Config
        +Auto-Register Plugin
        ---
        REQUIRES: brookesia_agent_manager
    }

    CozeAgent --> AgentManager : extends
    OpenAIAgent --> AgentManager : extends
    XiaozhiAgent --> AgentManager : extends
    AgentManager --> AgentHelper : depends on
```

---

## 8. Product Composition

### 8.1 Phone Products

```mermaid
graph TB
    subgraph "Phone Products"
        PP4["phone_p4_function_ev_board\n(ESP32-P4)\n1024x600"]
        PS3["phone_s3_lcd_ev_board\n(ESP32-S3)\n800x480"]
        PM5["phone_m5stack_core_s3\n(ESP32-S3)\n320x240"]
        PB3["phone_s3_box_3\n(ESP32-S3)\n320x240"]
    end

    subgraph "Hardware BSPs"
        BSP_P4[esp32_p4_function_ev_board\n5.0.*]
        BSP_S3[esp32_s3_lcd_ev_board\n4.0.*]
        BSP_M5[m5stack_core_s3\n3.0.*]
        BSP_B3[esp-box-3\n^3]
    end

    subgraph "Core"
        BC[brookesia_core]
    end

    subgraph "Apps"
        SQ[app_squareline_demo]
    end

    subgraph "sdkconfig Features"
        direction LR
        F1["AI Framework: ❌ disabled"]
        F2["Anim Player: ❌ disabled"]
        F3["Services: ❌ disabled"]
        F4["Speaker System: ❌ disabled"]
        F5["Phone System: ✅ enabled"]
        F6["GUI: ✅ enabled"]
    end

    PP4 --> BSP_P4
    PS3 --> BSP_S3
    PM5 --> BSP_M5
    PB3 --> BSP_B3

    PP4 --> BC
    PP4 --> SQ
    PS3 --> BC
    PS3 --> SQ
    PM5 --> BC
    PM5 --> SQ
    PB3 --> BC
    PB3 --> SQ
```

### 8.2 Speaker Product

```mermaid
graph TB
    subgraph "Speaker Product"
        SPK["speaker\n(ESP32-S3)\n360x360\nFull AI + Audio"]
    end

    subgraph "Hardware Components"
        ECHO[echoear\nAudio Board]
        BQ[bq27220\nBattery Mgmt]
        GMF_AI[gmf_ai_audio\nPatched]
    end

    subgraph "Core"
        BC[brookesia_core]
    end

    subgraph "All Apps"
        A1[app_settings]
        A2[app_ai_profile]
        A3[app_game_2048]
        A4[app_calculator]
        A5[app_timer]
        A6[app_pos]
        A7[app_usbd_ncm]
    end

    subgraph "Patched Dependencies"
        GMF_CORE["gmf_core 0.6.1 (fixed)"]
        WS["esp_websocket_client 1.5.0 (fixed)"]
        LVGL_PORT["esp_lvgl_port 2.6.0 (fixed)"]
    end

    subgraph "sdkconfig Features"
        direction LR
        F1["AI Framework: ✅ enabled"]
        F2["Anim Player: ✅ enabled"]
        F3["Services: ✅ enabled"]
        F4["Speaker System: ✅ enabled"]
        F5["Phone System: ❌ disabled"]
    end

    SPK --> BC
    SPK --> ECHO
    SPK --> BQ
    SPK --> GMF_AI
    SPK --> A1
    SPK --> A2
    SPK --> A3
    SPK --> A4
    SPK --> A5
    SPK --> A6
    SPK --> A7
    SPK --> GMF_CORE
    SPK --> WS
    SPK --> LVGL_PORT
```

---

## 9. Build System & Configuration Flow

```mermaid
flowchart TB
    subgraph "Build Configuration"
        direction TB
        SDK[sdkconfig.defaults\nTarget, features, fonts]
        KCONFIG[Kconfig Flags\nConditional compilation]
        IDF_COMP[idf_component.yml\nDependency declaration]
        CMAKE[CMakeLists.txt\nComponent registration]
    end

    subgraph "Dependency Resolution"
        LOCAL[Local override_path\nMonorepo relative paths]
        GIT[Git References\nRemote repository fetch]
        REGISTRY[ESP Component Registry\nespressif/* packages]
    end

    subgraph "Build Output"
        FIRMWARE[Firmware Binary]
        PARTITION[Partition Table]
        SPIFFS[SPIFFS Data\nAnimations, Assets]
    end

    SDK --> KCONFIG
    KCONFIG --> CMAKE
    IDF_COMP --> LOCAL
    IDF_COMP --> GIT
    IDF_COMP --> REGISTRY
    LOCAL --> CMAKE
    GIT --> CMAKE
    REGISTRY --> CMAKE
    CMAKE --> FIRMWARE
    CMAKE --> PARTITION
    CMAKE --> SPIFFS
```

### Standalone Product Build (phone_p4 example)

```mermaid
flowchart LR
    subgraph "Standalone Repo"
        PROJ[phone_p4_function_ev_board/]
        MAIN[main/\n- main.cpp\n- idf_component.yml]
        SDKCONF[sdkconfig.defaults]
        PART[partitions.csv]
    end

    subgraph "Fetched from esp-owlet via git"
        BC[brookesia_core\ncore/brookesia_core]
        APP[brookesia_app_squareline_demo\napps/brookesia_app_squareline_demo]
    end

    subgraph "Fetched from ESP Registry"
        BSP[esp32_p4_function_ev_board\n5.0.*]
        LVGL[lvgl 9.2.*]
        BOOST_D[esp-boost 0.3.*]
        OTHER[Other espressif/* deps]
    end

    PROJ --> MAIN
    PROJ --> SDKCONF
    PROJ --> PART
    MAIN -->|git: esp-owlet.git\npath: core/brookesia_core| BC
    MAIN -->|git: esp-owlet.git\npath: apps/...| APP
    MAIN -->|ESP Component Manager| BSP
    BC -->|ESP Component Manager| LVGL
    BC -->|ESP Component Manager| BOOST_D
    BC -->|ESP Component Manager| OTHER
```

---

## 10. Data Flow

### 10.1 Plugin Registration Pattern

```mermaid
sequenceDiagram
    participant App as Application Plugin
    participant Core as brookesia_core
    participant SM as Service Manager
    participant AM as Agent Manager

    Note over App,AM: Build-time: Kconfig enables auto-registration

    App->>Core: idf_component_register()
    Core->>Core: Compile with CONFIG flags

    Note over App,AM: Runtime: Plugin Discovery

    SM->>SM: Initialize Worker Threads
    SM->>SM: Start RPC Server
    AM->>SM: Register via Service Helper
    App->>Core: Register via App Registry

    Note over App,AM: Runtime: Event Flow

    AM->>SM: Subscribe to events
    SM->>AM: Dispatch events (pub/sub)
    AM->>App: Trigger UI updates
    App->>Core: Update LVGL display
```

### 10.2 Phone System Runtime Flow

```mermaid
sequenceDiagram
    participant Main as main.cpp
    participant BSP as Board Support Package
    participant LVGL as LVGL Display
    participant Phone as Phone System
    participant Style as Stylesheet
    participant Apps as App Registry

    Main->>BSP: bsp_display_start_with_config()
    Main->>BSP: bsp_display_backlight_on()
    Main->>LVGL: LvLock::registerCallbacks()
    Main->>Phone: new Phone()
    Main->>Style: new Stylesheet(resolution)
    Main->>Phone: addStylesheet() / activateStylesheet()
    Main->>Phone: begin()
    Main->>Apps: initAppFromRegistry()
    Main->>Apps: installAppFromRegistry()
    Main->>LVGL: lv_timer_create(clock_update)

    loop Every 1 second
        LVGL->>Phone: Update status bar clock
    end
```

---

## Directory Structure Summary

```
esp-owlet/
├── agent/                              # 🤖 AI Agent Components
│   ├── brookesia_agent_coze/           #    Coze API + WebSocket
│   ├── brookesia_agent_helper/         #    Type-safe definitions
│   ├── brookesia_agent_manager/        #    Agent lifecycle management
│   ├── brookesia_agent_openai/         #    OpenAI API + P2P
│   └── brookesia_agent_xiaozhi/        #    Xiaozhi API + OTA
│
├── apps/                               # 📦 Application Components
│   ├── brookesia_app_ai_profile/       #    AI profile management
│   ├── brookesia_app_calculator/       #    Calculator utility
│   ├── brookesia_app_game_2048/        #    2048 game
│   ├── brookesia_app_pos/              #    Point of Sale
│   ├── brookesia_app_settings/         #    Device settings
│   ├── brookesia_app_squareline_demo/  #    SquareLine UI demo
│   ├── brookesia_app_timer/            #    Timer application
│   └── brookesia_app_usbd_ncm/         #    USB NCM networking
│
├── core/                               # ⚙️ Core Framework
│   └── brookesia_core/                 #    Monolithic core (v0.6.0-beta2)
│       ├── ai_framework/               #    AI agent & expression framework
│       ├── gui/                        #    LVGL wrapper, animation, style
│       ├── services/                   #    Built-in NVS storage service
│       └── systems/                    #    Phone & Speaker system shells
│
├── expression/                         # 🎭 Expression Components
│   └── brookesia_expression_emote/     #    Emote & animation management
│
├── service/                            # 🔧 Service Components
│   ├── brookesia_service_audio/        #    Audio playback & encoding
│   ├── brookesia_service_helper/       #    CRTP-based helper library
│   ├── brookesia_service_manager/      #    Plugin lifecycle + RPC + events
│   ├── brookesia_service_nvs/          #    NVS key-value storage
│   ├── brookesia_service_sntp/         #    NTP/Timezone service
│   └── brookesia_service_wifi/         #    WiFi state machine
│
├── utils/                              # 🛠️ Utility Library
│   └── brookesia_lib_utils/            #    Task scheduler, profilers, logging
│
├── products/                           # 📱 Product Configurations
│   ├── phone/                          #    Phone form-factor products
│   │   ├── phone_p4_function_ev_board/ #    ESP32-P4 (1024x600)
│   │   ├── phone_s3_lcd_ev_board/      #    ESP32-S3 LCD (800x480)
│   │   ├── phone_m5stack_core_s3/      #    M5Stack Core S3 (320x240)
│   │   └── phone_s3_box_3/             #    ESP-BOX-3 (320x240)
│   └── speaker/                        #    Speaker form-factor product
│       ├── main/                       #    Speaker main application
│       ├── common_components/          #    Patched/custom components
│       ├── bootloader_components/      #    Custom bootloader hooks
│       └── spiffs/                     #    SPIFFS filesystem data
│
├── examples/                           # 📖 Example Projects
│   └── service_console/                #    Service + Agent console demo
│
└── docs/                               # 📄 Documentation
```

---

---

# 中文版本 Architecture Document

## 目录

- [1. 概述](#1-概述)
- [2. 高层架构](#2-高层架构)
- [3. 分层图](#3-分层图)
- [4. 组件依赖关系图](#4-组件依赖关系图)
- [5. 核心组件内部结构](#5-核心组件内部结构)
- [6. 服务层架构](#6-服务层架构)
- [7. Agent 层架构](#7-agent-层架构)
- [8. 产品组合](#8-产品组合)
- [9. 构建系统与配置流程](#9-构建系统与配置流程)
- [10. 数据流](#10-数据流)

---

## 1. 概述

ESP-Owlet（基于 ESP-Brookesia）是一个单仓库（monorepo）框架，用于在乐鑫芯片（ESP32-P4、ESP32-S3 等）上构建 AIoT 产品。它提供了模块化的、基于插件的架构，包括：

- **GUI 框架**：基于 LVGL 9.2 构建
- **AI 框架**：支持多供应商 LLM Agent（OpenAI、Coze、小智）
- **服务层**：提供 RPC、事件发布/订阅和插件生命周期管理
- **系统外壳**：支持手机和音箱两种产品形态
- **应用插件系统**：模块化的 UI 应用

---

## 2. 高层架构

（请参考上方英文版本的 Mermaid 图表，图表内容通用）

---

## 3. 分层说明

| 层级 | 组件 | 说明 |
|------|------|------|
| **产品层** | phone_p4, phone_s3, speaker 等 | 硬件特定的构建配置，包含 BSP、sdkconfig、分区表 |
| **应用层** | brookesia_app_* (8个应用) | 可插拔的 UI 应用，全部依赖 brookesia_core |
| **Agent 层** | brookesia_agent_* (5个组件) | AI/LLM 集成，支持 OpenAI、Coze、小智 |
| **表情层** | brookesia_expression_emote | 表情和动画管理 |
| **核心层** | brookesia_core | 单体核心组件，包含 GUI、AI 框架、系统和服务 |
| **服务层** | brookesia_service_* (6个组件) | 基于插件的微服务，包含 RPC、事件、音频、WiFi 等 |
| **工具层** | brookesia_lib_utils | 任务调度器、状态机、插件系统、性能分析、日志 |
| **外部依赖** | LVGL、ESP-IDF、Boost、GMF、BSP | 第三方和乐鑫官方库 |

---

## 4. 组件版本一览

| 组件 | 版本 | 主要依赖 |
|------|------|----------|
| brookesia_core | 0.6.0-beta2 | lvgl 9.2.*, esp-boost 0.3.*, gmf_core ^0.6 |
| brookesia_lib_utils | 0.7.5 | cmake_utilities 0.*, esp-boost 0.4.* |
| brookesia_service_manager | 0.7.4 | brookesia_lib_utils 0.7.* |
| brookesia_service_helper | 0.7.5 | brookesia_service_manager 0.7.* |
| brookesia_service_audio | 0.7.2 | brookesia_service_helper 0.7.*, av_processor |
| brookesia_service_wifi | 0.7.4 | brookesia_service_helper 0.7.*, esp_wifi_remote (P4) |
| brookesia_service_sntp | 0.7.1 | brookesia_service_helper 0.7.* |
| brookesia_service_nvs | 0.7.2 | brookesia_service_helper 0.7.* |
| brookesia_agent_manager | 0.7.3 | brookesia_agent_helper 0.7.*, service_sntp, service_audio |
| brookesia_agent_helper | 0.7.0 | brookesia_service_helper 0.7.* |
| brookesia_agent_coze | 0.7.2 | brookesia_agent_manager 0.7.*, esp_websocket_client |
| brookesia_agent_openai | 0.7.3 | brookesia_agent_manager 0.7.*, esp_peer |
| brookesia_agent_xiaozhi | 0.7.1 | brookesia_agent_manager 0.7.* |
| brookesia_expression_emote | 0.7.3 | brookesia_service_helper 0.7.*, esp_emote_expression |

---

## 5. 关键设计模式

### 5.1 插件自动注册模式

所有 Agent 和 Service 组件使用 Kconfig 控制的自动注册机制：

```
CONFIG_BROOKESIA_AGENT_MANAGER_ENABLE_AUTO_REGISTER=y
```

在 CMakeLists.txt 中通过链接时符号（link-time symbol）实现：

```cmake
target_link_libraries(${COMPONENT_LIB} INTERFACE "-u ${PLUGIN_SYMBOL}")
```

### 5.2 条件编译模式

核心组件通过 Kconfig 标志控制功能模块的编译：

- `CONFIG_ESP_BROOKESIA_ENABLE_AI_FRAMEWORK` - AI 框架
- `CONFIG_ESP_BROOKESIA_ENABLE_GUI` - GUI 模块
- `CONFIG_ESP_BROOKESIA_ENABLE_SERVICES` - 服务模块
- `CONFIG_ESP_BROOKESIA_ENABLE_SYSTEMS` - 系统模块
- `CONFIG_ESP_BROOKESIA_SYSTEMS_ENABLE_PHONE` - 手机系统
- `CONFIG_ESP_BROOKESIA_SYSTEMS_ENABLE_SPEAKER` - 音箱系统

### 5.3 依赖解析模式

组件依赖通过三种方式解析：

1. **override_path**：单仓库内的本地相对路径引用
2. **git + path**：从远程 Git 仓库获取特定子目录
3. **ESP Component Registry**：从乐鑫组件注册表获取

---

## 6. 产品与核心的关系

### Phone P4（作为独立仓库构建）

```
phone_p4_function_ev_board/  （独立仓库）
└── main/idf_component.yml
    ├── brookesia_core ──────── git: esp-owlet.git → core/brookesia_core
    ├── brookesia_app_squareline_demo ── git: esp-owlet.git → apps/...
    └── esp32_p4_function_ev_board ──── ESP Component Registry
```

### Speaker（在单仓库内构建）

```
products/speaker/  （单仓库内）
└── main/idf_component.yml
    ├── brookesia_core ──────── override_path: ../../../core/brookesia_core
    ├── 7 个应用组件 ────────── override_path: ../../../apps/...
    ├── echoear, bq27220 ───── override_path: ../common_components/...
    └── 多个外部依赖 ────────── ESP Component Registry（部分固定版本）
```
