---
order: 4
icon: mingcute:android-fill
---

# Android 實體設備

::: info 注意

1. 從 Android 10 開始，Minitouch 在 SELinux 為 `Enforcing` 模式時不再可用，請切換至其他觸控模式，或將 SELinux **臨時**切換為 `Permissive` 模式。
2. 由於 Android 生態極為複雜，可在 MAA `設定` - `連接設定` 中嘗試將 `連接配置` 修改為 `通用模式` 或 `兼容模式` 或 `第二解析度` 或 `通用模式（屏蔽異常輸出）`，直到某個模式可以正常使用。
3. 由於 MAA 僅支援 `16:9` 比例的解析度，所以非 `16:9` 或 `9:16` 螢幕比例的設備需要強制修改解析度，這包含大多數現代設備。若被連接設備螢幕解析度比例原生為 `16:9` 或 `9:16`，則可跳過 `更改解析度` 部分。
4. 請將設備導航方式切換為除 `全面屏手勢` 以外的方式，如 `經典導航鍵` 等以避免誤操作。
5. 請將遊戲內設定中的 `異形屏UI適配` 一項調整為 0 以避免任務出錯。
:::

::: tip
典型的 `16:9` 比例的解析度有 `3840*2160` (4K)、`2560*1440` (2K)、`1920*1080` (1080P)、`1280*720` (720P)。
:::

## 下載、運行 adb 調試工具並連接設備

1. 下載 [adb](https://dl.google.com/android/repository/platform-tools-latest-windows.zip) 並解壓。
2. 打開解壓後的資料夾，清空地址欄並輸入 `cmd` 後按回車。
3. 在彈出的命令提示符視窗中輸入 `adb`，若顯示大量英文幫助文本則運行成功。
4. 手機開啟 `USB 調試`，每個品牌的手機進入方式可能不同，請善用搜尋引擎。廠商可能會提供有關 USB 調試的額外選項，如 MIUI 中的 `USB 安裝` 和 `USB 調試（安全設置）`，請同時開啟。
5. 將手機通過數據線連接至電腦，在剛剛的命令提示符視窗中輸入以下命令。

   ```bash
   adb devices
   ```

- 成功執行後會顯示已連接 `USB 調試` 設備的信息。

  - 連接成功的例子：

    ```bash
    List of devices attached
    VFNDU1682100xxxx        device
    ```

  - **`device` 前的英文數字組合為設備序列號，同時也作為 MAA 的 `連接地址`。**

- 現代安卓設備進行 `USB 調試` 需在被調試設備上點擊彈窗授權，若未授權則例子如下：

  ```bash
  List of devices attached
  VFNDU1682100xxxx        unauthorized
  ```

- 若無論如何都提示未授權或設備序列號後顯示 `offline`，則需重啟設備及電腦後重試。如仍未解決問題，可刪除當前用戶個人資料夾下的 `.android` 資料夾並再次重啟後重試，具體位置請自行搜尋。

## 更改解析度

::: tip
手機螢幕解析度為 `短邊*長邊`，而非電腦顯示器的 `長邊*短邊`。具體數值請根據目標設備自行確定。
:::

- 如果上文設備列表內僅有一台設備，則可直接運行以下命令更改/還原解析度。

  ```bash
  # 查看當前解析度
  adb shell wm size
  # 還原預設解析度
  adb shell wm size reset

  # 更改解析度為 720p
  adb shell wm size 720x1280
  # 更改解析度為 1080p
  adb shell wm size 1080x1920
  ```

- 若存在多台設備，則需在 `adb` 和 `shell` 中間添加參數 `-s <目標設備序列號>`，例子如下。

  ```bash
  # 查看當前解析度
  adb -s VFNDU1682100xxxx shell wm size
  # 還原預設解析度
  adb -s VFNDU1682100xxxx shell wm size reset

  # 更改解析度為 720p
  adb -s VFNDU1682100xxxx shell wm size 720x1280
  # 更改解析度為 1080p
  adb -s VFNDU1682100xxxx shell wm size 1080x1920
  ```

- 部分設計不規範的應用可能在還原解析度後內容佈局仍然錯亂，一般重啟對應應用或設備即可解決。

::: danger 注意
強烈建議在**下次重啟設備前**還原解析度，否則因設備而定可能會導致不可預料的後果，~~包括但不限於佈局混亂，觸控錯位，應用閃退，無法解鎖等~~。
:::

## 自動化更改解析度

1. 在 MAA 目錄下新建兩個文本檔案，分別在其中填入以下內容。

   ```bash
   # 調整解析度為 1080p
   adb -s <目標設備序列號> shell wm size 1080x1920
   # 降低螢幕亮度（可選）
   adb -s <目標設備序列號> shell settings put system screen_brightness 1
   ```

   ```bash
   # 還原解析度
   adb -s <目標設備序列號> shell wm size reset
   # 提高螢幕亮度（可選）
   adb -s <目標設備序列號> shell settings put system screen_brightness 20
   # 返回桌面（可選）
   adb -s <目標設備序列號> shell input keyevent 3
   # 鎖屏（可選）
   adb -s <目標設備序列號> shell input keyevent 26
   ```

2. 將第一個檔案重命名為 `startup.bat`，第二個檔案重命名為 `finish.bat`。

   - 如果重命名後沒有彈出修改擴展名的二次確認對話框，且檔案圖示沒有變化，請自行搜尋“Windows 如何顯示檔案擴展名”。

3. 在 MAA 的 `設定` - `連接設定` - `開始前腳本` 和 `結束後腳本` 中分別填入 `startup.bat` 和 `finish.bat`。

## 連接 MAA

### 有線連接

::: tip
使用有線連接不需要任何 IP 地址或埠，只需要 `adb devices` 給出的設備序列號。
:::

1. 將上文獲取到的目標設備序列號填入 MAA `設定` - `連接設定` - `連接地址` 中。
2. Link Start!

### 無線連接

- 請確保設備與電腦處在同一區域網環境下且能互相通信。諸如 `AP 隔離`、`訪客網路` 等設定會阻止設備間通信，具體請查閱對應路由器文檔。
- 無線調試在設備重啟後失效，需要重新設定。

#### 使用 `adb tcpip` 開啟無線埠

1. 在剛剛的命令提示符視窗中輸入以下命令以開啟無線調試。

   ```bash
   adb tcpip 5555
   # 存在多台設備則添加參數 -s 以指定序列號
   ```

2. 查看設備 IP 地址。

   - 進入手機 `設定` - `WLAN`，點擊當前已連接的無線網路查看 IP 地址。
   - 各類品牌設備設定位置不同，請自行查找。

3. 將 `<IP>:5555` 填入 MAA `設定` - `連接設定` - `連接地址` 中，如 `192.168.1.2:5555`。
4. Link Start!

#### 使用 `adb pair` 開啟無線埠

::: tip
`adb pair` 無線配對，即使用安卓 11 及更新版本中開發者選項內的 `無線調試` 進行配對後連接，與 `adb tcpip` 相比可以避免有線連接。
:::

1. 進入手機開發者選項，點擊 `無線調試` 並開啟，點擊確定，點擊 `使用配對碼配對設備`，在配對完成前不要關閉出現的彈窗。

2. 進行配對。

   1. 在命令提示符中輸入 `adb pair <設備彈窗給出的 IP 地址和埠>`，按回車。
   2. 輸入 `<設備彈窗給出的六位配對碼>`，按回車。
   3. 視窗出現 `Successfully paired to <IP:埠>` 等內容，同時設備上的彈窗自動消失，底部已配對的設備中出現計算機名稱。

3. 將當前設備螢幕上給出的 `<IP 地址和埠>` 填入 MAA `設定` - `連接設定` - `連接地址` 中，如 `192.168.1.2:11451`，**一定和剛剛填寫的不一樣**。
4. Link Start!

#### 使用 root 權限開啟無線埠

~~都接觸到 root 了還用得著看這段文檔嗎~~

1. 下載、安裝 [WADB](https://github.com/RikkaApps/WADB/releases) 並授予其 root 權限。
2. 打開 WADB，啟動無線 adb。
3. 將 WADB 提供的 IP 地址及埠填入 MAA `設定` - `連接設定` - `連接地址` 中，如 `192.168.1.2:5555`。
4. Link Start!