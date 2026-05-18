# PowerShell West 腳本使用指南（簡潔版）

## 段落1：使用腳本啟用 West 環境並構建 Zephyr 項目

`powershell_to_run_west.ps1` 腳本用於臨時設置 PowerShell 環境，使您能夠在命令行界面 (CLI) 中使用 `west` 工具來構建 Zephyr 應用項目。

### 如何使用

1. **導航到項目目錄**：  
   ```
   cd c:\ncs\v3.1.0\zephyr\samples\net\gptp
   ```

2. **運行腳本（設置環境）**：  
   ```
   .\powershell_to_run_west.ps1
   ```

3. **只構建項目**（不燒錄）：  
   ```
   west build --build-dir build --domain gptp
   ```  
   完全重新構建：  
   ```
   west build --build-dir build --domain gptp --pristine
   ```

4. **修改程式碼後一鍵完成 Build + Flash + Reset**：  
   ```
   west flash --build-dir build --domain gptp
   ```  
   此指令會自動依序完成：  
   - **Build** — 重新編譯有更動的程式碼  
   - **Flash** — 將固件燒錄到 nRF 設備  
   - **Reset** — 板子重置，應用程式立即開始運行  

   > 首次使用請先安裝 nrfutil device 插件：`.\install_nrfutil_device.ps1`

5. **監視設備輸出（GDB attach）**：  
   ```
   west attach --build-dir build --domain gptp
   ```  
   連接後輸入 `c` 繼續執行應用程式：  
   ```
   (gdb) c
   ```  
   > JLink 路徑 `C:\Program Files\SEGGER\JLink_V886\` 已由腳本自動加入 PATH。  
   > 按 Ctrl+C 退出監視模式。

### 注意事項
- 環境設置僅在當前 PowerShell 會話中有效，關閉終端後需重新運行腳本。
- 如果遇到腳本執行策略錯誤，請參考段落2。

---

## 段落2：修復執行策略錯誤

如果遇到 "execution of scripts is disabled on this system" 錯誤：

1. 以管理員身份開啟 PowerShell，執行：  
   ```
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

2. 驗證：  
   ```
   Get-ExecutionPolicy
   ```

3. 再次運行腳本：  
   ```
   .\powershell_to_run_west.ps1
   ```

> **要恢復限制**：`Set-ExecutionPolicy -ExecutionPolicy Restricted -Scope CurrentUser`
