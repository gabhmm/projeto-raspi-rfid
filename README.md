#  Sistema de Controle de Acesso e Monitoramento — Sala Restrita AAA

> **Projeto de Solução Distribuída (IoT + Aplicação Web + Análise de Dados)**  
> Controle de acesso físico e digital por RFID para proteção de propriedade intelectual sob acordo de confidencialidade (**NDA — Non-Disclosure Agreement**).

---

##  Identificação do Grupo (Integrantes)

| Nome Completo | ID / RM / Matrícula | Função / Responsabilidade no Projeto |
| :--- | :--- | :--- |
| **[Gabriel M Magalhaes]** | `[1137910]` | Backend Flask, Banco de Dados SQLite & API REST |
| **[Guilherme Matte Embarach]** | `[1137953]` | Backend Flask, Banco de Dados SQLite & API REST |
| **[Pedro Arthur Silvestri]** | `[1140338]` | Front-end Web (Gerenciamento & Monitoramento em Tempo Real) |
| **[Brayan Lima]** | `[1132051]` | Sistema Embarcado Raspberry Pi (RFID, Atuadores & Cache Offline) & Documentação |
| **[Adriano Cesar]** | `[1138080]` | Análise de Dados com Pandas e PDF (Jupyter Notebook) |

---

##  Contexto e Objetivo do Trabalho

### O Problema
Um estúdio de desenvolvimento de jogos fechou pela primeira vez um contrato de *outsourcing* com uma **publisher AAA** para desenvolver o design de personagens confidenciais que ainda não foram anunciados ao público. Devido a um rigoroso contrato de **NDA (*Non-Disclosure Agreement*)**, qualquer vazamento acarretaria severas penalidades contratuais e financeiras. Para proteger os materiais, o estúdio isolou uma sala física de acesso restrito e rede local controlada.

### A Solução (Objetivo Geral)
Desenvolver uma solução distribuída capaz de controlar o acesso físico à sala restrita via crachás **RFID**, centralizar os registros em uma **aplicação conectada (Flask + SQLite)**, oferecer um **sistema web de gerenciamento de colaboradores/permissões**, exibir um **painel de monitoramento em tempo real para uma empresa de segurança privada** e disponibilizar dados estruturados (`.csv`) para auditoria e **análise de dados com Pandas**.

---

##  Arquitetura e Funcionalidades Principais

```mermaid
graph LR
    Pi["Raspberry Pi + RFID"] -->|POST /api/validar| Server["🖥️ Servidor Flask + SQLite"]
    Pi -->|GET /api/colaboradores (Boot + 3 min)| Server
    Admin["Administrador"] -->|Login + CRUD Colaboradores/Tags| Web["🌐 Interface Web"]
    Seguranca[" Segurança Privada"] -->|Monitoramento Ao Vivo (Polling 2s)| Web
    Web -->|API JSON| Server
    Pandas["Análise Pandas"] -->|GET /api/eventos.csv| Server
```

1. **Aplicação Conectada (`server/`):**
   - Servidor RESTful em **Python (Flask)** com banco de dados relacional **SQLite (`acesso.db`)** atuando como a versão única da verdade do sistema.
   - Máquina de estados que classifica cada leitura de crachá em:
     - `ENTRADA` (1ª entrada no dia: *"Bem-vindo, {nome}"* | retorno no mesmo dia: *"Bem-vindo de volta, {nome}"*);
     - `SAIDA` (*"Até logo, {nome}"*);
     - `NEGADO` (colaborador cadastrado, porém inativo ou sem permissão de acesso à sala do projeto);
     - `INVASAO` (tentativa de acesso com tag RFID desconhecida/não cadastrada).
   - Cálculo em tempo real de presença (`na_sala`), se já compareceu no dia (`entrou_hoje`) e do **tempo diário acumulado de permanência** (`tempo_hoje_segundos`).

2. **Sistema de Gerenciamento de Acessos (`client/templates/colaboradores.html`):**
   - Autenticação administrativa (`POST /login`).
   - Cadastro, listagem, edição e exclusão de colaboradores (`nome`, `matricula`, `tag_uid`, `cargo`, `acesso_sala`, `ativo`).
   - Alteração rápida de permissão de sala e visualização do tempo diário de permanência.

3. **Página de Monitoramento em Tempo Real (`client/templates/monitor.html`):**
   - Voltada para a **empresa de segurança privada**, atualizada automaticamente a cada **2 segundos** (*polling*).
   - Exibe colaboradores atualmente dentro da sala, alertas críticos recentes (`INVASAO` e `NEGADO`), últimas entradas, últimas saídas, acessos negados e tentativas de invasão.
   - Inclui barra de **Simulador Rápido de Leitura RFID** para testes de bancada.

4. **Suporte à Operação Offline da Raspberry Pi & Exportação Pandas:**
   - Rota `GET /api/colaboradores` sincronizada pela Raspberry Pi na inicialização e a cada **3 minutos** (`cache.json`).
   - Rota `GET /api/eventos.csv` para exportação direta dos logs de acesso para análise no **Pandas**.

---

## Estrutura do Repositório

```text
raspi-projeto/
├── client/                        # Front-end Web (Gerenciamento & Monitoramento)
│   ├── static/
│   │   ├── css/styles.css         # Tema Dark Mode (Centro de Controle AAA)
│   │   └── js/                    # Scripts de integração (api.js, colaboradores.js, monitor.js)
│   └── templates/                 # Telas HTML (login.html, colaboradores.html, monitor.html)
├── server/                        # Backend Flask + SQLite
│   ├── app.py                     # Rotas HTTP, Autenticação e API REST
│   ├── config.py                  # Configurações de ambiente e credenciais
│   ├── database.py                # Repositório SQLite3 e inicialização automática
│   ├── models.py                  # Entidades com Dataclasses e Type Hints
│   └── services.py                # Regras de negócio, validação RFID e cálculo de permanência
├── tests/                         # Testes automatizados de integração
│   └── test_server.py             # Suíte unittest validando rotas, regras e CSV
├── api contratos.md               # Especificação enxuta do contrato de API v1
├── guia_testes_api.md             # Documentação detalhada para testes via Postman/Insomnia
├── requirements.txt               # Dependências Python do projeto
└── README.md                      # Documentação principal do repositório
```

---

## Como Executar o Projeto

### 1. Configurar e ativar o ambiente virtual
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### 2. Iniciar o Servidor Flask
Na raiz do projeto, execute:
```powershell
.\.venv\Scripts\python.exe -m server.app
```
O servidor ficará disponível em:
- **Acesso Local:** `http://127.0.0.1:5000`
- **Rede Local (Raspberry Pi):** `http://<IP_DO_SERVIDOR>:5000`

### 3. Credenciais de Acesso ao Painel Web
- **URL de Login:** `http://127.0.0.1:5000/login`
- **Usuário:** `admin`
- **Senha:** `admin`

---

## Como Executar os Testes Automatizados e Testar a API

Para rodar a suíte de testes automatizados (`unittest`):
```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```

Para testar manualmente cada endpoint pelo **Postman** ou **Insomnia**, consulte o guia completo com todos os exemplos de JSON em:
- 📄 **[guia_testes_api.md](./guia_testes_api.md)**
- 📄 **[api contratos.md](./api%20contratos.md)**
