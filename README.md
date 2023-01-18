# coursera-ci-cd

Com GitHub Actions, não é necessário realizar a configuração em sites externos. Tudo é feito dentro do arquivo *.yaml*. O conceito mais básico é o de *workflow*, uma série de procedimentos automatizados, representados como jobs e steps. Podemos ter workflows para CI, CD, entre outros. Cada job contém alguns steps, que contêm actions.

## Como Funciona?

Crie um repositório chamado "*.github/workflows*" na raiz do projeto e crie pelo menos um arquivo yaml. Dependendo de como você configura sua branch, cada action vai rodar em determinado momento. Um *Event* é uma ação que dispara um Job. Mais de um job podem ocorrer em paralelo ou em sequência. Cada Job possui um *Runner*, um servidor onde o job é realizado, numa plataforma ou SO. Runners contêm Steps, shell's com comandos e ações. Steps podem ser: lint, build, publish.

**Exemplo de Evento**:

```bash
on:
    pull request:
        types: [opened, reopened]
        branches:
            - main
```

Roda sempre que um PR é aberto/reaberto para dentro da main

```bash
on:
    push:
        branches:
            - main
```

Roda sempre que um push é feito em direção à main, incluindo um merge de um PR.

```bash
on:
    release:
        types: [publish]
```

Roda sempre que uma release é publicada.

**Jobs**: Série de steps que usam o mesmo Runner. Podemos ter mais de um job por workflow. Por padrão, são executados em paralelo. Mas podemos ter um campo "*needs*" para informar de qual outro job o atual precisa para executar - se o outro falhar, o atual não executa. Cada job possui um Runner, Services (opcional) e uma série de steps.

**Runner**: Além de definirmos o SO onde o serviço roda, podemos informar também algum tipo de container.

```bash
jobs:
    build:
        runs-on: ubuntu-latest
        container: python:3.9-slim
```

**Services**: Containers onde os jobs rodam. Podemos instanciar bancos de dados, filas de mensagens, etc.
**Steps**: São tarefas contendo um ou mais comando shell. Pode ter um nome, mostrado nos relatórios, e um id. Pode possuir ações, com o identificador *uses*, ou comandos shell, com o identificador *run*. Também podemos definir o ambiente, com a palavra chave *env*.

```bash
steps:
    - name: Run unit tests
    id: testing
    run: nosetests
    env:
        DATABASE_URI: "redis://redis:6397"
        FLASK_APP: servce:app
```

**Actions**: Procedimentos que podem ser executados dentro de steps, com a palava chave uses. O marketplace do GitHub Actions possui diversas disponíveis. Usamos a palavra chave *with* seguido de um par nome-valor para especificar alguns argumentos. Algumas ações podem usar *args*. Leia a documentação das Actions para entender as possibilidades de trabalho com elas.

```bash
steps:
    name: Upload code coverage
    uses: codecov/codecov-action@v3
    with:
        version: "v0.1.13"
```

## Exemplo de Workflow

```bash
name: CI Build
on:
    pull_request:
        branches: [master]
jobs:
    build:
        runs-on: ubuntu-latest
        container: python:3.9-slim
    services:
        redis:
            image: redis:6-alpine
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip wheel
          pip install -r requeriments.txt
      - name: Run unit tests with nose
        run: nosetests
        env:
            DATABASE_URI: "redis://redis:6397"
      - name: Upload code coverage
        uses: codecov/codecov-action@v2
        with:
            version: "v0.1.13"
```

## Diferenças CI-CD

Continuous Integration, continuamente integra seu código com a main/master/trunk, para ter certeza de que não haverá nenhum problema ao fazer o merge. Continuous Delivery é uma série de práticas desenvolvidas para assegurar que o código possa ser disponibilizado/implantado na produção com segurança, entregando o código num ambiente parecido com o de produção. Delivery, diferente do Deployment, entrega o código para um ambiente parecido com o de produção. Isso é automatizado. Segundo Martin Fowler, Continuous Delivery é uma disciplina que permite que o software seja entregue à produção a qualquer momento.

**Princípios chaves**:

* Qualidade Embutida: Asegurar a qualidade embutida em cada passo do processo. Continuamente revisar o código.
* Trabalhar em Pequenas Porções: Criar histórias curtas, para dividir o trabalho. Continuamente integrar pequenas mudanças.
* Pessoas para RESOLVER Problemas: Pessoas são úteis para resolver problemas, mas não para realizar tarefas repetitivas. Com TDD, você pode manualmente fazer o pull e testar o código. Deixe que os computadores façam essas tarefas repetitivas.
* Busque Melhoria Contínua: CD permite que trabalhemos também na melhoria contínua. Podemos saber onde estamos e quando as coisas quebram.
* TODOS São Responsáveis: Se algo quebra, não podemos buscar um culpado, mas entender como melhorar o processo para isso nunca mais acontecer.
