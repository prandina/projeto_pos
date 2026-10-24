# Casos de Teste de Integração — Exceções e Limites

## 1. Objetivo
Avaliar o comportamento da integração entre o **Serviço de Pedidos** e o **Gateway de Pagamento** (`POST /api/v1/payments`) em situações excepcionais, valores-limite, indisponibilidades de rede e falhas de persistência/comunicação entre os componentes, verificando se as exceções são tratadas adequadamente, se há resiliência no fluxo e se a integridade do estado do pedido é preservada.

---

## 2. Premissas
- Os códigos de status HTTP e payloads de erro indicados utilizam como hipótese o padrão REST/OpenAPI e precisam ser validados contra o contrato formal da API do Gateway.
- Limites de valor transacional (`amount`), tamanho de campos e regras de *idempotency-key* (prevenção de cobrança duplicada) precisam ser confirmados junto à documentação da integração.
- Os cenários de falha de comunicação e persistência dependem de recursos de simulação no ambiente de teste (como *WireMock*, *Mountebank* ou simuladores de degradação de rede).

---

## 3. Casos de Teste

### CT-05 — Campo obrigatório enviado vazio ou ausente
**Objetivo:** Verificar se a integração valida e rejeita payloads com campos obrigatórios ausentes ou com strings vazias antes de acionar a adquirente/gateway.
- **Pré-condições:** Serviço de Pedidos e Gateway de Pagamento ativos e integrados.
- **Requisição:** O Serviço de Pedidos envia para `POST /api/v1/payments` uma requisição com o campo obrigatório `payment_method` ou `card_token` vazio (`""`) ou ausente no JSON.
- **Resultado HTTP esperado:** `400 Bad Request` ou `422 Unprocessable Content`.
- **Resultado esperado nos componentes envolvidos:** A camada de validação da API rejeita o payload imediatamente. O Serviço de Pedidos não altera o status do pedido para "Em Processamento", registrando a falha de validação em log e mantendo o pedido no estado "Pendente de Pagamento".
- **Justificativa:** Garantir que requisições malformadas não trafeguem até a credenciadora de cartão, evitando chamadas desnecessárias à API externa e falhas em cascata.

---

### CT-06 — Valor de transação no limite permitido (*Boundary Value*)
**Objetivo:** Verificar o comportamento da integração diante de um valor de pagamento situado exatamente nos limites mínimo e máximo autorizados para a transação.
- **Pré-condições:** Conhecer os limites operacionais de valor por transação (ex: valor mínimo R$ 0,01 e máximo R$ 50.000,00).
- **Requisição:** Enviar uma requisição de pagamento com o campo `amount` configurado exatamente no valor limite permitido (ex: `amount: 50000.00`).
- **Resultado HTTP esperado:** `200 OK` ou `201 Created` (com status da transação `APPROVED`).
- **Resultado esperado nos componentes envolvidos:** O Gateway processa o valor exato sem arredondamentos ou erros de precisão decimal. O Serviço de Pedidos recebe a confirmação e persiste o registro no banco com a precisão exata do valor aprovado.
- **Justificativa:** Garantir que os tipos de dados monetários (tipicamente *bigdecimal* ou *integers* em centavos) sejam processados e armazenados corretamente nos dois lados da integração sem problemas de arredondamento.

---

### CT-07 — Tipo de dado incorreto no payload de transação
**Objetivo:** Verificar se a API de integração rejeita campos com tipos de dados incompatíveis com o contrato esperado.
- **Pré-condições:** Tipos de dados definidos no contrato OpenAPI da integração (ex: `amount` como *numeric*, `installments` como *integer*).
- **Requisição:** Enviar o campo `installments` (parcelas) com uma string em vez de um número inteiro (ex: `"installments": "tres"`).
- **Resultado HTTP esperado:** `400 Bad Request`.
- **Resultado esperado nos componentes envolvidos:** O *parser* da API rejeita a requisição antes do processamento de negócio. Nenhum evento de pagamento é publicado no mensageiro/barramento e nenhum log de transação financeira é gerado incorretamente.
- **Justificativa:** Evitar erros não tratados de conversão de tipos (*TypeCastException*) nos microsserviços internos ao desserializar os payloads.

---

### CT-08 — Falha no componente de persistência do Gateway/Serviço
**Objetivo:** Verificar o comportamento do fluxo de pagamento quando o banco de dados do Serviço de Pedidos ou do Gateway apresenta falha no momento de registrar o resultado da transação.
- **Pré-condições:** Ambiente de testes configurado para simular *timeout* ou falha de escrita no banco de dados (*Database Connection Timeout*) durante o fluxo do `POST`.
- **Requisição:** Enviar uma solicitação de pagamento válida com o banco de dados sob falha simulada.
- **Resultado HTTP esperado:** `500 Internal Server Error` ou `503 Service Unavailable`.
- **Resultado esperado nos componentes envolvidos:** A API captura a exceção e encerra a transação com *rollback*. Não é informado sucesso falso ao usuário. Caso o Gateway externo já tenha aprovado a cobrança, o mecanismo de compensação/estorno automático (*Saga Pattern/TCC*) é engatilhado.
- **Justificativa:** Garantir a consistência ACID/eventual do sistema e evitar o cenário em que o cliente é cobrado, mas o pedido não é registrado na base.

---

### CT-09 — Requisição duplicada com chave de idempotência (*Idempotency Key*)
**Objetivo:** Verificar como a integração trata a reexecução de pagamentos com a mesma chave de idempotência para prevenir cobranças em duplicidade.
- **Pré-condições:** Regra de idempotência configurada no cabeçalho da requisição (`X-Idempotency-Key`).
- **Requisição:** Enviar duas requisições idênticas consecutivas contendo o mesmo cabeçalho `X-Idempotency-Key: "UUID-12345"`.
- **Resultado HTTP esperado:** 
  - Primeira requisição: `200 OK` / `201 Created`.
  - Segunda requisição: `200 OK` (retornando a mesma resposta armazenada sem reprocessar a cobrança) ou `409 Conflict`.
- **Resultado esperado nos componentes envolvidos:** O Gateway identifica a chave duplicada na camada de *cache* ou banco e não efetua uma nova captura no cartão de crédito, retornando os dados da transação já efetuada.
- **Justificativa:** Evitar cobranças duplas no cartão do cliente motivadas por retentativas automáticas (*retries*) de rede na integração entre o front-end e o back-end.

---

### CT-10 — Timeout e inconsistência de resposta do Gateway externo
**Objetivo:** Verificar a resiliência do Serviço de Pedidos quando o Gateway de Pagamento demora mais tempo para responder do que o limite tolerado (*Read Timeout*).
- **Pré-condições:** Simulador de API (ex: WireMock) configurado para introduzir um *delay* de 10 segundos na resposta do Gateway (limite do cliente configurado em 3 segundos).
- **Requisição:** O Serviço de Pedidos envia a requisição de pagamento e aguarda o retorno.
- **Resultado HTTP esperado:** `504 Gateway Timeout` retornado pelo Serviço de Pedidos para o chamador.
- **Resultado esperado nos componentes envolvidos:** O Serviço de Pedidos aborta a conexão síncrona ao atingir 3 segundos, altera o pedido para o status "Aguardando Confirmação de Pagamento" e agenda uma consulta assíncrona (*Polling* ou consulta à fila de DLQ/Webhook) para harmonizar o estado real do pagamento sem travar a interface do usuário.
- **Justificativa:** Validar a resiliência e o desacoplamento da arquitetura em cenários de degradação e instabilidade de parceiros externos.

---

## 4. Premissas e Dúvidas para Validação

Antes de executar os testes automatizados ou manuais, é necessário validar com a equipe de arquitetura/backend:
1. Quais campos do payload de pagamento aceitam formato *nullable* ou string vazia?
2. Qual é a regra exata e o tempo de expiração do cabeçalho de idempotência (`X-Idempotency-Key`)?
3. Quais os códigos de erro padrão (mínimo e máximo de tentativas de *retry*) definidos no contrato com o Gateway?
4. Como é feito o fluxo de compensação/estorno caso ocorra uma falha de persistência interna após a aprovação da adquirente?
5. Qual é o tempo de *timeout* limite configurado nas rotas de pagamento do API Gateway (`circuit breaker`)?

---

## 5. Observações

- Os casos descritos foram elaborados com foco em resiliência, exceções e tratamento de limites na comunicação entre serviços de checkout e gateways financeiros.
- A validação efetiva desses cenários requer a utilização de ferramentas de simulação de APIs (mocks) com injeção de falhas e latência controlada.
- A documentação e estrutura destes casos visam assegurar o cumprimento dos critérios de **Semântica** e **Linguagem** no padrão do **STEMA**.