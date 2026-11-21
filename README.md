# WrtShiftSync 📱✨

Bem-vindo ao portal privado de sincronização de horários! Esta ferramenta foi concebida para simplificar a organização pessoal: pega no ficheiro PDF da escala de trabalho e transforma-o automaticamente num formato de calendário padrão (`.ics`), pronto a integrar em qualquer dispositivo (telemóvel, tablet ou computador) de forma rápida, privada e sem atritos.

---

## 🔒 Parte 1: Para quem usa (Simplicidade, Privacidade & Multiplataforma)

A privacidade e a soberania sobre os teus dados são prioridades absolutas. Esta aplicação foi desenhada sob uma filosofia de **confiança zero para o exterior e controlo total para o utilizador**:

- **Onde param os meus dados?** O PDF carregado é processado em tempo real e **eliminado de imediato** assim que a leitura termina: toca no disco só durante a leitura, num ficheiro temporário em memória. Os dados extraídos ficam apenas em memória, durante 30 minutos por sessão, e perdem-se também quando o serviço reinicia. Os horários não são guardados em base de dados, não existe histórico e nada é partilhado com terceiros. O código não escreve nomes nem números de colaborador nos registos.
- **Transparência:** o tratamento de dados limita-se ao descrito nesta secção: o PDF é eliminado após a leitura, os dados extraídos existem apenas em memória e os horários não são guardados.
- **Quem tem acesso?** O acesso ao pipeline é restrito e protegido por autenticação baseada em chave alfanumérica, com limitação do número de pedidos por minuto, sessões individuais com token e validade de 30 minutos, e limite de tamanho no upload.
- **Compatibilidade Multiplataforma Inteligente:** Não é uma ferramenta só para iPhone. A interface adapta as instruções ao dispositivo:
  - **iPhone / iPad (Safari):** orienta o utilizador a adicionar a ferramenta ao ecrã principal como Web App (PWA) de acesso instantâneo.
  - **iPhone / iPad (outro browser):** apresenta as instruções adequadas a esse caso.
  - **Android:** suporta navegação web universal e a importação do `.ics` no calendário do dispositivo.
  - **Computador:** funciona em qualquer browser moderno, com o `.ics` a abrir na aplicação de calendário instalada.
- **Como funciona na prática?**
  1. Acedes ao link seguro da plataforma.
  2. Introduzes a chave de acesso autorizada.
  3. Selecionas o PDF da escala mensal (*os dados são processados de forma volátil e segura*).
  4. Escolhes o teu perfil e descarregas o ficheiro `.ics` para sincronização imediata com o teu calendário (Apple Calendário, Google Calendar, Outlook ou qualquer aplicação que aceite o formato iCalendar).

---

## ⚙️ Parte 2: Arquitetura Técnica & Stack Open-Source (Power Users & Admin)

Esta secção documenta os pilares tecnológicos do backend, a infraestrutura de self-hosting e o ecossistema de dependências.

### 1. Stack Tecnológica de Código Aberto
As bibliotecas Python do motor do pipeline usam licenças permissivas (MIT, Apache 2.0 e BSD), garantindo liberdade de execução técnica sem contaminação de licenças copyleft:
- **Backend & API:** **FastAPI** (MIT) e **Uvicorn** (BSD 3-Clause) para alta performance assíncrona, com **python-multipart** (Apache 2.0) para o upload e **slowapi** (MIT) para a limitação de pedidos.
- **Motor de Extração de PDF:** **pdfplumber** (MIT) para parsing estruturado de tabelas de horário, seguido de uma validação final de integridade dos dados extraídos.
- **Geração de Calendário:** **ics** (Apache 2.0) para criação de eventos conformes com a norma RFC 5545 (com UIDs determinísticos anti-duplicação).
- **Templates & UI:** **Jinja2** (BSD 3-Clause) e **Tailwind CSS** (MIT) para uma interface responsiva nativa.
- **Serviço auxiliar de webhooks:** **Flask** (BSD 3-Clause) com **Flask-Limiter** (MIT), integrado com a **Evolution API** e **PostgreSQL**, para a gestão de pedidos de acesso através do WhatsApp.

### 2. Infraestrutura & Serviços (*Self-Hosted*)
O projeto corre como uma stack **Docker Compose** com seis serviços numa rede interna dedicada:
- **Web app (FastAPI/Uvicorn):** o motor de extração e geração do `.ics`.
- **Proxy Reverso & Encriptação:** **Nginx Proxy Manager (NPM)** para terminação de SSL/TLS (certificados Let's Encrypt). Só as portas 80 e 443 estão expostas.
- **DNS Dinâmico:** gestão de endereçamento dinâmico resiliente.
- **Evolution API + PostgreSQL:** integração com o WhatsApp; a base de dados serve exclusivamente esta integração, guarda só a instância do WhatsApp (mensagens, chats e contactos não são guardados) e **não guarda horários**.
- **Serviço de webhooks (Flask):** recebe eventos do grupo de acessos, regista o estado de cada pedido e notifica os administradores.
- **Ambiente de Execução:** contentorização isolada via Docker.

### 3. Segurança & Configuração
- **Segredos fora do código:** todas as chaves, tokens e passwords são lidos de um ficheiro `.env` que nunca é versionado, e o código não tem valores por defeito para segredos.
- **Controlo de acesso:** autenticação por chave alfanumérica e limitação de pedidos por minuto, mais restritiva nos pontos de upload e validação da chave.
- **Dados voláteis:** o PDF é guardado num ficheiro temporário (`/tmp` em memória) eliminado logo após a leitura, e os dados extraídos existem apenas em memória, em sessões com token e validade de 30 minutos.
- **Parser isolado:** a leitura do PDF corre num subprocesso com limites de tempo, CPU e memória, e o upload tem tamanho máximo (3 MB por defeito).
- **Webhook:** pedidos autenticados, comandos de admin só para administradores do grupo, gravação atómica do registo de membros e retenção de 60 e 180 dias.
- **Dados de infraestrutura fora do Git:** sessões do WhatsApp, base de dados e volumes do proxy não são versionados.
- **Isolamento de Processos:** contentores sem root, com sistema de ficheiros só de leitura, sem capacidades Linux e com limites de memória e processos; redes Docker bridge separadas por função.

### 4. Estrutura do Repositório
```
main.py              Web app (FastAPI): upload do PDF, validação da chave, descarga do .ics
parser.py            ScheduleParser: extração e validação dos turnos a partir do PDF
generator.py         CalendarGenerator: geração do ficheiro .ics
templates/           Interface web (index.html)
app.py               Serviço Flask de webhooks (integração WhatsApp / Evolution API)
Dockerfile           Imagem Python usada pela web app e pelo serviço de webhooks
docker-compose.yml   Definição da stack completa
requirements-*.in/.txt Dependências Python (web e webhook), com hashes
```

### 5. Execução (administradores)
1. Criar um ficheiro `.env` na raiz do projeto com os segredos necessários (nunca o enviar para o Git).
2. Arrancar a stack: `docker compose up -d --build`

---

## ⚖️ Parte 3: Aviso Legal & Propriedade Intelectual (Legal Disclaimer)

© **2026 Luís Duarte. Todos os direitos reservados.**

- **Natureza Independente:** O **WrtShiftSync** é uma ferramenta utilitária independente, concebida e desenvolvida a título pessoal e em âmbito privado, fora do horário de trabalho, sem utilização de meios, informação ou instruções de qualquer entidade empregadora ou empresarial.
- **Isenção de Afiliação:** Este projeto não é endossado, patrocinado nem oficialmente afiliado a qualquer entidade, empresa ou marca, e não utiliza o nome de nenhuma.
- **Proteção de Propriedade Intelectual:** Todos os direitos relativos à conceção do pipeline, arquitetura de parsing, código proprietário de suporte e identidade visual pertencem exclusivamente ao autor. A engenharia reversa, redistribuição não autorizada, descompilação ou integração corporativa sem consentimento prévio e expresso são estritamente proibidas nos termos da legislação aplicável.
