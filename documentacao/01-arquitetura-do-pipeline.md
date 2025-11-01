# Arquitetura da Solução (Como Funciona)

O sistema é composto por 3 workflows independentes no n8n (`/workflows`) que são acionados em sequência por webhooks.

## Estágio 1: Agente Criador de Testes
Este workflow é o ponto de partida do pipeline.

1.  **Gatilho:** Um formulário (`On form submission`) ou Webhook. O dev insere o `System prompt` (contexto do teste), a `URL da planilha` e o `Path do webhook` do agente-alvo.
2.  **Seleção de Modo (IF):** O dev escolhe "Novo teste" ou "Repete o mesmo".
    * **Se "Novo teste":** O workflow apaga os dados de *todas* as planilhas (testes antigos e resultados antigos).
    * **Se "Repete o mesmo":** O workflow apaga *apenas* os resultados antigos ("Análises", "Revisão", "Resultado"), preservando os testes já criados na planilha "Testes Detalhados".
3.  **Geração (IA):** (Apenas se for "Novo teste") Um Agente de IA (`Basic LLM Chain` + `Grok-4 Fast`) usa o contexto para gerar a bateria de testes.
4.  **Parse (Python):** Um nó de Código Python (`Divide cada teste`) "quebra" o texto da IA em itens JSON individuais para cada teste.
5.  **Ação:** Salva os novos casos de teste na planilha **"Testes Detalhados"**.
6.  **Próximo Estágio:** Dispara um Webhook (`Envia pra testar`) para acionar o Estágio 2.

## Estágio 2: Agente Testador de LLMs (O Simulador)
Este workflow simula as conversas com o agente-alvo.

1.  **Gatilho:** Recebe o Webhook do Estágio 1.
2.  **Leitura:** Puxa os casos de teste da planilha **"Testes Detalhados"**.
3.  **Loop:** Para cada caso de teste, ele inicia um loop (`Loop Over Items`).
4.  **Execução (IA + Ferramenta):** Um `AI Agent` (o "Test Runner") simula o usuário. Ele usa uma ferramenta customizada (`agente_principal`) que é um nó de código (JavaScript/Axios) que faz a chamada de API real para o "Agente-Alvo" que está sendo testado.
5.  **Memória:** Uma memória (`Postgres Chat Memory`) é usada para manter o contexto da conversa, turno após turno.
6.  **Ação:** Ao final de cada teste, o workflow puxa o histórico completo da conversa do Postgres (`Puxa histórico`) e salva o log na planilha **"Análises"**.
7.  **Próximo Estágio:** Ao final de *todos* os testes, dispara um Webhook (`Envia pra revisão`) para acionar o Estágio 3.

## Estágio 3: Agente Revisador (O Gerente de QA)
Este workflow é o cérebro analítico do pipeline.

1.  **Gatilho:** Recebe o Webhook do Estágio 2.
2.  **Fase A: Revisão Individual**
    * Lê todos os logs de conversa da planilha **"Análises"**.
    * Entra em um loop (`Loop Over Items`).
    * Para cada conversa, um `AI Agent` (o "Crítico Individual") a avalia e gera um `score` e uma `justificativa`.
    * Salva essa revisão individual na planilha **"Revisão"**.
3.  **Fase B: Relatório Consolidado (Após o loop)**
    * Lê *todas* as revisões individuais da planilha **"Revisão"**.
    * Agrega (`Aggregate3`) todos os dados em um único bloco de texto.
    * Envia este bloco para um `AI Agent` final (o "Gerente de QA"), que usa um modelo poderoso (`Gemini 2.5 Pro`) e um prompt massivo de "Análise Consolidada".
    * Este agente gera o relatório final, calculando um **Score Geral (0-100)** e analisando **Padrões de Erro**.
    * **Resultado Final:** Salva o relatório completo na planilha **"Resultado"**.
