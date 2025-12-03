
```markdown
# OctoPrint + Obico no Orange Pi Zero 2W

Guia completo e testado (2025) para instalar o **OctoPrint com Obico** no **Orange Pi Zero 2W**, já que a imagem oficial do Obico **não dá boot** nesse modelo.

A imagem pré-pronta do Obico foi feita para o Orange Pi Zero 2 (placa maior), não para o Zero 2W. A solução mais estável é instalar manualmente o OctoPrint em uma imagem oficial do Orange Pi Zero 2W.

## Requisitos

- Orange Pi Zero 2W (1GB, 1.5GB, 2GB ou 4GB)
- Cartão MicroSD de boa qualidade (mínimo 16GB, classe 10 ou superior)
- Fonte 5V ≥ 2A com cabo de boa qualidade
- Acesso à internet (Wi-Fi ou cabo Ethernet)

## Passo 1: Baixar e gravar a imagem oficial

1. Acesse a página oficial:  
   http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-Zero-2W.html

2. Baixe uma das imagens recomendadas:
   - **Debian 12 Bookworm** (mais estável)
   - ou **Ubuntu 22.04/24.04**

3. Grave no cartão usando:
   - **Raspberry Pi Imager** (recomendado)
   - ou Balena Etcher

Dica: no Raspberry Pi Imager, clique no ícone de engrenagem e pré-configure:
   - Hostname: `orangepi`
   - Usuário: `orangepi` / Senha: escolha a sua
   - Wi-Fi (SSID e senha)

## Passo 2: Primeiro boot e acesso via SSH

1. Insira o cartão e ligue o Orange Pi Zero 2W
2. Descubra o IP no seu roteador ou use:
   ```bash
   nmap -sn 192.168.1.0/24
   ```
3. Conecte via SSH:
   ```bash
   ssh orangepi@SEU_IP
   ```
   Senha padrão (se não mudou): `orangepi`

## Passo 3: Atualizar o sistema

```bash
sudo apt update && sudo apt upgrade -y
sudo dpkg --configure -a
sudo apt update
sudo apt install python3 python3-pip python3-venv python3-dev git curl -y
```

## Passo 4: Criar usuário dedicado para o OctoPrint

```bash
sudo adduser --disabled-password --gecos "" octoprint
sudo usermod -a -G tty,dialout octoprint
echo "octoprint ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/octoprint
```

## Passo 5: Instalar o OctoPrint

```bash
sudo su - octoprint
python3 -m venv ~/OctoPrint
source ~/OctoPrint/bin/activate
pip install --upgrade pip setuptools wheel
pip install octoprint
exit
```

## Passo 6: Criar serviço systemd (inicia automaticamente)

```bash
sudo nano /home/octoprint/octoprint.service
```

Cole o conteúdo abaixo:

```ini
[Unit]
Description=OctoPrint - The snappy web interface for your 3D printer
After=network-online.target
Wants=network-online.target

[Service]
User=octoprint
WorkingDirectory=/home/octoprint
ExecStart=/home/octoprint/OctoPrint/bin/octoprint serve
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Salve (Ctrl+O → Enter → Ctrl+X) e ative o serviço:

```bash
sudo cp /home/octoprint/octoprint.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable octoprint
sudo systemctl start octoprint
```

## Passo 7: Acessar o OctoPrint

Abra no navegador:
```
http://SEU_IP:5000
```

Faça o wizard de configuração inicial.

## Passo 8: Instalar o plugin Obico (The Spaghetti Detective)

1. No OctoPrint, vá em **Settings → Plugin Manager → Get More**
2. Procure por: `Obico for OctoPrint`
3. Instale
4. Faça login ou crie sua conta Obico
5. Linke o túnel (é tudo automático)

Pronto! Agora você tem OctoPrint + monitoramento remoto funcionando perfeitamente no Orange Pi Zero 2W.

## Ver status do serviço

```bash
sudo systemctl status octoprint
journalctl -u octoprint -f    # ver logs em tempo real
```

## Atualizar OctoPrint no futuro

```bash
sudo systemctl stop octoprint
sudo su - octoprint
source ~/OctoPrint/bin/activate
pip install --upgrade octoprint
exit
sudo systemctl start octoprint
```

## Problemas comuns

| Sintoma                  | Solução provável                          |
|--------------------------|-------------------------------------------|
| LED vermelho fixo        | Fonte fraca ou cartão SD ruim             |
| Não conecta no Wi-Fi     | Pré-configurar no Raspberry Pi Imager     |
| Não acessa porta 5000    | Verifique se o serviço está rodando       |
| Erro de permissão serial | Usuário octoprint precisa estar no grupo dialout |

## Links úteis

- Site oficial Orange Pi Zero 2W: http://www.orangepi.org
- Obico (ex-Spaghetti Detective): https://www.obico.io
- Documentação OctoPrint: https://docs.octoprint.org

Qualquer dúvida, abra uma issue aqui no repositório!
