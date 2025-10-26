

# 在 Windows 上安装 C++ 编译器

如果要对 Windows 进行 C++ 开发，建议安装 Microsoft Visual C++ (MSVC)编译器。

1. 若要安装 MSVC，请打开 VS Code 终端(CTRL + `)并在以下命令中粘贴:

   ```
   winget install Microsoft.VisualStudio.2022.BuildTools --force --override "--wait --passive --add Microsoft.VisualStudio.Workload.VCTools --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.Windows10SDK"
   ```

2. **注意**: 可以使用 Visual Studio 生成工具中的 C++ 工具集以及 Visual Studio Code 以编译、生成并验证任何 C++ 代码库，前提是同时具有有效的 Visual Studio 许可证(社区版、专业版或企业版)，且正积极将其用于开发该 C++ 代码库。

## 正在验证编译器安装

1. 在 Windows“开始”菜单中键入‘开发人员’以打开 **VS 的开发人员命令提示**。

2. 在 VS 的开发人员命令提示中键入 `cl` 以检查 MSVC 安装。你应该会看到包含版本和基本使用情况说明的版权消息。

   > **注意**: 要从命令行或 VS Code 使用 MSVC，必须从 **VS 的开发人员命令提示** 运行。普通 shell (例如 PowerShell、 Bash)或 Windows 命令提示符未设置必要的路径环境变量。

## 其他编译器选项

如果面向的是 Windows 中的 Linux，请查看[在 VS Code 中使用 C++ 和 适用于 Linux 的 Windows 子系统(WSL)](https://code.visualstudio.com/docs/cpp/config-wsl)。或者，可[在带 MinGW 的 Windows 上安装 GCC](https://code.visualstudio.com/docs/cpp/config-mingw)。

# 在 Windows 上安装 C++ 编译器

如果要对 Windows 进行 C++ 开发，建议安装 Microsoft Visual C++ (MSVC)编译器。

1. 若要安装 MSVC，请打开 VS Code 终端(CTRL + `)并在以下命令中粘贴:

   ```
   winget install Microsoft.VisualStudio.2022.BuildTools --force --override "--wait --passive --add Microsoft.VisualStudio.Workload.VCTools --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.Windows10SDK"
   ```

2. **注意**: 可以使用 Visual Studio 生成工具中的 C++ 工具集以及 Visual Studio Code 以编译、生成并验证任何 C++ 代码库，前提是同时具有有效的 Visual Studio 许可证(社区版、专业版或企业版)，且正积极将其用于开发该 C++ 代码库。

## 正在验证编译器安装

1. 在 Windows“开始”菜单中键入‘开发人员’以打开 **VS 的开发人员命令提示**。

2. 在 VS 的开发人员命令提示中键入 `cl` 以检查 MSVC 安装。你应该会看到包含版本和基本使用情况说明的版权消息。

   > **注意**: 要从命令行或 VS Code 使用 MSVC，必须从 **VS 的开发人员命令提示** 运行。普通 shell (例如 PowerShell、 Bash)或 Windows 命令提示符未设置必要的路径环境变量。

## 其他编译器选项

如果面向的是 Windows 中的 Linux，请查看[在 VS Code 中使用 C++ 和 适用于 Linux 的 Windows 子系统(WSL)](https://code.visualstudio.com/docs/cpp/config-wsl)。或者，可[在带 MinGW 的 Windows 上安装 GCC](https://code.visualstudio.com/docs/cpp/config-mingw)。

# 使用开发人员命令提示符重新启动

正在使用带有 MSVC 编译器的 Windows 机器，因此需要从开发人员命令提示符中启动 VS Code，以便所有环境变量都能正确设置。要使用开发人员命令提示符重新启动:

1. 通过在 Windows 开始菜单中键入 "developer" 来打开 VS 的开发人员命令提示提示。选择 VS 的开发人员命令提示提示，它将自动导航到当前打开的文件夹。
2. 在命令提示符中键入 "code"，然后按 Enter。这应该重新启动 VS Code 并将你带回此演练。