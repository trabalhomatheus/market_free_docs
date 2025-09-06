# Documentação de Testes – MS Produtos

## 1. Objetivo

Garantir que as operações de CRUD do microserviço **Produtos** funcionem corretamente, respeitando as regras de negócio, integridade dos dados e retornos esperados via **gRPC**.

---

## 2. Escopo

Serviços cobertos nos testes:

* `CreateProduto`
* `GetProduto`
* `UpdateProduto`
* `DeleteProduto`
* `ListProdutos`

---

## 3. Ambiente de Testes

* **Banco de Dados:** MongoDB
* **Interface de comunicação:** gRPC
* **Porta do serviço:** `50051`

---

## 4. Pré-condições

* MongoDB acessível e limpo antes dos testes.
* Servidor gRPC ativo.
* Cliente de teste configurado para consumir os métodos gRPC.

---

## 5. Casos de Teste

### 5.1 CreateProduto

**Objetivo:** Validar criação de um produto.

**Entrada válida:**

```json
{
  "nomeProduto": "Notebook",
  "descricao": "Ultrabook 16GB RAM",
  "qtdEstoque": 10,
  "preco": 7500,
  "categoria": "Eletrônicos"
}
```

**Saída esperada:**

* Produto criado com `id` gerado automaticamente.
* Campos `dataCriacao` e `dataAtualizacao` preenchidos.

**Partições de equivalência:**

* Todos os campos válidos → sucesso.
* Campo obrigatório ausente (`nomeProduto`, `preco`, etc.) → erro.
* Tipo inválido (`preco: "abc"`) → erro.

---

### 5.2 GetProduto

**Objetivo:** Buscar produto pelo `id`.

**Cenários:**

* ID existente → retorna produto.
* ID inexistente → retorna objeto vazio `{}`.
* ID inválido (`"123"`) → erro de formato.

---

### 5.3 UpdateProduto

**Objetivo:** Atualizar um produto existente.

**Entrada válida:**

```json
{
  "id": "<id_existente>",
  "nomeProduto": "Notebook Gamer",
  "qtdEstoque": 8
}
```

**Saída esperada:**

* Produto atualizado com `dataAtualizacao` alterada.
* Campos não enviados permanecem inalterados.

**Cenários:**

* Atualização bem-sucedida.
* ID inexistente → `{}`.
* Campos inválidos (`preco: -100`) → erro.

---

### 5.4 DeleteProduto

**Objetivo:** Excluir produto por `id`.

**Cenários:**

* ID existente → `{ "success": true }`
* ID inexistente → `{ "success": false }`
* ID inválido → erro

---

### 5.5 ListProdutos

**Objetivo:** Listar todos os produtos cadastrados.

**Saída esperada:**

* Array com todos os produtos cadastrados.
* Lista vazia se não houver registros.

---

## 6. Critérios de Aceitação

* Todos os métodos devem responder conforme o contrato definido em `products.proto`.
* Nenhum campo obrigatório pode ser nulo ou ausente.
* O campo `dataAtualizacao` deve mudar sempre que um produto for alterado.
* Exclusões devem refletir imediatamente na listagem.

---
