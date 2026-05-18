# PowerShell West 腳本使用指南

## 段落1：使用腳本啟用 West 環境並構建 Zephyr 項目

`powershell_to_run_west.ps1` 腳本用於臨時設置 PowerShell 環境，使您能夠在命令行界面 (CLI) 中使用 `west` 工具來構建 Zephyr 應用項目。

### 如何使用
1. **導航到項目目錄**：  
   打開 PowerShell 並切換到您的 Zephyr 項目目錄，例如：  
   ```
   cd c:\ncs\v3.1.0\zephyr\samples\net\gptp
   ```

2. **運行腳本**：  
   執行以下命令來設置環境（包括 west 和 nrfutil）：  
   ```
   .\powershell_to_run_west.ps1
   ```  
   腳本會臨時將 nRF Connect 工具鏈的路徑添加到 `PATH` 和 `PYTHONPATH`，並驗證 `west` 和 `nrfutil` 是否可用。

3. **構建項目**：  
   設置完成後，您可以在同一 PowerShell 會話中使用 `west` 命令。例如：  
   ```
   west build --build-dir build --domain gptp
   ```  
   如果需要完全重新構建（清理舊文件），可以使用 `--pristine` 選項：  
   ```
   west build --build-dir build --domain gptp --pristine
   ```  
   **注意**：對於多域項目，請使用根 `build` 目錄（而非子域目錄如 `build/gptp`），並指定正確的 `--domain`。錯誤的命令（如 `west build --build-dir build/gptp --domain gptp`）會導致 "domain not found" 錯誤。

4. **燒錄（Flash）到設備**：  
   修改程式碼（如 `main.c`）後，只需執行以下**單一指令**即可完成所有工作：  
   ```
   west flash --build-dir build --domain gptp
   ```  
   此指令會自動依序完成：  
   - **Build** — 重新編譯有更動的程式碼  
   - **Flash** — 將固件燒錄到 nRF 設備  
   - **Reset** — 板子重置，應用程式立即開始運行  

   如果有多個設備連接，可以指定序列號：  
   ```
   west flash --build-dir build --domain gptp --sn <serial_number>
   ```  
   **注意**：如果首次使用，需要安裝 nrfutil device 插件。您可以運行提供的安裝腳本：  
   ```
   .\install_nrfutil_device.ps1
   ```  
   或手動運行：  
   ```
   nrfutil install device
   ```  
   確保設備已連接並正確配置。

5. **監視（Monitor）設備輸出 / 重置並運行應用**：  
   燒錄後，可以監視設備的串口輸出以查看應用程式的運行日誌（這也會重置板子並啟動應用）：  
   ```
   west attach --build-dir build --domain gptp
   ```  
   **注意**：`west attach` 使用 JLink runner，需要 `JLinkGDBServer`。  
   已確認 JLink 位於 `C:\Program Files\SEGGER\JLink_V886\`，腳本 `powershell_to_run_west.ps1` 已自動將此路徑加入 PATH。  
   連接後，在 GDB 提示符 `(gdb)` 中輸入 `c` 並按 Enter 以繼續執行應用程式：  
   ```
   (gdb) c
   ```  
   安裝 JLink 後，`west attach` 應該可以正常工作。如果您不想安裝 JLink，請考慮使用 nRF Connect for VS Code 的內建監視功能。  

   如果有多個設備，可以指定序列號：  
   ```
   west attach --build-dir build --domain gptp --sn <serial_number>
   ```  
   這將連接設備並顯示即時輸出。按 Ctrl+C 退出監視模式。

### 注意事項
- 環境設置僅在當前 PowerShell 會話中有效。關閉終端後，需要重新運行腳本。
- 如果遇到腳本執行錯誤，請參考段落2。

## 段落2：修復執行策略錯誤

## 步驟

1. **以管理員身份打開 PowerShell**  
   右鍵點擊 PowerShell 並選擇 "以管理員身份運行"。

2. **檢查當前執行策略**  
   運行以下命令：  
   ```
   Get-ExecutionPolicy
   ```

3. **設置執行策略以允許腳本**  
   根據您的需求選擇以下選項之一：

   a) **僅當前用戶（推薦用於開發）**：  
      ```
      Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
      ```

   b) **所有用戶（更寬鬆，請謹慎使用）**：  
      ```
      Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
      ```

   c) **允許所有腳本（最不安全，不推薦）**：  
      ```
      Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
      ```

4. **驗證更改**  
   運行以下命令確認：  
   ```
   Get-ExecutionPolicy
   ```

5. **再次嘗試運行腳本**  
   ```
   .\powershell_to_run_west.ps1
   ```

## 註釋
- **RemoteSigned**：允許本地創建的腳本和已簽名的遠程腳本。
- 如果未指定 `-Scope`，此更改僅對當前會話臨時有效。
- 要永久更改，請使用 `-Scope LocalMachine`（需要管理員權限）。
- **要恢復限制**：運行 `Set-ExecutionPolicy -ExecutionPolicy Restricted -Scope CurrentUser`。

如果按照這些步驟後腳本仍失敗，問題可能不是執行策略，而是模組路徑或其他環境問題。