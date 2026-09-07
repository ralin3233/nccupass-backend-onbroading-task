# 資料庫連線與管理工具

在政大通的專案中，我們主要使用 **MySQL 8.0** 作為關聯式資料庫。

在 Onboarding 階段，團隊已經為你建置好了開發測試用的資料庫服務；若你想在自己電腦本地啟動 MySQL，我們也提供了 Docker 一鍵啟動指令。

---

## 1. 取得團隊開發資料庫連線資訊

請向你的 Mentor 或在 Discord `#onboarding-help` 索取個人專屬的連線帳密，格式通常如下：

```dotenv
# .env 範例格式
DB_HOST=dev-db.nccupass.internal  # 或團隊提供的測試主機 IP
DB_PORT=3306
DB_USER=nccu_onboarding_user
DB_PASSWORD=your_secure_password
DB_NAME=nccu_memo_db
```

---

## 2. 推薦 GUI 資料庫工具：DBeaver

我們強烈推薦安裝 **DBeaver Community**（免費開源、跨平台、功能強大）：

1. 前往 [DBeaver 官方網站下載安裝](https://dbeaver.io/download/)。
2. 開啟 DBeaver，點擊左上角的 **新連線圖示 (Plug icon)**。
3. 選擇 **MySQL**，點擊下一步。
4. 填入連線參數：
   - **Host**：`DB_HOST`
   - **Port**：`3306`
   - **Database**：`DB_NAME`
   - **Username**：`DB_USER`
   - **Password**：`DB_PASSWORD`
5. 點擊左下角的 **Test Connection（測試連線）**，若提示需要下載驅動，點擊 **Download** 即可。
6. 看到連線成功後，點擊 **Finish**。

---

## 3. 備用方案：使用 Docker 在本地啟動 MySQL (選配)

如果你想在完全離線或自己電腦上獨立跑 MySQL，可以在專案中建立 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: nccupass_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: nccu_memo_db
      MYSQL_USER: nccu_user
      MYSQL_PASSWORD: nccu_password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

在終端機執行一行指令啟動：
```bash
docker compose up -d
```
即可在本地 `localhost:3306` 擁有完整的 MySQL 8.0 環境！
