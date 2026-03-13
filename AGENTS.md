# Neural Amp Modeler Plugin - Agent Guidelines

This document provides instructions and context for agentic coding tools (like opencode, Cursor, or Copilot) operating in the Neural Amp Modeler (NAM) repository.

## Project Structure
- `NeuralAmpModeler/`: Main plugin source code (iPlug2-based).
- `NeuralAmpModelerCore/`: (Submodule) The core DSP engine for neural network modeling.
- `AudioDSPTools/`: (Submodule) Common DSP utilities (filters, noise gates, etc.).
- `iPlug2/`: (Submodule) The audio plugin framework.
- `eigen/`: (Submodule) Linear algebra library used by NAM.

---

## Build, Lint, and Test Commands

### Initialization
Before building for the first time or after updating submodules:
```bash
bash setup_container.sh
```
This script initializes submodules and downloads the necessary iPlug2 SDKs and prebuilt libraries.

### Build Commands
The project uses platform-specific scripts in the `NeuralAmpModeler/scripts` directory.

- **macOS (Xcode):**
  ```bash
  cd NeuralAmpModeler/scripts && ./makedist-mac.sh full zip
  ```
- **Windows (MSBuild/Visual Studio):**
  ```bash
  cd NeuralAmpModeler\scripts && .\makedist-win.bat full zip
  ```
- **Web (Emscripten):**
  ```bash
  cd NeuralAmpModeler/scripts && ./makedist-web.sh
  ```

### Formatting (Linting)
The project uses `clang-format` with LLVM style (plus custom brace wrapping).
```bash
bash format.bash
```
This formats all `.h` and `.cpp` files in the repository. Do not commit unformatted code.

### Testing
This repository currently lacks a unit testing framework. Verification is primarily performed through:
1. **Manual Validation:** Build and run the standalone application or load the VST3/AU in a DAW.
2. **VST3 Validation:** Use `pluginval` (downloadable tool) to validate the VST3 plugin:
   ```bash
   pluginval --skip-gui-tests --validate-in-process --validate path/to/NeuralAmpModeler.vst3
   ```
3. **VST3PluginTestHost:** Use the official Steinberg test host for deeper VST3 protocol checks.

---

## Code Style Guidelines

### Formatting and Layout
- **Formatter:** `clang-format` (configured in `.clang-format`).
- **Indentation:** 2 spaces (no tabs).
- **Line Length:** 120 columns.
- **Braces (Allman-style):** Braces should be on a new line for classes, functions, and control statements (if/for/while).
- **Pointer/Reference Alignment:** Left-aligned (e.g., `int* pData`, `std::string& name`).

### Naming Conventions
- **Classes/Structs/Types:** `PascalCase` (e.g., `NeuralAmpModeler`, `ResamplingNAM`).
- **Constants:** `kCamelCase` (e.g., `kNumParams`, `kDCBlockerFrequency`).
- **Member Variables:** `mCamelCase` (e.g., `mInputGain`, `mStagedModel`).
- **Function/Method names:** `PascalCase` (e.g., `ProcessBlock`, `OnParamChange`).
- **Internal Helper Methods:** Prefixed with underscore `_PascalCase` (e.g., `_StageModel`, `_ApplyDSPStaging`).
- **Local Variables:** `camelCase` (e.g., `scaleFactor`, `mainArea`).
- **Pointers:** Prefix with `p` (e.g., `pGraphics`, `pData`).
- **Enums and Tags:** `kCamelCase` (e.g., `kInputLevel`, `kMsgTagLoadedModel`).
- **Macros:** `SCREAMING_SNAKE_CASE`.

### Imports (Includes)
- Use `#pragma once` in all headers.
- **Order:**
  1. Standard library headers (`<algorithm>`, `<vector>`, etc.).
  2. Submodule headers (`"../AudioDSPTools/..."`, `"../NeuralAmpModelerCore/..."`).
  3. Local project headers (`"Colors.h"`, `"ToneStack.h"`).
  4. iPlug2 specific headers (often require specific ordering, wrapped in `// clang-format off/on`).

### Namespaces
- Frequent use of `using namespace iplug;` and `using namespace igraphics;` at the top of `.cpp` files is common.
- Avoid large namespaces in headers; prefer explicit scoping.

### Error Handling
- **DSP Layer:** Throw `std::runtime_error` for fatal logic errors in non-real-time paths (e.g., during model staging).
- **UI Layer:** Use `_ShowMessageBox` to communicate errors to the user.
- **Return Values:** Many internal methods return `std::string` messages on failure (empty string on success).

### Best Practices
- **Memory Management:** Use `std::unique_ptr` for managing lifetimes of DSP modules and UI controls.
- **Concurrency:** Use `std::atomic<bool>` for flags that communicate between the UI and DSP threads (e.g., `mShouldRemoveModel`).
- **iPlug2 Integration:** Adhere to the `IPlug` lifecycle (e.g., `OnReset`, `OnParamChange`, `ProcessBlock`).
- **Floating Point:** Use `float` for GUI coordinates and some DSP, but `double` is preferred for high-precision audio parameters and core processing unless performance dictates otherwise.

---

## Tool-Specific Rules
### Cursor / Copilot
- No specific `.cursorrules` or `.github/copilot-instructions.md` files are present.
- Follow the `.clang-format` and `.vscode/settings.json` configurations.
- Agents should proactively run `bash format.bash` before proposing commits.

---

## Common Tasks for Agents
1. **Adding a Parameter:**
   - Add to `EParams` enum in `NeuralAmpModeler.h`.
   - Initialize in the constructor of `NeuralAmpModeler.cpp`.
   - Handle in `OnParamChange`.
   - Update UI layout in `mLayoutFunc`.
2. **DSP Logic:**
   - Core neural network logic belongs in `NeuralAmpModelerCore`.
   - Plugin wrapper logic (resampling, gain, noise gate) belongs in `NeuralAmpModeler.cpp/h`.
3. **UI Improvements:**
   - Modify `mLayoutFunc` in `NeuralAmpModeler.cpp`.
   - Use `IVStyle` and `IVColorSpec` for consistent appearance.
