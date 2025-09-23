# Documentação de Testes – MS Logs

## 1. Introdução
O objetivo desta documentação é descrever os testes a serem realizados no microserviço de logs (ms-logs). O sistema é desenvolvido em Node.js (ESM) com Express 5 e utiliza gRPC para comunicação com o serviço de produtos.
---

## 2. Escopo dos Testes
Os testes contemplam:
- **CreateLog**
- **ListLogs**

---

## 3. Ambiente de Testes
- **Banco de Dados**: MongoDB  
- **Interface de comunicação**: gRPC  
- **Porta do serviço**: 50051  

---

## 4. Casos de Teste

### 4.1 CreateLog
**Caso**: Validar criação de logs.  

**Entrada válida**:
```json
{
  "service": "auth",
  "message": "Usuário realizou login",
  "user": "admin",
  "log": "{\"ip\":\"127.0.0.1\"}"
}

**Saída esperada**:
- **Log criado com id gerado automaticamente.  
- **Campo timestamp preenchido automaticamente.  
- **Campo log convertido corretamente para objeto { "ip": "127.0.0.1" }.

### 4.1 ListLogs
**Caso**: Validar listagem de logs com paginação.

**Cenários**:
- Log criado com id gerado automaticamente.  
- Campo timestamp preenchido automaticamente.  
- Campo log convertido corretamente para objeto { "ip": "127.0.0.1" }.
- Página inicial (page=1, limit=2) -> retorna até 2 registros, hasPrev=false
- Página intermediária (page=2, limit=2) -> retorna registros correspondentes, hasPrev=true, hasNext=true
- Página inexistente (page=99) -> retorna lista vazia, hasNext=false
- Ordenação -> logs retornados em ordem decrescente por timestamp
- Limite fora do range (limit=0 ou limit=200) -> ajustado automaticamente para 1 e 100

 **Saída esperada**:
- Array de logs no formato definido em logs.proto
- Metadados preenchidos corretamente (total, page, limit, totalPages, hasNext, hasPrev)

---

## 5. Critérios de Aceitação
- Todos os métodos devem responder conforme o contrato definido em logs.proto
- Nenhum campo obrigatório pode ser nulo ou ausente  
- O campo timestamp deve ser preenchido automaticamente na criação do log
- A paginação deve respeitar os limites definidos (1 ≤ limit ≤ 100)

