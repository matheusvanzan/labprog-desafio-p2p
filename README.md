# Desafio: descubra os segredos da rede

Bem-vindos ao desafio! A turma foi dividida em **seis grupos** e cada grupo
**inventa o seu próprio segredo**: uma frase que só o grupo conhece. Ninguém
conhece os segredos dos outros, e o
objetivo da aula é que, no fim, **todos os seis grupos conheçam todos os seis
segredos**, sem que ninguém digite, copie ou envie um segredo na mão.

Não existe servidor central. Cada grupo constrói **um nó** da rede: um servidor
**Java** que mostra os segredos que o nó já conhece e um robô **Python** que
sai procurando os outros nós na rede da sala e pedindo os segredos deles.
Um painel projetado pelo professor só **observa** o que está acontecendo.

O contrato HTTP é igual para todos, então o robô Python de um grupo funciona
com o servidor Java de **qualquer** outro grupo.

## Objetivo

Cada grupo entrega **dois programas independentes**, rodando na mesma máquina:

1. um **servidor Java com Spring Boot**, com uma rota que diz que o servidor
   está no ar e outra que devolve os segredos que o nó conhece;
2. um **script Python**, que procura os outros nós na rede, pega os segredos
   deles e avisa o painel do professor como está o seu progresso.

No fim, os seis nós devem ter descoberto uns aos outros e trocado segredos até
que cada um tenha os seis, tudo automático.

## Organização dos grupos

| Grupo | Segredo | Identificador |
| --- | --- | --- |
| 1 | segredo do grupo 1 | `secret-1` |
| 2 | segredo do grupo 2 | `secret-2` |
| 3 | segredo do grupo 3 | `secret-3` |
| 4 | segredo do grupo 4 | `secret-4` |
| 5 | segredo do grupo 5 | `secret-5` |
| 6 | segredo do grupo 6 | `secret-6` |

Cada grupo inventa uma **frase secreta** e a coloca no seu próprio script Python,
no formato do objeto `Secret` descrito mais abaixo. O `id` é sempre
`secret-` seguido do **número do seu grupo**. **Esse é o único segredo que o seu
grupo conhece no início.** Não contem a frase para os outros grupos: ela só pode
chegar a eles pela rede.

## Como a atividade funciona

1. O professor entrega o **endereço do painel**, algo como
   `http://IP-DO-PROFESSOR:8000`, e a lista de grupos.
2. **Primeiro passo: criem o servidor Java e deixem ele respondendo.** Ele é a
   "porta de entrada" do seu nó. Com o servidor no ar, o seu grupo aparece no
   painel como **Servidor Java: online**. O painel procura os servidores sozinho
   na rede, então **vocês não precisam informar o seu IP ao professor**.
3. Depois, construam e testem o script Python.
4. Todos os computadores precisam estar na **mesma rede do professor**. No
   momento combinado, os seis grupos deixam os seus programas rodando ao mesmo
   tempo.
5. Os nós se encontram e trocam segredos. O painel mostra o progresso em tempo
   real, de 0 a 36.
6. A atividade termina quando o painel marca **36 de 36**. O professor valida
   ao vivo.

## Como o nó funciona

```
 Máquina do grupo
 ┌──────────────────────────────────────────────────┐
 │  Script Python                  Servidor Java    │
 │  procura nós, pede   escreve    mostra os        │
 │  os segredos e  ──── arquivo ──► segredos que    │
 │  avisa o painel      (sem HTTP)  o nó conhece    │
 └──────┬──────────────────────────────▲────────────┘
        │ GET /api/secrets              │ GET /api/secrets
        ▼                               │
  Java dos outros grupos          Python dos outros grupos
```

- O **Python** é quem sai procurando. Ele pergunta os segredos aos servidores
  Java dos outros grupos e guarda tudo o que descobriu.
- O **Java** só responde. Ele não procura ninguém, não chama nenhuma API e não
  decide nada: apenas mostra os segredos que o nó já conhece, **inclusive os
  que o Python descobriu**. É assim que um nó repassa os segredos que aprendeu:
  quem chegar nele depois já encontra tudo.
- Python e Java **não conversam por HTTP** entre si, nem por `localhost`. O
  Python precisa entregar ao Java o que descobriu por outro caminho (um arquivo,
  por exemplo). A escolha é do grupo e deve ser justificada em poucas linhas
  (veja "Entregáveis").

Como o Java repassa o que o Python descobriu, **você não precisa visitar todos os
nós da rede**. Se você encontra um nó que já conhece os segredos dos outros,
acabou: o objetivo é ter os seis, e não falar com todo mundo.

## Preparando o ambiente

Vocês precisam de **JDK 21** e do **VS Code** com o *Extension Pack for Java* e
o *Spring Boot Extension Pack*, como no desafio de migração, e de **Python 3.10
ou superior** com a biblioteca `requests` (`pip install requests`).

Criem o projeto Java pelo **Spring Initializr** (Gradle, Java 21, Jar, com a
dependência **Spring Web**) e rodem com `.\gradlew.bat bootRun` (Linux, macOS
ou WSL: `./gradlew bootRun`).

Para o servidor Java subir na porta do desafio, coloquem no
`application.properties`:

```properties
server.port=12000
```

## Contrato da API Java (igual para todos)

Todos os grupos implementam **exatamente** estas duas rotas. Os nomes dos campos
valem com maiúsculas e minúsculas.

### Onde cada coisa fica

| O quê | URL |
| --- | --- |
| API Java de cada nó (a que o Python procura nos outros grupos) | `http://IP-DO-NÓ:12000/api/secrets` |
| Envio de status ao painel (feito pelo Python) | `POST http://IP-DO-PROFESSOR:8000/api/status` |

A porta `12000` é a mesma para todos os grupos. O que muda de um nó para outro é
só o IP.

### Objeto `Secret`

| Campo | Tipo | Observações |
| --- | --- | --- |
| `id` | texto | `secret-N`, com `N` de 1 a 6 |
| `origin_group` | número inteiro | grupo de origem, de 1 a 6 (o mesmo `N` do `id`) |
| `content` | texto | o conteúdo do segredo |

```json
{
  "id": "secret-3",
  "origin_group": 3,
  "content": "a reunião secreta será na sala 305"
}
```

O `id` identifica o segredo: dois objetos com o mesmo `id` são o mesmo segredo.
O `content` deve ser guardado e repassado **exatamente como veio**, sem mudar
nada (nem espaço, nem maiúscula), porque o painel confere o seu hash.

### `GET /api/health`

Diz que o servidor Java está no ar. Ela **não depende do Python**: tem que
responder assim que o servidor subir, antes de o Python gravar qualquer coisa. É
com ela que o painel marca o seu grupo como **Servidor Java: online**.

Exemplo: `GET /api/health`. Retorna `200`:

```json
{
  "status": "ok",
  "service": "p2p-secrets",
  "group": 3
}
```

| Campo | Tipo | Observações |
| --- | --- | --- |
| `status` | texto | sempre `"ok"` |
| `service` | texto | sempre `"p2p-secrets"` |
| `group` | número inteiro | número do seu grupo, de 1 a 6 (escrevam no código do Java) |

### `GET /api/secrets`

Exemplo: `GET /api/secrets`. Retorna `200` com os segredos que o nó conhece
**agora**:

```json
{
  "service": "p2p-secrets",
  "group": 3,
  "secrets": [
    { "id": "secret-3", "origin_group": 3, "content": "a reunião secreta será na sala 305" },
    { "id": "secret-1", "origin_group": 1, "content": "a senha do wifi do laboratório é azul-42" }
  ]
}
```

| Campo | Tipo | Observações |
| --- | --- | --- |
| `service` | texto | sempre `"p2p-secrets"`: é assim que o Python reconhece que achou um nó do desafio |
| `group` | número inteiro | número do seu grupo |
| `secrets` | lista de `Secret` | todos os segredos que o nó conhece; sem repetir `id` |

Se o nó ainda não conhece nada, `secrets` vem vazio. Não há `POST`: quem
descobre os segredos é o Python, perguntando.

### Exemplo do arquivo `segredos.json`

É o arquivo que o Python grava e o Java lê. Ele tem **exatamente o mesmo
formato** que a rota devolve, então o Java pode devolver o conteúdo do arquivo
sem mexer em nada:

```json
{
  "service": "p2p-secrets",
  "group": 3,
  "secrets": [
    { "id": "secret-3", "origin_group": 3, "content": "a reunião secreta será na sala 305" },
    { "id": "secret-1", "origin_group": 1, "content": "a senha do wifi do laboratório é azul-42" }
  ]
}
```

Teste com o navegador ou com `curl`:

```bash
curl http://localhost:12000/api/secrets
```

## Painel do professor

O painel **observa** a rede. Ele **não** faz parte dela: não ajuda a descobrir
nós e não distribui segredos. Em tempo real, ele mostra:

- os seis grupos e se cada um está **ativo** ou **inativo**;
- quantos segredos cada grupo tem, de 0 a 6;
- o progresso global, de 0 a 36 (6 grupos x 6 segredos);
- quantos nós cada grupo já encontrou;
- o horário da última atualização de cada grupo.

O painel **nunca recebe o texto** dos segredos, e o professor também não os
conhece. Cada grupo informa o `id` e o **hash SHA-256** do `content` de cada
segredo que tem. O painel considera o hash que **o próprio grupo N informou** para
o `secret-N` como o correto, e só conta a cópia de outro grupo se o hash for
**igual**. Ou seja: o segredo precisa chegar sem nenhuma alteração. O professor
também pode consultar o `GET /api/secrets` do seu nó para conferir se o que você
informou é o que ele realmente guarda.

### `POST /api/status` (no painel)

Quem envia é o **script Python**, para `http://IP-DO-PROFESSOR:8000/api/status`,
a **cada 5 segundos**:

```json
{
  "group": 3,
  "timestamp": "2026-11-02T19:42:10Z",
  "discovered_nodes": 4,
  "secrets": [
    { "id": "secret-3", "sha256": "9c1e...a7f0" },
    { "id": "secret-1", "sha256": "41bd...03c2" }
  ]
}
```

| Campo | Tipo | Observações |
| --- | --- | --- |
| `group` | número inteiro | número do seu grupo, de 1 a 6 |
| `timestamp` | texto | data e hora UTC, formato ISO 8601 |
| `discovered_nodes` | número inteiro | quantos **outros** nós o seu script encontrou, de 0 a 5 |
| `secrets` | lista | um item por segredo que o seu nó **tem agora**, com `id` e `sha256` |

O hash é o SHA-256 dos bytes UTF-8 do `content`, em hexadecimal minúsculo:

```python
import hashlib
hashlib.sha256(content.encode("utf-8")).hexdigest()
```

Resposta `200`: `{ "status": "received", "validated": ["secret-3"], "rejected": [] }`.
`validated` são os `id` cujo hash conferiu e `rejected` os que não conferiram.
Use isso para depurar. Dados inválidos retornam `400` com
`{ "error": "invalid_report", "message": "..." }`.

Cada envio é um **retrato completo** do que o grupo tem agora. Um grupo aparece
como **inativo** se ficar mais de 20 segundos sem enviar. Se o painel estiver
fora do ar, **o seu script continua funcionando normalmente**: só tenta de novo
no próximo envio.

## O que vocês vão implementar

Usando o que vimos em aula:

- **Java:** um `@RestController` com dois métodos `@GetMapping`: o `/api/health`,
  que devolve um `HashMap` fixo, e o `/api/secrets`, que **lê o arquivo JSON**
  gravado pelo Python e devolve o conteúdo dele;
- **Python:** um script simples com `requests`, usando `for`, `if`, listas e
  dicionários, que procura os outros nós, pede os segredos deles e envia o
  status ao painel;
- a **troca de informação entre Python e Java**, sem HTTP.

### Dicas (o que ainda não vimos)

- No Python, `requests.get(url, timeout=0.3)` desiste depois de 0,3 segundo. Sem
  `timeout`, um IP que não responde trava o seu script. Coloque a chamada dentro
  de `try`/`except`, porque a maioria dos IPs da rede não vai responder.
- Um jeito simples de procurar nós é percorrer os IPs da rede (do `.1` ao `.254`)
  e tentar a rota do desafio em cada um. Ferramentas de varredura de portas, como
  o `nmap`, também são permitidas, mas o resultado precisa chegar ao seu script
  Python automaticamente.
- O jeito mais simples de dividir o trabalho: o Python grava no arquivo
  **exatamente o JSON que a rota deve devolver** (com `service`, `group` e
  `secrets`), e o Java só lê o arquivo e devolve. Assim o Java não precisa montar
  nada.
- Como ler um arquivo JSON em Java com Spring Boot não vimos em aula: pesquisem
  (por exemplo, "spring boot ler arquivo json"). A biblioteca de JSON já vem junto
  com o Spring.
- Java e Python precisam apontar para o **mesmo arquivo**. Combinem um caminho
  que funcione para os dois, lembrando que cada um roda a partir de uma pasta
  diferente.
- No Python, grave primeiro em um arquivo temporário e troque o nome no fim
  (`os.replace`). Assim o Java nunca lê um arquivo pela metade.
- O `content` pode ter acentos. Leiam e gravem os arquivos em UTF-8.
- Cada IP que não responde faz o seu script esperar o `timeout` inteiro. Com
  `timeout=0.3` e 254 IPs, uma varredura completa pode levar mais de um minuto.
  Pensem em como deixar isso mais rápido (menos IPs, `timeout` menor, várias
  tentativas ao mesmo tempo) e em avisar o painel enquanto a busca ainda não terminou.
- Depois de um tempo, vale a pena parar de procurar. Pensem em quando.

## Regras gerais (valem para todos os grupos)

1. Existem **seis grupos** e **seis segredos**, um para cada grupo no início.
2. Cada grupo começa conhecendo **apenas o seu próprio segredo**.
3. É **proibido** colocar no código, em arquivo ou em variável os IPs dos
   outros participantes. A descoberta é automática.
4. Os segredos são obtidos **automaticamente**, sem `curl`, Postman ou qualquer
   intervenção manual durante a execução oficial.
5. O que o seu nó aprendeu deve aparecer no `GET /api/secrets` do **seu** Java,
   para os outros poderem pegar de você.
6. Segredo repetido **não pode** aparecer duas vezes no `GET /api/secrets`.
7. Se um nó estiver fora do ar ou demorar a responder, o seu script **continua**
   com os outros. Ele não pode travar nem encerrar.
8. É **proibido** usar o painel para descobrir nós ou pegar segredos, e também
   qualquer outro servidor externo compartilhado entre os grupos.
9. Python e Java **não se comunicam por HTTP/TCP**, nem por `localhost`.
10. O Java não faz chamadas para outros servidores. Quem sai procurando é o
    Python.
11. Não sobrecarreguem a rede da sala: usem `timeout` curto e esperem alguns
    segundos entre as rodadas.

## Integração

1. Rodem **primeiro o servidor Java** e abram `http://localhost:12000/api/health`
   no navegador: deve aparecer o JSON com `"status": "ok"`. Depois abram
   `http://localhost:12000/api/secrets`. Antes de o Python gravar o arquivo, a
   resposta pode vir vazia: tudo bem. Peçam ao professor para confirmar que o painel
   mostra o seu grupo com **Servidor Java: online**.
2. Rodem o **script Python**. Ele deve gravar o seu segredo e o painel deve mostrar
   o seu grupo com 1 segredo.
3. Descubram o **IP da sua máquina** (`ipconfig` no Windows, "Endereço IPv4";
   `ifconfig` ou `ip a` no Linux/macOS). Peçam a um colega que abra
   `http://SEU-IP:12000/api/secrets`. Se não abrir, o **firewall** está
   bloqueando: no Windows, permitam o Java (ou a porta `12000`) em redes privadas.
4. Testem o seu script Python contra o servidor Java de **outro grupo** e o
   contrário, antes da execução oficial.
5. Na execução oficial, subam os dois programas e acompanhem o painel.

## Problemas comuns

| Sintoma | O que verificar |
| --- | --- |
| O painel não mostra o seu Servidor Java como online | o servidor não subiu (veja o terminal); `/api/health` não existe ou devolve outro JSON; computador em outra rede que a do professor; firewall bloqueando a porta `12000` |
| O painel marca o grupo como inativo no meio da busca | a varredura demorou mais de 20 segundos sem enviar o status; envie o status também durante a busca |
| Os outros grupos não conseguem abrir o seu Java | servidor com outra porta que não `12000`; firewall; computador em outra rede ou Wi-Fi com isolamento de clientes |
| O script não encontra ninguém | os outros ainda não subiram; rede errada no script (confira o prefixo do seu IP); `timeout` curto demais ou sem `try`/`except` |
| O script trava ou fica muito lento | falta `timeout` nas chamadas; esperar muito por IPs que não respondem |
| O script acha o próprio nó | falta ignorar o seu próprio `group` |
| O painel mostra um segredo rejeitado | o `content` foi alterado no caminho (espaço, quebra de linha, codificação); `id` fora do formato `secret-N`; o grupo de origem ainda não informou o hash dele |
| O seu Java mostra `secrets: []` mesmo com o Python rodando | o Java não está lendo o mesmo arquivo/pasta que o Python grava; arquivo ainda não existe |
| O Java dá erro ao ler o arquivo | o Python estava gravando na hora; troquem o arquivo de uma vez com `os.replace` |
| Um segredo aparece duas vezes | o script guardou o mesmo `id` mais de uma vez; use um dicionário indexado pelo `id` |
| O painel mostra o grupo como inativo | o script parou, ou o envio demorou mais de 20 segundos; a URL do painel está errada |

## Critério de sucesso

O desafio está concluído quando, ao mesmo tempo:

- os seis grupos estão rodando o servidor Java **e** o script Python;
- cada um dos seis nós mostra os **seis** segredos no seu `GET /api/secrets`;
- o painel exibe **36 de 36** segredos validados;
- tudo isso aconteceu **sem configurar IPs** e **sem enviar segredos na mão**.

O professor pode derrubar e reiniciar o Java ou o Python de um grupo durante a
execução. Depois de reiniciado, o nó deve voltar a ter os seis segredos sozinho.

## Entregáveis

1. O projeto do **servidor Java**, com instruções para rodar.
2. O **script Python**, com instruções para rodar.
3. Um arquivo **`DECISOES.md`** curto explicando:
   - como o seu script **procura** os outros nós;
   - qual mecanismo vocês usaram para o Python entregar os segredos ao Java e
     **por quê**;
   - quando o seu script **para de procurar** e por quê;
   - o que acontece se o seu Java ou o seu Python reiniciar.
4. A **demonstração ao vivo** na execução oficial.

Não passem a frase do seu segredo para outros grupos: ela só pode chegar a
eles pela rede.

## Critérios de avaliação

| Critério | Peso | O que o professor observa |
| --- | --- | --- |
| Contrato da API Java | 25 | rotas `/api/health` e `/api/secrets`, campos e códigos corretos, porta `12000`, funciona com o script de outros grupos |
| Descoberta automática | 20 | encontra os nós sem IPs digitados, ignora IPs que não respondem e o próprio nó, não trava |
| Obtenção e repasse dos segredos | 20 | chega aos seis segredos, sem repetidos, repassa o que aprendeu pelo seu Java |
| Python e Java sem HTTP | 15 | troca de informação funcionando, justificada em `DECISOES.md` |
| Painel | 10 | envio a cada 5 segundos, hashes corretos, o script segue funcionando com o painel fora do ar |
| Demonstração | 10 | os dois programas sobem, o grupo explica o que acontece e o nó se recupera depois de reiniciar |

Os seis grupos dependem uns dos outros para chegar a 36 de 36, então ajudar um
colega a depurar a integração faz parte do exercício. Já o **código** não se
compartilha: cada grupo escreve o seu.

Bom desafio!
