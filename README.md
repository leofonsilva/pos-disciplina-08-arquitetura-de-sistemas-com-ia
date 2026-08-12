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
