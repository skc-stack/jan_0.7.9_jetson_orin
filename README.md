# Jan AI Desktop Application v0.7.9 - Build & Install Guide for Jetson Orin Nano

## 目標機器
- **IP**: 192.168.5.29
- **帳號**: kghsai
- **架構**: ARM64 (aarch64) - NVIDIA Jetson Orin Nano
- **OS**: Ubuntu 22.04 with GNOME Desktop

---

## 前置準備

### 1. SSH 連線測試
```bash
ssh kghsai@192.168.5.29
```

### 2. 系統環境確認
```bash
# 確認架構
uname -m
# 預期輸出: aarch64

# 確認記憶體
free -h
# 建議: 至少 8GB RAM

# 確認磁碟空間
df -h /home
# 建議: 至少 50GB 可用空間
```

---

## 系統依賴安裝

### Node.js 20.x
```bash
# 移除舊版 nodejs (如果存在)
sudo apt-get remove -y nodejs npm libnode-dev

# 安裝 Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# 驗證
node --version  # 應為 v20.x.x
```

### Yarn 4.x (使用 Corepack)
```bash
# 啟用 corepack 並啟動 yarn 4.x
sudo corepack enable
sudo corepack prepare yarn@4.5.3 --activate

# 驗證
yarn --version  # 應為 4.x.x
```

### Rust (用於 Tauri)
```bash
# 安裝 Rust (如果尚未安裝)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# 驗證
rustc --version
cargo --version
```

### GTK3 & WebKitGTK 開發庫 (GUI 必備)
```bash
sudo apt-get update
sudo apt-get install -y \
    libgtk-3-dev \
    libjavascriptcoregtk-4.1-dev \
    libwebkit2gtk-4.1-dev \
    libsoup-3.0-dev \
    libgbm-dev \
    libasound2-dev \
    libpango1.0-dev \
    libatk1.0-dev \
    libcairo2-dev

# 額外必要工具
sudo apt-get install -y \
    build-essential \
    pkg-config \
    libssl-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
```

---

## 下載 Jan v0.7.9 原始碼

```bash
# 移除舊版本 (如果存在)
rm -rf ~/jan-0.7.9

# 下載 v0.7.9 原始碼
cd ~
wget https://github.com/janhq/jan/archive/refs/tags/v0.7.9.tar.gz

# 解壓縮
tar -xzf v0.7.9.tar.gz
rm v0.7.9.tar.gz
```

---

## 更新版本號 (可選)

官方 v0.7.9 原始碼中的版本號是 0.6.599，若要顯示正確版本可手動修改：

```bash
# 修改 Cargo.toml
sed -i 's/version = "0.6.599"/version = "0.7.9"/g' ~/jan-0.7.9/src-tauri/Cargo.toml

# 修改 tauri.conf.json
sed -i 's/"version": "0.6.599"/"version": "0.7.9"/g' ~/jan-0.7.9/src-tauri/tauri.conf.json
```

---

## 安裝專案依賴

```bash
cd ~/jan-0.7.9

# 安裝 node_modules
yarn install
```

---

## 下載必要二進位檔案

Jetson Orin 需要下載 ARM64 專用的 uv、bun、sqlite-vec 等工具：

```bash
yarn download:bin
```

---

## 建立資源目錄結構

```bash
# 建立必要目錄
mkdir -p ~/jan-0.7.9/src-tauri/resources/pre-install
mkdir -p ~/jan-0.7.9/src-tauri/resources/bin

# 複製 uv (Python 環境管理器)
cp -r ~/jan-0.7.9/scripts/dist/uv-aarch64-unknown-linux-gnu ~/jan-0.7.9/src-tauri/resources/bin/

# 複製 bun (JavaScript 執行環境)
cp ~/jan-0.7.9/scripts/dist/bun-linux-aarch64/bun ~/jan-0.7.9/src-tauri/resources/bin/bun

# 複製 sqlite-vec
cp ~/jan-0.7.9/scripts/dist/vec0.so ~/jan-0.7.9/src-tauri/resources/bin/sqlite-vec.so

# 建立 pre-install 佔位檔案（避免 glob 錯誤）
touch ~/jan-0.7.9/src-tauri/resources/pre-install/.gitkeep
```

---

## 編譯擴展套件

```bash
cd ~/jan-0.7.9

# 先 pack core 套件
cd core
yarn pack
cd ..

# 編譯 extensions (產生 .tgz 檔案)
yarn build:extensions
```

---

## 編譯圖示

```bash
cd ~/jan-0.7.9
yarn build:icon
```

---

## 編譯 Web 前端

```bash
cd ~/jan-0.7.9
export IS_TAURI=true
yarn build:web
```

---

## 編譯 jan-cli (CLI 工具)

```bash
source ~/.cargo/env
cd ~/jan-0.7.9/src-tauri

# 編譯 jan-cli
cargo build --release --features cli --bin jan-cli -j 1

# 複製到 resources/bin
cp ~/jan-0.7.9/src-tauri/target/release/jan-cli ~/jan-0.7.9/src-tauri/resources/bin/jan-cli
```

---

## 編譯完整 Jan GUI 應用程式

```bash
source ~/.cargo/env
cd ~/jan-0.7.9/src-tauri

# 完整編譯 (包含 GUI)
cargo build --release -j 1
```

---

## 驗證編譯結果

```bash
# 檢查 Jan binary
ls -la ~/jan-0.7.9/src-tauri/target/release/Jan

# 檢查 jan-cli 版本
~/jan-0.7.9/src-tauri/target/release/jan-cli --version
```

預期輸出：
```
Jan: ELF 64-bit LSB pie executable, ARM aarch64
jan 0.7.9
```

---

## 建立桌面捷徑

### 1. 複製圖示到桌面
```bash
cp ~/jan-0.7.9/src-tauri/icons/128x128.png ~/Desktop/jan-icon.png
```

### 2. 建立 .desktop 檔案
```bash
cat > ~/Desktop/jan.desktop << 'EOF'
[Desktop Entry]
Name=Jan AI
Comment=Jan AI Desktop Application v0.7.9
Exec=/home/kghsai/jan-0.7.9/src-tauri/target/release/Jan
Icon=/home/kghsai/Desktop/jan-icon.png
Terminal=false
Type=Application
Categories=AI;Chat;Utility;
StartupWMClass=Jan
EOF

chmod +x ~/Desktop/jan.desktop
```

---

## 啟動 Jan GUI

### 檢查桌面環境
```bash
# 確認有 X server 運行
ps aux | grep Xorg

# 確認目前使用者有桌面會話
w
# 應該顯示 :1 或 :0 的登入會話
```

### 啟動 Jan
```bash
# 方式一：直接啟動（需要有 GUI 桌面）
DISPLAY=:1 /home/kghsai/jan-0.7.9/src-tauri/target/release/Jan

# 方式二：背景執行
DISPLAY=:1 nohup /home/kghsai/jan-0.7.9/src-tauri/target/release/Jan > /tmp/jan_gui.log 2>&1 &

# 方式三：透過 desktop 捷徑
# 在 GNOME 桌面點擊 jan.desktop
```

---

## 驗證 Jan 運行狀態

```bash
# 檢查程序
pgrep -a -f "target/release/Jan"

# 查看日誌
tail -20 /tmp/jan_gui.log

# 查看詳細程序狀態
cat /proc/$(pgrep -f "release/Jan")/status | grep -E "Name|State|VmRSS|Threads"
```

正常運行狀態：
- State: S (sleeping)
- VmRSS: 約 150-200 MB
- Threads: 多執行緒 (含 WebKit 子進程)

---

## 常見問題排除

### 1. Cargo 編譯被系統終止 (SIGTERM)
症狀：編譯到一半突然終止，exit code 為 -15
可能原因：系統程序管理問題、記憶體不足、磁碟空間不足
解決方案：
- 嘗試重新開機後再編譯
- 使用 `cargo clean` 清理後重新編譯
- 降低並行 jobs 數量：`-j 1`

### 2. GTK 初始化失敗
症狀：`Failed to initialize GTK` 錯誤
解決方案：
- 確認 DISPLAY 環境變數已設定 (DISPLAY=:1)
- 確認 X server 正常運行
- 確認 GTK3 開發庫已正確安裝

### 3. Web 前端編譯失敗 (@janhq/core 找不到)
解決方案：
```bash
# 重新 pack core 套件
cd ~/jan-0.7.9/core
yarn pack

# 重新安裝 extensions
cd ~/jan-0.7.9
yarn build:extensions
```

### 4. Yarn 4.x 權限問題
解決方案：
```bash
sudo corepack enable
sudo corepack prepare yarn@4.5.3 --activate
```

### 5. 缺少圖示檔案
症狀：`failed to open icon /32x32.png: No such file or directory`
解決方案：
```bash
yarn build:icon
```

### 6. 缺少資源檔案
症狀：`resource path resources/bin/uv-aarch64-unknown-linux-gnu doesn't exist`
解決方案：
```bash
yarn download:bin
mkdir -p src-tauri/resources/pre-install
touch src-tauri/resources/pre-install/.gitkeep
cp -r scripts/dist/uv-aarch64-unknown-linux-gnu src-tauri/resources/bin/
```

---

## Jan 版本資訊

- **編譯版本**: 0.7.9
- **二進位位置**: `~/jan-0.7.9/src-tauri/target/release/Jan`
- **CLI 位置**: `~/jan-0.7.9/src-tauri/target/release/jan-cli`
- **資料目錄**: `~/.local/share/Jan/`
- **API Server**: `http://127.0.0.1:1337` (啟動後自動運行)

---

## 快速指令總結

```bash
# 1. SSH 連線
ssh kghsai@192.168.5.29

# 2. 下載並解壓縮 v0.7.9
wget https://github.com/janhq/jan/archive/refs/tags/v0.7.9.tar.gz
tar -xzf v0.7.9.tar.gz && rm v0.7.9.tar.gz

# 3. 安裝依賴
cd ~/jan-0.7.9
yarn install
yarn download:bin

# 4. 準備資源
mkdir -p src-tauri/resources/pre-install src-tauri/resources/bin
cp -r scripts/dist/uv-aarch64-unknown-linux-gnu src-tauri/resources/bin/
cp scripts/dist/bun-linux-aarch64/bun src-tauri/resources/bin/bun
cp scripts/dist/vec0.so src-tauri/resources/bin/sqlite-vec.so
touch src-tauri/resources/pre-install/.gitkeep

# 5. 編譯
yarn build:icon
cd core && yarn pack && cd ..
yarn build:extensions
export IS_TAURI=true && yarn build:web
source ~/.cargo/env && cd src-tauri && cargo build --release --features cli --bin jan-cli -j 1
cp target/release/jan-cli src-tauri/resources/bin/jan-cli
cargo build --release -j 1

# 6. 建立桌面捷徑
cp src-tauri/icons/128x128.png ~/Desktop/jan-icon.png
cat > ~/Desktop/jan.desktop << 'EOF'
[Desktop Entry]
Name=Jan AI
Comment=Jan AI Desktop Application v0.7.9
Exec=/home/kghsai/jan-0.7.9/src-tauri/target/release/Jan
Icon=/home/kghsai/Desktop/jan-icon.png
Terminal=false
Type=Application
Categories=AI;Chat;Utility;
StartupWMClass=Jan
EOF
chmod +x ~/Desktop/jan.desktop

# 7. 啟動 Jan
DISPLAY=:1 nohup ~/jan-0.7.9/src-tauri/target/release/Jan > /tmp/jan_gui.log 2>&1 &

# 8. 驗證
pgrep -a -f "release/Jan"
```

---

## 參考資源

- [Jan GitHub](https://github.com/janhq/jan)
- [Jan Official Site](https://jan.ai/)
- [Jetson Orin Nano](https://developer.nvidia.com/embedded/jetson-orin-nano)
