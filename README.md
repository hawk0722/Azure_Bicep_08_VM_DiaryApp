# Bicep Sample

## Azure Resource

This includes of the following resources:

- Virtual network
- Network interface
- Virtual machine
- Network security group
- Public IP address
- Key Vault

## OS and Middleware

- Linux (Ubuntu)
- Apache
- MySQL
- PHP

This includes of the following resources:

## SystemConfiguration

![SystemConfiguration](/img/SystemConfiguration.svg)

```
/var/www/html/diary-app/
├── index.php              # 投稿一覧ページ
├── config.php             # DB接続設定
├── db/init.sql            # データベース初期化SQL
├── css/style.css          # スタイル
├── entries/               # 投稿の作成・編集・削除
│   ├── create.php
│   ├── edit.php
│   └── delete.php
└── templates/             # ヘッダー・フッター共通部品
    ├── header.php
    └── footer.php
```

## Instructions

Make sure have the [Azure CLI and Bicep](https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/install)

1. Update parameter in main.bicep
2. To log in Azure, Run the following command.

```bash:bash
az login

az account show
```

3. To Create resource group, Run the following command.

```bash:bash
az group create --name <your resource group name> --location <location>

# az group create --name rg-diary-dev --location japaneast
```

4. To Deploy Azure resources, Run the following command.<br>
   ※There should be main.bicep and cloud-init.txt in the current directory.

```bash:bash
az deployment group create --template-file main.bicep --resource-group <your resource group name> --parameters adminPassword='<login password for the Azure VM>' dbPassword='<login password for Database>'

# az deployment group create --template-file main.bicep --resource-group rg-diary-dev --parameters adminPassword='P@ssw0rd1234!' dbPassword='P@ssw0rd1234!'
```

5. Upload diary-app.tar.gz to Azure VM.

6. After login into the virtual machine(App), Run the following command.

```bash:bash(App)
sudo tar -xzvf ~/diary-app.tar.gz -C /var/www/html/

sudo rm /var/www/html/index.html

sudo sed -i "s/\$vaultName = 'kv-diary-dev-.*';/\$vaultName = 'kv-diary-dev-<***>';/" /var/www/html/diary-app/config.php
sudo sed -i "s/\$host = '.*';/\$host = '10.0.2.4';/" /var/www/html/diary-app/config.php

sudo systemctl restart apache2

ssh azureuser@10.0.2.4 // go to db server

```

7. After login into the virtual machine(DB), Run the following command.

```bash:bash(DB)
sudo sed -i 's/^bind-address\s*=.*/bind-address = 0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf

sudo systemctl restart mysql

sudo mysql -u root

# SQL
CREATE USER 'root'@'%' IDENTIFIED BY 'P@ssw0rd1234!';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
FLUSH PRIVILEGES;
exit

exit // go to app server
```

8. After login into the virtual machine(App), Run the following command.

```bash:bash(App)
mysql -h 10.0.2.4 -u root -p < /var/www/html/diary-app/db/init.sql

sudo mysql -h 10.0.2.4 -u root -p -D diary -e "SHOW DATABASES;"
```

9 . Enter http://publicIPAddress/diary-app in your browser.

## Notes

- The deployment was tested on windows.

```

```
