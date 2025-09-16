AVB-Disabler-with-Rescue: 分离式AVB禁用与救砖模块
 
A separated module for disabling Android Verified Boot (AVB) and auto-recovering from boot failures
 
一、项目介绍（Project Intro）
 
本项目是一套针对Android设备的分离式AVB禁用与救砖解决方案，核心解决两大问题：
 
1. 安全禁用Android Verified Boot（AVB）验证，突破系统签名限制，支持自定义模块安装；
2. 应对AVB禁用可能导致的“变砖”风险，提供自动+手动双重恢复机制，保障设备可用性。
 
相比传统方案，本项目通过“适配层+应急层+校验层”优化，覆盖更多设备型号、降低操作门槛，同时强化安全管控，仅推荐用于合法的开发测试场景（禁用AVB会降低设备安全性，请勿用于日常主力设备的敏感场景）。
 
二、核心功能（Core Features）
 
1. AVB禁用模块（AVB-Disabler）
 
- 多设备适配：自动识别设备型号，加载对应 vbmeta 分区路径（支持Samsung S22、Xiaomi Mi13、Google Pixel7等主流机型，含动态分区设备）；
- 预安装校验：自动检测Android版本（需8.0+）、Magisk版本（需20.4+）、Bootloader解锁状态，不符合条件则终止安装；
- 安全提醒：安装时通过Magisk弹窗提示风险，需用户确认后继续，避免误操作；
- 属性伪装：伪造系统属性（如 ro.boot.verifiedbootstate=green ），规避部分系统对AVB状态的检测。
 
2. 救砖模块（Brick-Rescue）
 
- 自动备份：首次启动时自动备份原始 vbmeta 分区至 /data/adb/avb_backup/ ，为恢复预留基础；
- 智能启动检测：30秒超时判定（适配低配设备），同时检测 sys.boot_completed 属性与Magisk服务状态，减少误判；
- 分级恢复机制：
- 3次启动失败：自动进入安全模式重试；
- 5次启动失败：自动恢复原始 vbmeta 并禁用AVB模块，重启后设备恢复正常；
- 离线应急恢复：提供 offline_recovery.zip （含Windows/Linux/macOS版Fastboot工具），无ADB连接时可通过电脑操作；
- 状态日志：生成 /sdcard/boot_failure_report.txt ，记录失败原因与恢复建议，便于排查问题。
 
3. 组合安装包（AVB-Disabler-With-Rescue）
 
- 一键双装：一次刷入即可完成AVB禁用模块与救砖模块的安装；
- 增量更新：检测已安装模块版本，仅更新差异文件，节省安装时间；
- 设备白名单：安装前匹配 docs/device_list.md 中的已验证设备，不支持机型会提示风险。
 
三、环境要求（Requirements）
 
1. 设备端：
- Android 8.0及以上版本；
- Bootloader已解锁；
- 已安装Magisk 20.4及以上（需正常激活，支持Magisk Hide）；
- 预留至少100MB存储空间（用于备份 vbmeta 和日志文件）。
2. 电脑端（可选，用于应急恢复）：
- 安装Android SDK Platform Tools（含ADB、Fastboot工具）；
- Windows/Linux/macOS系统均可（离线恢复包适配多系统）。
 
四、安装步骤（Installation Guide）
 
方式1：组合包一键安装（推荐）
 
1. 下载项目根目录下的 AVB-Disabler-With-Rescue.zip ；
2. 打开Magisk Manager → 点击“模块” → “从存储安装” → 选择上述ZIP文件；
3. 等待安装完成（期间会弹窗提示风险，点击“确认”）；
4. 重启设备，模块自动生效（可通过Magisk Manager查看模块状态）。
 
方式2：分别安装模块（进阶）
 
1. 先下载 AVB-Disabler.zip ，按上述步骤在Magisk中安装，重启设备；
2. 再下载 Brick-Rescue.zip ，重复安装步骤，再次重启；
3. 验证：进入 /data/adb/modules/ 目录，确认 avbdisabler 和 brickrescue 两个文件夹存在，即为安装成功。
 
五、手动恢复操作（Manual Recovery）
 
当设备无法启动（如卡在开机logo），可通过以下方式手动恢复：
 
1. 通过ADB恢复（推荐，需设备开启ADB调试）
 
1. 电脑连接设备，打开命令提示符（Windows）或终端（Linux/macOS）；
2. 输入指令：
bash  
adb shell sh /data/adb/modules/brickrescue/system/bin/avb_recovery --force
 
3. 设备会自动恢复原始 vbmeta 并重启，重启后AVB禁用模块已被禁用，设备恢复正常。
 
2. 通过Recovery模式恢复（无ADB时）
 
1. 长按设备“电源键+音量上键”，进入TWRP或其他自定义Recovery；
2. 打开“文件管理器”，删除 /data/adb/modules/avbdisabler 文件夹；
3. 重启设备，AVB禁用模块失效，设备恢复默认启动流程。
 
3. 通过Fastboot恢复（极端情况，设备无法进入系统/Recovery）
 
1. 电脑解压 emergency/offline_recovery.zip ，找到对应系统的Fastboot工具；
2. 设备进入Fastboot模式（长按“电源键+音量下键”），连接电脑；
3. 执行指令（需提前准备设备原始 vbmeta.img ，可从官方固件中提取）：
bash  
# Windows系统
fastboot.exe flash vbmeta vbmeta_original.img

# Linux/macOS系统
./fastboot flash vbmeta vbmeta_original.img
 
4. 指令执行完成后，输入 fastboot reboot 重启设备。
 
六、常见问题（Troubleshooting）
 
Q1：安装后设备无法启动？
 
A1：进入TWRP删除 /data/adb/modules/avbdisabler ，或执行 adb shell sh /data/adb/modules/brickrescue/system/bin/avb_recovery --force ，恢复原始 vbmeta 。
 
Q2：救砖模块不工作，启动失败后未自动恢复？
 
A2：
 
1. 确认Magisk已正常激活（进入 /data/adb/magisk/ ，查看 magiskd.pid 文件是否存在）；
2. 检查 /data/adb/brick_rescue 目录权限是否为 0755 （可通过Recovery修改权限）。
 
Q3：提示“Device not supported”（设备不支持）？
 
A3：查看 docs/device_list.md 确认设备是否在已验证列表中；若为小众机型，可手动编辑 AVB-Disabler/device_adapt/vbmeta_paths.conf ，添加设备的 vbmeta 分区路径（需自行查询设备分区信息）。
 
Q4：恢复 vbmeta 后，Magisk提示“未安装”？
 
A4：需重新刷入Magisk补丁包（可通过Fastboot刷写 magisk_patched-xxx_boot.img ，或通过Recovery刷入Magisk安装包）。
 
七、文件结构（File Structure）
 
plaintext  
AVB-Disabler-with-Rescue/
├── AVB-Disabler/                # AVB禁用模块
│   ├── device_adapt/            # 设备适配目录
│   │   ├── vbmeta_paths.conf    # 机型-分区路径配置
│   │   └── prop_override.sh     # 设备专属属性脚本
│   ├── common/                  # 核心脚本
│   │   ├── post-fs-data.sh      # 启动后执行脚本
│   │   └── system_hook/         # 系统Hook脚本
│   ├── system/                  # 系统配置
│   ├── check_compatibility.sh   # 兼容性检测脚本
│   └── module.prop              # 模块信息
│
├── Brick-Rescue/                # 救砖模块
│   ├── emergency/               # 应急恢复目录
│   │   ├── offline_recovery.zip # 离线恢复包
│   │   └── fastboot_cmd.txt     # Fastboot指令模板
│   ├── status/                  # 状态日志目录
│   ├── system/bin/              # 核心工具（avb_stealthd、diag_system等）
│   └── module.prop              # 模块信息
│
├── AVB-Disabler-With-Rescue/    # 组合安装包
│   ├── modules/                 # 子模块压缩包
│   ├── verify/                  # 校验目录（MD5、兼容性检测）
│   └── scripts/                 # 安装脚本
│
├── docs/                        # 文档目录
│   ├── troubleshooting.md       # 故障排查指南
│   └── device_list.md           # 已验证设备列表
│
├── .gitignore                   # Git忽略文件配置
├── LICENSE                      # 开源许可证（见“九、许可证”）
└── README.md                    # 本自述文件
 
 
八、注意事项（Notes）
 
1. 合法性与安全性：
- 仅用于合法的开发测试，禁止用于破解他人设备、侵犯隐私或其他违法场景；
- 禁用AVB后，设备易受恶意软件攻击，请勿存储银行卡、密码等敏感信息；
- 操作前务必备份设备数据（推荐使用TWRP全量备份，或通过电脑导出重要文件）。
2. 兼容性补充：
- 支持Android 14及 vbmeta v3 格式；
- 动态分区设备需确保 device_adapt/vbmeta_paths.conf 中已配置对应路径（如 /dev/block/mapper/vbmeta ）；
- 部分品牌（如华为、OPPO）的设备可能有额外锁机制，需先解除对应限制（如华为的FRP锁）。
3. 权限说明：
- 本项目所有者为“好无聊”，使用者仅享有“了解、使用、非破坏性改进”权限，禁止商业售卖、二次授权或修改核心逻辑（如拆分模块、改变恢复机制）。
 
九、许可证（License）
 
本项目采用 MIT许可证（MIT License），具体条款如下：
 
- 允许自由使用、复制、修改、合并、发布、分发、 sublicense 和/或销售本项目的副本；
- 需在所有副本或重要部分中保留原始版权声明和许可证声明；
- 作者不对项目的使用后果承担责任，项目按“现状”提供，无任何担保（包括但不限于适销性、特定用途适用性的担保）。
 
完整许可证文本见项目根目录的 LICENSE 文件。
 
十、联系方式（Contact）
 
若遇到未覆盖的设备适配问题，或发现功能Bug，可通过以下方式反馈：
 
- GitHub Issues：直接在项目仓库提交Issue（需附设备型号、Android版本、操作步骤及错误日志）；
- 邮件：[cailin44@outlook.com]。
 
反馈时请注明“AVB-Disabler-with-Rescue反馈”，以便快速定位问题。



Description of English version


AVB-Disabler-with-Rescue: Separated AVB Disabling & Boot Recovery Module
 
A separated module for disabling Android Verified Boot (AVB) and auto-recovering from boot failures
 
1. Project Introduction (Project Intro)
 
This project is a separated solution for AVB disabling and boot recovery designed for Android devices, addressing two core issues:
 
1. Securely disable Android Verified Boot (AVB) verification to bypass system signature restrictions and support custom module installation;
2. Mitigate the "brick risk" caused by AVB disabling by providing dual recovery mechanisms (automatic + manual) to ensure device usability.
 
Compared with traditional solutions, this project optimizes compatibility across more device models, lowers operational barriers, and enhances security controls through an "adaptation layer + emergency layer + verification layer". It is only recommended for legal development and testing scenarios (Disabling AVB reduces device security; do not use it for sensitive scenarios on daily primary devices).
 
2. Core Features (Core Features)
 
1. AVB Disabling Module (AVB-Disabler)
 
- Multi-device Adaptation: Automatically identifies the device model and loads the corresponding  vbmeta  partition path (supports mainstream models like Samsung S22, Xiaomi Mi13, Google Pixel7, including dynamic partition devices);
- Pre-installation Verification: Automatically detects Android version (Android 8.0+ required), Magisk version (Magisk 20.4+ required), and Bootloader unlock status. Installation will be terminated if the conditions are not met;
- Security Reminder: Pops up a risk notice via Magisk during installation, requiring user confirmation to proceed and avoiding accidental operations;
- Property Spoofing: Spoofs system properties (e.g.,  ro.boot.verifiedbootstate=green ) to bypass AVB status detection by some systems.
 
2. Boot Recovery Module (Brick-Rescue)
 
- Automatic Backup: Automatically backs up the original  vbmeta  partition to  /data/adb/avb_backup/  on first boot, laying the foundation for recovery;
- Intelligent Boot Detection: 30-second timeout judgment (adapted for low-end devices), which detects both the  sys.boot_completed  property and Magisk service status to reduce misjudgments;
- Tiered Recovery Mechanism:
- 3 boot failures: Automatically enter safe mode for a retry;
- 5 boot failures: Automatically restore the original  vbmeta  and disable the AVB module, and the device returns to normal after reboot;
- Offline Emergency Recovery: Provides  offline_recovery.zip  (including Fastboot tools for Windows/Linux/macOS), allowing operation via computer when there is no ADB connection;
- Status Logging: Generates  /sdcard/boot_failure_report.txt  to record failure causes and recovery suggestions for easy troubleshooting.
 
3. Combined Installation Package (AVB-Disabler-With-Rescue)
 
- One-click Dual Installation: Complete the installation of both the AVB disabling module and the boot recovery module with a single flash;
- Incremental Updates: Detects the version of installed modules and only updates differential files to save installation time;
- Device Whitelist: Matches verified devices in  docs/device_list.md  before installation, and issues a warning for unsupported models.
 
3. Environment Requirements (Requirements)
 
1. Device-side:
 
- Android 8.0 or higher;
- Unlocked Bootloader;
- Magisk 20.4 or higher installed (must be activated normally and support Magisk Hide);
- At least 100MB of storage space reserved (for backing up  vbmeta  and log files).
 
2. Computer-side (Optional, for Emergency Recovery):
 
- Android SDK Platform Tools installed (including ADB and Fastboot tools);
- Windows/Linux/macOS systems are all supported (the offline recovery package is compatible with multiple systems).
 
4. Installation Steps (Installation Guide)
 
Method 1: One-click Installation with Combined Package (Recommended)
 
1. Download  AVB-Disabler-With-Rescue.zip  from the project root directory;
2. Open Magisk Manager → Tap "Modules" → "Install from Storage" → Select the above ZIP file;
3. Wait for the installation to complete (a risk notice will pop up during the process, tap "Confirm");
4. Reboot the device, and the modules will take effect automatically (you can check the module status via Magisk Manager).
 
Method 2: Separate Module Installation (Advanced)
 
1. First, download  AVB-Disabler.zip , install it via Magisk following the above steps, and reboot the device;
2. Then download  Brick-Rescue.zip , repeat the installation steps, and reboot again;
3. Verification: Navigate to the  /data/adb/modules/  directory, and confirm that the two folders  avbdisabler  and  brickrescue  exist (indicating successful installation).
 
5. Manual Recovery Operations (Manual Recovery)
 
If the device fails to boot (e.g., stuck at the boot logo), you can manually recover it through the following methods:
 
1. Recovery via ADB (Recommended, Requires ADB Debugging Enabled on the Device)
 
1. Connect the device to the computer, open Command Prompt (Windows) or Terminal (Linux/macOS);
2. Enter the command:
bash  
adb shell sh /data/adb/modules/brickrescue/system/bin/avb_recovery --force
 
3. The device will automatically restore the original  vbmeta  and reboot. After rebooting, the AVB disabling module will be disabled, and the device will return to normal.
 
2. Recovery via Recovery Mode (When There is No ADB Connection)
 
1. Press and hold the device's "Power Button + Volume Up Button" to enter TWRP or other custom Recovery modes;
2. Open "File Manager" and delete the  /data/adb/modules/avbdisabler  folder;
3. Reboot the device, the AVB disabling module will become invalid, and the device will resume the default boot process.
 
3. Recovery via Fastboot (Extreme Cases: Unable to Enter System/Recovery)
 
1. Extract  emergency/offline_recovery.zip  on the computer and find the Fastboot tool corresponding to your system;
2. Boot the device into Fastboot mode (press and hold "Power Button + Volume Down Button") and connect it to the computer;
3. Execute the command (you need to prepare the original  vbmeta.img  of the device in advance, which can be extracted from the official firmware):
bash  
# For Windows
fastboot.exe flash vbmeta vbmeta_original.img

# For Linux/macOS
./fastboot flash vbmeta vbmeta_original.img
 
4. After the command is executed, enter  fastboot reboot  to restart the device.
 
6. Frequently Asked Questions (Troubleshooting)
 
Q1: The device fails to boot after installation?
 
A1: Enter TWRP to delete  /data/adb/modules/avbdisabler , or execute  adb shell sh /data/adb/modules/brickrescue/system/bin/avb_recovery --force  to restore the original  vbmeta .
 
Q2: The boot recovery module does not work, and no automatic recovery occurs after boot failure?
 
A2:
 
1. Confirm that Magisk is normally activated (navigate to  /data/adb/magisk/  and check if the  magiskd.pid  file exists);
2. Check if the permission of the  /data/adb/brick_rescue  directory is  0755  (you can modify the permission via Recovery).
 
Q3: The prompt "Device not supported" appears?
 
A3: Check  docs/device_list.md  to confirm whether the device is in the verified list; for niche models, you can manually edit  AVB-Disabler/device_adapt/vbmeta_paths.conf  and add the  vbmeta  partition path of the device (you need to query the device's partition information by yourself).
 
Q4: After restoring  vbmeta , Magisk prompts "Not installed"?
 
A4: You need to re-flash the Magisk patched package (you can flash  magisk_patched-xxx_boot.img  via Fastboot, or flash the Magisk installation package via Recovery).
 
7. File Structure (File Structure)
 
plaintext  
AVB-Disabler-with-Rescue/
├── AVB-Disabler/                # AVB Disabling Module
│   ├── device_adapt/            # Device Adaptation Directory
│   │   ├── vbmeta_paths.conf    # Device Model - Partition Path Configuration
│   │   └── prop_override.sh     # Device-Specific Property Script
│   ├── common/                  # Core Scripts
│   │   ├── post-fs-data.sh      # Post-Boot Execution Script
│   │   └── system_hook/         # System Hook Scripts
│   ├── system/                  # System Configuration
│   ├── check_compatibility.sh   # Compatibility Check Script
│   └── module.prop              # Module Information
│
├── Brick-Rescue/                # Boot Recovery Module
│   ├── emergency/               # Emergency Recovery Directory
│   │   ├── offline_recovery.zip # Offline Recovery Package
│   │   └── fastboot_cmd.txt     # Fastboot Command Template
│   ├── status/                  # Status Log Directory
│   ├── system/bin/              # Core Tools (avb_stealthd, diag_system, etc.)
│   └── module.prop              # Module Information
│
├── AVB-Disabler-With-Rescue/    # Combined Installation Package
│   ├── modules/                 # Sub-module ZIP Files
│   ├── verify/                  # Verification Directory (MD5, Compatibility Check)
│   └── scripts/                 # Installation Scripts
│
├── docs/                        # Documentation Directory
│   ├── troubleshooting.md       # Troubleshooting Guide
│   └── device_list.md           # Verified Device List
│
├── .gitignore                   # Git Ignore File Configuration
├── LICENSE                      # Open-Source License (See "9. License")
└── README.md                    # This README File
 
 
8. Notes (Notes)
 
1. Legality and Security:
 
- Only use for legal development and testing; it is prohibited to use it for cracking others' devices, invading privacy, or other illegal scenarios;
- After disabling AVB, the device is vulnerable to malware attacks; do not store sensitive information such as bank cards and passwords;
- Be sure to back up device data before operation (it is recommended to use TWRP for full backup or export important files to a computer).
 
2. Compatibility Supplements:
 
- Supports Android 14 and  vbmeta v3  format;
- For dynamic partition devices, ensure that the corresponding path (e.g.,  /dev/block/mapper/vbmeta ) is configured in  device_adapt/vbmeta_paths.conf ;
- Devices of some brands (such as Huawei and OPPO) may have additional lock mechanisms, and the corresponding restrictions (such as Huawei's FRP lock) need to be removed first.
 
3. Permission Statement:
 
- The owner of this project is "好无聊" (Hao Wuliao). Users only have the rights to "understand, use, and make non-destructive improvements" to the project. Commercial sales, sub-licensing, or modification of core logic (such as splitting modules, changing recovery mechanisms) are prohibited.
 
9. License (License)
 
This project adopts the MIT License. The specific terms are as follows:
 
- Permission is granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software;
- The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software;
- THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY.
 
The full license text can be found in the  LICENSE  file in the project root directory.
 
10. Contact Information (Contact)
 
If you encounter unsupported device adaptation issues or find functional bugs, you can provide feedback through the following methods:
 
- GitHub Issues: Submit an Issue directly in the project repository (you need to attach the device model, Android version, operation steps, and error logs);
- Email: [cailin44@outlook.com].
 
Please indicate "AVB-Disabler-with-Rescue Feedback" when providing feedback to facilitate quick problem localization.