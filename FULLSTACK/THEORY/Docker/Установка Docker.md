Можно установить через консоль. Если у тебя есть `winget`, открой PowerShell **от имени администратора** и выполни:

```powershell
winget install -e --id Docker.DockerDesktop
```

После установки перезагрузи компьютер или хотя бы выйди/зайди в Windows, потом открой Docker Desktop один раз, дождись статуса “Docker is running”.

Проверка:

```powershell
docker --version
docker compose version
```