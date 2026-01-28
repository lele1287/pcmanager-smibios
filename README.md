# pcmanager-smibios-proxy

一个可直接在 **Visual Studio 2022** 下打开的 CMake + vcpkg 解决方案，构建一个同名 `version.dll` 代理，
在首次被调用时**懒安装**对 `Kernel32!GetSystemFirmwareTable` 的 Hook，并向 `'RSMB'` 提供伪造的 SMBIOS Type 1 信息：

- Manufacturer: `HUAWEI`
- Product Name: `FLMH-XX`
- Version: `M1010`
- Serial Number: `2VBBB2481480888`
- UUID: `20240816-105F-AD36-7653-105FAD367657`
- Wake-up Type: `Power Switch`
- SKU Number: `C233`
- Family: `MateBook`

> 仅用于兼容性研究与自测。不要在违反许可或法律的场景使用。

## 构建

```powershell
# 可选：生成与系统一致的导出（带序号）
python scripts/gen_def.py --arch x64 --out src/version.def

# CMake 配置与构建（x64）
cmake -S . -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE="%VCPKG_ROOT%\scripts\buildsystems\vcpkg.cmake"
cmake --build build --config Release --target version smbios_check
```

## 一键验证

将 `version.dll` 与 `smbios_check.exe` 放在**同一目录**（`build/bin/Release`），执行：

```powershell
cd build\bin\Release
./smbios_check.exe
```

若输出如下即表示生效：

```
Manufacturer : HUAWEI
Product Name : FLMH-XX
Version      : M1010
Serial Number: 2VBBB2481480888
Wakeup Type  : 0x06
SKU Number   : C233
Family       : MateBook
```

## CI（GitHub Actions）

工作流会：
1. 从当前 Runner 的 `System32`/`SysWOW64` 的 **version.dll** 自动生成 `src/version.def`（带**名称+序号**）
2. 构建你的代理 `version.dll`
3. 用 `pefile` 比对**你构建出的导出**与**系统 version.dll 导出**，确保一致

## 参考
- `GetSystemFirmwareTable`、`EnumSystemFirmwareTables`、RSMB/ACPI 语义与返回规范（Microsoft Learn）
- `RawSMBIOSData` 结构与示例（Microsoft Learn）
- `DllMain` 最佳实践：避免在 DllMain 内进行 Loader 相关操作，采用懒初始化（Microsoft Learn）
```
