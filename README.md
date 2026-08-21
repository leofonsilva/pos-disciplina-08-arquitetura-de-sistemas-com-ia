# Pós Disciplina 08 - Arquitetura de Sistemas com IA

## Introdução
Pendente...

## Módulos

### Módulo 01: Fundamentos de Arquitetura AI-First

#### **Projeto:** [Vitalis Pharma - Trial Forge](module-01)

**Tecnologias utilizadas:**
- **LLM (Large Language Model)** - Motor com raciocínio estruturado
- **AI Architecture Canvas** - Artefato para organização de decisões arquiteturais
- **Framework de Três Perguntas** - Critério objetivo para decidir quando utilizar IA

**Conceitos abordados:**
- **Arquitetura AI-First vs. Tradicional:** Sistemas tradicionais são determinísticos e previsíveis; sistemas AI-First incorporam um componente não determinístico (modelo generativo) capaz de produzir respostas diferentes para entradas semelhantes, exigindo novas abordagens de arquitetura.
- **Cinco Pilares da Arquitetura AI-First:**
  - **Não Determinismo por Design:** Testes não podem validar igualdade absoluta; devem considerar faixas aceitáveis e níveis mínimos de confiança.
  - **Approval Gates (Human in the Loop):** Decisões críticas exigem aprovação humana antes da execução.
  - **Observabilidade e Trilha de Auditoria:** Cada decisão, raciocínio e ação do agente precisa ser registrado para reconstrução posterior.
  - **Custo e Latência como Restrições de Primeira Classe:** Impacto financeiro e tempo de resposta devem ser considerados desde o início do projeto.
  - **Degradação Graciosa:** O sistema deve possuir mecanismos de fallback quando o modelo apresenta baixa confiança ou indisponibilidade.
- **Framework de Decisão (Três Perguntas):**
  1. Existe uma regra finita capaz de resolver mais de 90% dos casos reais? → Se sim, use lógica determinística.
  2. O erro é caro e irreversível? → Se sim, o agente produz recomendações, mas exige Approval Gate.
  3. O comportamento muda conforme o contexto? → Se sim, utilize agente inteligente; se não, mantenha regras determinísticas.
- **Decomposição de Tarefas Híbridas:** Processos complexos devem ser divididos em subtarefas menores, cada uma avaliada individualmente pelo framework.
- **Trade-offs Arquiteturais:** Quatro eixos competem entre si - latência, custo, precisão e performance/throughput. Não é possível maximizar todos simultaneamente; cada componente possui seu próprio orçamento.

**Aplicação prática:**
No contexto da Vitalis Pharma, o Trial Forge é uma plataforma baseada em agentes de IA para modernizar a produção de documentos de estudos clínicos (protocolos, Termos de Consentimento, Clinical Study Reports). O desafio não é escolher o modelo de linguagem, mas construir uma arquitetura capaz de colocá-lo em produção com responsabilidade real sobre os resultados. O diagrama de referência é composto por Gateway (autenticação, roteamento), Orquestrador (coordenador determinístico), Modelo + Tools + RAG (núcleo não determinístico), Approval Gate (controle de decisões críticas) e uma camada transversal de Observabilidade. O framework das três perguntas é aplicado para decidir, por exemplo, que a validação de formulários obrigatórios deve ser determinística (regra finita), enquanto a geração do Termo de Consentimento exige agente com Approval Gate (erro caro e irreversível).

### Módulo 02: Arquiteturas Single-Agent

#### **Projeto:** [Vitalis Pharma - Trial Forge](module-02)

**Tecnologias utilizadas:**
- **LLM (Large Language Model)** - Motor com raciocínio estruturado
- **ReAct (Reasoning + Acting)** - Padrão de ciclo entre raciocínio e ação
- **Reflection** - Padrão de autocrítica e revisão da própria resposta
- **Tool Calling** - Mecanismo de chamada de ferramentas externas com contratos tipados
- **MCP (Model Context Protocol)** - Protocolo padronizado para comunicação com ferramentas

**Conceitos abordados:**
- **Anatomia do Agente:** Quatro componentes fundamentais - Memória, Planejamento, Ferramentas e Ação. Um agente bem projetado não é aquele que possui todos no nível máximo, mas aquele que utiliza exatamente o necessário para sua tarefa.
- **Memória de Curto e Longo Prazo:** Curto prazo (contexto da requisição atual, como RAM) e longo prazo (persistente entre sessões, como armazenamento em disco). Memória persistente exige mecanismos de recuperação seletiva (busca vetorial) e cuidados com compliance (LGPD, retenção de dados).
- **Planejamento:** Decomposição de tarefas complexas em etapas menores, incluindo raciocínio estruturado, decomposição em subobjetivos, autocrítica e reflexão. Quanto mais o agente analisa antes de agir, maior a qualidade, mas também maior o custo e latência.
- **Ferramentas:** Funções ou serviços externos que o agente pode acionar (calculadoras, buscas, APIs, bancos de dados). O modelo não executa diretamente a ferramenta - ele propõe a chamada, e a aplicação mantém controle sobre a execução.
- **Ação:** Passo executado no mundo real após interpretação do contexto. Em situações de alto risco, a ação mais importante pode ser interromper o processo e solicitar aprovação humana.
- **Padrão ReAct:** Ciclo contínuo de quatro etapas - Pensamento (analisa o que sabe e o que falta), Ação (utiliza ferramenta), Observação (recebe resultado), Decisão (avalia se já possui contexto suficiente). O número de iterações nunca é conhecido antecipadamente.
- **Padrão Reflection:** Após produzir uma resposta, o agente a revisa criticamente procurando inconsistências, erros ou omissões. Não busca novas informações - trabalha exclusivamente sobre o que já produziu. Execução e reflexão devem ser chamadas independentes (prompts separados).
- **Mecanismos de Contenção:** Limite máximo de iterações, orçamento de custo/tempo, critérios de confiança. Sem esses limites, um agente pode permanecer indefinidamente tentando resolver um problema.
- **Tool Calling:** Quatro etapas - Definição (esquema com nome, finalidade, parâmetros tipados), Decisão (modelo escolhe ferramenta e preenche parâmetros), Execução (aplicação valida e executa), Retorno (resultado estruturado). Esquemas devem ser explícitos e com tipagem rigorosa para evitar ambiguidades.
- **Ferramentas de Leitura vs. Escrita:** Leitura consulta informações sem alterar estado (busca, cálculo). Escrita altera estado externo (criar registros, notificar sistemas). Ferramentas de escrita com alto impacto exigem Approval Gate antes da execução.
- **MCP (Model Context Protocol):** Protocolo aberto que padroniza a descrição de ferramentas, fornecimento de contexto e retorno estruturado, permitindo interoperabilidade entre diferentes agentes e ferramentas.

**Aplicação prática:**
No Trial Forge, o agente responsável pela geração da seção de assentimento do Termo de Consentimento é calibrado com memória exclusivamente temporária (cada protocolo processado independentemente), ciclo ReAct com limite de 4 iterações, única ferramenta (busca regulatória na base da Anvisa/FDA), e ação restrita à geração de rascunho (nunca publica autonomamente). O agente não utiliza Reflection (validação humana via Approval Gate é considerada suficiente) nem memória de longo prazo (não há benefício em armazenar protocolos anteriores). A ferramenta de busca recebe parâmetros tipados (tema em texto livre, jurisdição restrita a "Anvisa" ou "FDA") e retorna o texto da cláusula com referência exata (resolução e artigo) para rastreabilidade. O ciclo ReAct permite que o agente identifique se o estudo envolve menores de idade, consulte a cláusula de assentimento, e somente então produza a seção correspondente - adaptando seu comportamento conforme o contexto descoberto durante a execução.

**Comandos executados:**
```bash
cd module-02
node agent-components-demo.js
node react-agent-prototype.js
```

### Módulo 03: Arquiteturas Multi-Agent

#### **Projeto:** [Vitalis Pharma - Trial Forge](module-03)

**Tecnologias utilizadas:**
- **LLM (Large Language Model)** - Motor com raciocínio estruturado
- **A2A (Agent-to-Agent Protocol)** - Protocolo para comunicação entre agentes autônomos
- **Teorema CAP** - Princípio de sistemas distribuídos para decisão entre consistência e disponibilidade
- **Padrão Saga** - Mecanismo de ações compensatórias para falhas tardias

**Conceitos abordados:**
- **Limite do Agente Único:** Um agente generalista que acumula múltiplos domínios (ICF, protocolo, CSR) torna-se difícil de auditar, evoluir e depurar. A divisão em especialistas é necessária quando tarefas exigem vocabulários, ferramentas ou níveis de risco distintos.
- **Agente vs. Ferramenta:** Ferramenta executa função determinística com esquema fixo. Agente interpreta contexto, toma decisões, utiliza ferramentas e pode reconhecer quando uma tarefa está fora de seu domínio. Comunicação entre agentes exige protocolos específicos (A2A) porque ambos os lados possuem autonomia.
- **Seis Padrões de Orquestração:**
  - **Sequential:** Agentes executam em sequência, cada um dependendo da saída do anterior. Adequado quando existe dependência lógica entre atividades.
  - **Parallel:** Agentes executam simultaneamente. Reduz latência quando atividades são independentes. Exige consolidação posterior dos resultados.
  - **Supervisor:** Agente central coordena especialistas, distribui tarefas e consolida resultados. Não substitui os especialistas - apenas coordena.
  - **Hierarchical:** Evolução do Supervisor com múltiplos níveis de coordenação (árvore). Orquestrador principal encaminha para supervisores de domínio, que gerenciam seus próprios especialistas. Isola responsabilidades e facilita expansão.
  - **Group Chat:** Agentes compartilham mesma conversa, debatem e chegam a decisões por consenso. Moderador apenas organiza a ordem das interações. Útil quando diferentes perspectivas precisam ser reconciliadas.
  - **Handoff:** Agente transfere completamente a responsabilidade para outro especialista quando identifica que a tarefa ultrapassa seu domínio. O novo agente continua do ponto onde o processo foi interrompido.
- **Teorema CAP em Agentes:** Diante de falha de comunicação, o sistema precisa escolher entre consistência (aguardar até ter certeza da informação correta) e disponibilidade (continuar respondendo com informações parciais). Diferentes componentes podem fazer escolhas distintas conforme o risco associado.
- **Mecanismos de Tolerância a Falhas:** Timeout explícito (nenhuma espera indefinida), Retry com limite máximo (tentativas controladas), Idempotência (repetição não produz efeitos duplicados).
- **Padrão Saga:** Quando uma falha tardia invalida parte do trabalho, não é necessário reiniciar todo o processo. Executam-se ações compensatórias apenas nas etapas afetadas, na ordem inversa da execução original. Cada etapa possui sua própria estratégia de desfazer.
- **Controle Otimista de Versão:** Cada agente informa qual versão do documento utilizou. Ao tentar gravar, o sistema verifica se aquela versão continua sendo a mais recente. Caso outro agente já tenha atualizado, a gravação é rejeitada e o conflito deve ser resolvido explicitamente.

**Aplicação prática:**
No Trial Forge, três agentes especialistas são criados: Agente ICF (linguagem acessível ao participante, seções condicionais), Agente de Protocolo (metodologia técnica, critérios de inclusão/exclusão) e Agente CSR (síntese estatística, conformidade ICH E3). O Supervisor coordena a execução: primeiro o protocolo é gerado (Sequential), depois ICF e CSR trabalham em paralelo (Parallel), e ao final o supervisor consolida os resultados. O padrão Hierarchical é aplicado com um orquestrador principal que encaminha solicitações para supervisores de domínio (protocolos, consentimento, relatórios). O Handoff é utilizado quando o agente ICF identifica um tema de bioética complexo e transfere a responsabilidade para um agente especializado em bioética. O Teorema CAP é aplicado: se o agente CSR deixar de responder, o supervisor pode optar por consistência (aguardar) ou disponibilidade (prosseguir com lacuna registrada). O padrão Saga é utilizado quando uma inconsistência regulatória é descoberta tardiamente - apenas os agentes afetados executam compensações, preservando o trabalho válido já realizado.

**Comandos executados:**
```bash
cd module-03
node trialforge-message-queue-prototype.js
```

### Módulo 04: Padrões de Design AI-Específicos

#### **Projeto:** [Vitalis Pharma - Trial Forge](module-04)

**Tecnologias utilizadas:**
- **LLM (Large Language Model)** - Motor com raciocínio estruturado
- **RAG (Retrieval Augmented Generation)** - Recuperação de contexto antes da geração
- **Embeddings** - Representação vetorial para busca semântica
- **BM25** - Algoritmo de busca lexical para correspondência exata de termos
- **Semantic Cache** - Cache baseado em similaridade semântica de perguntas
- **Prompt Cache** - Reutilização de contexto processado anteriormente
- **Response Streaming** - Exibição gradual de respostas para reduzir percepção de espera

**Conceitos abordados:**
- **Padrões de RAG:**
  - **Basic RAG:** Quatro etapas - fragmentação (chunking) dos documentos, geração de embeddings, armazenamento vetorial, recuperação dos fragmentos semanticamente mais próximos e envio ao modelo. Separação entre recuperação e geração.
  - **Hybrid Search:** Combinação de busca vetorial (similaridade semântica) com busca lexical (BM25 para correspondência exata de palavras, códigos, números de resolução). Amplia precisão e cobertura.
  - **Multi-Index:** Múltiplos índices documentais organizados por domínio (ex: um índice para Anvisa, outro para FDA, outro para referências científicas). Cada consulta identifica qual índice é mais relevante.
  - **Agentic RAG:** Quando a qualidade da recuperação fica abaixo do esperado, o agente executa automaticamente novas consultas com abordagens diferentes, ampliando a estratégia de busca até encontrar contexto suficiente (limite de tentativas).
- **Roteamento:**
  - **Intent-Based Routing:** Identifica a intenção da requisição antes de qualquer inferência (gerar protocolo, ICF, CSR, consulta regulatória). Decide para qual fluxo a solicitação deve seguir. Acontece antes do Model Router.
  - **Model Router:** Decide qual modelo utilizar para cada tarefa, equilibrando custo, latência e qualidade. Tarefas simples vão para modelos menores e mais baratos; tarefas complexas para modelos mais sofisticados. Decisão baseada em classificação prévia da complexidade.
- **Otimização:**
  - **Semantic Cache:** Perguntas semanticamente equivalentes reutilizam respostas armazenadas. A pergunta é transformada em embedding e comparada com perguntas anteriores. Similaridade acima do limiar → resposta do cache (nenhuma chamada ao modelo). Reduz custo e latência.
  - **Prompt Cache:** Reutiliza contexto processado anteriormente (documentos longos, System Prompts, definições de ferramentas). A chamada ao modelo continua, mas o reprocessamento do contexto repetido é eliminado. Complementar ao Semantic Cache.
  - **Response Streaming:** Não reduz tempo real de geração, mas reduz percepção de espera ao exibir os primeiros resultados gradualmente. Útil para respostas longas (documentos, relatórios).
- **Approval Gate e Controle de Confiança:**
  - **Approval Gate:** Componente que interrompe o fluxo para validação humana. Pode ser síncrono (aguarda aprovação antes de prosseguir, para ações irreversíveis) ou assíncrono (prossegue e revisa depois, para ações reversíveis).
  - **Confidence Threshold:** Limiar que converte confiança probabilística em decisão arquitetural. Acima do limiar → prossegue automaticamente. Abaixo do limiar → encaminha para validação humana. Deve ser recalibrado continuamente.
  - **Audit Trail:** Registro permanente de todas as decisões: agente/pessoa responsável, data/hora, versão do prompt, versão do modelo, limiar de confiança, resultado final. Registros imutáveis (correções criam novos eventos, não apagam os anteriores).

**Aplicação prática:**
No Trial Forge, o fluxo integrado de padrões é executado sequencialmente: (1) Intent-Based Routing identifica se a solicitação é uma consulta rotineira ou geração de CSR; (2) Semantic Cache verifica se a pergunta já foi respondida anteriormente (se sim, resposta imediata); (3) Model Router seleciona modelo pequeno para consultas simples ou modelo avançado para CSR; (4) RAG recupera contexto com Multi-Index (Anvisa/FDA), Hybrid Search (vetorial + BM25), e Agentic RAG com até 3 tentativas se a recuperação for insuficiente; (5) Response Streaming exibe a resposta gradualmente; (6) Confidence Threshold avalia a confiança da resposta; (7) Approval Gate (síncrono) é acionado obrigatoriamente para o CSR, independentemente da confiança, devido ao alto risco regulatório; (8) Audit Trail registra permanentemente toda a execução. O Semantic Cache é utilizado para perguntas rotineiras sobre protocolos, mas nunca para CSR (risco de desatualização). O Prompt Cache é utilizado para reutilizar o contexto do protocolo clínico em diferentes consultas.

**Comandos executados:**
```bash
cd module-04
node trialforge-gateway-prototype.js
```

### Módulo 05: Arquitetura Enterprise

#### **Projeto:** [Vitalis Pharma - Trial Forge Platform](module-05)

**Tecnologias utilizadas:**
- **LLM (Large Language Model)** - Motor com raciocínio estruturado
- **Kubernetes** - Orquestração de contêineres para cargas permanentes
- **Serverless** - Modelo de execução sob demanda para cargas esporádicas
- **Edge Computing** - Processamento próximo ao usuário para baixa latência e privacidade
- **KServe** - Plataforma para inferência de modelos em Kubernetes
- **OpenTelemetry** - Instrumentação para observabilidade distribuída
- **Open Policy Agent** - Controle de políticas centralizado

**Conceitos abordados:**
- **Arquitetura Enterprise:** A plataforma deixa de atender um único estudo e passa a ser compartilhada por dezenas de equipes, países e projetos simultaneamente. O maior desafio não é a execução individual, mas o compartilhamento eficiente da infraestrutura.
- **Quatro Componentes do Stack:**
  - **API Gateway:** Ponto único de entrada para todos os estudos. Centraliza autenticação, controle de acesso, limitação de taxa, observabilidade e gerenciamento de modelos.
  - **Orquestração (Kubernetes):** Administra quantas instâncias de cada serviço permanecem ativas, ampliando ou reduzindo conforme a demanda. A plataforma adapta sua capacidade ao comportamento real da carga.
  - **Serviços Compartilhados:** Embeddings, modelos de linguagem, cache e recuperação de contexto existem uma única vez e são reutilizados por todos os estudos. Evita reimplementação dos mesmos serviços por diferentes equipes.
  - **Observabilidade Unificada:** Logs, métricas, trilhas de auditoria e eventos compõem uma visão única da plataforma. Cada execução continua identificada individualmente, mas toda a infraestrutura de registro é compartilhada.
- **Três Princípios da Arquitetura Enterprise:**
  - **Loose Coupling:** Reduz dependências entre diferentes partes. Cada estudo evolui seus componentes sem impactar os demais.
  - **Clear Interfaces:** Cada serviço compartilhado expõe contratos explícitos. Consumidores dependem apenas das interfaces, não dos detalhes internos.
  - **Policy-Driven Control:** Regras de autorização, limites de custo e utilização são controladas por políticas centralizadas (ex: Open Policy Agent), avaliadas dinamicamente durante a execução.
- **Observabilidade para IA:**
  - **Prompt como Código:** Prompts são versionados como código-fonte. Cada alteração é uma nova versão com histórico preservado. Diff compara versões; Replay reproduz execuções antigas com diferentes versões.
  - **Metadados de IA:** Registro de tokens de entrada/saída, modelo utilizado, tempo até primeiro token, tempo total, documentos recuperados no RAG.
  - **Deriva de Qualidade:** A qualidade do modelo pode degradar silenciosamente mesmo com infraestrutura saudável. É necessário monitorar prompts, versões, documentos recuperados e qualidade das respostas separadamente da infraestrutura.
- **Modelos de Implantação:**
  - **Kubernetes:** Mantém serviços permanentemente disponíveis. Ideal para cargas constantes e previsíveis. Custo contínuo mesmo sem utilização.
  - **Serverless:** Ambiente criado apenas quando uma requisição chega. Cobrança baseada no trabalho efetivamente realizado. Ideal para cargas esporádicas ou com grandes variações. Desafio: cold start (carregamento do modelo, inicialização da GPU).
  - **Edge Computing:** Processamento próximo ao usuário ou no dispositivo. Reduz latência (distância física) e preserva privacidade (dados sensíveis não saem do dispositivo). Não elimina necessidade de auditoria centralizada.
- **Controle Inteligente de Custos:**
  - **Model Cascading:** O modelo mais barato tenta responder primeiro. Se a confiança da resposta for insuficiente, escala para o próximo nível (mais caro, mais sofisticado). A decisão de escalar ocorre depois da execução, não antes (diferente do Model Router).
  - **Controle por Tenant:** Cada estudo possui seu próprio orçamento. A verificação acontece antes de qualquer processamento (no Gateway), interrompendo requisições que excederiam o limite.
  - **Dois Sinais de Confiança:** Qualidade da recuperação (RAG) + confiança da resposta do modelo. Ambos são usados para decidir quando escalar na cascata.
- **Evaluation Gate:** Antes de promover um novo modelo, ele é executado contra um Golden Set (conjunto conhecido de perguntas e respostas). A nova versão só é promovida se mantiver desempenho compatível com a tolerância definida. Protege contra regressões de qualidade e configurações incorretas.

**Aplicação prática:**
No Trial Forge em escala enterprise, o API Gateway centraliza autenticação e roteamento para todos os estudos clínicos da Vitalis Pharma. O Kubernetes mantém os serviços centrais (orquestração, autenticação, componentes compartilhados) continuamente disponíveis. Processamentos ocasionais (tarefas de apoio sob demanda) são executados em Serverless para reduzir custos. Funcionalidades com dados sensíveis podem utilizar Edge Computing para preservar privacidade. A observabilidade unificada registra prompts versionados (cada alteração é rastreada via Diff e Replay), metadados de IA (tokens, modelo, tempo, documentos RAG) e monitora deriva de qualidade separadamente da infraestrutura. O Model Cascading é aplicado: consultas rotineiras começam pelo modelo mais barato; se a confiança for insuficiente, escalam para modelos mais sofisticados (exceto CSR, que vai direto ao modelo mais avançado por risco regulatório). Cada estudo possui seu próprio orçamento, verificado no Gateway antes de qualquer processamento. O Evaluation Gate bloqueia automaticamente a promoção de novos modelos ou configurações que causem regressão de qualidade no Golden Set.

**Comandos executados:**
```bash
cd module-05
node model-eval-gate-prototype.js
node manipulation-guardrail-prototype.js
node trialforge-model-tiering-prototype.js
```
