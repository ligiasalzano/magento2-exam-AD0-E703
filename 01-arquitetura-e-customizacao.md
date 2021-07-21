---
title: Arquitetura Magento e Técnicas de Customização
description: Arquitetura e técnicas de customização do Magento 2
permalink: /arquitetura-e-customizacao
---

1. Descrever a arquitetura baseada em módulos da Magento
2. Descrever a estrutura de diretórios Magento 
3. Utilizar XML de configuração e escopo de variáveis
4. Demonstrar como usar a injeção de dependência (Dependency Injection - DI) 
5. Demonstar habilidade no uso de plugins
6. Configurar `event observers` e trabalhos agendados (scheduled jobs)
7. Utilizar o CLI 
8. Demonstrar habilidade com o gerenciamento de cache
{:toc}

## Descrever a arquitetura baseada em módulos do Magento

Os módulos Magento 2 utilizam a arquitetura MVVM. Que tem 3 camadas:
- Model: Lógica de negócio e contado com a database.
- View: Estrutura e layout
- ViewModel: Liga as camadas Model e View

Em seu nível mais alto, a arquitetura do produto Magento consiste no código do produto principal mais módulos opcionais. Esses módulos opcionais aprimoram ou substituem o código básico do produto.
[A arquitetura do Magento 2 consiste em 4 camadas](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/ALayers_intro.html):
- Presentation Layer: Contém _view elements_, como layouts, blocos, _templates_ e _controllers_
- Service Layer: Intermediário entre as camadas _Presentation_ e _Domain_. Executa os contratos de serviço (que permitem adicionar ou modificar a regra de negócio).
- Domain Layer: Responsável pela lógica de negócio.
- Persistence Layer: Descreve um _resource model_ que é responsável por recuperar e modificar dados na _database_ usando CRUD.

**As áreas do Magento**

Há 7 áreas no Magento: ([referência](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_and_areas.html#magento-area-types))

- `adminhtml` - A área adminhtml inclui o código necessário para o gerenciamento da loja. Refere-se ao painel administrativo (admin ou backend) do Magento. 
- `frontend` - Refere-se à parte da loja na qual haverá a interação com o cliente (usuário final)
- `global` (ou `base`) - Engloba todas as áreas e é usado como _fallback_ para arquivos ausentes nas áreas `frontend` e `adminhtml`
- `crontab` - É carregada quando as _crons_ são executadas. A classe `\Magento\Framework\App\Cron` sempre carrega a área `crontab`.
- `webapi_rest` - Esta área responde pelas chamadas _REST_ da _API_ do Magneto 2.
- `webapi_soap` – Refere-se às chamadas _SOAP_.
- `graphql` - Responde pelas chamadas GRAPHQL (essa área foi inserida a partir do Magento 2.3)

Nem todas as áreas estão disponíveis todo o tempo. Por exemplo, a `crontab` é usada apenas ao executar tarefas cron.

**Desenvolvimento de módulos**
Para mais detalhes, descrevi sobre a criação de módulos [aqui](https://ligiasalzano.github.io/magento2-exam-AD0-E702/arquitetura-e-customizacao#quais-s%C3%A3o-os-principais-passos-para-adicionar-um-novo-m%C3%B3dulo)

### Descreva as limitações do módulo. 
Os módulos são colocados em diretórios localizados em dois diretórios principais:
- app/code
- vendor

Logo, todos os arquivos de um módulo ficam dentro do seu respectivo diretório, desde classes à templates e layouts.

Um outro ponto é que, quando desabilitamos um módulo, ele se torna inativo. Porém as entradas e tabelas no banco de dados não são removidas.

Para verificar o status de um módulo, você pode utilizar o comando:
```
bin/magento module:status Vendor_Module
```
E podemos desabilitar o módulo com `module:disable` e habilitar com `module:enable`.


#### Como os diferentes módulos interagem uns com os outros? Quais efeitos colaterais podem surgir dessa interação?

O Magento segue alguns padrões que ajudam na interação entre os módulos.

- PSR-4 em sua declaração, diz que os `namespaces` devem corresponder ao caminho do arquivo até a classe.
- O arquivo `/composer.json`, na raiz da aplicação, auxilia no carregamento automático das classes
- O uso de injeção de dependência
- O uso de [contratos de serviços](https://alankent.me/2014/10/31/magento-2-service-contract-patterns/) (dentro de `Vendor/Module/Api`) e de classes que representam dados dentro de `Vendor/Module/Api/Data`

Conseguimos extender ou sobreescrever praticamente qualquer classe no Magento. Um problema desta interação entre módulos é que podem ocorrer quebras e efeitos colaterais. Quando as recomentações oficiais da Magento são seguidas, esses problemas podem ser evitados.

## Descrever a estrutura de diretórios Magento

Os módulos podem ser encontrados dentro da pasta `vendor` ou em `app/code`. Coloquei uma descrição sobre os diretórios do Magento [aqui](https://ligiasalzano.github.io/magento2-exam-AD0-E702/arquitetura-e-customizacao#descrever-a-estrutura-de-diret%C3%B3rios-do-magento).


### Descreva como localizar diferentes tipos de arquivos no Magento e onde encontrar arquivos JavaScript, HTML e PHP.

Os módulos do core do Magento são encontrados, em uma instalação típica usando o composer, dentro de `/vendor/Magento`. Em uma instalação a partir do repositório Magento Open Source, geralmente utilizada pela comunidade de contribuidores, os módulos core são encontrados em `/app/code/Magento`. Alguns arquivos JS e CSS da instalação nativa do Magento podem ser localizados no diretório `/lib`.

Lembrando que os arquivos relacionados com o frontend serão colocados dentro do tema ou dentro do módulo em `app/code/[Vendor]/[Module]/view/`. 
Há ainda uma subdivisão no diretório `/view`. Dentro dele, os arquivos JavaScript são colocador na pasta `/view/web` e os arquivos HTML (`.phtml`) ficam dentro do diretório `/view/templates`. Arquivos `.php` são colocados nos outros diretórios do módulo (nunca dentro de `/view`).

## Utilizar XML de configuração e escopo de variáveis

### Determine como usar os arquivos de configuração.
O Magento divide as configurações em vários arquivos. Dessa forma, evita um arquivo muito grande (como ocorria no Magento 1). E cada arquivo é carregado conforme o Magento precisa.
Estes arquivos são colocados dentro do diretório `etc` do módulo. Muitos arquivos de configurações podem ser restringidos de acordo com a `area` de interesse. Por exemplo, o `di.xml` pode ser restrito ao `frontend` ou ao `adminhtml`, basta colocá-lo em um subdiretório com o nome correspondente à área de interesse. Quando não se define uma área, ele será válido globalmente.

> Dica: Busque, como referência, o arquivo de algum módulo existente no core do Magento. Você pode copiá-lo e usá-lo como base.

### Quais arquivos de configuração correspondem a diferentes funcionalidades?

**Alguns arquivos de configuração importantes:**
- `module.xml`: Configuração do módulo. Nele estão o nome do módulo, as dependências em relação à outros módulos e a versão. Este é um arquivo obrigatório.
- `acl.xml`: Inclui o módulo na árvore de permissões à recursos. Possibilita restringir o módulo à certos grupos de usuários no admin.
- `config.xml`: Aqui são colocadas as configurações padrão do módulo. As entradas de configuração podem ser marcadas como criptografadas aqui.
- `view.xml`: É semelhando ao `config.xml`. Aqui especificamos valores para configurações de `design`
- `crontab.xml`: Define ações realizadas por agendamento. Há um serviço que nos ajuda nesta configuração, o [Crontab Guru](https://crontab.guru/).
- `di.xml`: Configura as injeções de dependências. Aqui definimos plugins, `virtual types`, sobreescrita de classes, argumentos para construtores e ligamos as classes concretas às suas `interfaces`.
- `events.xml`: Neste arquivo registramos os nossos `observers`.
- `[area]/routes.xml`: Neste arquivo criamos as rotas para o módulo. Aqui definimos o `frontname`, que é usado na construção da url para o `controller` e o `route ID`, que é usado na nomenclatura dos `handles` de layout.
- `adminhtml/system.xml`: Usado para configurações do módulo. Nele são especificados `tabs`, `sections`, `groups` e `fields` que são encontrados em `Stores > Configurations`.
- `adminhtml/menu.xml`: Usamos este arquivo para criar um novo menu no painel administrativo, na barra lateral.
- `email_templates.xml`: Especifica templates de email.
- `indexer.xml`: Configração de índices.
- `mview.xml`: É usado para rastrear mudanças no banco de dados para uma determinada entidade. Veja mais [aqui na documentação](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/indexing.html#m2devgde-mview).
- `webapi.xml`: Configurações de acesso de APIs e rotas.
- `widget.xml`: Configura `widgets` para serem usados em produts, páginas CMS e blocos CMS.

Existem muitos outros arquivos de configuração no Magento, você pode ver a [lista que existe no devdocs](https://devdocs.magento.com/guides/v2.2/config-guide/config/config-files.html).

> Também é possível criar arquivos de configuração customizados. Dê uma olhada neste [artigo](https://www.atwix.com/magento-2/working-with-custom-configuration-files/).

## Demonstrar como usar a injeção de dependência (Dependency Injection - DI)

Se uma classe precisa de outras classes para funcionar, usamos a injeção de dependências para injetar as classes que precisamos na classe que estamos construíndo.
Leia mais sobre injeção de dependências [aqui](https://alankent.me/2014/06/07/magento-2-dependency-injection-the-m2-way-to-replace-api-implementations/).

### Descreva a arquitetura e abordagem de injeção de dependência do Magento.

A injeção de dependência é um padrão de design que permite declarar dependências de objetos no Magento 2. A utilização da DI permite desenvolver um código mais estrutural e independente tornando o processo de codificação mais conveniente.
A injeção de dependência é feita no Magento através de um construtor (função `__construct()`), no qual todas as dependências são especificadas como argumentos.

**ObjectManager**
O `ObjectManager` é a unidade interna de armazenamento de objetos do Magento e raramente deve ser acessada diretamente. Ele torna possível a implementação do princípio de composição sobre herança.
Na [documentação do Magento](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/depend-inj.html#object-manager), a definição de ObjectManager é: "uma classe de serviço do Magento que instancia objetos no início do processo de _bootstrapping_".

> O Magento proíbe o uso direto do ObjectManager no seu código porque oculta as dependências reais de uma classe. Veja as [regras de uso](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/object-manager.html#usage-rules).

Responsabilidades do `ObjectManager`:
- Criação de objetos _factories_ e _proxies_
- Instanciar automaticamente parâmetros em construtores de classe
- Gerenciar dependências instanciando a classe de preferência quando um construtor solicita sua interface
- Implementar o padrão _singleton_ retornando a mesma instância compartilhada de uma classe quando solicitado

**[Proxies](https://devdocs.magento.com/guides/v2.2/extension-dev-guide/proxies.html)**
As _proxies_ são usadas para a criação de objetos que demandam mais tempo para carregar. Nesses casos, uma classe _proxy_ lento carrega a classe. 
A _proxy_ é especificada no `di.xml` em uma classe como `\Magento\Catalog\Model\Product\Proxy`. 
> _Proxies_ não devem ser especificadas no método construtor.

**Factories**
Para objetos que precisam se criados todas as vezes que são usados, existem as _factories_. Para especificar uma _factory_, inclua a palavra _Factory_ no final da classe ou interface. Exemplo: `\Magento\Catalog\Api\Data\ProductInterfaceFactory`.
Caso você precise, você pode criar a _factory_. Veja como exemplo a classe: `: vendor/magento/module-paypal/Model/IpnFactory.php`.

**Como os objetos são identificados no Magento?**
Como a injeção de dependência acontece automaticamente no construtor, o Magento deve lidar com a criação de classes. Como tal, a criação de classe acontece no momento da injeção ou com uma _factory_.

**Por que é importante ter um processo centralizado de criação de objetos?**
Ter um processo centralizado para criar objetos facilita muito o teste. Ele também fornece uma interface simples para substituir objetos e modificar os existentes.

### Identifique como usar arquivos de configuração DI para personalizar o Magento.

Para fazer essas personalizações no Magento 2, você precisa usar o arquivo de configuração `di.xml`. Que pode ser global ou específico de uma área.

**Plugins**
Os plugins nos dão três possibilidades para manipular funções públicas:
- podemos envolver as funções de outra classe (`around`),
- adicionar um método `before` para modificar os argumentos de entrada ou
- adicionar um método `after` para modificar a saída.

**Preferences**
São usadas para substituir classes inteiras e, também, para ligar uma classe concreta à sua interface.

**Como substituir uma classe nativa?**
> Se houver a possibilidade de usar _plugin_, evite sobreescrever uma classe. Isso pode gerar conflitos.

Para substituir uma classe nativa, use uma entrada `<preference />` para especificar o nome da classe existente (a barra invertida anterior \ é opcional) e a classe a ser substituída.

```xml
<config>
    <preference for="Magento\Catalog\Api\Data\ProductInterface" type="YourVendor\Catalog\Model\Product" />
</config>
```

**Como injetar uma classe em outro objeto**
Use uma entrada `<type/>` com name `providers` (`<argument name="providers" xsi:type="object">\Path\To\Your\Class</argument>`) dentro do nó `<arguments/>`.

Exemplo: `Magento\Sales\Model\ResourceModel\Provider\UpdatedIdListProvider` é a nossa classe que queremos injetar e `vendor/magento/module-sales/Model/ResourceModel/Provider/NotSyncedDataProvider.php`é o objeto de classe no qual vamos injetar a nossa classe.

```xml
<type name="Magento\Sales\Model\ResourceModel\Provider\NotSyncedDataProvider">
 <arguments>
   <argument name="providers" xsi:type="array">
       <item name="default" xsi:type="string">Magento\Sales\Model\ResourceModel\Provider\UpdatedIdListProvider</item>
   </argument>
 </arguments>
</type>
```

**[Virtual Type](https://alanstorm.com/magento_2_object_manager_virtual_types/)**
_Virtual type_ é usado para reduzir a redundância de classes PHP. Ele permite criar uma instância de uma classe existênte modificando os argumentos do construtor.
Neste [tópico](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/build/di-xml-file.html#type-configuration) do devdocs, a configuração dos _virtual types_ é explicada.

```xml
<virtualType name="pdfConfigDataStorage" type="Magento\Framework\Config\Data">
    <arguments>
        <argument name="reader" xsi:type="object">Magento\Sales\Model\Order\Pdf\Config\Reader</argument>
        <argument name="cacheId" xsi:type="string">sales_pdf_config</argument>
    </arguments>
</virtualType>
<type name="Magento\Sales\Model\Order\Pdf\Config">
    <arguments>
        <argument name="dataStorage" xsi:type="object">pdfConfigDataStorage</argument>
    </arguments>
</type>
```

**[Argumentos do construtor](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/build/di-xml-file.html#constructor-arguments)**
É possível modificar quais objetos são injetados em classes específicas, direcionando o nome do argumento para associá-lo à nova classe. É possível injetar uma classe em qualquer outro construtor de classes no `di.xml`

Também é possível injetar sua classe em outro objeto através de uma entrada `<type />`, dessa forma:
```xml
<type name="My\Custom\ClassName">
    <arguments>
        <argument name="object" xsi:type="object">MyExtension\PerhapsNotInstalled\ClassThatMightNotExist</argument>
    </arguments>
</type>
```
Temos mais detalhes sobre isso [aqui](https://magento.stackexchange.com/a/240269) e [aqui](https://inchoo.net/magento-2/overriding-classes-magento-2/).


## Demonstar habilidade no uso de plugins

- Existem 3 tipos de plugins: _before_, _after_ e _around_. São úteis para modificar a entrada, saída ou execução de um método existente (cuidado com os plugins do tipo _around_).
- Plugins funcionam apenas em métodos públicos (não em privados ou protegidos)
- Plugins não funcionam em classes virtuais e finais, métodos finais, estáticos e não públicos, construtores e objetos instanciados antes da inicialização do `Magento\Framework\Interception`.
- Eles são configurados no `di.xml`
- Plugins podem ser usados em interfaces, classes abstratas ou classes pai. Os métodos de plugin serão chamados para qualquer implementação dessas abstrações.


### Demonstrar como projetar soluções complexas usando o ciclo de vida do plug-in

Quando criamos um plugin, o Magento gera uma classe que vai "envelopar" o algo do plugin. Por exemplo, se você cria um plugin para a classe `\Magento\Catalog\Model\Product`, o Magento gera automaticamente a classe `\Magento\Catalog\Model\Product\Interceptor`. Todos os métodos dentro da classe alvo serão representados no _interceptor_. 
Então o Magento localiza e executa os plugins com a classe ` vendor/magento/framework/Interception/Interceptor.php`.

Na [documentação do Magento](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/plugins.html) temos mais detalhes sobre os plugins.

**Plugins Before**
Servem para modificar os argumentos de _input_ de um método. O plugin `before` recebe o objeto (que é a classe alvo do plugin) e os argumentos que o método original recebe. E retorna um array com os atributos do método original, que serão enviados para o próximo plugin ou o próprio método alvo.

**Plugins After**
Estes modificam o retorno do método alvo. Ele recebe como argumento, o objeto da classe alvo, o resultado do método alvo do plugin e os argumentos do método observado.

**Plugins Around**
Os plugins do tipo `around` possuem controle total sobre o _input_ e _output_ de uma função. O método original é passado como um _callback_ e, por padrão, é nomeado `$proceed`. 
A recomendação do Magento é evitar o uso desse tipo de plugin. O motivo é porque, facilmente, pode-se alterar acidentalmente as funções principais do sistema ao omitir uma chamada para o `$proceed`. Ele também torna o processo de _debugging_ mais complicado.


### Como múltiplos plugins interagem e como sua ordem de execução pode ser controlada?
No arquivo `di.xml`, registramos os plugins. E lá ainda podemos definir a prioridade deles diante outros plugins, com a propriedade `sortOrder`. Existe, ainda, a opção de desabilitar um plugin de outro módulo, usando a propriedade booleana `disabled`.

```xml
<type name="Magento\Checkout\Block\Checkout\LayoutProcessor">
    <plugin name="ProcessPaymentConfiguration" disabled="true"/>
</type>
```


### Como você depura um plug-in se ele não funcionar?
1. Primeiro, desabilite o puglin e verifique se isso remove o bug
2. Verifique se o `di.xml` está correto. Busque se há erros de sintaxe.
3. Confira se o plugin não está marcado, em algum lugar, como desabilitado.
4. A classe algo está correta? E a classe do plugin?
5. O plugin foi registrado em um nó `<plugin type="...">`?
6. A classe que você está tentando alterar é do tipo `final`?
7. O método alvo é público? (Verifique as [limitações de plugins](https://devdocs.magento.com/guides/v2.2/extension-dev-guide/plugins.html#limitations).)
8. O nome fa função do seu plugin está escrito corretamente? Exemplo: `beforeMethodName`, `afterMethodName` ou `aroundMethodName`.
9. A classe que você está tentando alterar foi criada segundo os padrões do Magento? Usa DI e `ObjectManager`?

> Existem módulos que utilizam em seu `core`, classes que não seguem os padrões de desenvolvimento do Magento.

### Identifique os pontos fortes e fracos dos plugins. 
Pontos fortes: 
- Facilita a modificação de código existente (como o `core` do Magento)
- São usados para seguir o princípio de responsabilidade única ao separar cada funcionalizada em sua própria área.

Ponto fraco:
- Desenvolvedores podem prejudicar funcionalidades importantes ao usá-los sem conhecer as consequências das alterações, principalmente em plugins do tipo `around`.

### Quais são as limitações na utilização de plugins para personalização? 
- Plugins só funcionam em métodos públicos
- Plugins não funcionam em classes ou métodos finais.
- Não podem ser usados em `Virtual Types`

Leia mais [aqui](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/plugins.html#limitations).

### Em quais casos os plugins devem ser evitados?
Os plugins devem ser evitados em situações em que o uso de um `observer` funcione. Os eventos funcionam bem quando o fluxo de dados não precisa ser modificado.

## Configurar event observers e trabalhos agendados (`scheduled jobs`)

 - Os `observers` observam eventos que são disparados quando ocorre alguma ação. 
 - Uma característica em comum entre `observers` e `scheduled jobs` é que ambos não devem modificar dados que passamm por eles ([Magento guidelines](https://devdocs.magento.com/guides/v2.3/coding-standards/technical-guidelines.html#14-events)).
 
 > Se você precisa modificar dados em um método, é melhor usar os plugins `after` ou `before`.

### Demonstrar como configurar os observers. 
**Como você faz seu observer ser somente ativado no frontend ou backend?**

Um `observer` é registrado no arquivo `events.xml`, que fica localizado dentro do diretório `/etc`. Caso o arquivo `events.xml` seja colocado em um subdiretório nomeado `frontend` ou `backend` o `observer` ficará restrito à área correspondente.

Após isso, é criada uma classe dentro do diretório `Observers` (este nome é usado como convenção). A classe do `observer` deve implementar a interface: `\Magento\Framework\Event\ObserverInterface`.

> No DevDocs temos este [artigo](https://devdocs.magento.com/guides/v2.3/ext-best-practices/extension-coding/observers-bp.html) com as melhores práticas no uso dos `observers`.

### Demonstrar como configurar um trabalho agendado. 

Para configurar um trabalho agendado, você precisa configurá-lo no arquivo `crontab.xml`. Este arquivo fica diretamente dentro da pasta `/etc` (ele não pode ser colocado em subdiretórios).

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Cron:etc/crontab.xsd">
    <group id="default">
        <job name="custom_cronjob" instance="Magento\SampleMinimal\Cron\Test" method="execute">
            <schedule>* * * * *</schedule>
        </job>
    </group>
</config>
```

O nó `group` define à qual grupo uma dada rotina pertence. Isto serve para agrupar funcionalidades em grupos lógicos. Para a maior parte dos trabalhos agendados, usamos o grupo `default`.
Configurações de grupos são feitas no aquivo `cron_groups.xml`.

A configuração do trabalho em si é feita no nó `job`. Aqui definimos um nome único para o trabalho, a classe que o representa (`instance`) e o nome do método que será executado. Dentro do nó `job` existe o nó `schedule` que é onde configuramos a rotina em que aquele trabalho será executado (se será a cada hora, ou todo dia 15, ou a cada 6 meses...).
O site [crontab.guru](https://crontab.guru/) fornece uma ferramenta para auxiliar na criação desta rotina.

**Quais parâmetros são usados na configuração, e como esta configuração pode interagir com a configuração do servidor?**

As variáveis de ambiente do sistema podem ser usadas para alterar as informações de configuração da loja. 
[Ver sobre aqui](https://devdocs.magento.com/guides/v2.4/config-guide/prod/config-reference-var-name.html)


### Identificar a função e o uso apropriado dos eventos disponíveis automaticamente

O Magento possui uma série de eventos já definidos, que cobrem diversas funcionalidades. 
O carregamento de dados é uma das ações mais comuns de se usar eventos e `observers`. 

Alguns eventos disponíveis:

**Models:**
> Acionados em `\Magento\Framework\Model\AbstractModel`.
- [$eventPrefix]_load_before
- [$eventPrefix]_load_after
- [$eventPrefix]_save_before
- [$eventPrefix]_save_after
- [$eventPrefix]_save_commit_after
- [$eventPrefix]_delete_before
- [$eventPrefix]_delete_after
- [$eventPrefix]_delete_commit_after

> O prefixo do evento é definido no `model`, veja este como exemplo: `vendor/magento/module-catalog/Model/Product.php`

**Controllers:**

- controller_action_predispatch_[ROUTE_NAME]
- controller_action_predispatch_[FULL_ACTION_NAME]
- controller_action_postdispatch_[ROUTE_NAME]
- controller_action_postdispatch_[FULL_ACTION_NAME]
- controller_action_layout_render_before_[FULL_ACTION_NAME]


> [Lista de todos os eventos disparados](https://cyrillschumacher.com/magento2-list-of-all-dispatched-events/)

**Como disparar um evento**
Primeiro, injete uma instânica da classe `\Magento\Framework\Event\ManagerInterface` no construtor. Então, você poderá disparar o evento, no momento desejado, desta forma:
```php
$this->eventManager->dispatch('event_name_goes_here', ['parameter' => 'array']);
```

## Utilizar o CLI

### Descreva o uso dos comandos `bin/magento` no ciclo de desenvolvimento

**Quais comandos estão disponíveis?**

Podemos ver a lista de comandos disponíveis utilizando o comando `bin/magento list` ou apenas `bin/magento`. Uma outra dica é que, se usar `--help` no final de qualquer comando, é mostrado uma ajuda sobre ele e suas opções.

Veja como criar novos comandos [aqui](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cli-cmds/cli-howto.html).

Alguns comandos importantes:
- `cache:status`: Lista os tipos de cache e seus status (se está habilitado ou desabilitado).
- `cache:flush` e `cache:clean`: Destrói ou libera o cache. Diferenças entre estes serão explicadas na próxima seção.
- `cache:flush`: Libera ou destrói o cache.
- `deploy:mode:show`: Mostra o atual modo de deploy: `default`, `production` ou `develeper`.
- `dev:query-log:enable`: Habilita o log das `queries`.
- `indexer:info`: Detalhes sobre os índices.
- `indexer:reindex`: Inicia o processo de `reindex`.
- `module:enable`: Usado para habilitar um módulo.
- `module:status`: Mostra se um módulo está habilitado ou não.
- `setup:upgrade`: Sincroniza alterações com o banco de dados (tabelas criadas ou removidas, versões de módulos alteradas, etc).
- `setup:di:compile`: Compila classes e injeção de dependências.

**Como os comandos são usados no ciclo de desenvolvimento?**

Os comandos CLI fornecem um ponto de entrada seguro para a realização de operações que podem não ser seguras para serem executadas no painel de administração do Magento. O acesso SSH deve ser uma maneira segura de examinar o status de autorização de um usuário.

O Magento provê comandos para fazer deploy de conteúdos estáticos, atualizar o banco de dados, compilar as classes com injeção de dependências, habilitar/desabilitar módulos etc.
Alguns comandos são frequentemente utilizados por serem mais ágeis do que utilizar a interface do painel administrativo (como limpar o cache, por exemplo).

### Demonstrar a capacidade de criar um processo de deploy. 

O processo de deploy possui algumas etapas:
- O modo de manutenção é habilitado
- Copia os arquivos para o destino de implantação
- Habilitar módulos / aplicar configuração de implantação
- Executa a compilação de dependências
- Constrói ativos estáticos
- Atualiza o banco de dados
- O modo de manutenção é desabilitado

**Como a aplicação se comporta nos diferentes modos de deploy, e como esses comportamentos afetam a abordagem do deploy para o código PHP, assets frontend, etc.?**

A aplicação Magento comporta-se diferente dependendo do modo de deploy que está ativo, conforme segue:

`Default`:
- Híbrido entre os modos `production` e `developer`
- Links simbólicos são estabelecidos na pasta `pub/static`
- `Exceptions` não são mostradas para o usuário, apenas logadas na pasta `var/log`
- Arquivos estáticos são gerados em tempo real e são vinculados simbolicamente dentro do diretório `var/view_preprocessed`

`Developer`:
- Baixa performance
- Links simbólicos são estabelecidos na pasta `pub/static`
- Os erros são mostrados ao usuário e o registro é detalhado.
- Magento constrói código automaticamente para plug-ins (interceptores), fábricas, etc. como faz nos outros modos
- Arquivos estáticos são gerados em tempo real e são vinculados simbolicamente dentro do diretório `var/view_preprocessed`

`Production`:
- Melhor performance
- Os arquivos estáticos devem ser pré-compilados, pois nenhuma compilação acontecerá instantaneamente.
- Erros são apenas logados na pasta `var/log` e não são mostrados para o usuário

Para mais informações sobre os modos do Magento, tem [esta referência](https://devdocs.magento.com/guides/v2.4/config-guide/bootstrap/magento-modes.html) no DevDocs.

## Demonstrar habilidade com o gerenciamento de cache

### Descrever os tipos de cache e as ferramentas usadas para gerenciar o cache. 
Como você adiciona um conteúdo
dinâmico em páginas servidas com o full page cache?

### Descrever como operar a limpeza de cache.
Como você poderia limpar o cache? Em qual situação você limpa o
cache/descarrega o armazenamento do cache?

### Descreva como limpar o cache programaticamente.
Quais mecanismos estão disponíveis para limpar todo ou parte
do cache?
