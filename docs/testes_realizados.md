# CityVision AI: Relatório de Testes, Desafios e Otimizações de Engenharia

Este documento cataloga a jornada de testes, identificação de falhas arquiteturais, otimização de hardware local e respectivas correções aplicadas durante o desenvolvimento e evolução do **CityVision AI**, um sistema multiagente autônomo para planejamento urbano sustentável.

---

## 🖥️ Ambiente de Execução (Servidor Local)
- **Processador:** Intel Core i5-12450H
- **Memória RAM:** 24 GB DDR4
- **GPU Dedicada:** NVIDIA GeForce RTX 3050
- **Motor de Inferência:** LM Studio executando `google/gemma-3-4b`
- **Orquestração:** n8n (Self-hosted) + Tavily Search API

---

## Fase 1: Validação Funcional e Primeiras Correções (v1.0)

### 1. Alucinação Temporal e Coordenadas Históricas
- **Desafio Identificado:** O modelo local (`google/gemma-3-4b`) tendia a inserir o ano de 2023 ou datas específicas nos relatórios devido aos limites temporais de sua base de treinamento.
- **Correção Aplicada:** Implementação de uma **"Trava Temporal"** rigorosa em todas as *System Messages* dos agentes, proibindo expressamente o uso de anos específicos e determinando o uso exclusivo de uma linguagem atemporal (presente e futuro).

### 2. Poluição de Dados na Pesquisa Web (Tavily Search API)
- **Desafio Identificado:** A busca web genérica trazia ruído informacional de outras cidades (como empresas de engenharia em São Paulo ou comércios em Portugal), induzindo os agentes a alucinarem fatos incorretos sobre o local pesquisado.
- **Correção Aplicada:**
  1. Criação de um protocolo de validação estrita no **Agente Pesquisador**.
  2. Implementação do gatilho condicional `DADOS_INUTEIS`: caso a pesquisa retorne dados irrelevantes, o agente descarta a busca web e orienta os agentes especialistas a focarem estritamente nos dados iniciais fornecidos pelo solicitante.

### 3. Conflitos de Silos na Execução Paralela (Falta de Alinhamento)
- **Desafio Identificado:** Na primeira execução em paralelo, os especialistas operavam de forma desconexa (ex.: o Urbanista propunha adensamento sem base geotécnica; o Economista bloqueava o orçamento sem diálogo prévio).
- **Correção Aplicada:** Adoção de uma **Arquitetura Iterativa de 3 Rodadas com Caos Controlado**.
  - O Orquestrador foi configurado para atuar como um gestor crítico e diretivo.
  - Nas Rodadas 2 e 3, os agentes recebem as diretrizes corretivas do Orquestrador e ajustam suas propostas, garantindo convergência técnica e financeira antes do relatório final.

---

## Fase 2: Otimização de Performance, Tokens e Anti-Alucinação (v2.0)

Durante os testes de carga da v1.0 nas 3 rodadas completas (23 chamadas ao LLM), o sistema apresentava um **tempo médio de execução de ~20 minutos**, além de estourar o limite padrão de tokens e inventar cifras financeiras quando o usuário omitia o orçamento. Abaixo estão os diagnósticos e as otimizações estruturais aplicadas:

### 4. Resolução Definitiva do Estouro de Contexto e Gargalo de VRAM (De 16.384 para 4.096 Tokens)
- **Desafio Identificado:** Com o acúmulo progressivo de textos brutos entre as rodadas (`Rodada 1 + Orquestrador + Rodada 2 + Orquestrador...`), a requisição ultrapassava o limite de **8.192 tokens**, atingindo **9.104 tokens** (Erro 400 no LM Studio). A tentativa paliativa inicial de expandir o *Context Length* (`n_ctx`) para **16.384 tokens** saturou a VRAM da **RTX 3050**, forçando *offload* parcial do KV Cache para a memória RAM DDR4 e CPU (i5-12450H), o que derrubou drasticamente a velocidade de inferência e elevou o tempo total do fluxo para ~20 minutos.
- **Correção Aplicada (Compressão por Ata Executiva):**
  1. Eliminação do repasse do texto bruto dos 6 agentes (`Aggregate`) para os 6 agentes da rodada seguinte (que gerava um "efeito bola de neve").
  2. Agora, apenas o **Orquestrador** lê a íntegra das 6 propostas em cada rodada e emite uma **Ata Diretiva Sintetizada** (~180 palavras), que é repassada aos especialistas nas Rodadas 2 e 3.
  3. Com a redução do pico de requisição de **9.104 para ~2.400 tokens (-73%)**, a janela de contexto (`n_ctx`) no LM Studio foi reduzida para **4.096 tokens** com *Flash Attention*, mantendo o modelo e o KV Cache **100% alocados na VRAM da RTX 3050**.

### 5. Padronização de Prompts ("One-Standard") e Controle de Geração
- **Desafio Identificado:** Assimetria severa nos *System Prompts* — enquanto agentes como Engenheiro e Sociólogo possuíam prompts enxutos (~1,5 KB), os agentes de Urbanismo e Mobilidade possuíam prompts extensos (~10 KB) e saídas longas sem limite de tokens, aumentando a latência de leitura (*prompt evaluation*) e geração.
- **Correção Aplicada:**
  - Padronização de todos os agentes para prompts compactos (~1,2 KB a 1,5 KB) em Markdown estruturado.
  - Substituição de nós pesados de *AI Agent* por nós diretos de **`Basic LLM Chain` (`chainLlm`)**.
  - Aplicação de limites rígidos de geração nos nós do modelo local: `temperature: 0.1` no Pesquisador, `temperature: 0.2` e `maxTokens: 350` nos Especialistas e Orquestradores, e `maxTokens: 900` no Apresentador.

### 6. Estruturação do Formulário de Entrada e Precisão no Tavily
- **Desafio Identificado:** O formulário inicial possuía apenas 2 campos genéricos (`briefing` e `contexto`), fazendo com que descrições longas de problemas fossem enviadas inteiras na query da API do Tavily, reduzindo a precisão da busca web.
- **Correção Aplicada:** Refatoração do gatilho inicial (`On form submission`) para **4 campos estruturados e obrigatórios**:
  1. `local_regiao` (Bairro e Cidade) — enviado de forma isolada e limpa para a query do Tavily (`{{ $json.briefing.trim() }} infraestrutura OR problemas urbanos OR notícias locais`) com `max_results: 3`.
  2. `motivo` (Motivo da revitalização).
  3. `detalhes_proposta` (Proposta e condicionantes do local).
  4. `orcamento_estimado` (Orçamento estimado ou faixa de investimento).

### 7. Correção de Roteamento de Variáveis e Blindagem Financeira no Apresentador
- **Desafio Identificado:**
  1. O dossiê validado pelo `Pesquisador` na Rodada 1 era sobrescrito nos nós `Edit Fields Rodada 2` e `Edit Fields Rodada 3`.
  2. Na Rodada 3, a referência de memória apontava para o `Aggregate` da Rodada 1 em vez do `Aggregate1` da Rodada 2.
  3. O **Agente Apresentador** recebia apenas o texto resumido do último Orquestrador, sem acesso direto às propostas técnicas finais (`Aggregate2`) nem ao orçamento original do cliente, o que levava ao uso de *placeholders* ou à invenção de valores monetários.
- **Correção Aplicada:**
  - Preservação integral da variável `contexto` (contendo Motivo, Proposta, Orçamento e Dossiê do Pesquisador) ao longo das 3 rodadas.
  - Injeção direta de 4 blocos consolidados no **Agente Apresentador**: `LOCAL SOLICITADO`, `BRIEFING E ORÇAMENTO ORIGINAL`, `PROPOSTAS TÉCNICAS FINAIS DOS 6 ESPECIALISTAS` (`Aggregate2`) e `APROVAÇÃO FINAL DO ORQUESTRADOR`.
  - Expansão do **Protocolo de Dado Ausente (Zero Alucinação Financeira)** para todos os agentes: é terminantemente proibido inventar ou estimar valores em Reais (R$) não fornecidos pelo usuário. Quando não há orçamento numérico detalhado, o plano apresenta alocação estratégica por fases (Curto, Médio e Longo Prazo) e mecanismos de financiamento (PPPs, outorga onerosa, fundos municipais).

---

## 📊 Resumo Comparativo de Performance (v1.0 vs. v2.0 Otimizada)

| Métrica / Indicador | Versão 1.0 (Inicial) | Versão 2.0 (Otimizada) | Impacto Obtido |
| :--- | :--- | :--- | :--- |
| **Ciclos de Debate Multiagente** | 3 Rodadas (23 chamadas LLM) | **3 Rodadas Mantidas (23 chamadas LLM)** | Preservação de 100% da profundidade metodológica |
| **Pico de Tokens no Contexto** | 9.104 tokens (Erro 400 em 8k) | **~2.400 tokens** | **Redução de ~73%** no consumo de contexto |
| **Janela do LM Studio (`n_ctx`)** | 16.384 tokens (Swap em RAM DDR4) | **4.096 tokens (Flash Attention)** | Execução **100% dentro da VRAM da RTX 3050** |
| **Tamanho dos System Prompts** | Até 10 KB por agente | **Padronizados em ~1,3 KB** | **Redução de ~75%** nos tokens de instrução |
| **Alucinação Temporal / Financeira** | Ocasional em inputs incompletos | **Zero (Bloqueio por Governança)** | Fidelidade estrita aos dados do solicitante |
| **Tempo Médio de Execução** | ~20 minutos | **~4 a 5 minutos** | **Ganho de velocidade de 4x a 5x** |
