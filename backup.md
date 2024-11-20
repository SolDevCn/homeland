要使用 GoBackup 将 PostgreSQL 数据库备份到 Cloudflare R2 中，首先需要确保你具备以下环境和权限：

### 1. **安装 GoBackup**

GoBackup 是一个开源工具，允许你备份 PostgreSQL 数据库到不同的存储后端，包括 Cloudflare R2。你可以从 GoBackup 的 GitHub 页面（[GoBackup GitHub](https://gobackup.github.io/)）获取安装和使用说明。

### 2. **创建 Cloudflare R2 存储桶**

在使用 Cloudflare R2 之前，需要在 Cloudflare 创建一个存储桶，并获取存储桶的 **Access Key** 和 **Secret Key**。
如何创建 Access Key 和 Secret Key， 参见： https://developers.cloudflare.com/r2/api/s3/tokens/

1. 登录到 [Cloudflare 控制台](https://dash.cloudflare.com/)，选择 R2。
2. 创建一个新的 R2 存储桶，选择适合的存储区域。
3. 获取 **Access Key** 和 **Secret Key**，这将在配置 GoBackup 时使用。

### 3. **配置 GoBackup 使用 Cloudflare R2**

GoBackup 支持通过 `s3` 存储后端配置将备份文件上传到 Cloudflare R2，因为 R2 是兼容 Amazon S3 API 的。要使 GoBackup 将备份上传到 Cloudflare R2，需要在配置中指定 Cloudflare R2 的连接信息。

#### 配置步骤：

1. **创建 GoBackup 配置文件**：
   GoBackup 需要一个配置文件来定义备份的详细信息，包括数据库连接信息和云存储的配置。配置文件通常是一个 YAML 格式的文件。

   创建一个名为 `gobackup.yml` 的文件，内容如下：

   ```yaml
   models:
     soldevcn:
       schedule:
         cron: "* * * * *"
       databases:
         postgresql:
           type: "postgresql"
           host: "localhost"
           port: 5432
           username: "__USERNAME__"
           password: "__PASSWORD__"
           database: "__DATABASE_NAME__"
       compress_with:
         type: tgz
       storages:
         store:
           type: r2
           bucket: "__BUCKET_NAME__"
           path: "__PATH__"
           account_id: "__ACCOUNT_ID__"
           access_key_id: "__ACCESS_KEY_ID__"
           secret_access_key: "__SECRET_ACCESS_KEY__"
           endpoint: "__ENDPOINT__"
   ```

   - `access_key_id` 和 `secret_access_key` 是你在 Cloudflare R2 控制台中创建的访问密钥。
   - `endpoint` 是 Cloudflare R2 的 endpoint 地址，一般为 `https://<account_id>.r2.cloudflarestorage.com`。
   - `bucket` 是你创建的 R2 存储桶的名称。

2. **运行 GoBackup 执行备份**：
   配置完成后，可以使用以下命令运行 GoBackup 来执行备份：

   ```bash
   gobackup perform   <== 这是单次触发
   gobackup start     <== 这是启用守护进程模式
   ```

   这将触发 GoBackup 连接到 PostgreSQL 数据库，生成备份文件，并将备份文件上传到 Cloudflare R2 存储桶中。

### 4. **验证备份**

在 Cloudflare R2 控制台中检查存储桶，确认备份文件已经成功上传。

### 5. **测试备份的文件是否可以正常导入**

```bash
psql -h localhost -p 5432 -U postgres -d __DATABASE_NAME -f path_to_the_sql_file.sql
```

### 小结

通过以上步骤，你可以成功将 PostgreSQL 数据库备份到 Cloudflare R2 中。只需确保 GoBackup 配置正确，并设置好定期备份任务，就能保持数据库备份的安全和可靠。
