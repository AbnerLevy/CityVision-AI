# 🏙️ CityVision-AI: Sistema Multiagentes de Planejamento Urbano Sustentável

O **CityVision AI** é um sistema de consultoria autônomo em planejamento urbano sustentável baseado em **Arquitetura Multiagente Iterativa**, orquestrado via **n8n**, executando modelos de linguagem locais via **LM Studio** e integrado a dados web em tempo real. Desenvolvido como projeto prático para o curso de **Engenharia de Prompt** da **Residência Tecnológica Accenture + Porto Digital**.

O objetivo da ferramenta é apoiar gestores públicos, urbanistas e comunidades no diagnóstico socioambiental e no planejamento de intervenções urbanas sustentáveis, integrando dados locais e inteligência artificial generativa.

---

## 🚀 Arquitetura do Sistema

O sistema foi desenhado para simular uma mesa de reuniões multidisciplinar real, onde agentes especializados em estrutura em pilares inteligentes e autônomos, debatem, entram em conflito técnico-financeiro, recebem correções de um Orquestrador e chegam a um **Consenso Final** em um ciclo de 3 rodadas iterativas. 

1. **Filtro Web Inteligente (Tavily + Pesquisador):** 
   Utiliza a API de busca web integrada a um agente dedicado que lê os resultados brutos da internet. Caso encontre dados irrelevantes ou de outras cidades, ele emite o gatilho de rejeição (`DADOS_INUTEIS`), evitando alucinações e garantindo que o sistema foque no caso real (como os desafios da Rua Imperial, em Recife).
2. **Arquitetura Multiagente em Paralelo (3 Rodadas Iterativas):** 
   Uma equipe composta por 6 especialistas (**Urbanista, Engenheiro, Mobilidade, Sustentabilidade, Sociólogo e Economista**) que analisam o problema em paralelo, debatendo ativamente, confrontando ideias e ajeitando divergências de silo ao longo de três ciclos de reuniões.
3. **Orquestrador Crítico:** 
   O gestor de IA que atua entre as rodadas com rigor diretivo. Ele corrige rumos, veta propostas utópicas, cobra estimativas reais de Capex/Opex, saneamento básico e parâmetros técnicos objetivos da equipe.
4. **Agente Apresentador Executivo:** 
   O comunicador oficial do sistema que lê o consenso final aprovado e sintetiza todo o processo em um **Relatório Executivo limpo, sem placeholders e pronto para uso institucional** (apresentações para gestores públicos e bancas acadêmicas).

**workflow:**  

[Formulário de Entrada] ──► [Tavily Search API] ──► [Agente Pesquisador (Filtro Anti-Lixo / "DADOS_INUTEIS")] ──► [Rodada 1: Execução Paralela] ──► [Orquestrador (Crítica e Ajuste)] ──► [Rodada 2: Execução Paralela] ──► [Orquestrador (Validação de Custos)] ──► [Rodada 3: Consenso Final] ──► [Agente Apresentador (Relatório Executivo)]

---

## 👥 A Equipe de Agentes Especialistas

Cada agente possui um prompt especializado ("System Message") com restrições rígidas de escopo, tom técnico e uma **trava temporal contra alucinações**:

1. **Urbanista:** Focado em zoneamento misto, cidade de 15 minutos e escala humana.
2. **Engenheiro de Infraestrutura:** Focado em viabilidade construtiva, macrodrenagem, saneamento e Capex.
3. **Especialista em Mobilidade:** Focado em ciclomobilidade, transporte ativo e desenho universal.
4. **Ambientalista:** Defensor de Soluções Baseadas na Natureza (NBS) e mitigação de ilhas de calor.
5. **Sociólogo Urbano:** Focado em habitação de interesse social (HIS) e mitigação de gentrificação.
6. **Economista Urbano:** O guardião do orçamento, avaliando ROI, teto orçamentário e fontes de financiamento (PPPs).
7. **Orquestrador-Chefe:** O gestor que analisa os conflitos de silo entre os especialistas e dita ordens corretivas a cada rodada.
8. **Agente Apresentador:** Sintetiza todo o processo em um **Relatório Executivo final** pronto para gestores públicos.

---

## 🛡️ Principais Diferenciais Técnicos

* **Execução 100% Local (Privacy & Cost-Effective):** Roda modelos open-source otimizados (`google/gemma-3-4b`) através do LM Studio com aceleração de hardware local.
* **Filtro Inteligente de Pesquisa Web:** O Agente Pesquisador analisa criticamente os resultados do Tavily. Caso venham dados irrelevantes de outras cidades, ele emite um gatilho de rejeição (`DADOS_INUTEIS`), evitando alucinações na base de dados dos especialistas.
* **Ciclo de Auto-Correção (3 Rodadas):** Os agentes não geram apenas uma resposta estática; eles debatem, erram, recebem críticas severas do Orquestrador e convergem para um plano viável.
* **Blindagem Temporal:** Instruções rígidas impedem que os modelos insiram anos específicos (como 2023 ou 2026) em diagnósticos urbanos.

---

## 📊 Exemplo de Output Real (Estudo de Caso: Recife - Rua Imperial)
O sistema foi testado para requalificação de vazios urbanos e problemas de drenagem, gerando automaticamente um plano diretor orçado em R$ 42.500.000,00, integrando engenharia de macrodrenagem, ciclovias, áreas verdes (NBS) e conselhos comunitários participativos.

👉 **[Clique aqui para ler o Relatório Executivo Completo]([./docs/output_rua_imperial.md](https://github.com/AbnerLevy/CityVision-AI/blob/main/docs/teste_output.md))**

---

## 🛠️ Tech Stack
* **Orquestração:** n8n (Self-hosted)
* **Runtime / Backend:** Node.js
* **LLM Engine:** LM Studio (`google/gemma-3-4b`)
* **Busca Web:** Tavily AI API
* **Linguagem de Lógica:** JavaScript Expressions & Markdown
