# Entendendo a Importância da Modelagem e Segurança na Construção de APIs

## 📌 Sobre o desafio

Este repositório foi criado como parte do desafio do curso **Entendendo
a Importância da Modelagem e Segurança na Construção de APIs**.

Como já possuo conhecimentos sobre criação e consumo de APIs, o objetivo
principal deste estudo é aprofundar os conhecimentos relacionados à
**segurança de APIs**, entendendo vulnerabilidades, formas de ataque e
boas práticas para proteção de endpoints, dados e serviços.

O foco não é apenas fazer uma API funcionar, mas entender como
construí-la de maneira **segura, previsível e preparada para ambientes
de produção**.

------------------------------------------------------------------------

## 🎯 Objetivos

-   Aprofundar conhecimentos em segurança de APIs;
-   Entender os principais riscos apresentados pelo OWASP;
-   Diferenciar autenticação de autorização;
-   Evitar exposição indevida de dados;
-   Proteger endpoints contra abuso;
-   Entender rate limiting e ataques de força bruta;
-   Aplicar boas práticas de versionamento;
-   Conhecer ferramentas para testes de segurança;
-   Incorporar segurança desde a modelagem da API.

------------------------------------------------------------------------

## 🔐 Segurança de APIs

Uma API não deve depender de seus endpoints permanecerem escondidos.

Mesmo que uma aplicação possua apenas uma interface web, um usuário pode
inspecionar as requisições pelo navegador, identificar endpoints e
reproduzi-los utilizando ferramentas como:

``` text
Postman
cURL
Burp Suite
OWASP ZAP
```

Por isso, devemos assumir que um possível atacante conhece:

``` text
GET    /api/users
GET    /api/users/{id}
POST   /api/users
PUT    /api/users/{id}
DELETE /api/users/{id}
```

A segurança deve estar nas regras implementadas no servidor.

> Conhecer um endpoint não deve ser suficiente para conseguir acessar
> dados ou executar uma operação sem autorização.

------------------------------------------------------------------------

## 🔑 Autenticação x Autorização

Embora estejam relacionadas, autenticação e autorização resolvem
problemas diferentes.

### Autenticação

Responde à pergunta:

> **Quem está realizando a requisição?**

Algumas tecnologias utilizadas:

-   Bearer Token;
-   API Key;
-   JWT;
-   OAuth 2.0;
-   OpenID Connect;
-   sessões;
-   mTLS.

Exemplo:

``` http
Authorization: Bearer <token>
```

### Autorização

Responde à pergunta:

> **Esse usuário pode executar esta operação neste recurso?**

Imagine:

``` http
GET /api/users/10
```

Um usuário autenticado não deve automaticamente possuir acesso ao
usuário `10`.

A aplicação precisa verificar se aquele usuário possui permissão para
acessar o recurso solicitado.

------------------------------------------------------------------------

## 🚨 BOLA / IDOR

Um dos problemas mais importantes em APIs é o acesso indevido a objetos
através da manipulação de identificadores.

Exemplo:

``` http
GET /api/reservas/100
```

O usuário altera manualmente:

``` http
GET /api/reservas/101
```

Se a aplicação retornar uma reserva pertencente a outra pessoa sem
verificar a autorização, existe uma vulnerabilidade.

### ❌ Implementação conceitualmente insegura

``` php
$reserva = Reserva::findOrFail($id);

return response()->json($reserva);
```

Encontrar o registro não significa que o usuário autenticado pode
acessá-lo.

### ✅ Implementação com autorização

Em Laravel, uma alternativa é utilizar **Policies**:

``` php
$this->authorize('view', $reserva);
```

A aplicação deve validar autorização no servidor para cada recurso
protegido.

------------------------------------------------------------------------

## 🛡️ OWASP API Security

O **OWASP API Security Project** mantém referências sobre
vulnerabilidades específicas de APIs.

O OWASP API Security Top 10 é um excelente ponto de partida para estudar
segurança aplicada.

Entre os temas abordados estão:

-   Broken Object Level Authorization;
-   Broken Authentication;
-   Broken Object Property Level Authorization;
-   Unrestricted Resource Consumption;
-   Broken Function Level Authorization;
-   Unrestricted Access to Sensitive Business Flows;
-   Server Side Request Forgery (SSRF);
-   Security Misconfiguration;
-   Improper Inventory Management;
-   Unsafe Consumption of APIs.

📚 Referência:

[OWASP API Security Project](https://owasp.org/www-project-api-security)

------------------------------------------------------------------------

## 📦 Exposição excessiva de dados

Uma API deve retornar somente os dados necessários.

### ❌ Evitar

``` json
{
  "id": 10,
  "nome": "João",
  "email": "joao@example.com",
  "password": "...",
  "remember_token": "...",
  "internal_id": "...",
  "secret": "..."
}
```

Mesmo que o frontend não utilize determinados campos, eles continuam
disponíveis na resposta HTTP.

### Laravel

Em Laravel, é possível controlar respostas utilizando:

-   API Resources;
-   DTOs;
-   `$hidden`;
-   serializers;
-   transformação explícita dos dados.

Exemplo:

``` php
return [
    'id' => $user->id,
    'name' => $user->name,
];
```

O ideal é definir explicitamente o contrato de saída da API.

------------------------------------------------------------------------

## 📝 Mass Assignment

Outro cuidado importante é impedir que clientes alterem propriedades que
não deveriam controlar.

Considere:

``` json
{
  "name": "João",
  "email": "joao@example.com",
  "is_admin": true
}
```

Se todos os campos recebidos forem enviados diretamente para o Model, um
cliente pode tentar modificar propriedades internas.

### ❌ Evitar

``` php
User::create($request->all());
```

### ✅ Preferir dados validados

``` php
$data = $request->validated();

User::create($data);
```

Além disso, Models Laravel devem possuir corretamente configurados:

``` php
$fillable
```

ou

``` php
$guarded
```

------------------------------------------------------------------------

## ⚡ Rate Limiting

Rate limiting restringe a quantidade de requisições realizadas em
determinado período.

Exemplo:

``` text
60 requisições por minuto
```

Quando o limite é excedido:

``` http
HTTP/1.1 429 Too Many Requests
```

Isso ajuda a proteger a aplicação contra:

-   abuso;
-   força bruta;
-   scraping excessivo;
-   consumo exagerado de recursos;
-   sobrecarga de serviços.

Endpoints especialmente sensíveis incluem:

``` text
POST /login
POST /forgot-password
POST /reset-password
POST /verify-code
POST /token
```

------------------------------------------------------------------------

## 🔨 Ataques de força bruta

Um atacante pode realizar milhares de tentativas contra endpoints de
autenticação.

Exemplo:

``` text
POST /login
```

Tentativas:

``` text
usuario@email.com : 123456
usuario@email.com : password
usuario@email.com : admin
...
```

Algumas medidas de proteção:

-   rate limiting;
-   MFA;
-   bloqueios temporários;
-   monitoramento;
-   CAPTCHA quando apropriado;
-   políticas adequadas de autenticação;
-   alertas de comportamento suspeito.

------------------------------------------------------------------------

## 🎫 Segurança de Tokens

Tokens devem ser tratados como credenciais.

Boas práticas incluem:

``` text
✔ Não armazenar tokens em repositórios Git
✔ Utilizar HTTPS
✔ Definir permissões/scopes
✔ Permitir revogação
✔ Utilizar expiração quando apropriado
✔ Rotacionar credenciais
✔ Não registrar tokens completos em logs
✔ Utilizar variáveis de ambiente ou cofres de segredo
```

Um token comprometido pode permitir que outra pessoa execute operações
com as permissões associadas a ele.

------------------------------------------------------------------------

## 🌐 HTTPS

APIs em produção devem utilizar HTTPS.

Sem TLS, informações podem ser expostas durante a comunicação.

Isso inclui:

``` text
tokens
cookies
credenciais
dados pessoais
payloads
```

HTTPS protege os dados **em trânsito**, mas não substitui autenticação,
autorização ou validação.

------------------------------------------------------------------------

## 🌍 CORS não é autenticação

CORS controla quais origens podem realizar determinadas requisições
através de navegadores.

Por exemplo:

``` http
Access-Control-Allow-Origin: https://app.exemplo.com
```

Porém, CORS **não impede que alguém faça uma requisição diretamente
utilizando**:

``` bash
curl https://api.exemplo.com/api/users
```

Portanto:

> CORS não deve ser utilizado como mecanismo de autenticação ou
> autorização.

------------------------------------------------------------------------

## 🧪 Pentest em APIs

Pentest busca identificar vulnerabilidades através de testes
controlados.

Alguns pontos importantes:

``` text
Autenticação
Autorização
Manipulação de IDs
Manipulação de parâmetros
Rate limiting
Validação de entrada
Exposição de dados
Tokens
Headers
Uploads
Endpoints antigos
Configurações incorretas
```

Ferramentas úteis:

-   Postman;
-   Burp Suite;
-   OWASP ZAP;
-   cURL.

Os testes devem ser realizados somente em ambientes e sistemas para os
quais exista autorização.

------------------------------------------------------------------------

## 🗃️ Logs e informações sensíveis

Logs são importantes para auditoria e investigação de incidentes.

Porém, não devem armazenar segredos desnecessariamente.

### ❌ Evitar

``` text
Authorization: Bearer eyJhbGciOi...
password=123456
token=abc123
```

### ✅ Preferir

``` text
user_id=42
endpoint=/api/reservas
status=403
ip=...
timestamp=...
```

Quando dados sensíveis forem necessários para algum processo de
observabilidade, devem existir políticas adequadas de mascaramento,
retenção e acesso.

------------------------------------------------------------------------

## 🇧🇷 LGPD e APIs

APIs frequentemente processam dados pessoais.

Alguns princípios importantes relacionados à proteção desses dados:

-   retornar apenas informações necessárias;
-   restringir acesso;
-   utilizar HTTPS;
-   proteger credenciais;
-   evitar dados pessoais desnecessários em logs;
-   monitorar acessos;
-   definir políticas de retenção;
-   possuir procedimentos para incidentes.

Segurança de API também faz parte da proteção dos dados tratados pelo
sistema.

------------------------------------------------------------------------

## 🔄 Versionamento

APIs evoluem.

Uma estratégia comum é utilizar versões:

``` text
/api/v1/reservas
/api/v2/reservas
```

Uma migração pode seguir:

``` text
1. Criar a v2
2. Manter temporariamente a v1
3. Documentar alterações
4. Comunicar consumidores
5. Monitorar utilização da v1
6. Definir período de descontinuação
7. Remover a versão antiga
```

Endpoints antigos esquecidos também podem representar riscos de
segurança.

------------------------------------------------------------------------

## 🚪 API Gateway e Microservices

Em arquiteturas distribuídas:

``` text
                    ┌── Users Service
Cliente → API Gateway ── Orders Service
                    └── Payment Service
```

O gateway pode implementar:

``` text
Autenticação
Rate Limiting
Roteamento
Logs
Observabilidade
Políticas
Validação
```

Porém, os microsserviços também devem aplicar suas próprias regras de
autorização e segurança quando necessário.

Isso cria uma estratégia de **defesa em profundidade**.

------------------------------------------------------------------------

## 🏗️ Modelagem também é segurança

Uma boa modelagem reduz oportunidades de erros de segurança.

Antes de criar um endpoint, algumas perguntas importantes são:

``` text
Quem pode acessar?

Quem pode alterar?

Quais campos podem ser enviados?

Quais campos podem ser retornados?

Existe informação sensível?

Precisa de rate limiting?

Como será auditado?

O endpoint precisa realmente existir?
```

Segurança começa antes da implementação.

------------------------------------------------------------------------

## 🔎 Checklist de segurança

Antes de publicar uma API:

``` text
[ ] HTTPS configurado
[ ] Autenticação implementada
[ ] Autorização por recurso
[ ] Inputs validados
[ ] Outputs controlados
[ ] Rate limiting configurado
[ ] Tokens protegidos
[ ] Segredos fora do código
[ ] Logs sem informações sensíveis
[ ] CORS configurado corretamente
[ ] Dependências atualizadas
[ ] Endpoints documentados
[ ] APIs antigas removidas
[ ] Tratamento de erros adequado
[ ] Testes de segurança realizados
```

------------------------------------------------------------------------

## 🧠 Principal aprendizado

Um dos conceitos mais importantes deste estudo é:

> **Uma API não é segura porque seus endpoints estão escondidos.**

Um usuário pode abrir o DevTools do navegador e visualizar:

``` text
Request URL
Request Method
Headers
Payload
Response
```

Também pode reproduzir e modificar essas requisições fora da aplicação.

Portanto, uma API deve continuar segura mesmo quando alguém conhece seus
endpoints e tenta manipulá-los.

A segurança precisa existir no **backend** através de autenticação,
autorização, validação, limitação de recursos, monitoramento e boas
práticas de desenvolvimento.

------------------------------------------------------------------------

## 📚 Referências

### OWASP

-   [OWASP](https://owasp.org)

### API Security

-   [API Security Articles](https://apisecurity.io)

### OpenAPI

-   [OpenAPI Initiative](https://www.openapis.org)
-   [Swagger](https://swagger.io)
-   [Swagger OpenAPI Specification](https://swagger.io/specification)

### Testes

-   [Postman](https://www.postman.com/postman/workspace/postman-team-collections/overview)

### Desenvolvimento

-   [Java](https://www.java.com/pt-BR/download/help/develop.html)
-   [Spring Boot](https://spring.io/projects/spring-boot)

### API Management

-   [Google Cloud -
    Apigee](https://cloud.google.com/training/apigee/?hl=pt)

------------------------------------------------------------------------
