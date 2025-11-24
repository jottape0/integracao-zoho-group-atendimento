### 🎯 Objetivo

O objetivo principal é transformar dados de conversas (texto e transcrições de áudio) em métricas estruturadas de atendimento, intenção, e satisfação, enriquecendo o perfil do cliente no CRM.

### 🌟 Visão Geral do Fluxo

O workflow é ativado por um Webhook do Group Atendimento após o finalização de um chat. Ele segue uma sequência de etapas para coleta, processamento, análise via Inteligência Artificial, e registro final dos dados:

    Gatilho (Webhook): Recebe o evento de finalização de chat do Group Atendimento.

    Coleta de Dados: Busca as mensagens completas do chat, incluindo informações do atendente e do contato.

    Processamento de Mídia: Identifica e baixa gravações de áudio para transcrição (usando OpenAI).

    Formatação de Conversa: Combina texto do chat e transcrições de áudio em um formato único para a IA.

    Análise por AI Agent: Utiliza um modelo de linguagem (via OpenRouter) para analisar a conversa, extraindo resumo, intenção, sentimentos, riscos, e feedbacks em um formato JSON estruturado.

    Enriquecimento de Dados:

        Extrai o customerCode do nome do contato.

        Consulta o Microsoft SQL Server para obter o idClienteZoho a partir do customerCode.

        Mapeia o sentimento do cliente (positivo, negativo, neutro) para os valores padrão do campo "Status Cliente" no Zoho CRM.

        Busca e corrige o ownerId do atendente para corresponder ao ID de usuário no Zoho CRM.

    Integração com Zoho CRM: Envia o registro de Sucesso do Cliente, contendo todos os dados analisados e enriquecidos, para o Zoho CRM, garantindo que a informação seja registrada apenas para clientes com um idClienteZoho válido e, opcionalmente, que o cliente 
    esteja classificado como "Satisfeito" no atendimento

  ### 🧩 Módulos e Tecnologias-Chave
| Módulo | Tecnologia | Função Principal no Fluxo | 
| :--- | :--- | :--- | 
| Webhook| n8n Base | Gatilho inicial ao finalizar um chat no Group Atendimento. |
| HTTP Request | n8n Base | "Comunicação com APIs externas (Group Atendimento, Zoho CRM, Zoho OAuth)." |
| Code | n8n Base | Scripts para: 1. Formatar a conversa. 2. Extrair áudios. 3. Combinar textos/transcrições. 4. Extrair e estruturar o JSON da IA. 5. Corrigir e-mails e mapear ID do Zoho. 6. Mapear satisfação do cliente. |
| OpenAI | n8n Langchain | Transcrição de mensagens de áudio. |
| OpenRouter Chat Model | n8n Langchain | Provedor do Modelo de Linguagem (LLM) para a análise de conversas. |
| AI Agent | n8n Langchain | "Orquestra a tarefa de análise, garantindo o JSON de saída e a aplicação de regras." |
| Microsoft SQL | n8n Database | Busca o ID do Cliente no Zoho (idClienteZoho) a partir do código interno (customerCode). |
| If | n8n Base | "Implementa lógica condicional para: 1. Prosseguir apenas se o idClienteZoho for encontrado. 2. Opcionalmente, prosseguir apenas se a satisfação for positiva (depende da configuração final da branch)." |

### 🛠️ Pré-requisitos e Configurações
Para replicar este fluxo, são necessárias as seguintes credenciais e configurações nos respectivos nós:

    Group Atendimento:

        access-token para buscar chats e informações de usuários (Configurado em HTTP Request(chats), HTTP Request1, etc.).

    OpenAI:

        API Key para o nó Transcribe a recording.

    OpenRouter:

        API Key para o nó OpenRouter Chat Model (utilizado pelo AI Agent).

    Zoho CRM:

        refresh_token, client_id, e client_secret no nó Gera o access token para autenticação OAuth 2.0.

        A URL da API do Zoho no nó HTTP Request final (https://www.zohoapis.com/crm/v8/...).

    Microsoft SQL:

        Credenciais de acesso ao banco de dados e a query de seleção no nó Microsoft SQL.

### 📈 Análise Detalhada do AI Agent

O nó AI Agent é o centro da inteligência do fluxo. Ele é configurado com a seguinte estrutura de prompt, forçando a saída para um JSON específico:

Instrução Principal:

    "Você é um assistente especializado em análise de atendimentos (mensagens de chat + áudios transcritos). Analise todo o conteúdo recebido. Retorne apenas um JSON válido em uma única linha (sem quebras de linha, sem texto fora do JSON)."

Estrutura do JSON Esperado:

    resumo

    intencao_cliente

    sentimento_cliente

    sentimento_atendente

    objeções_principais

    oportunidades

    proximos_passos

    riscos

    feedback_cliente (incluindo nota 0-10, pontos_positivos, pontos_a_melhorar, evidencias)

    feedback_atendente (incluindo nota 0-10, pontos_positivos, pontos_a_melhorar, evidencias)

    palavras_chave

    extratos_relevantes

    status
