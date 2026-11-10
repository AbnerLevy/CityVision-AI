# CityVision AI: Relatório de Testes, Desafios e Correções Aplicadas

Este documento cataloga a jornada de testes, identificação de falhas arquiteturais e respectivas correções aplicadas durante o desenvolvimento do **CityVision AI**, um sistema multiagente autônomo para planejamento urbano.

## 1. Alucinação Temporal e Coordenadas Históricas

- **Desafio Identificado:** O modelo local (`google/gemma-3-4b`) tendia a inserir o ano de 2023 ou datas específicas nos relatórios devido aos limites temporais de sua base de treinamento.
- **Correção Aplicada:** Implementação de uma **"Trava Temporal"** rigorosa em todas as System Messages dos agentes, proibindo expressamente o uso de anos específicos e determinando o uso exclusivo de uma linguagem atemporal (presente e futuro).

## 2. Poluição de Dados na Pesquisa Web (Tavily Search API)

- **Desafio Identificado:** A busca web genérica trazia "lixo" informacional de outras cidades (como empresas de engenharia em São Paulo ou comércios em Portugal), fazendo com que os agentes alucinassem fatos incorretos sobre o local pesquisado.
- **Correção Aplicada:**
  1. Criação de um protocolo de validação no Agente Pesquisador.
  2. Implementação do gatilho condicional `DADOS_INUTEIS`: caso a pesquisa retorne dados irrelevantes, o sistema descarta a busca web e orienta os especialistas a focarem estritamente no briefing original do cliente.

## 3. Conflitos de Silos na Execução Paralela (Falta de Alinhamento)

- **Desafio Identificado:** Na primeira execução em paralelo, os especialistas operavam de forma desconexa (ex: o Urbanista propunha adensamento sem base geotécnica; o Economista bloqueava o orçamento sem diálogo prévio).
- **Correção Aplicada:** Adoção de uma **Arquitetura Iterativa de 3 Rodadas com Caos Controlado**.
  - O Orquestrador foi reescrito para atuar como um gestor crítico e severo.
  - Nas Rodadas 2 e 3, os agentes recebem as críticas do Orquestrador e são obrigados a ajustar suas propostas, garantindo a convergência técnica e financeira antes do relatório final.

## 4. Estouro de Contexto no LM Studio (Erro 400 - Exceed Context Size)

- **Desafio Identificado:** Com o acúmulo progressivo de dados entre as rodadas (Rodada 1 + Orquestrador + Rodada 2 + Orquestrador...), a requisição ultrapassou o limite padrão de **8.192 tokens**, atingindo 9.104 tokens e gerando falha na API local.
- **Correção Aplicada:** Expansão da janela de contexto (*Context Length* / `n_ctx`) diretamente nas configurações do LM Studio para **16.384 tokens**, acomodando com folga o histórico completo de debate da equipe.

## 5. Presença de Placeholders Genéricos no Apresentador

- **Desafio Identificado:** O Agente Apresentador gerava o relatório final contendo campos em branco ou marcadores genéricos (ex: `[Valor Definido pelo Orquestrador]`) em vez de incorporar os dados numéricos debatidos.
- **Correção Aplicada:** Atualização da diretriz do Apresentador proibindo terminantemente o uso de placeholders. O prompt foi ajustado para forçar a extração, humanização e inserção direta dos valores reais de Capex, Opex e teto orçamentário (como o orçamento consolidado de R$ 42.500.000,00 para a Rua Imperial).
