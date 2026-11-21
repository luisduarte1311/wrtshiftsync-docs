# Privacidade e tratamento de dados

> Documento técnico e aviso de privacidade do WrtShiftSync. Descreve o que o código faz hoje. Não é aconselhamento jurídico.

## 1. Quem é o responsável

O serviço é privado, mantido a título pessoal por Luís Duarte, fora do horário laboral. O responsável pelo tratamento é o administrador do serviço (o autor). Contacto para qualquer pedido: mensagem ao administrador do grupo de WhatsApp "Acessos".

## 2. Aviso de privacidade

**Que dados trata**
O PDF da escala que carregas (nomes, números de colaborador, secção, turnos, férias e ausências). Os dados servem apenas para gerar o ficheiro de calendário que pedes. O PDF é lido num ficheiro temporário em memória e apagado logo a seguir. Os dados extraídos ficam em memória durante 30 minutos e são depois eliminados. Não são guardados em base de dados nem em disco.

**Registos técnicos**
Os registos técnicos servem para segurança e prevenção de abuso. Neste momento o servidor não regista o endereço IP de quem o visita, porque a infraestrutura só vê o endereço interno do Docker. Os registos incluem data, caminho pedido e agente do browser e não têm um prazo fixo em dias. O código não escreve nomes nem números de colaborador nos registos; só o número de colaboradores lidos. Um erro inesperado ao ler o PDF pode, em teoria, deixar no registo um fragmento do texto que o provocou.

**Grupo de acessos (WhatsApp)**
Para gerir pedidos de acesso, guardamos o número de telefone de quem entra no grupo e o estado do pedido, durante 60 dias (pedido pendente) ou 180 dias (chave entregue). Quando alguém que não é administrador tenta um comando de administração, registamos apenas os 3 últimos dígitos do número.

**Base legal**
Interesse legítimo do responsável (segurança do serviço e gestão dos pedidos de acesso) e, quanto ao PDF, o interesse legítimo de quem o carrega em obter o seu próprio calendário. A escala contém dados de colegas; não os usamos para outro fim nem os guardamos.

**Partilha**
Não partilhamos dados com terceiros, além dos serviços técnicos necessários ao funcionamento (WhatsApp, DuckDNS, Let's Encrypt).

**Os teus direitos**
Podes pedir acesso, correção ou apagamento dos teus dados ao administrador do grupo "Acessos". Respondemos no prazo de um mês. Também podes reclamar à CNPD (www.cnpd.pt).

O PDF e os dados extraídos não permitem pesquisa por pessoa: expiram sozinhos.
