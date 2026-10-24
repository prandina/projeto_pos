# Prompt 01: Geração de Casos de Teste de Integração (Cenários Funcionais)

## Instrução para a LLM

Atue como um **Engenheiro de Qualidade de Software (QA Senior)** especializado em arquitetura de microsserviços e comunicação via APIs REST.

### Contexto
Estamos documentando e validando a integração síncrona via HTTP/REST entre dois componentes principais de uma plataforma de e-commerce:
1. **Serviço Origem:** `Order Service` (Serviço de Pedidos - Checkout)
2. **Serviço Destino:** `Payment Gateway API` (Gateway de Pagamento)

### Objetivo
Gere a documentação completa dos **Casos de Teste de Integração para os Fluxos Funcionais Principais (Caminho Feliz e Regras Negociais Padrão)** na comunicação entre estes serviços ao realizar uma transação via `POST /api/v1/payments`.

---

### Directrizes Rigorosas de Documentação (Requisitos STEMA)
A documentação gerada deve estar pronta para ser auditada pela ferramenta **STEMA** e precisa seguir estritamente os critérios dos quatro pilares abaixo:

1. **Formatação:**
   - O documento final deve ser totalmente estruturado em **Markdown clean**.
   - Utilize tabelas organizadas para apresentar os passos de execução de cada teste.
   - Utilize blocos de código (`json`) devidamente indentados para representar as requisições (Request Payload) e respostas (Response Payload).
   - Mantenha títulos hierárquicos claros (`#`, `##`, `###`).

2. **Linguagem:**
   - Adote um tom profissional, técnico, impessoal e objetivo.
   - Utilize terminologia precisa de arquitetura de software e APIs (ex: *endpoints, HTTP status code, request header, response body, payload, idempotency*).
   - Evite ambiguidades ou instruções vagas (ex: substitua "envie os dados do cartão" por "envie uma requisição HTTP POST contendo o objeto JSON com os atributos do cartão").

3. **Semântica:**
   - Garanta consistência absoluta entre o fluxo do `Order Service` e a resposta do `Payment Gateway`.
   - Certifique-se de que o código de status HTTP (ex: `200 OK`, `201 Created`) seja estritamente condizente com o contrato da API e o estado do resultado esperado.
   - O resultado esperado do teste deve descrever tanto o comportamento do serviço destino quanto o impacto de retorno no serviço origem.

4. **Originalidade:**
   - **Não utilize placeholders genéricos** (ex: "teste1", "foo", "bar", "123456").
   - Construa dados sintéticos realistas, contextuais e representativos do domínio financeiro (nomes de clientes verossímeis, tokens mascarados válidos, valores decimais coerentes, UUIDs sintéticos válidos).

---

### Estrutura Esperada para Cada Caso de Teste Gerado:
- **ID do Teste:** (ex: `CT-INT-FUNC-01`)
- **Título do Cenário:**
- **Serviço Origem / Serviço Destino:**
- **Pré-condições:**
- **Passos de Execução:** (Tabela com: Passo | Ação | Componente Envolvido | Resultado Esperado do Passo)
- **Request Payload (`POST /api/v1/payments`):** (Bloco JSON)
- **Response Payload Esperado:** (Bloco JSON + Status Code HTTP)
- **Pós-condições / Critérios de Aceite:**