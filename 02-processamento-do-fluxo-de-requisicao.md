---
title: Processamento do fluxo de requisição
description: Processamento do fluxo de requisição do Magento 2
permalink: /processamento-do-fluxo-de-requisicao
---

1. Utilizar os modos de deploy da Magento e conhecer as etapas da inicialização da aplicação
2. Demonstrar habilidade para processar URLs do Magento
3. Demonstrar habilidade para personalizar rotas de requisição
4. Descrever o processo de inicialização do layout
5. Descrever a estrutura dos `block templates`
{:toc}


## Utilizar os modos de deploy da Magento e conhecer as etapas da inicialização da aplicação

### Identificar as etapas para a inicialização da aplicação. 

1. Ponto de entrada da aplicação: `pub/index.php`.
2. O bootstrap é inicializado: `Magento\Framework\App\Bootstrap->run()`.
3. É criada a aplicação HTTP (`\Magento\Framework\App\Http`): `Magento\Framework\App\Http\Interceptor->launch()`
4. É determinado o código da área do Magento
5. Baseado na área, o _front controller_ é criado (HTTP, Rest, Soap, etc): o _front controller_ vai direcionar a solicitação
6. `Magento\Framework\App\FrontController\Interceptor->dispatch()`: Aqui a lista de _routers_ é percorrida e cada um é verificado se corresponde à rota.
7. Se corresponder, a classr `\Magento\Framework\App\ActionInterface` é retornada e sua `action` é executada. 
8. O método `execute` do _controller_ é executado.
9. O `Result` disto é retornado para o `FrontController`
10. O `Result` é renderizado e definido na `Response`
11. Voltamos para  a classe `Http`.

**Como você projetaria uma personalização que deveria agir em cada requisição e capturar dados de saída, independentemente do controller?**
Seria uma boa oportunidade para usar os `observers`. Nesse caso, para o evento `controller_action_postdispatch`.


### Descrever como usar os modos Magento. 
**Entender os prós e contras usando o modo desenvolvedor/produção? Como você usa o modo default?**
Discutimos isso [aqui](https://ligiasalzano.github.io/magento2-exam-AD0-E703/arquitetura-e-customizacao#demonstrar-a-capacidade-de-criar-um-processo-de-deploy) e [aqui](https://ligiasalzano.github.io/magento2-exam-AD0-E702/processamento-do-fluxo-de-requisicao#descrever-como-usar-os-modos-magento).

**Como você ativa/desativa o modo de manutenção?**
- `bin/magento maintenance:enable`
- `bin/magento maintenance:disable`


### Descrever as responsabilidades do _front controller_.
O _front controller_ implementa a interface `\Magento\Framework\App\FrontControllerInterface` e é responsável por executar a lógica de negócios e retornar o resultado dela. Este resultado pode ser um simples layout HTML ou algo mais complexo, como um objeto para edição.

**Em quais situações o controller frontal é envolvido na execução e como ele é usado nas customizações do escopo?**
O `FrontController` é envolvido em todas as requisições que não sejam API. Ele é o ponto de entrada para o processamento do fluxo de requisição.
Ele é usado para transformar o _input_ de uma url e seus parêmetros em uma resposta HTML.
Não é usado em API ou no _console_.



## Demonstrar habilidade para processar URLs do Magento

### Descrever como a Magento processa uma dada URL. 

Há uma boa referência sobre o assunto no [DevDocs](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html).

**Como você identifica qual modulo e controller corresponde a uma certa URL?**

O Magento determina a área baseado no frontname (que é registrado no `routes.xml`). Se não houver um frontname que corresponda à rota, o padrão é usar a área `frontend`.
> `\Magento\Framework\App\AreaList::getCodeByFrontName`

Se não for uma requisição de API, o Magento analisa a URL usando o `\Magento\Framework\App\Router\Base::parseRequest`. A parte explorada é o `path`, que corresponde ao seguimento após o domínio.

A maior parte das URLs no magento podem ser divididas em 5 partes: `<store-url>/<front-name>/<controller-name>/<action-name>`
1. Primeiro a url da loja `store-url`.
2. Em seguida, o `frontname`. Exemplo: `catalog`.
4. Depois, o nome do `controller`. E consideramos o caminho até a classe, a partir da pasta `Controller`. Exemplo: em `product`.
5. Então o nome da classe da ação (`action-name`). Esta classe deve implementar a `ActionInterface` e possuir um método `exevute` 
6. Por fim, vem os parâmetros. Exemplo: `id/54`.


**O que é necessário para criar uma estrutura personalizada de URL?**
1. Crie uma nova classe _router_, na pasta `Controller`, que implementa a interface `\Magento\Framework\App\RouterInterface`.
2. Registre o _router_ no arquivo `di.xml`
3. Crie uma Action, instancia de `\Magento\Framework\App\ActionInterface`
4. Registre a rota da action no arquivo `routes.xml`

Há um exemplo passo a passo [aqui](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/routing.html#example-of-routing-usage).


### Descrever como é o processo de reescrita e sua função na criação de URL amigáveis.

**Como as URLs amigáveis são definidas, e como elas são customizadas?**
As URLs amigáves são apelidos para as URLs padrão do Magento. Ao invés de mostrar a url `htpps://loja.com/catalog/product/view/id/54`, o link para este produto será: `htpps://loja.com/meu-produto`. Essa reescrita pode ser feita automaticamente, se configurado.
O Magento usa a tabela `url_rewrite` para armazenar e mapear as URLs no banco de dados.
- `request_path`: "minha-url-amigavel"
- `target_path`: "catalog/product/view/id/54"

As URLs amigáveis vem do campo `url_key`, este atributo existe no cadastro de produtos, categorias e páginas cms.


### Descrever como funcionam as ações e resultados dos controllers. 

- Os `controllers` de ação implementam o a interface `\Magento\Framework\App\ActionInterface` e estendem a classe `\Magento\Framework\App\Action\Action` em algum ponto.
- Cada `controller` tem apenas uma única `action`.
- O método `execute` deve retornar `\Magento\Framework\Controller\ResultInterface` (para HTML, JSON ou outros tipos de output).

**Como os controladores interagem uns com os outros?**
- Um `controller` pode encaminhar para outra rota. Não é exibida nenhuma alteração na URL, mas um novo `controller` assume o processamento da URL. Isso reinicia o loop de correspondência do _router_ com novos argumentos. Ver: `\Magento\Framework\Controller\Result\Forward`.
- O `controller` pode redirecionar para outra URL. Ver: `\Magento\Framework\Controller\Result\Redirect`.

**Como os diferentes tipos de resposta são gerados?**
O Magento possui alguns tipos de resposta, todos implementam a interface `\Magento\Framework\Controller\ResultInterface`.
Usamos estas classes de resposta da seguinte forma:
1. Definimos qual o tipo a ser usado
2. Injetamos a `Factory` da classe no nosso `controller`.
3. Criamos uma instancia desta classe, definimos o que precisamos e à retornamos.

Os tipos de resposta já existentes no Magento são:
- `Magento\Framework\Controller\Result\Forward`
- `Magento\Framework\Controller\Result\Json`
- `Magento\Framework\Controller\Result\Raw`
- `Magento\Framework\Controller\Result\Redirect`
- `Magento\Framework\View\Result\Page`


## Demonstrar habilidade para personalizar rotas de requisição


### Descrever o roteamento e o fluxo da requisição na Magento. 

**Quando é necessário criar um novo roteador ou customizar os roteadores existentes?**
Quando temos grandes quantidades de personalizações de url ou situações que não são atendidas pela funcionalidade nativa do Magento.
Um exemplo de situação para criar um _router_ personalizado seria para um módulo de blog.

**Como você manipula páginas 404 customizadas?**
A página 404 é uma página CMS armazenada no banco de dados. Pode ser modificada usando HTML/CSS ou o Page Builder (no caso de ser Adobe Commerce).
Para modificações mais profundas, pode-se usar um `NoRouteHandler` na classe `\Magento\Framework\App\Router\NoRouteHandlerList`.

Classes que implementam a `Magento\Framework\App\Router\NoRouteHandlerInterface`:
- `Magento\Backend\App\Router\NoRouteHandler`
- `Magento\Framework\App\Router\NoRouteHandler`


## Descrever o processo de inicialização do layout

### Determinar como o layout é compilado. 

Todos os arquivos de layouts XML são mesclados em um XML que é armazenado no cache de layout. A sequência em que eles são carregados é determinada pelo nó `sequence` no `etc/module.xml`.

Para uma página particular o seu layout é definido por 3 principais componentes: 

**_Page layout_:**

Define o `page wireframe`, a estrutura, por exemplo, _one-column_, _two-column_ etc. 
São localizados em `<module_dir>/view/frontend/page_layout` ou `<theme_dir>/<Namespace>_<Module>/page_layout`. 
Exemplo de arquivo: `view/frontend/page_layout/2columns-left.xml`. 
Para utilizar um layout, ele precisa ser declarado no arquivo `layouts.xml`, que pode ser colocado em `<module_dir>/view/frontend/layouts.xml` ou `<theme_dir>/<Namespace>_<Module>/layouts.xml`

> O Magento possui, nativamente, 5 tipos de layout de página para o `frontend` (empty, 1column, 2columns-left, 2columns-right e 3columns) e 3 tipos para o `backend` (admin-empty, admin-1column, and admin-2columns-left).


**_Page configuration_**:

Detalha o conteúdo do `wireframe` da página, blocos (header, footer, etc), `meta information` e conteúdos da seção `<head>`. 
São localizados em `<module_dir>/view/frontend/layout` ou `<theme_dir>/<Namespace>_<Module>/layout`.

**_Generic layout_**:

Layouts genéricos que definem o conteúdo e a estrutura detalhada dentro da seção `<body>` da marcação da página HTML. 
Também ficam em `<module_dir>/view/frontend/layout` ou `<theme_dir>/<Namespace>_<Module>/layout`.

Para mais informações sobre os tipos de arquivos de layout, [clique aqui](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-types.html).


**Como você depuraria seus arquivos de layout.xml e verificaria que foram usadas as instruções de layout corretas?**

- Verifique se o seu layout XML está sendo processado. Isso pode ser feito com um breakpoint no `foreach` em `\Magento\Framework\View\Model\Layout\Merge::_loadFileLayoutUpdatesXml`.
- Verifique se o seu XML está correto, podes-se usar um validador de XML para isso.
- Usando o modo de desenvolvimento, verifique se algum erro é apresentado no `frontend`.
- Verifique os `handles` carregados na página.
- Verifique se o nome do XML está correto.
- No caso de um bloco com uma classe própria, verifique se a classe existe e está sendo carregada.
- Verifique se a saída HTML está sendo processada corretamente (método `toHtml`).

### Determinar como a saída HTML é renderizada. 
**Como a Magento descarrega a saída e quais mecanismos existem para acessar e customizar a saída?**

Layout rendering: `\Magento\Framework\View\Layout::getOutput()`.
Block renderer: `\Magento\Framework\View\Element\AbstractBlock::toHtml()`

Para customizar o `output`, você pode:

- Modificar o arquivo layout XML
- Sobrescrever templates phtml
- Adicionar um `layout handle` em `\Magento\Framework\View\Result\Page`.
- Sobrescrever o método `_toHtml` do bloco
- Criar uma `result class` customizada (estendendo a classe `\Magento\Framework\View\Result\Page`) e a aplicando no `controller`.


### Determinar o esquema de layout XML de um módulo. 
**Como você adiciona novos elementos em uma página introduzida por um certo módulo?**

Isso pode ser feito através do XML layout da página. Para tanto, é necessario saber qual `handle` será utilizado. 
Se não souber quais `handles` estão disponíveis, basta verificar o atributo classe da tag html `<body>` ou utiliar o método `\Magento\Framework\View\Layout\ProcessorInterface::getHandles()`. Com o handle em mãos, crie o xml com o mesmo nome.
No arquivo, basta utilizar as instruções de layout para adicionar ou modificar elementos.


### Demonstrar capacidade para usar o fallback do layout para personalizações e depuração.

> Uma forma prática de verificar o `fallback` é executar `$fallbackRule->getPatternDirs($params)` em `vendor/magento/framework/View/Design/FileResolution/Fallback/Resolver/Simple.php::resolveFile`

**Como você identifica exatamente qual arquivo de layout.xml é processado em um determinado escopo?**

- Verifique qual área está sendo carregada
- Localize o `layout handle`
- Busque no projeto arquivos com o mesmo nome

**Como a Magento trata arquivos de layout XML com os mesmos nomes em módulos diferentes?**

É utilizado o nó `sequence` para determinar a ordem dos arquivos que serão carregados.

### Identificar as diferenças entre os escopos admin e frontend. 
**Quais as diferenças existentes na inicialização do layout para o escopo de admin?**

Na área `adminhtml` o Magento ACL determina o que pode ou não pode ser exibido.


## Descrever a estrutura dos `block templates`

### Identificar e entender os templates root, empty.xml, e page_layout. 
**Como as estruturas da página são definidas, incluindo o número de colunas, quais containers básicos são apresentados, etc.?**

### Descrever a função de blocos e templates no fluxo de requisição. 
**Em quais situações você criaria um novo bloco ou um novo template?**
