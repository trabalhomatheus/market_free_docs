# Documentação de Testes – MS Pagamentos

## 1. Introdução
O objetivo desta documentação é descrever os testes a serem realizados no microserviço de pagamentos (`ms-pagamentos`).  
O serviço é desenvolvido em **Node.js**, utiliza **gRPC** como interface primária e **MongoDB** como banco de dados para persistência de transações.

Este microserviço é responsável por:
- Registrar pagamentos.
- Processar diferentes métodos de pagamento.
- Validar e retornar o status de transações.
- Consultar transações individuais.
- Registrar logs via serviço gRPC externo (`LogsService`).

---

## 2. Escopo dos Testes
Os testes contemplam:

- Métodos gRPC expostos pelo serviço:
  - `ProcessPayment`
  - `GetTransaction`
- Persistência correta das transações no MongoDB.
- Integração com o serviço gRPC de logs (`LogsService`).
- Tratamento de erros e respostas gRPC adequadas.
- Validação de entradas e regras de negócio:
  - Métodos de pagamento aceitos.
  - Valores válidos.
  - Pedido existente.
- Comportamento em cenários de falha:
  - Banco indisponível.
  - Falha no serviço de logs.
  - Transação não encontrada.

---

## 3. Ambiente de Testes
- **Node.js**: >= 18  
- **gRPC**: @grpc/grpc-js  
- **Banco de dados**: MongoDB 4.4+  
- **Porta padrão do serviço**: 50055  
- **Ferramentas**: Jest, grpcurl, Docker, Kubernetes/Minikube  
- **Dependências externas**:
  - Serviço gRPC `LogsService` (mockado ou real)

---

## 4. Casos de Teste

### 4.1 Processar Pagamento – Sucesso
**Caso:** Garantir que o pagamento é processado corretamente.  
- **Método**: `ProcessPayment`  
- **Entrada**:
```json
{
  "pedidoId": "123",
  "valor": 150.00,
  "metodo": "cartao"
}
```
- **Resultado Esperado**:
  - Status: `OK`
  - Transação criada com:
    - `status = "APROVADO"`
    - `dataTransacao` preenchida
  - Registro salvo no MongoDB
  - Log enviado ao serviço `LogsService`

---

### 4.2 Processar Pagamento – Método Inválido
**Caso:** Método de pagamento não aceito.  
- **Entrada**:
```json
{
  "pedidoId": "123",
  "valor": 150,
  "metodo": "cripto"
}
```
- **Resultado Esperado**:
  - Código gRPC: `INVALID_ARGUMENT`
  - Mensagem: `"Método de pagamento inválido"`
  - Nenhuma transação registrada

---

### 4.3 Processar Pagamento – Valor Inválido
**Caso:** Valor menor ou igual a zero.  
- **Resultado Esperado**:
  - Código gRPC: `INVALID_ARGUMENT`
  - Mensagem: `"Valor deve ser maior que zero"`

---

### 4.4 Processar Pagamento – Erro no Serviço de Logs
**Caso:** Serviço `LogsService` indisponível.  
- **Simulação:** Mock retornando erro.  
- **Resultado Esperado**:
  - Pagamento processado normalmente
  - Log não enviado
  - Um aviso é registrado nos logs internos do serviço

---

### 4.5 Buscar Transação – Sucesso
**Caso:** Recuperar uma transação existente.  
- **Método**: `GetTransaction`  
- **Entrada**:
```json
{ "id": "<id-existente>" }
```
- **Resultado Esperado**:
  - Código: `OK`
  - Corpo contendo:
    - `id`
    - `pedidoId`
    - `valor`
    - `status`
    - `metodo`
    - `dataTransacao`

---

### 4.6 Buscar Transação – Não Encontrada
- **Entrada**:
```json
{ "id": "id-invalido" }
```
- **Resultado Esperado**:
  - Código: `NOT_FOUND`
  - Mensagem: `"Transação não encontrada"`

---

### 4.7 Banco Indisponível
**Caso:** MongoDB fora do ar.  
- **Simulação:** Derrubar container `mongodb`.  
- **Resultado Esperado**:
  - `ProcessPayment` → retorna `INTERNAL`
  - `GetTransaction` → retorna `INTERNAL`
  - Logs internos registram a exceção

---

## 5. Estratégia de Teste
- **Testes unitários**:
  - Funções de validação
  - Funções auxiliares (`toProto`)
  - Lógica de criação de transações
- **Testes de integração**:
  - gRPC + MongoDB + LogsService (mockado)
- **Testes end-to-end**:
  - Execução via Docker Compose ou Kubernetes
  - Simulação completa: Pedido → Pagamento
- **Testes de contrato**:
  - Validação das mensagens gRPC (`payments.proto`)

---

## 6. Critérios de Aceitação
- Nenhuma operação deve retornar erro não tratado.
- As respostas devem seguir o contrato gRPC.
- Logs devem registrar eventos corretamente.
- Dados devem permanecer consistentes mesmo em falhas externas.

---

## 7. Riscos e Considerações
- Serviço de logs pode se tornar ponto crítico.
- Falhas no banco exigem políticas de retry em produção.
- Novos métodos de pagamento podem demandar novas validações.

