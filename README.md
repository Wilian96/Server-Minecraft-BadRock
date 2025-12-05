# Server-Minecraft-BadRock
Servidor para Minecraft BadRock

**Visão Geral**

- **Projeto:**: Servidor e ferramentas de deploy/execução para o painel Crafty (versão 4) adaptado para o servidor "Minecraft BadRock".
- **Conteúdo principal:**: instalador (`crafty-installer-4.0`), binários e scripts de execução em `minecraft/`, e o código fonte do painel em `minecraft/crafty-4/`.

**Estrutura do Repositório**

- **`crafty-installer-4.0/`**: Scripts e utilitários para instalação automática em várias distribuições Linux.
- **`minecraft/`**: Scripts de execução, serviço `systemd`, configuração Docker e o código do Crafty (`crafty-4`).
	- **`minecraft/crafty-4/`**: Código do painel (app, config, migrations, frontend, dockerfiles).
	- **`minecraft/crafty.service`**: Unidade `systemd` para executar o serviço como daemon.
	- **`minecraft/run_crafty.sh`** e **`minecraft/run_crafty_service.sh`**: scripts para execução manual e execução em modo daemon.

**O que foi feito / configurações aplicadas**

- **Serviço systemd:**: adicionada a unidade `minecraft/crafty.service` (arquivo em `minecraft/crafty.service`) que aponta para o script `run_crafty_service.sh`. Isso permite gerenciar o painel com `systemctl` e habilitar inicialização automática.
- **Scripts de execução:**: `run_crafty.sh` (modo foreground) e `run_crafty_service.sh` (modo demonizado com `-d`). Ambos ativam o ambiente virtual (`.venv`) antes de executar `python3 main.py`.
- **Instalador:**: pasta `crafty-installer-4.0/` contém scripts para instalar dependências do sistema, criar usuário `crafty` e preparar o ambiente em várias distribuições (Ubuntu, Debian, Fedora, etc.).
- **Docker / Compose:**: templates e `Dockerfile` e `docker-compose.yml` estão disponíveis em `minecraft/crafty-4/docker/` e `minecraft/docker/` para execução em contêiner.
- **Configuração do painel:**: arquivos de configuração estão em `minecraft/crafty-4/app/config/` (ex.: `default.json.example`) e as migrações de banco em `minecraft/crafty-4/migrations/`.

**Pré-requisitos**

- **Sistema:**: Distribuição Linux (de preferência baseada em Debian/Ubuntu ou CentOS/RHEL com adaptadores no instalador).
- **Pacotes:**: `python3`, `python3-venv`, `python3-pip`, `git`, `docker` e `docker-compose` (se usar Docker).

**Instalação manual (modo virtualenv)**

1. Clone o repositório (se ainda não):

```bash
git clone <repo-url> Server-Minecraft-BadRock
cd Server-Minecraft-BadRock
```

2. Criar e ativar o ambiente virtual na pasta `minecraft`:

```bash
cd minecraft
python3 -m venv .venv
source .venv/bin/activate
```

3. Instalar dependências do painel:

```bash
pip install -r crafty-4/requirements.txt
```

4. Ajustar configurações iniciais:

- Copie e edite o arquivo de configuração de exemplo: `minecraft/crafty-4/app/config/default.json.example` -> `minecraft/crafty-4/app/config/default.json`.
- Verifique as configurações de banco e credenciais em `minecraft/crafty-4/app/config/`.

5. Rodar o servidor (modo foreground):

```bash
bash run_crafty.sh
```

6. Rodar em modo daemon (para debugging leve):

```bash
bash run_crafty_service.sh
```

**Instalação como serviço systemd**

- Copie `minecraft/crafty.service` para `/etc/systemd/system/` (ou `/etc/systemd/user/` para serviço de usuário) e recarregue as unidades:

```bash
sudo cp minecraft/crafty.service /etc/systemd/system/crafty.service
sudo systemctl daemon-reload
sudo systemctl enable --now crafty.service
sudo systemctl status crafty.service
```

**Execução com Docker**

- Existem arquivos `docker-compose.yml` e `Dockerfile` dentro de `minecraft/crafty-4/docker/` e `minecraft/docker/`.
- Para rodar com Docker Compose (exemplo):

```bash
cd minecraft/crafty-4/docker
docker-compose up -d --build
```

Observação: ajuste volumes e variáveis de ambiente nos `docker-compose.yml` antes de subir.

**Configurações importantes e caminhos**

- **Unidade systemd:** `minecraft/crafty.service` (no repositório).
- **Scripts de execução:** `minecraft/run_crafty.sh`, `minecraft/run_crafty_service.sh`.
- **Código do painel:** `minecraft/crafty-4/`.
- **Configurações do app:** `minecraft/crafty-4/app/config/` (ex.: `default.json.example`).
- **Migrações DB:** `minecraft/crafty-4/migrations/`.
- **Logs & backups:** veja `minecraft/logs/` e `minecraft/docker/logs/` (conforme configuração do docker/compose).

**O que você precisa revisar / customizar após a instalação**

- Atualizar `default.json` com credenciais seguras e ajustes (senha do admin, endpoints, caminhos de logs).
- Verificar permissões do usuário `crafty` (se o serviço systemd for usado): arquivos e diretórios devem ser acessíveis por esse usuário.
- Configurar rotinas de backup para banco e uploads (script de backup, ou usar `docker` volumes e snapshots).

**Comandos úteis**

- Ativar venv e executar localmente:

```bash
cd /caminho/para/Server-Minecraft-BadRock/minecraft
source .venv/bin/activate
bash run_crafty.sh
```

- Gerenciar serviço systemd:

```bash
sudo systemctl start crafty.service
sudo systemctl stop crafty.service
sudo systemctl restart crafty.service
sudo journalctl -u crafty.service -f
```

- Executar com Docker Compose:

```bash
cd minecraft/crafty-4/docker
docker-compose up -d --build
docker-compose logs -f
```

**Resolução de problemas (quick checks)**

- Não inicia / erro de imports: verifique se o `.venv` está ativado e se as dependências foram instaladas.
- Erros de permissão: confirme o `User=` em `crafty.service` e ajuste permissões dos diretórios do projeto.
- Banco de dados ausente / migrações: execute as migrações que estiverem pendentes em `minecraft/crafty-4/migrations/`.
- Logs: use `journalctl -u crafty.service -f` para ver logs do systemd ou `docker-compose logs -f` para containers.

**Contribuição**

- Abra issues para problemas ou features.
- Para alterações no painel, faça fork, branch com nome descritivo e um pull request.

**Licença**

- Veja o arquivo `LICENSE` no repositório principal e também em `minecraft/crafty-4/`.

**Contato / Referências**

- Scripts de instalação e helper: `crafty-installer-4.0/` (considere usá-los para instalações automatizadas em servidores de produção).
- Código do painel e documentação adicional: `minecraft/crafty-4/README.md` (se existir) e arquivos `CONTRIBUTING.md` dentro da pasta do painel.

--

Se quiser, eu posso:

- adicionar exemplos práticos para `docker-compose.yml` (com volumes e variáveis recomendadas),
- gerar um arquivo de unit `systemd` alternativo ajustado para ambientes não-root, ou
- aplicar um template de `default.json` com campos comentados para facilitar a configuração inicial.

