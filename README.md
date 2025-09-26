# WSL-Ubuntu-Install-Docker

[![](https://github.com/TechTutoPPT/WSL-Ubuntu-Install-Docker/blob/main/cover.PNG)](https://youtu.be/RNIej1BsjoU)

承接上一影片, 即然已安裝了WSL Ubuntu, 那直接於Ubuntu內安裝Docker是否可行呢? 
尤其很多人都遇過安裝Windows版的Docker Desktop後很難卸除, 另外直接於Ubuntu內安裝Docker可有更多的Panel工具去管理容器, 
而答案是可行的, 但需一些技巧去配置, 請跟著我的腳步去體驗吧.
我們先快速確認已執行以下步驟安裝上WSL及Ubuntu(以系統管員身份開啟PowerShell並執行):
```
wsl --install
wsl --update
wsl --shutdown
```

然後按Win+R, 輸入ubuntu開啟Ubuntu CLI環境, 再執行以下指令去安裝相關套件:
```
sudo apt update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
```

執行以下指令去加入Docker官方的GPG金鑰:
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

以下指令去配置Docker軟件源:
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安裝Docker Engine:
```
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

啟動Docker並設定為自動啟動:
```
sudo systemctl start docker
sudo systemctl enable docker
```
驗證Docker是否安裝成功:
```
docker --version
```

假如想執行docker指令時免去加上sudo指令, 可執行以下內容將一般使用者加入docker群組:
```
sudo usermod -aG docker $USER
```
完成後重登或使用newgrp docker指令去使群組變更生效. 

至此已完成WSL Docker的安裝, 但你會發覺應用Docker服務時只能以本機(localhost)使用服務, 內聯網中的其他裝置未能連上, 
這是由於WSL與內聯網不是處於同一網段, 我們可以執行以下指令去查看本機及Ubuntu的IP地址:
本機: Win+R, 輸入cmd開啟終端, 再執行ipconfig, 查看對應網卡, 假如得出的結果是192.168.31.200
Ubuntu: 執行ip a | grep eth0假如得出的結果是172.29.85.119
現在要做的是為它施加一點魔法, 將內聯網其他裝置向本機索取的服務要求以端口轉發方式傳遞給WSL內的Docker容器.
以下是例子, 可因應自訂的端口去改變:
打開PowerShell(以管理員身份)執行以下命令將本機IP的80端口轉發到WLS容器IP的8080端口(若該服務設有多個端口需重複以下指令去映射不同的端口):
```
netsh interface portproxy add v4tov4 listenaddress=192.168.31.200 listenport=80 connectaddress=172.29.85.119 connectport=8080
```
另為免Windows內置的防火牆阻擋了端口的連接, 我們再於PowerShell中執行以下指令去添加防火牆規則:
```
New-NetFirewallRule -DisplayName "Allow Docker Port 80" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow
```
這樣內聯網的其他裝置於瀏覽器中輸入192.168.31.200:80便會經本機轉發到172.29.85.119:8080

規則有添加當然可以有刪減, 以下是刪減的方法:
列出現有轉發規則:
```
netsh interface portproxy show v4tov4
```
刪除特定端口轉發規則:
```
netsh interface portproxy delete v4tov4 listenaddress=192.168.31.200 listenport=80
```

透過防火牆規則名稱確認該規則是否存在:
```
Get-NetFirewallRule -DisplayName "Allow Docker Port 80"
```
刪除該規則:
```
Remove-NetFirewallRule -DisplayName "Allow Docker Port 8080"
```


