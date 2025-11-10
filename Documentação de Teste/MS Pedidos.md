# Documentação de Testes – MS Pedidos

## 1. Introdução
O objetivo desta documentação é descrever os testes aplicados ao microserviço de pedidos (`ms-pedidos`).
Este serviço é responsável pela criação e consulta de pedidos realizados pelos usuários, persistindo dados em MongoDB.

---

## 2. Escopo dos Testes
Os testes contemplam:
- Métodos expostos (`CreateOrder`, `GetOrder`, `ListOrders`).
- Persistência correta dos dados no banco.
- Respostas adequadas em cenários de falha e sucesso.
- Comportamento em integração com carrinho, pagamento ou estoque (mockados, se necessário).

---

## 3. Ambiente de Testes
- **Node.js**: >= 18
- **gRPC**: @grpc/grpc-js
- **Banco**: MongoDB 4.4+
- **Porta padrão**: 50054
- **Ferramentas**: Jest, Docker Compose, grpcurl

---

## 4. Casos de Teste

### 4.1 Criar Pedido – Sucesso
- **Método**: `CreateOrder`
- **Entrada**:
  ```json
  {
    "userId": "user-1",
    "itens": [{ "produtoId": "p1", "quantidade": 2, "preco": 29.9 }],
    "endereco": {},
    "pagamento": {}
  }
  ```
- **Resultado Esperado**:
  - Pedido salvo no banco.
  - Resposta inclui `orderId` e `status: "CREATED"`.

---

### 4.2 Criar Pedido – Sem Itens
- Resultado esperado: Erro `INVALID_ARGUMENT`. Nenhum pedido criado.

---

### 4.3 Buscar Pedido Existente
- **Método**: `GetOrder`
- **Resultado Esperado**: Retorna dados completos do pedido.

---

### 4.4 Buscar Pedido Inexistente
- Resultado esperado: `NOT_FOUND`.

---

### 4.5 Listar Pedidos do Usuário
- **Método**: `ListOrders`
- **Resultado Esperado**: Lista paginada com pedidos do usuário.

---

### 4.6 Banco Indisponível
- Resultado esperado: Erro `UNAVAILABLE` ou `INTERNAL`.

---

### 4.7 Falha em Serviço de Pagamento (Mock)
- **Resultado Esperado**:
  - Pedido não deve ser confirmado como pago.
  - Status adequado refletido no banco.

---

## 5. Estratégia de Teste
- **Unitários**: validações, cálculo de total.
- **Integração**: serviço + DB real com dependências mockadas.
- **E2E**: fluxo completo Carrinho → Pedido.
- **Contrato `.proto`**: validação de estrutura.

---

## 6. Critérios de Aceitação
- Nenhum pedido deve ser criado sem itens.
- Logs devem registrar falhas claramente.

---

## 7. Riscos e Considerações
- Falhas externas podem exigir fluxos compensatórios.
