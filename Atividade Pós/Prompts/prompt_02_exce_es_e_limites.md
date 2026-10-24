# Prompt 02: Geração de Casos de Teste de Integração (Exceções, Erros e Limites)

## Instrução para a LLM

Atue como um **Engenheiro de Qualidade de Software (QA Senior)** focado em Resiliência, Segurança e Testes de Borda/Integração.

### Contexto
Estamos testando a resiliência e a tratativa de erros na integração síncrona via HTTP/REST entre:
1. **Serviço Origem:** `Order Service` (Serviço de Pedidos - Checkout)
2. **Serviço Destino:** `Payment Gateway API` (Gateway de Pagamento)

### Objetivo
Gere a documentação técnica detalhada dos **Casos de Teste de Integração para Exceções, Condições de Borda e Falhas de Comunicação**, cobrindo especificamente:
1. **Timeout de Comunicação/Latência Excessiva** (Ex: Gateway demora mais que o limite configurado e responde com `504 Gateway Timeout` ou estouro de socket).
2. **Falha de Validação de Payload / Regra de Negócio de Borda** (Ex: Cartão recusado por limite excedido `422 Unprocessable Entity` ou dados malformados/tipos incorretos `400 Bad Request`).
3. **Instabilidade/Erro Interno do Gateway** (Ex: `500 Internal Server Error` no Gateway e validação da estratégia de fallback/retry no Order Service).

---

### Directrizes Rigorosas de Documentação (Requisitos STEMA)
A documentação gerada por você será submetida à avaliação da ferramenta **STEMA**. Para garantir a aprovação, certifique-se de atender aos 4 pilares:

1. **Formatação:**
   - Apresente o conteúdo em **Markdown impecável**.
   - Destaque erros, status HTTP e atributos com formatação inline (ex: `400 Bad Request`, `amount`).
   - Use blocos de código com sintaxe destacada (`json`) para payloads de erro e retornos da API.
   - Organize os cenários e tabelas com separação visual limpa.

2. **Linguagem:**
   - Empregue linguagem formal, técnica e sem margem para interpretações duvidosas.
   - Utilize vocabulário padrão de tratamento de exceções em integrações (ex: *circuit breaker, fallback, retry policy, timeout limit, payload validation error, error schema*).
   - Escreva passos de teste na voz ativa e imperativa.

3. **Semântica:**
   - Mantenha total coerência entre a falha simulada e a estrutura do payload de erro retornado pelo Gateway.
   - Garanta que as mensagens de erro no JSON (`message`, `code`, `details`) correspondam exatamente ao código de status HTTP retornado.
   - Descreva a ação defensiva esperada do `Order Service` ao receber cada tipo de erro (ex: não duplicar cobrança, reverter estado do pedido para 'Pendente').

4. **Originalidade:**
   - Crie mensagens e estruturas de erro ricas e detalhadas, similares às APIs de pagamento do mercado real (como Stripe, Adyen ou PagSeguro).
   - Evite exemplos óbvios ou simplistas. Inclua dados de borda interessantes (ex: valores zerados, moedas não suportadas, strings com caracteres especiais, limites de timeout em milissegundos).

---

### Estrutura Esperada para Cada Caso de Teste Gerado:
- **ID do Teste:** (ex: `CT-INT-EXC-01`)
- **Tipo de Exceção / Cenário de Borda:**
- **Serviço Origem / Serviço Destino:**
- **Comportamento Esperado da Rede/Gateway:** (Timeout, Erro de Validação, Queda)
- **Request Payload (`POST /api/v1/payments`):** (Bloco JSON com o dado de borda/inválido ou normal)
- **Response Payload de Erro:** (Bloco JSON do contrato de erro + HTTP Status Code)
- **Tratativa e Fallback no Order Service:** (Como o sistema origem reage para não quebrar a aplicação)