# Privacidade e tratamento de dados

> Documento técnico e aviso de privacidade do WrtShiftSync. Descreve o que o código faz hoje. Não é aconselhamento jurídico.

## 1. Quem é o responsável

O serviço é privado, mantido a título pessoal por Luís Duarte, fora do horário laboral. O responsável pelo tratamento é o administrador do serviço (o autor). Contacto para qualquer pedido: mensagem ao administrador do grupo de WhatsApp "Acessos".

## 2. Inventário

| Dado | Titulares | Onde fica | Quanto tempo | Base legal |
|---|---|---|---|---|
| PDF da escala (nomes, nº de colaborador, secção, turnos, férias e ausências) | O utilizador e os colegas que constam da escala | Ficheiro temporário em `/tmp` (memória, não disco) só durante a leitura; apagado a seguir, mesmo em caso de erro | Segundos | Interesse legítimo do utilizador que o carrega (calendário pessoal). Ver 4.1 |
| Dados extraídos do PDF | Idem | Memória do processo da web app, numa sessão com token aleatório | 30 minutos (`SESSION_TTL=1800`); perdem-se também ao reiniciar o contentor. **Nota:** enquanto `SESSION_MODE=compat` (A-07), existe ainda uma cópia sem prazo, indexada pela chave de acesso, que só desaparece ao reiniciar ou num novo upload com a mesma chave. Passa a 30 minutos com `strict` | Idem |
| Endereço IP do visitante | Qualquer visitante | **Não é registado neste ambiente.** Com o Docker Desktop, o proxy (NPM) e a web app só veem o endereço interno do Docker (`10.x.x.1`), não o do visitante. Esse endereço interno aparece no log do NPM e na memória do limite de pedidos. A web app deixou de o escrever nos seus logs. Se o serviço sair do Docker Desktop, o IP real passa a ser visto e este ponto tem de ser revisto | Memória: até reiniciar. Logs do NPM: sem prazo definido (crescem até serem apagados). Logs Docker da web app: rotação por tamanho (10 MB x 3) | Interesse legítimo (segurança e prevenção de abuso) |
| Registo de acessos do NPM (data, método, caminho, domínio, agente do browser e IP interno) | Qualquer visitante | Volume `npm/data/logs/` | Sem prazo definido. Não vimos rotação nos ficheiros de acesso | Interesse legítimo (segurança) |
| Números do grupo "Acessos" e estado do pedido | Quem entra no grupo; administradores | `data/group_members.json` | 60 dias (`pending`) e 180 dias (`key_sent`), purgados a cada evento autenticado | Interesse legítimo (gerir pedidos de acesso) |
| Pedido de acesso pelo site: nome, loja, telemóvel (com indicativo 351), estado e data | Quem preenche o cartão "Pedir acesso" | `data/group_members.json` (mesmo ficheiro, campos `nome`, `loja`, `origem: "site"`). O site só reencaminha o pedido ao bot por uma rede interna e não o guarda. Os administradores recebem também os dados numa mensagem de WhatsApp (enviada pela Evolution API) que fica no telemóvel de cada um, fora desta retenção e do `/apagar` | 60 dias (`pending`) e 180 dias (`key_sent`), como os números do grupo | Consentimento (caixa "Li e aceito" no cartão) |
| 3 últimos dígitos do remetente de comandos de admin tentados por não-admins | Qualquer pessoa que escreva ao bot | Log do webhook, nível ERROR, sem argumentos | Sem prazo em dias: rotação 10 MB x 3 | Interesse legítimo (segurança). Números completos não são registados |
| Números mascarados (3 últimos dígitos) nos restantes logs do webhook | Membros e admins | Log do webhook | Idem | Idem |
| Logs do contentor da Evolution API (eventos do WhatsApp) | Membros do grupo e quem escreve ao bot | Logs do Docker do contentor da Evolution (10 MB x 3) | Rotação por tamanho. **Não verificado:** podem conter números completos, porque a Evolution regista os eventos que recebe. Os logs dela também vêm com 3 horas de atraso (sem `TZ`) | Interesse legítimo (segurança) |
| Sessão WhatsApp do bot (credenciais da instância; sem mensagens, chats nem contactos) | O titular do número do bot | `evolution_instances/` e Postgres | Enquanto o bot existir | Necessário ao funcionamento do serviço |
| Hostname e IP públicos | O administrador | DuckDNS, Let's Encrypt | Enquanto o domínio existir | Necessário ao funcionamento do serviço |
| `data/group_members.json.corrupt-<data>` | Membros do grupo | Ficheiro posto de parte se o JSON ficar corrompido | Fica até ser apagado à mão (fora da purga) | Interesse legítimo |

O que **não** existe: base de dados de horários, histórico de uploads, cookies de rastreio, análise de utilização, partilha com terceiros além dos serviços técnicos acima (WhatsApp, DuckDNS, Let's Encrypt).

## 3. Aviso de privacidade (texto para `/privacidade`)

**Quem é o responsável**
O serviço é privado, mantido a título pessoal por Luís Duarte, fora do horário laboral. O responsável pelo tratamento é o administrador do serviço (o autor). Contacto para qualquer pedido: mensagem ao administrador do grupo de WhatsApp "Acessos".

**Que dados trata**
O PDF da escala que carregas (nomes, números de colaborador, secção, turnos, férias e ausências). Os dados servem apenas para gerar o ficheiro de calendário que pedes. O PDF é lido num ficheiro temporário em memória e apagado logo a seguir. Os dados extraídos ficam em memória durante 30 minutos e são depois eliminados (serviço em fase de testes: este prazo pode ser alterado). Enquanto o serviço estiver em modo de compatibilidade, existe também uma cópia temporária em memória, sem prazo, que desaparece quando o serviço reinicia ou quando carregas outro PDF com a mesma chave. Não são guardados em base de dados nem em disco. A escala contém também dados dos teus colegas. Ao carregá-la, confirmas que a usas só para obteres o teu calendário e que tens o direito de a usar. O teu consentimento não substitui o deles: tratamos esses dados com base no interesse legítimo e só para gerar o teu calendário.

**Registos técnicos**
Os registos técnicos servem para segurança e prevenção de abuso. Neste momento o servidor não regista o endereço IP de quem o visita, porque a infraestrutura só vê o endereço interno do Docker. Os registos incluem data, caminho pedido e agente do browser e não têm um prazo fixo em dias. O código não escreve nomes nem números de colaborador nos registos; só o número de colaboradores lidos. Um erro inesperado ao ler o PDF pode, em teoria, deixar no registo um fragmento do texto que o provocou.

**Grupo de acessos (WhatsApp)**
Para gerir pedidos de acesso, guardamos o número de telefone de quem entra no grupo e o estado do pedido, durante 60 dias (pedido pendente) ou 180 dias (chave entregue). Quando alguém que não é administrador tenta um comando de administração, registamos apenas os 3 últimos dígitos do número.

**Pedido de acesso pelo site**
Quando preenches o cartão "Pedir acesso", guardamos o teu nome, a loja e o número de telemóvel, o estado do pedido e a data, durante 60 dias (pedido pendente) ou 180 dias (chave entregue). Os administradores do grupo "Acessos" recebem estes dados por mensagem de WhatsApp para decidirem o pedido. A mensagem fica no WhatsApp de cada administrador até ele a apagar. O pedido só é enviado depois de marcares a caixa de aceitação. Se os dados estiverem errados, pede o apagamento e faz um novo pedido: um segundo pedido com o mesmo número não atualiza o primeiro. Pedimos o mínimo: nome, loja e telemóvel.

**Base legal**
Gerar o calendário a partir do PDF: interesse legítimo de quem o carrega em obter o seu próprio calendário. A escala contém dados de colegas; não os usamos para outro fim nem os guardamos. Segurança do serviço e gestão do grupo de acessos: interesse legítimo do responsável. Pedido de acesso (nome, loja e telemóvel): o teu consentimento, dado ao marcar a caixa; podes retirá-lo pedindo o apagamento ao administrador do grupo "Acessos". A caixa "Li e aceito" confirma que leste esta política.

**Partilha**
Não partilhamos dados com terceiros, além dos serviços técnicos necessários ao funcionamento (WhatsApp, DuckDNS, Let's Encrypt).

**Os teus direitos**
Podes pedir acesso, correção ou apagamento dos teus dados ao administrador do grupo "Acessos". Respondemos no prazo de um mês. Também podes reclamar à CNPD (www.cnpd.pt).

## 4. Pontos em aberto (não afirmar mais do que isto)

1. **Dados de colegas no PDF (D-01, pendente).** A escala traz dados de terceiros que não escolheram usar o serviço, e a lista mostra todos os colaboradores (D-02). A confirmação por escrito da política da empresa sobre usar a escala fora dos sistemas internos está por obter e reforçaria a base legal.
2. **Prazo dos logs.** Os logs do Docker rodam por tamanho (10 MB x 3). Os do NPM não têm prazo nem rotação visível. Se for preciso um prazo em dias, é preciso configurá-lo e atualizar este documento. Confirmado em 2026-11-18 com um pedido local (`curl`) e com um pedido de um telemóvel Android em dados móveis: o NPM registou `10.x.x.1` nos dois.
3. **A-07 em `compat`.** O prazo de 30 minutos só é totalmente verdadeiro depois de passar a `strict`.
4. **Logs da Evolution.** Não foram inspecionados; podem conter números completos de telefone. Falta ver o que registam e, se for preciso, reduzir o nível de log deles. Desde 2026-11-22 também enviam os avisos com nome, loja e telemóvel dos pedidos feitos pelo site.
5. **Erros do parser.** O código só escreve mensagens fixas e contagens, mas as exceções inesperadas (traceback) podem incluir texto do PDF. Não foi testado.
6. **IP real.** O Docker Desktop esconde-o à web app e ao NPM; o limite de pedidos é um balde único.
7. **Prova de aceitação (futuro).** A caixa "Li e aceito" (página inicial e cartão "Pedir acesso") só é obrigatória no browser: o servidor não sabe se foi marcada e não guarda prova. Se vier a ser preciso prova, exige um campo novo no `POST /` (contrato) e guardar essa informação; avaliar então o impacto na privacidade. Desde 2026-11-22 o servidor recusa pedidos do cartão sem `consentimento: true`, mas esse valor vem do browser: não é prova (qualquer cliente o pode enviar) e não é guardado.
8. **Ausências no PDF.** Na loja do utilizador a ausência aparece só como "A", sem indicar a causa (confirmado em 2026-11-21). O layout das outras lojas é desconhecido: se algum passar a indicar baixa médica, seria dado de saúde (RGPD, art. 9.º) e o interesse legítimo não bastaria. Rever se o serviço passar a ser usado noutras lojas.

## 5. Direitos e incidentes

**Acesso, correção, apagamento.** Pedido ao administrador do grupo "Acessos"; resposta no prazo de um mês. Para números do grupo, o administrador apaga o registo com o comando `/apagar <número com indicativo, só dígitos>` (responde "Feito" ou "Não encontrado"). Para pedidos feitos pelo site, o mesmo comando remove também o nome e a loja (é o mesmo registo); a mensagem de aviso que ficou no WhatsApp dos administradores tem de ser apagada à mão por cada um. O PDF e os dados extraídos não permitem pesquisa por pessoa: expiram sozinhos (ver 2).

**Fuga de dados.**
1. Conter: parar o contentor afetado (`docker compose stop <serviço>`), rodar a chave de acesso e o token da Evolution se houver suspeita de comprometimento.
2. Avaliar: que dados, de quem, durante quanto tempo, se há risco para os titulares.
3. Se houver risco, notificar a CNPD em 72 horas após ter conhecimento; se o risco for elevado, avisar também os titulares.
4. Registar o incidente e as medidas tomadas.
