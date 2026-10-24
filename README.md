📧 Outlook Account Checker

Ferramenta para verificação em massa de contas Microsoft/Outlook, com suporte a autenticação via protocolo Outlook Lite (Android), busca por palavra-chave nos e-mails e salvamento sem duplicatas.

⚠️ AVISO IMPORTANTE: Este projeto é disponibilizado apenas para fins educacionais e de pesquisa em segurança. O uso indevido, como acesso não autorizado a contas de terceiros, viola os Termos de Serviço da Microsoft e pode ser crime em diversas jurisdições (LGPD, CFAA, GDPR, etc.). Você é o único responsável pelo uso desta ferramenta.

---

📋 Índice

· Funcionalidades
· Requisitos
· Instalação
· Como Usar
· Formato dos Arquivos
· Fluxo de Autenticação
· Estados de Retorno
· Estrutura do Código
· Aviso Legal

---

✨ Funcionalidades

· ✅ Verificação via Outlook Lite — simula o cliente oficial com.microsoft.outlooklite (Android)
· ✅ Suporte a múltiplas threads — processamento paralelo (configurado para 4 threads)
· ✅ Busca por palavra-chave nos e-mails da conta (via SearchService do Outlook)
· ✅ Extração de metadados — nome, país, assunto, remetente, preview do e-mail
· ✅ Salvamento sem duplicatas — verifica o live.txt e palavraachada.txt antes de gravar
· ✅ Relatório final com contadores e tempo de execução
· ✅ Detecção de bloqueios — contas banidas, com identidade pendente, recuperação ou abuso
· ✅ Verificação por LastDeliveryTime — confirma se o e-mail é recente (últimos 3 minutos)

---

🧰 Requisitos

· Python 3.8+
· Bibliotecas:
  · requests
  · urllib3 (já incluído com requests, mas usamos disable_warnings)

Instalação das dependências

```bash
pip install requests
```

As demais bibliotecas (uuid, re, json, os, threading, time, queue, datetime, urllib.parse) já fazem parte da biblioteca padrão do Python.

---

🚀 Instalação

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/outlook-checker.git
cd outlook-checker

# 2. (Opcional) Crie um ambiente virtual
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows

# 3. Instale as dependências
pip install requests

# 4. Execute
python outlook_checker.py
```

---

🎮 Como Usar

1. Crie um arquivo combo.txt com as contas no formato email:senha, uma por linha:

```txt
usuario1@outlook.com:senha123
usuario2@hotmail.com:senha456
```

2. Execute o script:

```bash
python outlook_checker.py
```

3. O script irá solicitar:

```
📁 Nome do arquivo com combos: combo.txt
🔍 Palavra-chave para buscar: nota fiscal
```

4. O progresso será exibido em tempo real:

```
[1/2] 🟢 usuario1@outlook.com - LIVE: Live account but no emails found
[2/2] ✅ usuario2@hotmail.com - KEYWORD_FOUND: Found 3 emails with keyword
```

5. Ao final, serão gerados dois arquivos:

Arquivo Conteúdo
live.txt Todas as contas válidas (live) — formato login:senha
palavraachada.txt Contas que contêm a palavra-chave — formato login:senha palavra: X vezes: N

---

📁 Formato dos Arquivos

Entrada — combo.txt

```txt
user1@outlook.com:senha123
user2@hotmail.com:senha456
```

Saída — live.txt

```txt
user1@outlook.com:senha123
user2@hotmail.com:senha456
```

Saída — palavraachada.txt

```txt
user2@hotmail.com:senha456 palavra: nota fiscal vezes: 3
```

Sem duplicatas: o script carrega os logins já existentes em ambos os arquivos na inicialização e ignora repetições — podendo ser reexecutado sem sobrescrever ou duplicar dados.

---

🔐 Fluxo de Autenticação

O script replica o fluxo oficial do Outlook Lite (Android) em 7 etapas:

Etapa Endpoint Finalidade
1 odc.officeapps.live.com/odc/emailhrd/getidp Descobre o provedor de identidade (valida se é MSAccount)
2 login.microsoftonline.com/consumers/oauth2/v2.0/authorize Inicia o fluxo OAuth2 do consumidor
3 login.live.com (POST) Envia login + passwd + PPFT para autenticar
4 login.microsoftonline.com/consumers/oauth2/v2.0/token Troca o code por access_token
5 substrate.office.com/profileb2/v2.0/me/V1Profile Captura nome e país do perfil
6 outlook.live.com/searchservice/api/v2/query Busca e-mails pela palavra-chave
7 — Consolida resultados e grava arquivos

Headers importantes

· X-OneAuth-AppId: com.microsoft.outlooklite
· X-Office-Application: 145
· X-Office-Version: 3.11.0-minApi24
· User-Agent: Outlook-Android/2.0
· X-AnchorMailbox: CID:{CID}

---

📊 Estados de Retorno

Status Significado Cor
LIVE Conta válida, mas nenhum e-mail com a palavra-chave 🟢
KEYWORD_FOUND Conta válida e encontrou e-mails com a palavra ✅
FAILURE Senha incorreta, conta inválida, erro de conexão, etc. ❌
BAN Muitas tentativas — conta temporariamente bloqueada ⛔

Motivos comuns de FAILURE

· Not a valid MSAccount
· Account or password is incorrect
· Identity confirm required
· Recovery required
· Account abuse/locked
· Could not get access token

---

🧩 Estrutura do Código

```
├── outlook_checker.py
│   ├── generate_guid()             # Gera UUID v4
│   ├── load_accounts_from_file()   # Carrega combos do .txt
│   ├── check_account()             # Núcleo da verificação
│   └── class AccountChecker        # Orquestração multithread
│       ├── load_existing_results() # Evita duplicatas
│       ├── save_result()           # Grava com lock
│       ├── worker()                # Loop de cada thread
│       ├── process_accounts()      # Gerencia a fila
│       └── generate_report()       # Relatório final
└── main()                          # Ponto de entrada
```

Observações técnicas

· session.verify = False — desabilita verificação TLS (o endpoint login.live.com costuma recusar certificados em ambientes restritos).
· Queue + threading.Lock — garante thread-safety ao escrever em arquivos.
· time_limit_ago = now - timedelta(minutes=3) — verifica e-mails recentes na busca por palavra-chave.
· existing_live / existing_keyword — conjuntos em memória com os logins já processados.

---

⚖️ Aviso Legal

Este projeto foi criado exclusivamente para fins de estudo sobre:

· Fluxos de autenticação OAuth2 da Microsoft
· Segurança de APIs móveis (Outlook Lite)
· Programação concorrente com threads em Python

Não é permitido:

· ❌ Acessar contas sem autorização expressa do titular
· ❌ Comercializar dados obtidos
· ❌ Utilizar em esquemas de fraude, spam ou credential stuffing
· ❌ Violar os Termos de Serviço da Microsoft

O autor não se responsabiliza por qualquer uso indevido. Ao clonar ou executar este código, você assume total responsabilidade legal pelas suas ações.

🤝 Contribuições

Contribuições são bem-vindas para melhorias de código, refatoração e documentação, desde que mantidos os fins educacionais. Abra uma issue ou envie um pull request.

---

Feito com ❤️ para estudo de automação HTTP e segurança de APIs.
