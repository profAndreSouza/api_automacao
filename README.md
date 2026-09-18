# Orquestração de Dados e Fluxos IIoT com Node-RED

Bem-vindo ao laboratório prático da **Semana 05** da disciplina de **Automação Industrial**!

Nesta aula prática, toda a infraestrutura roda **100% conteinerizada via Docker** (3 contêineres independentes), sem necessidade de instalar pacotes ou dependências no computador do aluno:
1. **Eclipse Mosquitto (MQTT Broker):** Responsável pelo roteamento de mensagens na porta `1883` (TCP) e `9001` (WebSocket).
2. **Node-RED:** Ambiente de programação visual baseada em fluxos na porta `443` (HTTP).
3. **Simulador Flask (Fábrica Virtual / API):** Aplicação monolítica Python com motor de simulação de sensores industriais e painel SCADA web na porta `80` (HTTP padrão).

---

## Arquitetura dos 3 Contêineres Docker

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      REDE DOCKER (rede_automacao)                       │
│                                                                         │
│  ┌───────────────────────┐             ┌─────────────────────────────┐  │
│  │   SIMULADOR / API     │             │   ECLIPSE MOSQUITTO         │  │
│  │  (flask_simulator_app)│ ──────────► │   (mosquitto_broker)        │  │
│  │   http://localhost    │ MQTT :1883  │   Porta TCP: 1883           │  │
│  │         (:80)         │             │   Porta WS:  9001           │  │
│  └───────────────────────┘             └──────────────┬──────────────┘  │
│                                                       │                 │
│                                                       │ Subscrição      │
│                                                       │ fabrica/#       │
│                                                       ▼                 │
│                                        ┌─────────────────────────────┐  │
│                                        │          NODE-RED           │  │
│                                        │        (nodered_app)        │  │
│                                        │    http://localhost:443     │  │
│                                        │  [PAINEL LATERAL DEBUG 🪲]  │  │
│                                        └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Mapeamento de Portas

| Serviço | Porta no Host (Externa) | Porta no Contêiner (Interna) | Protocolo / URL |
| :--- | :--- | :--- | :--- |
| **API / Simulador Web** | `80` | `5000` | HTTP: [http://localhost](http://localhost) |
| **Node-RED** | `443` | `1880` | HTTP: [http://localhost:443](http://localhost:443) |
| **MQTT Broker (TCP)** | `1883` | `1883` | MQTT: `localhost:1883` |
| **MQTT Broker (WebSocket)** | `9001` | `9001` | WS: `ws://localhost:9001` |

> [!NOTE]
> Como o Node-RED está exposto na porta 443 usando HTTP simples (sem certificado SSL/TLS), utilize explicitamente o prefixo `http://` no navegador: `http://localhost:443`.

---

## Como Executar Localmente

Abra o terminal na pasta do projeto e execute:

```bash
docker compose up --build -d
```

Verifique se os 3 contêineres estão em execução:
```bash
docker compose ps
```

Você verá:
- `mosquitto_broker` (Up - portas 1883, 9001)
- `nodered_app` (Up - porta 443 -> 1880)
- `flask_simulator_app` (Up - porta 80 -> 5000)

---

## Deploy na AWS EC2 (Amazon Linux)

Para subir e executar a aplicação em uma instância EC2 rodando **Amazon Linux**, siga o passo a passo abaixo:

### 1. Instalação e Inicialização do Docker

Execute os seguintes comandos no terminal da instância:

```bash
sudo yum update

sudo yum install docker

sudo systemctl enable docker.service

sudo systemctl start docker.service

sudo usermod -aG docker $USER

newgrp docker
```

> [!TIP]
> Caso a sua versão do Amazon Linux não venha com o plugin `docker compose` integrado por padrão, você pode instalar o Docker Compose Plugin via comando:
> ```bash
> sudo mkdir -p /usr/local/lib/docker/cli-plugins/
> sudo curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
> sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
> ```

### 2. Configuração do Security Group na EC2

Certifique-se de que o **Security Group** associado à sua instância EC2 possui as seguintes regras de entrada (Inbound Rules) liberadas:
- **Porta 80 (TCP / HTTP):** Acesso à **API / Simulador Web**
- **Porta 443 (TCP / Custom HTTP):** Acesso ao **Node-RED**
- *(Opcional)* **Porta 1883 (TCP)** e **9001 (WS):** Caso queira conectar clientes MQTT externos diretamente ao Mosquitto.

### 3. Subir a Aplicação na EC2

Dentro da pasta do projeto na EC2:

```bash
docker compose up --build -d
```

### 4. Portas e URLs de Acesso na AWS

Com a aplicação rodando, acesse via navegador utilizando o **IPv4 Público** ou **DNS Público** da sua instância EC2:

- **API / Simulador Web (Porta 80):**
  - URL: `http://<IP_PUBLICO_EC2>` (ou `http://<IP_PUBLICO_EC2>:80`)
- **Node-RED (Porta 443):**
  - URL: `http://<IP_PUBLICO_EC2>:443`

> [!NOTE]
> Como o Node-RED está exposto na porta `443` utilizando HTTP simples (sem certificado SSL/TLS), utilize obrigatoriamente o prefixo `http://` no navegador: `http://<IP_PUBLICO_EC2>:443`. Se digitar `https://`, o navegador não conseguirá estabelecer a conexão.

---

## Acessos e Roteiro de Aula

### 1. Acessar a Fábrica Virtual (Simulador SCADA / API)
 Abra no navegador: **[http://localhost](http://localhost)** (ou [http://localhost:80](http://localhost:80))
- Acompanhe a telemetria em tempo real (Temperatura, Vibração, Pressão, Corrente).
- Observe as contagens de produção e taxa de qualidade.
- Use os botões de ação rápida para simular falhas e eventos (Superaquecimento, Vibração Excessiva, Parada de Emergência E-STOP, etc.).

### 2. Acessar o Node-RED e Importar o Fluxo
 Abra no navegador: **[http://localhost:443](http://localhost:443)**
1. Pressione `Ctrl + I` (ou Menu hambúrguer `☰` > **Import**).
2. Copie o conteúdo do arquivo [`flows_semana05.json`](file:///c:/projetos/Material/Automação%20Industrial/materiais/semana_05/nodered/flows_semana05.json) e cole na caixa de texto.
3. Clique em **Import** e em seguida no botão vermelho **Deploy** (canto superior direito).
4. Abra o **Painel lateral Debug** (ícone do inseto `🪲` ou atalho `Ctrl + G` seguido de `D`).

### 3. Observação dos Eventos e Depuração
- Com a aba Debug aberta no Node-RED, volte à tela da API/Simulador ([http://localhost](http://localhost)) e dispare os eventos.
- Observe a classificação de criticidade e os tratamentos nos nós:
  -  `[DEBUG] Telemetria Formatada`
  -  `[DEBUG] Alarme Crítico / Emergência`
  -  `[DEBUG] Produção & Qualidade`
  -  `[DEBUG] Todas Msg Brutas`

---

##  Como Parar os Contêineres

Para encerrar a execução dos serviços:
```bash
docker compose down
```

---

##  Subir em um Repositório Git Novo

Caso deseje disponibilizar esta pasta como um repositório Git independente:

```bash
# 1. Acesse a pasta semana_05
cd materiais/semana_05

# 2. Inicialize o repositório git local
git init -b main

# 3. Adicione todos os arquivos
git add .

# 4. Crie o primeiro commit
git commit -m "feat: lab semana 05 - Node-RED (443), API (80), MQTT (1883)"

# 5. Adicione a URL do seu novo repositório remoto (GitHub/GitLab)
git remote add origin https://github.com/SEU_USUARIO/SEU_NOVO_REPOSITORIO.git

# 6. Envie para o repositório
git push -u origin main
```

---

##  Estrutura de Diretórios da Semana 05

A estrutura foi modularizada para manter cada componente isolado em seu próprio diretório:

```
materiais/semana_05/
├── docker-compose.yml          # Orquestração dos 3 contêineres Docker
├── README.md                   # Guia rápido de execução do laboratório
│
├── api/                        # Aplicação Web / Simulador SCADA Flask
│   ├── Dockerfile              # Build da imagem da aplicação Python
│   ├── app.py                  # Servidor Flask e simulador de chão de fábrica
│   ├── requirements.txt        # Dependências Python (Flask, paho-mqtt)
│   ├── static/                 # Folhas de estilo CSS
│   └── templates/              # Painel de controle HTML (SCADA)
│
├── mqtt/                       # Broker MQTT (Eclipse Mosquitto)
│   └── config/
│       └── mosquitto.conf      # Arquivo de configuração e portas (1883 TCP, 9001 WS)
│
└── nodered/                    # Orquestrador de Fluxos IIoT
    └── flows_semana05.json     # Fluxo exportado pronto para importação
```
