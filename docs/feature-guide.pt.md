# Visão geral dos recursos do Launchpad

Este guia é uma visão geral concisa do kit inicial do Launchpad para potenciais compradores. Ele resume o que está incluído, o que requer serviços externos ou configuração e o que o kit não fornece automaticamente. O guia completo de recursos e o código-fonte do projeto adquirido permanecem como as referências autoritativas de implementação.

## O que o Launchpad inclui

O Launchpad é uma aplicação SaaS modular construída sobre uma única base de código compartilhada. Seus módulos opcionais abrangem faturamento, chat de IA, organizações, publicação, suporte, comunidade, notificações, marketing por e-mail, análise de dados, indicações, publicidade, SEO, integrações e armazenamento privado de arquivos. O assistente de configuração permite escolher quais módulos opcionais manter e um destino de implantação. Os componentes principais de conta, segurança, configurações e banco de dados compartilhado permanecem no lugar.

Em um checkout Git, a configuração usa sparse-checkout, de modo que os arquivos rastreados excluídos permanecem no histórico do Git. Em um ZIP adquirido, os arquivos de módulos excluídos são removidos do projeto extraído; eles não podem ser restaurados do histórico do Git porque o ZIP não contém metadados do repositório. Esta é uma configuração única, não uma ferramenta de alternância de módulos in-place. Faça backup do ZIP extraído antes de configurá-lo, caso queira recuperar o código-fonte excluído posteriormente.

O assistente grava os módulos selecionados e as preferências de implantação em
`launchpad.config.json`, e os segredos dos provedores no arquivo `.env` ignorado pelo Git.
Ele pode verificar as credenciais dos provedores selecionados quando você explicitamente escolhe **Testar
conexão**. Ele não cria contas de provedores, implanta infraestrutura
ou configura painéis externos. Você ainda precisa completar e manter
esses serviços por sua conta.

A área de plugins expõe um conjunto de componentes incluídos em uma lista de permissões; não é
um mercado de pacotes arbitrário e não baixa ou executa código de
plugins enviados. O serviço opcional de retransmissão de eventos pode ser implantado separadamente para
o evento atualmente suportado de atualização de perfil de conta. Não é um registro
de serviços geral, barramento de mensagens ou sistema de confirmação por consumidor.

## Contas e segurança

A autenticação suporta login por e-mail e login opcional com Google e GitHub.
Em desenvolvimento, o Mailpit pode receber e-mails de login quando nenhum provedor de e-mail está
configurado. Ambientes implantados precisam de um provedor de e-mail para login por e-mail
e e-mails transacionais. As páginas e APIs de administrador aplicam papéis no servidor.

A autenticação multifator opcional TOTP funciona com aplicativos
de autenticação compatíveis e inclui códigos de recuperação de uso único. As configurações de
segurança incluem gerenciamento de sessões e atividade de auditoria. O kit não inclui
login baseado em senha ou um fluxo de recuperação de senha.

A lista de verificação de integração de contas abrange configurações de perfil, segurança da conta
e o painel; é opcional e pode ser revisada.

## Faturamento e acesso pago

O módulo de faturamento suporta o Stripe Checkout para assinaturas configuradas e
compras únicas de Licença Vitalícia, além do Portal do Cliente Stripe. A verificação opcional de compras da
Apple App Store e do Google Play pode fornecer outra fonte de faturamento
quando os produtos necessários, credenciais, contas de loja e integração
de cliente nativo estiverem configurados. As rotas da loja estão indisponíveis até que suas
configurações sejam fornecidas; o Stripe pode continuar a operar independentemente.

O kit processa eventos de pagamento e assinatura verificados, registra o histórico de pagamentos
e lida com reembolsos e disputas. Ele **não** protege automaticamente
todos os recursos do seu produto apenas porque o faturamento está habilitado.
`paymentBlocked` é um sinal de faturamento, não um paywall universal. Cada recurso
pago deve aplicar o acesso no servidor; ocultar um botão no navegador
não é suficiente. Decida quais recursos são pagos e o que clientes restritos
ainda podem fazer, depois implemente e teste essas regras. O kit também não
inclui um fluxo de trabalho pronto de administrador para verificar pagamentos feitos
fora do Stripe.

O e-mail de faturamento usa o provedor de e-mail configurado, mas o faturamento não
requer e-mail para funcionar. A entrega de e-mails pode ser atrasada, falhar ou ser
duplicada após uma resposta de rede incerta; não é um sistema de
entrega exatamente uma vez.

## E-mail, lista de espera e marketing

O transporte de e-mail compartilhado suporta Resend, SendGrid e SMTP. É usado
para autenticação, avisos de faturamento, notificações, boletins e
mensagens de marketing. As identidades do remetente e as credenciais do provedor devem ser
configuradas. A verificação SMTP do assistente de configuração autentica sem enviar uma
mensagem.

A lista de espera de acesso antecipado localizada requer consentimento explícito. Seus status
são gerenciados manualmente: convidar ou marcar alguém como participante não cria
uma conta, não envia um convite e não concede acesso. O consentimento da lista de espera é separado
da inscrição no boletim.

O boletim suporta inscrições baseadas em consentimento, links de cancelamento de inscrição e
campanhas em texto simples criadas pelo administrador. A automação de marketing suporta
sequências curtas de e-mails para novos inscritos no boletim. Essas ferramentas não são um
editor visual de e-mails, um sistema de segmentação de audiência ou um serviço de
conformidade/legal. Elas não inscrevem automaticamente entradas da lista de espera nem enviam mensagens
por meio de notificações push e webhooks. Campanhas agendadas e automação
requerem um agendador externo, bem como um provedor de e-mail configurado.

## Assistente de IA e organizações

O assistente opcional do painel conecta-se a uma API de texto compatível com OpenAI configurada.
Ele armazena conversas no banco de dados da aplicação, aplica
limites de requisições da conta e suporta conversas pessoais e de organização.
Ele não inclui um modelo, não executa ferramentas e não toma ações autônomas.
Prompts e respostas são enviados ao provedor configurado e armazenados pela
aplicação; configure as configurações de privacidade e retenção do provedor e
divulgue esse processamento aos usuários.

As organizações fornecem espaços de trabalho compartilhados, papéis de membros e convites.
A autorização é verificada em relação à membresia da organização, mas isso é
tenancy de banco de dados compartilhado, não um banco de dados ou esquema separado para cada cliente.
Faturamento, configurações de conta, suporte, notificações, análise de dados, boletins
e conteúdo de publicação global não são automaticamente escopados à organização.

## Publicação, suporte e comunidade

Módulos separados fornecem um blog em Markdown localizado, um feed de atualizações de versão e
páginas de CMS. Os administradores gerenciam rascunhos e conteúdo publicado. As páginas
públicas incluem metadados localizados e suporte a sitemap. As ferramentas de publicação não
incluem publicação agendada, tradução automática, upload de imagens ou um
sistema de moderação de comentários.

O módulo de suporte fornece tickets de clientes e respostas de administradores. O
módulo de comunidade fornece um fórum de discussão. O diretório curado permite
que administradores gerenciem entradas públicas. Estes são recursos iniciais, não
integrações com uma mesa de ajuda hospedada ou uma rede social completa: anexos,
automação de atribuição/SLA, reações de fórum, respostas aceitas e perfis públicos de
usuários não estão incluídos.

## Notificações e dados do cliente

O painel tem uma caixa de entrada de notificações persistente e canais opcionais de entrega por e-mail, Web
Push e webhook assinado. As notificações push requerem chaves VAPID
e assinatura do navegador; os consumidores de webhook devem verificar assinaturas e
deduplicar eventos. Retentativas automáticas para notificações, marketing e
limpeza de armazenamento precisam de trabalhos agendados externamente.

As configurações da conta fornecem uma exportação JSON dos dados de conta suportados e um
link separado para o gerenciamento de faturamento. A exportação exclui credenciais,
sessões, segredos de assinatura e outros tokens sensíveis. É um download
em streaming, não um backup completo do banco de dados ou um serviço de arquivo em segundo plano.

## Análise de dados, localização e personalização

O painel inclui visões de conta e faturamento. A análise de dados de produto opcional
coleta eventos controlados por consentimento e fornece relatórios administrativos; eles não
são uma substituição para uma plataforma de análise de dados de terceiros. Atribuição,
rastreamento de marketing e cookies de indicação podem ter obrigações de consentimento
na sua implantação. O Launchpad não inclui uma plataforma de gerenciamento de consentimento.

A interface suporta 18 idiomas: inglês, árabe, bengali, alemão,
espanhol, francês, hindi, indonésio, italiano, japonês, coreano, holandês, polonês,
português, russo, turco, vietnamita e chinês. O inglês é o
catálogo de fallback e o idioma de fallback do assistente de configuração. Revise as traduções
e o texto legal para o seu público antes do lançamento.

O construtor de produtos edita um conjunto de seções da aplicação em lista de permissões,
navegação, texto localizado e configurações de marca. Ele não gera
código arbitrário, não cria rotas de aplicação e não altera a lógica de negócios. Tema,
movimento e outras preferências de interface pessoal são armazenados no navegador,
não sincronizados entre dispositivos.

## Armazenamento, indicações, publicidade e SEO

O armazenamento privado de arquivos opcional usa armazenamento de objetos compatível com S3. A
aplicação valida tipos e tamanhos de arquivos, usa URLs de upload/download de curta duração
e verifica a propriedade do arquivo. Configure um bucket privado e as credenciais
necessárias. O módulo não varre arquivos em busca de malware e a exclusão da conta
não remove automaticamente objetos remotos.

O módulo de indicações rastreia links de parceiros e comissões derivadas do Stripe.
Os pagamentos são realizados fora do Launchpad. Ele não calcula comissões
para compras da App Store ou do Google Play. A atribuição de indicação usa um
cookie de 30 dias; o Launchpad não tem um gerenciador de consentimento embutido, portanto avalie e
implemente os requisitos de consentimento aplicáveis.

As ferramentas de publicidade organizam informações de campanha e conta e importam
relatórios de CSV. Elas não se conectam a redes de anúncios para criar ou gerenciar campanhas ao vivo.
Os recursos de SEO fornecem metadados localizados, sitemaps, regras de robots,
dados estruturados e uma auditoria do banco de dados da aplicação; eles não são um
crawler da web, rastreador de classificação, ferramenta de pesquisa de palavras-chave ou gerenciador de redirecionamentos.

O catálogo de integrações relata se a configuração esperada está presente.
Ele não prova que as credenciais funcionam, não cria contas de provedores e não
garante a entrega.

## Implantação e responsabilidades operacionais

As implantações requerem PostgreSQL e variáveis de ambiente apropriadas,
incluindo conexões de banco de dados e `AUTH_SECRET`. Configure e-mail, Stripe,
faturamento de loja, armazenamento de objetos, IA, push e trabalhos agendados apenas para os
recursos que você habilita. As migrações de produção devem ser aplicadas usando o
processo de implantação documentado; mantenha ambientes de pré-visualização em um banco de dados isolado.

A aplicação inclui validação no lado do servidor, verificações de papel, controles de
segurança e registro de auditoria, mas não é um programa completo de operações de segurança.
Ela não fornece um WAF, varredura de malware, monitoramento de segurança externo
ou uma Política de Segurança de Conteúdo estrita. Configure um ingress confiável
e proteções de hospedagem, revise o modelo de ameaças e teste a aplicação
implantada antes do lançamento.

Para configuração detalhada, propriedade de dados, limites, migrações e
procedimentos operacionais, consulte o guia completo de recursos fornecido com o projeto.
