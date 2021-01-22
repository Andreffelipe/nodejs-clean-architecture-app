# Uma API Hapi.js básica seguindo os princípios de Arquitetura Limpa

## Começando (< 2mn)

```
git clone git@github.com:jbuget/nodejs-clean-architecture-app.git
cd nodejs-clean-architecture-app
npm install
npm test
npm start
```

acesse em seu navegador [http://localhost:3000/hello](http://localhost:3000/hello).

## Domain Driven Architectures



O design de software é uma coisa muito difícil. Com o passar dos anos, surgiu uma tendência de colocar a lógica de negócios, também conhecida como Domínio de Negócios, e com ela o Usuário, no coração de todo o sistema. Com base neste conceito, diferentes padrões arquitetônicos foram imaginados.

Um dos primeiros e principais foi apresentado por E. Evans em seu [Domain Driven Design approach](http://dddsample.sourceforge.net/architecture.html).

![DDD Architecture](/doc/DDD_architecture.jpg)

Com base nele ou ao mesmo tempo, outras arquiteturas de aplicativos surgiram como [Onion Architecture](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/) (by. J. Palermo), [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) (by A. Cockburn) or [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) (by. R. Martin).


Este repositório é uma exploração desse tipo de arquitetura, principalmente baseada em DDD e Clean Architecture, em uma aplicação JavaScript concreta e moderna.
 
## DDD e Clean Architecture


O aplicativo segue o Uncle Bob "[Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)" princípios e estrutura do projeto :

### camadas da arquitetura limpa

![Schema of flow of Clean Architecture](/doc/Uncle_Bob_Clean_Architecture.jpg)

### Anatomia do projeto

```
app 
 └ lib                              → Pasta raiz da aplicação 
    └ application                   → Camada de serviços da aplicação
       └ security                   → Interfaces de ferramentas de segurança (ex: AccessTokenManager.js,para gerar e decodificar o token de acesso OAuth)
       └ use_cases                  → Regras de negócios da aplicação
    └ domain                        → amada de negócios centrais da empresa, como objetos de modelo de domínio (Agregados, entidades, objetos de valor) e interfaces de repositório
    └ infrastructure                → Frameworks, drivers e ferramentas como Banco de Dados, Web Framework, mailing/logging/glue code etc.
       └ config                     → Arquivos de configuração de aplicativos, módulos e serviços
          └ service-locator.js      → Módulo que gerencia implementações de serviço por ambiente
       └ orm                        → Database ORMs middleware (Sequelize para SQLite3 ou PostgreSQL, Mongoose para MongoDB)
          └ mongoose                → Mongoose client and schemas
          └ sequelize               → Sequelize client and models
       └ repositories               → Implementação de interfaces de repositório de domínio
       └ security                   → Implementações de ferramentas de segurança (ex: JwtAccessTokenManager)
       └ webserver                  → Hapi.js Configuração do servidor web (server, routes, plugins, etc.)
          └ oauth                   → Hapi.js estratégias e esquemas de autenticação
          └ server.js               → Hapi.js definição de servidor
    └ interfaces                    → Adaptadores e formatadores para casos de uso e entidades para agências externas, como Banco de Dados ou a Web
       └ controllers                → Hapi.js manipuladores de rota
       └ routes                     → Hapi.js definições de rota
       └ serializers                → Objetos conversores que transformam objetos externos (ex: HTTP request payload) para objetos dentro (ex: Use Case request object)
 └ node_modules (generated)         → NPM dependências
 └ test                             → Pasta de origem para testes de unidade ou funcionais
 └ index.js                         → Principal ponto de entrada do aplicativo
```

### Fluxo de Controle

![Schema of flow of Control](/doc/Hapijs_Clean_Architecture.svg)

### The Dependency Rule

> A regra de substituição que faz essa arquitetura funcionar é a regra de dependência. Esta regra diz que as dependências do código-fonte só podem apontar para dentro. Nada em um círculo interno pode saber absolutamente nada sobre algo em um círculo externo. Em particular, o nome de algo declarado em um círculo externo não deve ser mencionado pelo código em um círculo interno. Isso inclui funções, classes. variáveis ​​ou qualquer outra entidade de software nomeada.
  
src. https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html#the-dependency-rule

### Server, Routes and Plugins

Servidor, rotas e plug-ins podem ser considerados como "código-encanador" que expõe a API ao mundo externo, por meio de uma instância do servidor Hapi.js.

A função do servidor é interceptar a solicitação HTTP e combinar a rota correspondente.

As rotas são objetos de configuração cujas responsabilidades são verificar o formato da solicitação e os parâmetros e, em seguida, chamar o bom controlador (com a solicitação recebida). Eles são registrados como Plugins.

Plugins são objetos de configuração que empacotam um conjunto de recursos (ex: autenticação e questões de segurança, rotas, pré-manipuladores, etc.) e são registrados na inicialização do servidor.   

### Controllers (a.k.a Route Handlers)

Os controladores são os pontos de entrada para o contexto do aplicativo.

Eles têm 3 responsabilidades principais:

1. Extraia os parâmetros (query ou body) da solicitação
2. Chame o bom caso de uso (camada de aplicativo)
3. Retorne uma resposta HTTP (com código de status e dados serializados)

### Use Cases

Um caso de uso é uma unidade de lógica de negócios.

É uma classe que deve ter um método `execute` que será chamado pelos controladores.

Ele pode ter um construtor para definir suas dependências (implementações concretas - a.k.a. _adapters_ - dos objetos _port_) ou seu contexto de execução.

**Seja cuidadoso! Um caso de uso deve ter apenas uma responsabilidade de negócios precisa!**

Um caso de uso pode chamar objetos na mesma camada (como repositórios de dados) ou na camada de domínio.

## Os 5 princípios do SOLID

### 1 — Single Responsability Principle:

Esse princípio parte do conceito de que, cada classe deve ter uma única responsabilidade. Ou seja, se temos uma classe de Cliente, por exemplo, não devemos ter uma função para fazer upload de arquivos. Portanto, nessa classe classe, os únicos métodos existentes devem fazer função única e exclusivamente referente ao Cliente, como por exemplo, cadastro de cliente, verificar status do cliente, informações do cliente, entre outros.

### 2 — Open/Closed Principle:

Deve ser capaz de extender a classe pai sem alterar seu funcionamento, ou seja, aberta para extensão, fechada para modificação.
Na prática, significa que:
Se temos uma classe de depósito em contas de um banco, por exemplo, podemos ter vários tipos de conta, certo? Conta poupança, salário, corrente, universitária, entre outros.
Vamos supor que, o banco irá criar uma conta nova, uma conta platinum com vários benefícios. A classe “depósito” sendo responsável por realizar o depósito em todos os tipos de conta, a unica alteração a se realizar seria adicionar a nova conta platinum e suas regras de negócio.
Porém, fazendo dessa forma, estamos ferindo o princípio do open/closed principle, pois a forma correta de fazer seria:
Ter uma classe abstrata depósito, e para cada tipo de conta, sua respectiva classe. Dentro da classe de cada tipo de conta, deveríamos ter suas funcionalidades e regras de negócio específicas. Assim, estaríamos extendendo a classe pai depósito, e fazendo as alterações somente na classe filha, sem alterar o funcionamento da classe pai.

### 3 — Liskov Substitution Principle:

É um princípio que está muito ligado ao Open/Closed Principle.
Uma subclasse não pode quebrar o comportamento da classe pai, e os objetos da classe filha podem ser usados no lugar dos objetos da classe pai. Ou seja, uma subclasse deve sobrescrever os métodos da classe pai, de tal maneira que não quebre o funcionamento da mesma.

### 4 — Interface Segregation Principle:

Muitas interfaces são melhores do que uma. Não se deve forçar classes a usar métodos que não utilizam.
Ou seja, criar várias interfaces independentes torna o código mais organizado e facilitará o reuso. Mesmo que existam muitos arquivos de interfaces, ainda sim é melhor do que ter poucas interfaces genéricas e com muitas funcionalidades.

### 5— Dependency Segregation Principle:

Depender de uma abstração e não de uma implementação, não depender de uma classe concreta diretamente no código. Injetar as dependências por meio de interfaces.
Imaginemos que, temos um projeto, em que existem vários arquivos que instanciam uma classe, e que usam várias funções dessa mesma classe. Porém, por algum motivo, preciso altera o nome da classe e de suas respectivas funções. Nesse caso, seria necessário ir em todos os arquivos e alterar o nome da função e da classe. O que fere o conceito do DSP, pois estaríamos dependendo de uma implementação.
Caso estivéssemos dependendo de uma abstração, seria necessário alterar o nome da função somente na interface, o que é fenomenal para a reutilização e manutenção do código. Ao instanciar uma nova classe, estamos trazendo todo o conteúdo da classe pai para a filha, o que é errado, pois o resto do código não precisa saber o conteúdo das outras classes.
