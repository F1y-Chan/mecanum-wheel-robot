# 参与贡献 / Contributing

感谢你对本项目的关注！无论是提交 Bug、改进文档还是贡献代码，都非常欢迎。

## 提交 Issue

提 Issue 前请先搜索是否已有相同问题。一个高质量的 Issue 应包含：

- **问题现象**：实际发生了什么，期望发生什么；
- **复现步骤**：尽可能精确，能稳定复现；
- **运行环境**：Keil MDK 版本、ARM Compiler 版本、开发板与电机型号、CAN 接线；
- **日志与截图**：编译报错全文、串口/裁判系统数据、OLED 报错码或波形截图。

## 提交 Pull Request

1. Fork 本仓库并从 `main` 拉出特性分支，例如 `feat/power-limit`、`fix/imu-calibration`；
2. 保持改动聚焦：一个 PR 只解决一个问题；
3. 提交前自检：
   - 工程可以正常编译（Keil `Rebuild` 无 Error）；
   - **不要提交编译产物**，仓库已通过 `.gitignore` 排除 `.o`、`.crf`、`.axf`、`.map` 等文件；
   - 不要提交本机绝对路径相关的文件（`*.uvguix.*`、`*.uvoptx`、`uv4.log` 等）；
   - 若改动了 `standard_robot.ioc`，请一并提交重新生成后的 `Inc/`、`Src/` 变化；
4. 提交信息建议使用 [Conventional Commits](https://www.conventionalcommits.org/)：`feat:`、`fix:`、`docs:`、`refactor:`、`chore:`；
5. 在 PR 描述中说明**改动内容、验证方式、对硬件行为的影响**。

## 代码分层约定

新增功能请遵守现有分层，不要把业务逻辑写进 `bsp/` 或 CubeMX 生成文件：

| 层次 | 目录 | 职责 |
| --- | --- | --- |
| 应用层 | `application/` | 任务逻辑与行为决策 |
| 板级支持层 | `bsp/boards/` | 外设封装，对上提供统一接口 |
| 组件层 | `components/` | 可复用算法与设备驱动 |

## 代码风格

- 变量使用小写下划线（`chassis_speed`），函数使用 `模块_动作`（`chassis_task_init`）；
- 常量使用全大写（`CHASSIS_MAX_SPEED`）；
- 缩进 4 空格；物理量注释单位（`rad/s`、`N·m`、`A`）；
- 新增 `.c/.h` 请加入 `MDK-ARM/standard_robot.uvprojx` 对应的分组，不要只放在目录里。

## 安全要求

本项目控制真实电机。涉及电流限幅、功率控制、保护逻辑、发射机构的 PR，请在描述中注明测试条件（是否架空、限流值、急停措施），未在实物上验证的改动请显式说明。

## 许可证

你提交的贡献将以本仓库的 [MIT License](LICENSE) 发布。
