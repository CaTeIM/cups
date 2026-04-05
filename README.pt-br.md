# 🐳 Servidor de Impressão CUPS - Imagem Docker Multi-Arquitetura

![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/CaTeIM/cups-docker/cups.yml?branch=main&style=for-the-badge)
![Docker Hub Pulls](https://img.shields.io/docker/pulls/cateim/cups?style=for-the-badge)
![Docker Image Size](https://img.shields.io/docker/image-size/cateim/cups/latest?style=for-the-badge)

*[🇺🇸 Read in English](README.md)*

Esta é uma imagem Docker multi-arquitetura do **[CUPS (Common Unix Printing System)](https://github.com/OpenPrinting/cups)**, construída sobre as bases mais recentes do **Ubuntu (Development)** e **Debian (Testing)**. O objetivo é fornecer um servidor de impressão com as versões mais recentes do CUPS, prontas para uso e fáceis de implantar em ambientes containerizados.

## 📚 Código-Fonte

Este projeto é de código aberto. O `Dockerfile`, o script de inicialização e o workflow de build do GitHub Actions estão todos disponíveis no repositório do projeto.

➡️ **[Repositório no GitHub: CaTeIM/docker-cups](https://github.com/CaTeIM/docker-cups)**

## 🐳 Tags Disponíveis

Este repositório constrói duas "trilhas" de imagem. A tag `latest` sempre aponta para a base Ubuntu.

| Tag | Base da Distro | Versão CUPS | Estabilidade |
| :--- | :--- | :--- | :--- |
| `latest`, `ubuntu`, `2.4.12` | Ubuntu 25.10 (Questing Quokka) | `2.4.12` | ⚠️ Development |
| `debian`, `2.4.10` | Debian 13 (Trixie) | `2.4.10` | ⚠️ Testing |

## ✨ Por que usar esta imagem?

-   ✅ **Sempre Atualizado**: Utiliza o método de instalação `apt-get` a partir dos repositórios oficiais do Ubuntu 25.10 e Debian 13, garantindo as versões mais recentes do CUPS.

-   ✅ **Multi-Distro**: Escolha entre uma base Ubuntu (`latest`) ou Debian (`debian`), dependendo da sua preferência.

-   🔒 **Segura**: O processo de build inclui a aplicação de todas as atualizações de segurança disponíveis (`apt-get upgrade`).

-   🖨️ **Pronta para Uso**: Inclui um conjunto completo de drivers de impressão (`printer-driver-all`, `hplip`, `openprinting-ppds`), tornando a maioria das impressoras plug-and-play.

-   🚀 **Multi-Arquitetura**: Construída para rodar nativamente em `linux/amd64` (PCs, Servidores Intel/AMD) e `linux/arm64` (Raspberry Pi, Orange Pi 5, etc.).

-   🔧 **Configuração Inteligente**: Possui um script de inicialização que configura um usuário administrador e prepara o CUPS para acesso remoto na primeira execução.

## ⚙️ Como Usar (Exemplo com `docker-compose.yml`)

A forma recomendada de usar esta imagem é com o Portainer Stacks ou `docker-compose`. Crie um arquivo `docker-compose.yml` com o seguinte conteúdo.

> **Nota:** Para impressoras USB funcionarem corretamente (detectar falta de papel, reconexão, etc.), o mapeamento dos volumes `/run/udev` e `/run/dbus` é **essencial**.

```yaml
services:
  cups:
    # Use 'latest' (Ubuntu), 'debian', ou tags de versão como '2.4.12'
    image: cateim/cups:latest
    container_name: cups
    # Libera acesso total do container aos dispositivos do sistema (obrigatório para USB)
    privileged: true
    restart: unless-stopped
    environment:
      # Defina aqui uma senha segura para o usuário 'admin' da interface web
      - ADMIN_PASSWORD=sua_senha_forte
      # Define o fuso horário
      - TZ=America/Sao_Paulo
    volumes:
      # --- Configuração e Dados ---
      - /srv/cups/config:/etc/cups
      - /srv/cups/logs:/var/log/cups
      - /srv/cups/spool:/var/spool/cups
      
      # --- Hardware e Sistema (CRÍTICO PARA USB) ---
      # Acesso físico às portas USB
      - /dev/bus/usb:/dev/bus/usb
      # Permite comunicação com serviços do sistema (corrige erros de ColorManager/DBus)
      - /run/dbus:/run/dbus:ro
      # Permite ao CUPS detectar eventos de hardware (ex: recolocar papel, abrir tampa)
      - /run/udev:/run/udev:ro
      
      # Sincroniza o relógio com o Host
      - /etc/localtime:/etc/localtime:ro
      
    # 'host' é a forma mais fácil de garantir a descoberta de impressoras na rede (AirPrint/Bonjour)
    # Se preferir 'bridge', certifique-se de expor a porta 631:631
    network_mode: host
```

### 🔑 Administração

  - Para acessar a interface web, use o endereço: `https://<IP_DO_SEU_SERVIDOR>:631`
  - Para acessar a área de **Administration**, use o login `admin` e a senha que você definiu na variável `ADMIN_PASSWORD`.

## 🐛 Solução de Problemas (USB e Impressoras "Host-Based")

Se você utiliza impressoras USB que dependem de firmware carregado pelo host (como HP LaserJet P1102, P1005, série 1020, etc.), você pode notar que **desligar e ligar a impressora** faz com que o CUPS pare de responder ou deixe trabalhos como "Retidos".

Isso ocorre porque o Container perde a referência do dispositivo USB quando a conexão elétrica cai.

### Solução Definitiva (Regra UDEV no Host)

Para que o Container reconecte automaticamente sempre que a impressora for reiniciada ou o cabo reconectado, crie uma regra `udev` no seu sistema hospedeiro (não no container).

1.  Descubra o ID da sua impressora com o comando `lsusb` (ex: `03f0:002a`).
2.  Crie o arquivo `/etc/udev/rules.d/99-fix-cups-usb.rules` com o conteúdo abaixo (substituindo pelo seu ID):

```bash
# Reinicia o container CUPS automaticamente ao detectar a conexão da impressora
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="SEU_VENDOR_ID", ATTR{idProduct}=="SEU_PRODUCT_ID", RUN+="/usr/bin/docker restart cups"
```

3.  Recarregue as regras: `udevadm control --reload-rules && udevadm trigger`

### Dica Operacional (Falta de Papel)

Se o papel acabar e o trabalho ficar retido, evite desligar a impressora.

1.  Coloque o papel.
2.  **Abra e feche a tampa do toner/cartucho.**
3.  O volume `/run/udev` mapeado no container detectará o evento e o CUPS retomará a impressão automaticamente.

---

*Este projeto não é oficialmente afiliado à OpenPrinting. Todo o crédito pelo CUPS vai para seus respectivos desenvolvedores.*
