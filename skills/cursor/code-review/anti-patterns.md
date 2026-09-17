# Anti-Padrões — Catálogo de Detecção

Este documento lista anti-padrões que a skill `code-review` deve detectar ativamente durante a revisão. Use como referência para aprofundar a análise quando necessário.

**Os conceitos são agnósticos a linguagem** — aplicam-se a qualquer stack (PHP, JavaScript, Python, Go, Ruby, etc.). Os exemplos mostram diferentes linguagens para ilustrar que os anti-padrões são universais.

---

## 1. Segredo Hardcoded

**O que é:**  
Credenciais, tokens, chaves de API ou senhas inseridos diretamente no código-fonte.

**Por que bloqueia:**  
Expõe segredos em versionamento; vazamento irreversível.

**Como detectar:**
- Strings que parecem tokens: `sk_live_`, `AKIA`, `ghp_`, `Bearer`, `password=`, `apiKey=`
- Padrões de hash/base64 longos em variáveis de configuração
- URLs com credenciais: `https://user:pass@host`

**Severidade:** BLOQUEADOR

**Exemplo (JavaScript):**

```javascript
// ❌ Bloqueador
const API_KEY = "sk_live_REDACTED_EXAMPLE";

// ✅ Correto
const API_KEY = process.env.API_KEY;
```

**Exemplo (PHP):**

```php
// ❌ Bloqueador
define('API_KEY', 'sk_live_REDACTED_EXAMPLE');

// ✅ Correto
define('API_KEY', getenv('API_KEY'));
// ou
$apiKey = $_ENV['API_KEY'];
```

---

## 2. Silent Fail

**O que é:**  
Captura de erro que não propaga, não registra ou apenas faz `console.log`.

**Por que bloqueia:**  
Falhas ficam invisíveis em produção; debugging impossível.

**Como detectar:**
- `catch` vazio ou com corpo só de `console.log`/`console.error`
- `try/catch` que não re-lança (`throw`) nem registra em log estruturado
- Callback de erro ignorado (ex.: `promise.catch(() => {})`)

**Severidade:** BLOQUEADOR (caminho crítico) | AVISO (fluxo secundário)

**Exemplo (Python):**

```python
# ❌ Bloqueador
try:
    process_payment(user)
except Exception:
    pass  # silent fail

# ❌ Bloqueador
try:
    process_payment(user)
except Exception as e:
    print(e)  # não propagado, não registrado

# ✅ Correto
try:
    process_payment(user)
except PaymentError as e:
    logger.error(f"Payment failed for user {user.id}", exc_info=True)
    raise
```

**Exemplo (PHP):**

```php
// ❌ Bloqueador
try {
    processPayment($user);
} catch (Exception $e) {
    // silent fail - catch vazio
}

// ❌ Bloqueador
try {
    processPayment($user);
} catch (Exception $e) {
    error_log($e->getMessage()); // não propagado
}

// ✅ Correto
try {
    processPayment($user);
} catch (PaymentException $e) {
    $logger->error('Payment failed', ['user_id' => $user->id, 'error' => $e->getMessage()]);
    throw $e;
}
```

---

## 3. God Function

**O que é:**  
Função com múltiplas responsabilidades, muitas linhas (>50), ou que faz "tudo".

**Por que importa:**  
Dificulta manutenção, teste e compreensão; viola SRP (Single Responsibility Principle).

**Como detectar:**
- Função com mais de 50 linhas (heurística)
- Múltiplos níveis de indentação (>4)
- Nome genérico (`process`, `handle`, `doStuff`)
- Faz validação + lógica de negócio + persistência + notificação no mesmo escopo

**Severidade:** AVISO (sugerir refatoração)

**Exemplo (JavaScript):**

```javascript
// ❌ Aviso: God Function
function handleUserRegistration(data) {
  // validação
  if (!data.email || !data.password) throw new Error("Invalid");
  
  // hash de senha
  const hashed = bcrypt.hashSync(data.password, 10);
  
  // persistência
  const user = db.users.create({ email: data.email, password: hashed });
  
  // envio de email
  sendEmail(user.email, "Welcome!");
  
  // log
  console.log("User registered:", user.id);
  
  // notificação para Slack
  notifySlack(`New user: ${user.email}`);
  
  return user;
}

// ✅ Correto: responsabilidades separadas
function registerUser(data) {
  validateRegistrationData(data);
  const hashedPassword = hashPassword(data.password);
  const user = createUser(data.email, hashedPassword);
  scheduleWelcomeEmail(user);
  logUserRegistration(user);
  return user;
}
```

**Exemplo (PHP):**

```php
// ❌ Aviso: God Function
function handleUserRegistration(array $data): User {
    // validação
    if (empty($data['email']) || empty($data['password'])) {
        throw new InvalidArgumentException('Invalid data');
    }
    
    // hash de senha
    $hashed = password_hash($data['password'], PASSWORD_BCRYPT);
    
    // persistência
    $user = $this->db->table('users')->insert([
        'email' => $data['email'],
        'password' => $hashed
    ]);
    
    // envio de email
    $this->sendEmail($user->email, 'Welcome!');
    
    // log
    error_log("User registered: {$user->id}");
    
    // notificação para Slack
    $this->notifySlack("New user: {$user->email}");
    
    return $user;
}

// ✅ Correto: responsabilidades separadas
function registerUser(array $data): User {
    $this->validateRegistrationData($data);
    $hashedPassword = $this->hashPassword($data['password']);
    $user = $this->createUser($data['email'], $hashedPassword);
    $this->scheduleWelcomeEmail($user);
    $this->logUserRegistration($user);
    return $user;
}
```

---

## 4. Teste que Testa o Mock

**O que é:**  
Teste que valida apenas se um stub/mock foi chamado, sem assertar comportamento ou resultado.

**Por que importa:**  
Não testa a regra de negócio real; passa mesmo com implementação incorreta.

**Como detectar:**
- Assert que só verifica `mock.called`, `stub.calledWith` sem validar output/efeito
- Teste sem asserção sobre estado final ou retorno
- Mock que substitui lógica de negócio inteira

**Severidade:** AVISO

**Exemplo:**

```javascript
// ❌ Aviso: testa o mock, não o comportamento
it("should call payment service", () => {
  const paymentService = sinon.stub();
  processOrder({ paymentService });
  
  expect(paymentService.called).toBe(true); // só verifica chamada
});

// ✅ Correto: testa o comportamento
it("should mark order as paid after successful payment", () => {
  const paymentService = sinon.stub().resolves({ status: "success" });
  const order = processOrder({ paymentService });
  
  expect(order.status).toBe("paid"); // valida efeito real
});
```

---

## 5. Feature Flag Ausente

**O que é:**  
Discovery marcou `Feature flag: Sim`, mas a implementação não contém controle condicional.

**Por que bloqueia:**  
Incapacidade de ativar/desativar a feature; rollout sem controle.

**Como detectar:**
- Discovery com `Feature flag: Sim` em **Premissas**
- Diff sem referência a flag (ex.: `isFeatureEnabled`, `featureFlags.newCheckout`, etc.)

**Severidade:** BLOQUEADOR

**Exemplo (TypeScript):**

```typescript
// ❌ Bloqueador: sem feature flag quando Discovery exigia
function checkout() {
  return newCheckoutFlow(); // direto, sem controle
}

// ✅ Correto
function checkout() {
  if (featureFlags.isEnabled("new-checkout")) {
    return newCheckoutFlow();
  }
  return legacyCheckoutFlow();
}
```

**Exemplo (PHP):**

```php
// ❌ Bloqueador: sem feature flag quando Discovery exigia
function checkout(): CheckoutResult {
    return $this->newCheckoutFlow(); // direto, sem controle
}

// ✅ Correto
function checkout(): CheckoutResult {
    if ($this->featureFlags->isEnabled('new-checkout')) {
        return $this->newCheckoutFlow();
    }
    return $this->legacyCheckoutFlow();
}
```

---

## 6. Log de Dados Sensíveis

**O que é:**  
Log que expõe PII (email, CPF, telefone), tokens, senhas ou dados financeiros.

**Por que bloqueia:**  
Violação de LGPD/GDPR; risco de vazamento em ferramentas de observabilidade.

**Como detectar:**
- Log contendo campos: `email`, `cpf`, `password`, `token`, `credit_card`, `ssn`, `phone`
- Serialização de objeto completo sem mascarar campos sensíveis

**Severidade:** BLOQUEADOR

**Exemplo (Ruby):**

```ruby
# ❌ Bloqueador
logger.info("User login: #{user.email}, password: #{user.password}")

# ❌ Bloqueador
logger.info("Payment data: #{payment.to_json}") # inclui credit_card

# ✅ Correto
logger.info("User login", user_id: user.id) # sem PII

# ✅ Correto (mascaramento)
logger.info("Payment data", card_last_4: payment.card_last_4, amount: payment.amount)
```

**Exemplo (PHP):**

```php
// ❌ Bloqueador
$logger->info("User login: {$user->email}, password: {$user->password}");

// ❌ Bloqueador
$logger->info("Payment data: " . json_encode($payment)); // inclui credit_card

// ✅ Correto
$logger->info("User login", ['user_id' => $user->id]); // sem PII

// ✅ Correto (mascaramento)
$logger->info("Payment data", [
    'card_last_4' => $payment->cardLast4,
    'amount' => $payment->amount
]);
```

---

## 7. TODO/FIXME no Caminho Crítico

**O que é:**  
Comentário `TODO`, `FIXME`, `HACK` ou `XXX` em código de produção (fora de testes).

**Por que importa:**  
Indica trabalho inacabado; pode ser bug latente.

**Como detectar:**
- Grep por `TODO`, `FIXME`, `HACK`, `XXX`, `TEMP` em arquivos de produção
- Comentários pendentes em fluxo principal (não em testes ou scripts auxiliares)

**Severidade:** AVISO (caminho crítico) | SUGESTÃO (fluxo secundário)

**Exemplo:**

```go
// ❌ Aviso
func ProcessPayment(order Order) error {
    // TODO: add retry logic
    return gateway.Charge(order.Amount)
}

// ✅ Correto: implementado ou documentado em issue
func ProcessPayment(order Order) error {
    return retryWithBackoff(func() error {
        return gateway.Charge(order.Amount)
    })
}
```

---

## 8. Dependência Não Declarada

**O que é:**  
Import de biblioteca que não está no manifesto de dependências do projeto.

**Por que bloqueia:**  
Build quebra em outros ambientes; dependência fantasma.

**Como detectar:**
- Cruzar imports/requires no diff com `package.json`, `requirements.txt`, `Gemfile`, `go.mod`, `pom.xml`
- Import de pacote não listado

**Severidade:** BLOQUEADOR

**Exemplo (JavaScript/Node.js):**

```javascript
// ❌ Bloqueador (se lodash não está em package.json)
import _ from "lodash";

// ✅ Correto: lodash listado em package.json dependencies
{
  "dependencies": {
    "lodash": "^4.17.21"
  }
}
```

**Exemplo (PHP/Composer):**

```php
// ❌ Bloqueador (se guzzlehttp/guzzle não está em composer.json)
use GuzzleHttp\Client;

// ✅ Correto: guzzlehttp/guzzle listado em composer.json
{
  "require": {
    "guzzlehttp/guzzle": "^7.0"
  }
}
```

---

## 9. PR Gigante Sem Contexto

**O que é:**  
Diff com mais de 500 linhas alteradas sem Discovery ou Plan referenciado.

**Por que importa:**  
Revisão superficial; escopo unclear; alto risco de regressão.

**Como detectar:**
- `git diff --stat` com total de linhas > 500
- Sem Discovery em `docs/discovery/`
- Sem Plan na conversa

**Severidade:** AVISO (recomenda documentação retroativa)

**Exemplo:**

```
# ❌ Aviso
$ git diff --stat
 45 files changed, 1820 insertions(+), 340 deletions(-)

(sem Discovery ou Plan)

# ✅ Correto
$ git diff --stat
 45 files changed, 1820 insertions(+), 340 deletions(-)

Discovery: docs/discovery/0003-migrar-auth-para-jwt.md
Plan: discutido na conversa (disponível no contexto)
```

---

## 10. Validação de Input Ausente

**O que é:**  
Endpoint de API, handler de evento ou formulário que não valida entrada antes de processar.

**Por que bloqueia:**  
Vulnerável a injection (SQL, XSS, command), crashes por tipo inesperado, SSRF.

**Como detectar:**
- Controller/handler que recebe `request.body` ou `event.data` e usa diretamente em query, comando ou lógica
- Sem validação de schema (Zod, Joi, Pydantic, etc.)
- Sem sanitização de strings para query/comando

**Severidade:** BLOQUEADOR (fronteira externa) | AVISO (fronteira interna)

**Exemplo (Python):**

```python
# ❌ Bloqueador: SQL injection
@app.route("/users/<user_id>")
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"  # sem sanitização
    return db.execute(query)

# ✅ Correto
@app.route("/users/<int:user_id>")
def get_user(user_id: int):
    if not isinstance(user_id, int) or user_id <= 0:
        abort(400)
    return db.query(User).filter_by(id=user_id).first()
```

**Exemplo (PHP):**

```php
// ❌ Bloqueador: SQL injection
function getUser($userId) {
    $query = "SELECT * FROM users WHERE id = $userId"; // sem sanitização
    return $this->db->query($query);
}

// ✅ Correto (prepared statement)
function getUser(int $userId): ?User {
    if ($userId <= 0) {
        throw new InvalidArgumentException('Invalid user ID');
    }
    
    $stmt = $this->db->prepare("SELECT * FROM users WHERE id = ?");
    $stmt->execute([$userId]);
    return $stmt->fetch();
}

// ✅ Correto (ORM)
function getUser(int $userId): ?User {
    return User::find($userId);
}
```

---

## 11. Rollback Impossível

**O que é:**  
Migração de banco, mudança de schema ou deploy que não permite rollback sem work manual.

**Por que importa:**  
Incidente em produção vira downtime; sem plano de reversão.

**Como detectar:**
- Migração que altera schema destrutivamente (drop column, rename sem compatibilidade)
- Deploy que remove endpoint antes de desativar consumidores
- Feature sem feature flag (rollback exige redeploy)

**Severidade:** AVISO

**Exemplo:**

```sql
-- ❌ Aviso: rollback impossível sem restore
ALTER TABLE users DROP COLUMN legacy_email;

-- ✅ Correto: compatível com rollback (deploy 1: parar de usar; deploy 2: remover)
-- Deploy 1: código para de ler legacy_email
-- Deploy 2 (após validação):
ALTER TABLE users DROP COLUMN legacy_email;
```

---

## 12. Observabilidade Ausente

**O que é:**  
Plan definiu métricas/logs/alertas, mas a implementação não instrumentou.

**Por que bloqueia:**  
Impossível monitorar sucesso/falha da feature; debugging cego.

**Como detectar:**
- Cruzar seção **Observabilidade** do Plan com o diff
- Ausência de log estruturado em pontos críticos
- Ausência de métrica (contador, timer, gauge) quando Plan definiu

**Severidade:** BLOQUEADOR (Plan obrigatório) | AVISO (Plan opcional)

**Exemplo (JavaScript):**

```javascript
// Plan: "Métrica: taxa de sucesso de checkout (checkout.success / checkout.attempt)"

// ❌ Bloqueador: sem instrumentação
function checkout(cart) {
  const order = processOrder(cart);
  return order;
}

// ✅ Correto
function checkout(cart) {
  metrics.increment("checkout.attempt");
  try {
    const order = processOrder(cart);
    metrics.increment("checkout.success");
    logger.info("Checkout completed", { order_id: order.id, amount: order.total });
    return order;
  } catch (error) {
    metrics.increment("checkout.failure");
    logger.error("Checkout failed", { cart_id: cart.id, error: error.message });
    throw error;
  }
}
```

**Exemplo (PHP):**

```php
// Plan: "Métrica: taxa de sucesso de checkout (checkout.success / checkout.attempt)"

// ❌ Bloqueador: sem instrumentação
function checkout(Cart $cart): Order {
    $order = $this->processOrder($cart);
    return $order;
}

// ✅ Correto
function checkout(Cart $cart): Order {
    $this->metrics->increment('checkout.attempt');
    
    try {
        $order = $this->processOrder($cart);
        $this->metrics->increment('checkout.success');
        $this->logger->info('Checkout completed', [
            'order_id' => $order->id,
            'amount' => $order->total
        ]);
        return $order;
    } catch (Exception $e) {
        $this->metrics->increment('checkout.failure');
        $this->logger->error('Checkout failed', [
            'cart_id' => $cart->id,
            'error' => $e->getMessage()
        ]);
        throw $e;
    }
}
```

---

## Resumo de Severidades por Anti-Padrão

| Anti-Padrão | Severidade Padrão |
|-------------|-------------------|
| Segredo hardcoded | BLOQUEADOR |
| Silent fail (caminho crítico) | BLOQUEADOR |
| Silent fail (secundário) | AVISO |
| God function | AVISO |
| Teste que testa o mock | AVISO |
| Feature flag ausente | BLOQUEADOR |
| Log de dados sensíveis | BLOQUEADOR |
| TODO/FIXME (caminho crítico) | AVISO |
| TODO/FIXME (secundário) | SUGESTÃO |
| Dependência não declarada | BLOQUEADOR |
| PR gigante sem contexto | AVISO |
| Validação de input ausente (fronteira externa) | BLOQUEADOR |
| Validação de input ausente (interna) | AVISO |
| Rollback impossível | AVISO |
| Observabilidade ausente (Plan obrigatório) | BLOQUEADOR |
| Observabilidade ausente (Plan opcional) | AVISO |

---

## Uso

Este catálogo é lido pela skill `code-review` quando necessário aprofundar a análise. Não precisa ser carregado inteiro no contexto; use progressive disclosure (link do SKILL.md aponta para cá).
