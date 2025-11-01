# 🚀 Projeto: Pipeline de QA Automatizado para Agentes de IA (Meta-Agente)

Este projeto é um pipeline de automação ponta-a-ponta desenhado para resolver um dos maiores gargalos no desenvolvimento de Agentes de IA: **o teste manual, lento e inconsistente**.

Esta ferramenta atua como um "Meta-Agente", um sistema de IAs que testa, avalia e gera relatórios de performance sobre outros agentes, garantindo qualidade e acelerando o ciclo de desenvolvimento.

### 🎬 Demonstração Rápida (Vídeo)

Devido à complexidade do pipeline completo (execução de ~6 minutos), a demonstração foi dividida em duas partes para melhor visualização:

* **[Parte 1: Geração dos Testes e Execução da Simulação](https://www.loom.com/share/9ebde053df9e48d79ae143305faf2299)**
    * *O que este vídeo mostra:* O formulário de setup, a geração automática dos casos de teste (Estágio 1) e a execução da simulação de conversa (Estágio 2).

* **[Parte 2: Análise, Relatório e Score Final](https://www.loom.com/share/96794539c87b4d9491396bd65eef76c0)**
    * *O que este vídeo mostra:* A revisão individual de cada teste, a análise consolidada do "Agente Gerente" e o relatório final com o Score (0-100) na planilha (Estágio 3).

---

## 1. O Problema (A Dor)

Testar agentes de IA (chatbots, assistentes de vendas, etc.) é um processo complexo:
* **Demorado:** Requer que um humano crie dezenas de cenários e prompts manualmente.
* **Inconsistente:** Um testador humano pode avaliar de forma diferente de outro, ou esquecer de testar "casos extremos" (edge cases).
* **Superficial:** É difícil para um humano simular 20 testes diferentes e depois analisar *padrões de erro* entre todas as conversas.

## 2. A Solução (O Remédio)

Construí um **pipeline de 3 estágios** no n8n que gerencia o ciclo de QA de forma 100% autônoma, usando um "Agente Gerador", um "Agente Simulador" e um "Agente Revisor".

**Principais Vantagens:**
* **Economia Drástica de Tempo:** Reduz o tempo de teste de horas para minutos.
* **Consistência Total:** Todos os testes são gerados e avaliados com os mesmos critérios rigorosos.
* **Análise Profunda:** O "Agente Gerente" final identifica padrões de erro que um humano jamais veria.
* **Recurso de Depuração (Re-teste):** O fluxo permite "Repetir os mesmos testes" após uma correção, garantindo que o bug foi resolvido.

---

## 3. Como Usar (Setup Essencial)

Para executar este pipeline, siga os 3 passos de configuração:

### Passo 1: Copie a Planilha Modelo
O pipeline usa o Google Sheets como banco de dados para mover dados entre os estágios.
* **Ação:** [**Clique aqui para fazer sua cópia da planilha modelo.**](https://docs.google.com/spreadsheets/d/1gqqWsqzD3CMjYye8QzEeWJrBo_fSD2Z33zWt2sdysMg/edit?usp=sharing)
* (Faça login na sua conta Google, vá em `Arquivo` > `Fazer uma cópia`).

### Passo 2: Importe os Workflows
Os 3 workflows JSON estão na pasta `/workflows` deste repositório.
* **Ação:** Baixe os 3 arquivos da pasta `/workflows` e importe-os na sua instância do n8n.

### Passo 3: Configure as Variáveis
* **Workflow 1 (Criador):** No gatilho (Formulário ou Webhook), insira a URL da *sua* cópia da planilha.
* **Workflow 2 (Testador):** No gatilho, insira a URL do seu Agente-Alvo (o webhook do agente que você quer testar). O nó `agente_principal` fará as chamadas para este endpoint.

---

## 4. Arquitetura do Sistema e Documentação

Toda a lógica, arquitetura dos 3 estágios e os prompts detalhados estão documentados na pasta `/documentacao`.

* **[LEIA AQUI: Documentação da Arquitetura do Pipeline](./documentacao/01-arquitetura-do-pipeline.md)**
* **[LEIA AQUI: Documentação dos Prompts](./documentacao/02-prompts-dos-agentes.md)**

---

## 5. Ferramentas Utilizadas (Tech Stack)

* **Orquestração:** **n8n** (workflows, gatilhos, loops)
* **Inteligência (LLMs):**
    * **LangChain Nodes** (`AI Agent`, `Basic LLM Chain`, `Structured Output Parser`)
    * **OpenRouter** (para acesso a múltiplos modelos)
    * **Grok-4 Fast** (para geração de testes e revisão individual)
    * **Google Gemini 2.5 Pro** (para a análise consolidada final)
* **Banco de Dados:**
    * **Google Sheets** (usado como banco de dados de pipeline)
    * **PostgreSQL** (usado para a memória de chat (`Postgres Chat Memory`))
* **Lógica Customizada:**
    * **Python** (no nó `Code` para parsear o output de texto do LLM)
    * **JavaScript** (no nó `ToolCode` para criar a ferramenta `agente_principal` via `axios`)
* **Comunicação:**
    * **Webhooks** e **HTTP Request** (para acionar os workflows em sequência)

---

## 6. Estrutura do Repositório

Aqui está a organização de pastas do projeto para fácil navegação:

<pre>
/
├── README.md (Este arquivo, o "hub" do projeto)
│
├── /workflows/
│   ├── Agente Criador de Testes.json
│   ├── Agente Testador de LLMs.json
│   └── Agente Revisador do teste.json
│
├── /documentacao/
│   ├── 01-arquitetura-do-pipeline.md (Explicação detalhada dos 3 estágios)
│   └── 02-prompts-dos-agentes.md (Os prompts completos de cada Agente de IA)
│
├── /exemplos/
│   └── relatorio-final-exemplo.png (Screenshot do relatório final na planilha)
│
└── .gitignore
</pre>

---

## 7. Contato

Conecte-se comigo para discutir automação, IA ou oportunidades profissionais:

* **LinkedIn:** `https://www.linkedin.com/in/eduardo-sousa-dev12`
* **E-mail:** `eduardodesousasilva12@gmail.com`
