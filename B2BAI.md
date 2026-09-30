# B2bAI Sales

Pacote de implantação e auditoria inicial — 30/09/2026 (UTC).

## Estado real

Fork da base DeskcommCRM criado para o B2bAI. Código auditado: 50d14bd983bf35a878876451399e8832ff8e9ee0. Licença MIT e copyright de Rafael Melgaço preservados em LICENSE.

O CRM ainda não está implantado. Não há banco B2bAI conectado, WhatsApp pareado nem agente publicado. Nenhum serviço pago foi contratado e nenhuma chamada paga de IA foi executada.

## Decisão de implementação

Usar a marca própria nativa e o motor comercial existente. O núcleo opera CRM, agenda, IA, acompanhamento e passagem para atendimento humano. n8n entra para integrações externas quando necessário; não duplicar o motor de atendimento.

A marca é configuração em tempo de execução: platform_branding para a instalação e organizations.settings.branding para cada empresa. APP_NAME e APP_ACCENT_HEX são semente inicial e piso de rollback; depois o banco prevalece. Não trocar strings do núcleo nem reconstruir imagens só para mudar a marca.

Nome inicial: B2bAI Sales. Cor provisória: #6366f1. Não foi apresentada como identidade visual definitiva. Sem logotipo inventado; o produto pode usar o nome em texto.

## Configuração inicial

Partir do modelo completo .env.hostgator.example para uma instalação nova e preencher os dados reais em arquivo privado. Usar estas opções na primeira instalação:

~~~dotenv
APP_NAME="B2bAI Sales"
APP_ACCENT_HEX="#6366f1"
SIGNUP_MODE=so_convite
SENTRY_DSN=off
~~~

Esse trecho é apenas a personalização; não substitui o modelo completo. Credenciais de Supabase, domínio, administrador e IA ficam fora do GitHub. O instalador gera os segredos técnicos. Em instalação existente, configurar marca por /admin/marca e conferir a política de cadastro na tela: o banco prevalece sobre o arquivo.

## Auditoria do código atual

| Área | Evidência | Conclusão e limite |
| --- | --- | --- |
| Arquitetura | package.json; app/; workers/ | Next 16, React 19, TypeScript 6, Supabase; app, worker e scheduler separados. |
| Marca própria | lib/branding/; docs/white-label.md | Nome, cor e logos configuráveis; não exige fork visual do núcleo. |
| Autenticação | lib/auth/require-role.ts; lib/auth/rate-limit.ts | Guards por papel e limitadores de login/cadastro/tokens existem. Cobertura integral de todas as rotas não foi certificada. |
| Integrações | lib/api/auth-dual.ts; lib/mcp/auth.ts | Rotas habilitadas aceitam token dsk_; identidade da empresa vem do token. Não presumir que todas as rotas aceitam Bearer. |
| WhatsApp | lib/waha/webhook-auth.ts; Caddyfile | Assinatura incorreta é recusada. Sem assinatura, o padrão admite ingestão: WAHA Core pode não assinar. Webhook global bloqueado pelo proxy público; manter comunicação interna e proteger rota com token. |
| Acompanhamento | lib/followup/gatilho-etapa.ts | Gatilho por mudança de etapa implementado; documentação antiga que o chama pendente está desatualizada. |
| Banco | supabase/baseline.sql; tests/invariants/ | Instalação nova usa baseline e extensões requeridas. Não aplicar esse baseline em banco de outro projeto. RLS não foi executada nesta sessão. |
| Operação | docker-compose.prod.yml; hostgator-setup-kit/ | Exige processos persistentes, agendador e canal funcionando. Uma página publicada sozinha não entrega a operação completa. |
| Segredos | .env.example; lib/env.ts | INTERNAL_CRON_SECRET, IMPERSONATE_COOKIE_SECRET e LGPD_SIGNING_KEY já aparecem no modelo atual. |

A auditoria é inicial, por leitura e testes selecionados. Não é certificação de segurança ou de conformidade jurídica.

## Validação executada

Dependências instaladas com pnpm 9.15.9 e lockfile congelado. O pnpm 11 disponível no ambiente ignorava os overrides; a instalação foi refeita com a versão declarada pelo projeto, sem alterar o lockfile.

- TypeScript completo: exit 0; necessário aumentar o heap para 6 GB após o primeiro processo esgotar memória.
- ESLint completo: exit 0.
- 12 arquivos selecionados, 186 testes aprovados: resolução de marca, marca por organização, saídas, rate limit, webhook, tokens MCP, gatilho por etapa, publicação de acompanhamento, proteção anterior à ativação e autenticação de cron.
- Ambiente: Node 24.19.0. O CI upstream usa Node 22; resultados locais não certificam esse ambiente.
- Não executados: suíte unitária completa, build, Docker, invariantes reais de banco e E2E com Supabase/WhatsApp. Docker não está disponível neste ambiente.

## Agente comercial B2bAI — rascunho

Objetivo: receber interessados, entender o processo, registrar a oportunidade e encaminhar para diagnóstico comercial. Usar o texto abaixo no agente depois de conectar o canal e validar a credencial.

~~~text
Você atende interessados na B2bAI, empresa de implementação de inteligência artificial ponta a ponta para negócios.

A B2bAI trabalha com sistemas personalizados, geração e visualização de dados, centralização das informações, apoio à decisão e automação. As áreas incluem comercial, atendimento, financeiro, cobrança, operação, RH e jurídico.

Conduza uma conversa curta e natural, em português brasileiro. Faça uma pergunta por vez e aproveite o que a pessoa já contou.

Entenda a empresa e seu segmento, o processo que mais consome tempo ou perde oportunidades, como funciona hoje, quais sistemas usa, o volume aproximado e o resultado que deseja alcançar. Pergunte urgência e quem participa da decisão quando isso ajudar o diagnóstico. Não transforme a conversa em formulário longo.

Registre um resumo objetivo: empresa, segmento, problema, processo atual, sistemas, volume informado, resultado esperado, urgência e próximo passo. Diferencie informações fornecidas de pontos ainda desconhecidos.

Quando houver interesse em avançar, ofereça uma conversa de diagnóstico. Só confirme data e horário depois de verificar disponibilidade e receber confirmação de reserva. Sem agenda configurada, informe que a equipe combinará o horário.

Propostas, preços, descontos, prazos de implantação e garantias de resultado dependem de avaliação da equipe. Quando não houver informação validada, explique que o diagnóstico define o escopo e encaminhe a dúvida.

Se a pessoa pedir atendimento humano, houver reclamação, negociação contratual ou informação essencial ausente, encaminhe para a equipe com o contexto já coletado. Não prometa retorno em prazo que não esteja cadastrado.
~~~

## Funil proposto para o piloto

Novo → Contatado → Em diagnóstico → Qualificado → Proposta/negociação → Ganho ou Perdido.

Configurar exatamente uma etapa ganha e uma perdida. Mapear os sete passos do agente conforme a tela exigir. A proposta acima é configuração preparada, ainda não aplicada.

Acompanhamentos ficam em rascunho até definir a política comercial. Preparar dois casos: diagnóstico iniciado e interrompido; proposta sem resposta. Sem envio automático ou intervalos comerciais inventados.

## Aceite do piloto

1. Interessado pede automação comercial: atendimento pergunta pelo processo, sem inventar oferta.
2. Interessado fornece contexto: oportunidade e resumo aparecem no CRM certo.
3. Pergunta preço e prazo: resposta encaminha para diagnóstico, sem valor inventado.
4. Pede horário: reserva só é confirmada com recibo da agenda; sem agenda, encaminha para equipe.
5. Pede uma pessoa: atendimento humano recebe o resumo e o agente deixa de competir com ele.
6. Solicita parar mensagens: não recebe acompanhamento.
7. Reentrega de evento: não duplica contato, oportunidade ou envio.
8. Organização A não enxerga dados da B: provar com banco e sessões reais.
9. Repetir o ciclo por WhatsApp real e conferir CRM, agenda e logs; testes com mocks não encerram esse aceite.

## Próxima execução e dependências

Identificar acesso a um servidor existente capaz de rodar Docker e os processos persistentes. Conferir recursos e disponibilidade antes de instalar; se exigir nova contratação, aprovar custo antes.

Com acesso: criar ambiente B2bAI isolado, instalar pelo kit oficial, aplicar a marca, conectar banco próprio, concluir onboarding, preparar agente e base de conhecimento pelas telas. Conectar WhatsApp exige pareamento pelo dono do número. Ativar IA exige credencial válida e orçamento autorizado. Publicar atendimento somente após teste e confirmação do canal.

Não reutilizar o banco AI Creator Platform, staging ou produção de outro projeto. Não contratar WAHA Plus, hospedagem, APIs ou outros serviços sem aprovação de custo.

## Atualizações

Manter upstream melgarafael/DeskcommCRM e a camada de configuração B2bAI separada. Atualização do produto segue release publicada, backup e teste de atualização; não instalar main automaticamente em produção. A marca no banco sobrevive à troca de imagem. Customizações futuras de código exigem validar o caminho de distribuição para não serem sobrescritas pelo update.sh.
