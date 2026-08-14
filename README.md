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
