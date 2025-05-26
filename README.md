# 📋 Relatório da Análise

---

## 1. 🛣️ Estrutura e Organização das Rotas

- Utilizar o prefixo `auth` nas rotas `/login` e `/register` em vez de `users`, por maior clareza semântica.
- A rota `/login` transmite credenciais em Base64 via header, o que é inseguro. Recomendado enviar via body da requisição.
- Implementar **rate limiting** com middleware `throttle` para prevenir ataques de força bruta.
- Utilizar `Route::resource` para reduzir linhas de código e agrupar rotas de CRUD.
- Evitar aninhamento excessivo de rotas e aplicar **model binding automático**:
  ```php
  Route::get('users/{user}/account', [AccountController::class, 'show']);
  Route::post('users/{user}/cards', [CardController::class, 'store']);
  ```

---

## 2. 🏢 Registro de Empresa e Usuário

- A função `UserController::register()` não invoca `$request->validated()`.
- O use case `CreateFirstUser` viola o princípio de responsabilidade única.
- O tipo do usuário deveria estar definido no `CreateFirstUserParams`.
- Ausência de rollback em falhas de persistência.
- Status HTTP de criação deve ser `201 Created`, não `200 OK`.

---

## 3. 🔐 Login dos Usuários

- O comentário da função `Login::handler()` está incorreto.
- O Auth do Laravel pode ser usado em vez de repository para recuperar o usuário autenticado.
- Falha de padronização na nomeclatura das variaveis na importação `create_token`.

---

## 4. 🛠️ Atualização de Dados da Empresa

- A função `CompanyController::update()` contém lógica que deveria estar em um Use Case.
- Os dados da requisição não são validados.

---

## 5. 👥 Listagem e Obtenção de Usuários

### Listagem
- A função `Retrieve::handle()` usa `whereRaw` com dados do request — risco de **SQL Injection**.
- `UserController::index()` não trata os dados de resposta — risco de exposição de dados sensíveis.

### Obtenção de Usuário
- Classe `UseCases\show` tem nome com letra minúscula (violando convenção).
- Uso de variáveis sem nomes descritivos (`$a`, `$b`, `$c`).

---

## 6. ➕ Criação de Usuário

- `CreateRequest` não é validado via `$request->validated()`.
- Status HTTP incorreto (`200 OK` ao invés de `201 Created`).

---

## 7. 💳 Manipulação de Contas Bancárias

### Criação de Conta
- `AccountController::register()` não retorna status correto nem os dados criados.
- `Register::findUser()` realiza validações indevidas no contexto do Use Case.

### Obtenção de Conta
- `Find::requestUrl()` deve ser `"accounts/{$this->externalId}"`.
- `Find::findAccountData()` contém lógica de validação que viola responsabilidades.

### Ativação e Bloqueio de Conta
- `UpdateStatus::requestUrl()` incorreto. Deve retornar: `"accounts/{$this->externalId}/status/{$this->newStatus}"`.
- Falta de rollback em erros durante chamadas à API externa.

---

## 8. 🧾 Criação de Cartão para Usuário

- Ausência de uma classe `Request` para validação dos dados de entrada.

---

## 9. ✅ Testes

- Alguns testes esperam códigos de status incorretos para requisições não autorizadas.

---

## 10. ✨ Sugestões Arquiteturais

- Implementar o uso de enums para guardar dados estaticos, evitando o uso de codigos "chumbados" `('MANEGER', 'ACTIVE', 'BLOCK')`
- Criar *domains* específicos para encapsular regras e validações de contas bancárias.
- Adicionar cache nas rotinas de consulta ao banco de dados, melhorando a desempenho do sistema
