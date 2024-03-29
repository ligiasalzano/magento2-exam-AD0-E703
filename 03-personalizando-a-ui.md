---
title: Personalizando a Interface de Usuário (UI) do Magento
description: Personalizando a Interface de Usuário (UI) do Magento
permalink: /personalizando-a-ui
---

1. Demonstrar habilidade para usar temas e a estrutura de `templates`
2. Descrever como usar os blocos
3. Demonstrar habilidade de usar layout e `XML schema`
4. Uso do JavaScript na Magento
{:toc}

## Demonstrar habilidade para usar temas e a estrutura de `templates`

### Demonstrar capacidade para customizar a UI da Magento usando temas. 

- Os temas são localizados em `app/design`.
- Existem temas para o `frontend` e temas para o `adminhtml`
- Para criar um tema, são necessários os arquivos `theme.xml` e `registration.php`
- O arquivo `etc/view.xml` é requerido apenas se não houver um tema pai

Leia mais sobre a estrutura de temas do Magento [aqui](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/themes/theme-structure.html).

**Quando você deveria criar um novo tema?** 
Um tema possui um conjunto de personalizações no design do Magento e é criado conforme a necessidade de ter estas personalizações. Geralmente estendemos um tema existente (em última instância, os temas estentem o tema Magento Branck, existente no core da aplicação).
Trabalhamos no tema copiando e modificando templates existentes, criando XMLs para ajustar o layout e customizando o css (.less).

**Como você define a hierarquia do tema para seu projeto?**
O tema pai é especificado no nó `parent` dentro do arquivo `theme.xml`.


### Demonstrar a habilidade para customizar/depurar templates usando o processo de fallback do template. 

**Como você identifica qual é o exato arquivo de tema usado em diferentes situações?**
- Podemos habilitar as dicas de template do Magento (`bin/magento dev:template-hints:enable`)
- Pode-se buscar no projeto por alguma string do HTML (tradução, classe etc)
- Os arquivos de template podem serm `.phtml`, `.html` ou `.xhtml`

**Como você pode sobrepor arquivos nativos?**
- Encontre o template que quer sobrescrever
- Dentro do seu tema, crie uma pasta para o módulo ao qual o arquivo pertence. O formato é `Vendor_Module`
- Dentro deste diretório, copie o arquivo e o caminho para ele a partir da pasta `view/frontend`
- Obs: Se o arquivo estiver dentro de outro tema, que você está estendendo, então o arquivo e a estrutura de pastas dele


## Descrever como usar os blocos

### Demonstrar compreensão na arquitetura de blocos e seu uso no desenvolvimento.

**Quais objetos são acessíveis a partir do bloco? Qual é o papel característico de um bloco?**
Os blocos são classes PHP que fornecem informações para os templates `.phtml`. Eles ficam dentro do diretório `Block`, dentro do módulo.
A classe padrão dos blocos é a `Magento\Framework\View\Element\Template`.

> Nunca use `ObjectManager` em arquivos de template!


### Identificar as etapas no ciclo de vida de um bloco.

**Em que casos você colocaria seu código nos métodos `_prepareLayout()`, `_beforeToHtml()` e `_toHtml()`?**

> O bloco abstrato é encontrado em: vendor/magento/framework/View/Element/AbstractBlock.php

Etapas:
- criação do bloco (`__construct`)
- método `setLayout`
- Quando estiver pronto para obter o HTML:
  - é chamado o `toHtml`
  - é disparado o evento `view_block_abstract_to_html_before`
  - se houver um cache, o template não é carregado
  - `_beforeToHtml`
  - `_toHtml`
  - `_afterToHtml` (independete do cache)
  - é disparado o evento `view_block_abstract_to_html_after`
- O HTML é retornado

> Os plugins só podem modificar o fluxo de dados de métodos públicos. O melhor ponto para colocar um plugin é no método `toHtml`.

**Como você usaria os eventos disparados no bloco abstrato?**
Anteriormente, usava-se para modificar o fluxo de dados, porém as diretrizes do Magento recomendam usar plugins para isso.
Portanto, casos de uso destes eventos são limitados.

### Descrever como os blocos são renderizados e cacheados
O método `toHtml` é responsável por renderizar o bloco como HTML. 
Isso envolve o carregamento do template (se um template estiver sendo usado) e, se existe um chache, ele é retornado.

Um bloco é cacheado quando o tipo de cache `block_html` está habilitado e o `cache lifetime` não é `null`. Por padrão o `lifetime` não é definido.
Quando há cache, uma chave é gerada (`$this->getCacheKey()`). Parâmetros SID (IDs de sessão) são buscados no HTML e substituídos por marcadores antes da string ser salva.

### Identificar o uso de diferentes tipos de blocos
**Quando você usaria os tipos de bloco "sem template"? Em que situação você deve usar um bloco de template ou outros tipos de bloco?**

Tipos de blocos sem template:
- `Template`: Renderiza o HTML para o usuário - `vendor/magento/framework/View/Element/Template.php`.
- `Text`: Imprime uma string - `vendor/magento/framework/View/Element/Text.php`.
- `ListText`: Retorna o `output` de cada bloco filho, semelhante à um `container` - `vendor/magento/framework/View/Element/Text/ListText.php`.

Os casos de uso de blocos sem template são poucos. O bloco do tipo `Text` pode ser usado para mostrar uma string que foi definida pelo layout XML.

Existem vários tipos de blocos que estendem o tipo básico de templade de bloco. No diretório `vendor/magento/framework/View/Element` podemos encontrar alguns.

## Demonstrar habilidade de usar layout e `XML schema`

### Descrever os elementos do esquema de layout XML da Magento, incluindo as principais diretivas XML. 
**Como você usa as diretivas XML em suas customizações?**

O DevDocs possui um guia sobre o uso do layout XML e pode ser consultado [aqui](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/layout-overview.html).

Principais instruções de layout:
- `<block />`: cria um novo bloco, uma unidade que renderiza algum conteúdo distinto. A classe padrão de um bloco é a `Magento\Framework\View\Element\Template`. É recomendado sempre usar o atributo `name`, caso contrario será definido de forma aleatória.
- `<container />`: agrupa vários blocos e especifica a tag HTML e a classe que vai envelopar os blocos filhos. Também é recomendado usar `name`.
- `<arguments>` e `<argument>`: define um valor para o array `$data` do bloco.
- `<referenceBlock/>`: referencia um bloco existente para modificá-lo.
- `<referenceContainer/>`: referencia um container existente para modificá-lo.
- `<move/>`: permite mover um bloco entre containers.
- `<remove/>`: remove um bloco
- `<update/>`: adiciona um layout handle.


### Descrever como criar um novo arquivo de layout XML

Dentro do diretório `layout`, crie um arquivo .xml com o nome do `handle` que deve ser modificado. Por exemplo: `cms_index_index.xml`.
Crie o nó `<page>` e, dentro dele, as demais instruções do XML.

[Neste link](https://devdocs.magento.com/guides/v2.4/frontend-dev-guide/layouts/xml-manage.html), temos instruções sobre como realizar algumas tarefas comuns de customização de layout.

### Descrever como transpor variáveis do layout para o bloco

Isto pode ser feito através dos nós `<arguments>` e `<argument>`. Eles chamam o método `setData` no bloco.
E, dessa forma, pode-se recuperar esses valores posteriormente através do método `getData()`.

> Os nós `<arguments>` e `<argument>` também são usados para definir os `ViewModels`.

## Uso do JavaScript na Magento

### Descrever diferentes tipos e usos de módulos JavaScript
**Quais módulos JavaScript são adequados para cada tarefa?**

O Magento usa o RequireJS, criando um sistema modular e estensível.
Módulos JS podem estender um UI Component, ser customizados ou ser um jQuery widget. Também podem ser carregados ou embedados diretamente na página usando uma tag `<script>`. A tag `<script type="text/x-magento-init">` permite passar um JSON de configuração que carrega e inicializa o módulo com configurações geradas pelo lado do servidor.

Usar JavaScript and UI Components para fazer a interação entre o website e a aplicação PHP, como carregar dados específicos do cliente, permite que o Magento mantenha uma página totalmente em cache. É possível deixar a página toda cacheada e carregar informações específicas usando uma requisição AJAX.
E para demandas pontuais de manipulação de DOM, pode-se usar um módulo que cria um widget jQuery.

Já o atributo `data-mage-init` inicializa o módulo Javascript com a configuração fornecida pela tag. Pode ser usado para alterações pontuais.

### Descrever os UI components
**Em qual situação você deveria usar o UiComponent versus um módulo JavaScript comum?**

O uso de UI Components tem o propósito de criar um sistema modular e reutilizável para renderizar o layout nas áreas de front-end e adminhtml. 
Um novo componente pode ser criado e reutilizado quantas vezes forem necessárias.

### Descrever o uso do `requirejs-config.js`, `x-magento-init` e `data-mage-init` 

**`requirejs-config.js`:**
O RequireJS melhora os tempos de carregamento da página porque permite que o JavaScript seja carregado em segundo plano. Isto é, ele permite o carregamento assíncrono de JavaScript.
As configurações do RequireJS são feitas no arquivo `requirejs-config.js`. Nele podemos carregar arquivos e módulos JS e criar `aliases` para caminhos de módulos, por exemplo. No [DevDocs](https://devdocs.magento.com/guides/v2.4/javascript-dev-guide/javascript/requirejs.html#requirejs-config) a estrutura deste arquivo é descrita.

**`x-magento-init`:**
O `x-magento-init` é usado para inicializar um módulo JavaScript com parâmetros específicos.

Exemplo:
```html
 <div id="<carousel_name>" class="carousel">
     <div class="item">Item 1</div>
     ...
     <div class="item">Item n</div>
 </div>

 <script type="text/x-magento-init">
     {
         "#<carousel_name>": {
             "carousel": {"option": value}
         }
     }
 </script>
```

**`data-mage-init`:**
É similar ao `x-magento-init`, só que usado como uma tag HTML.

Exemplo (mesma funcionalidade do exemplo acima):
```html
  <div data-mage-init='{"carousel":{"option": value}}'>
     <div class="item">Item 1</div>
     ...
     <div class="item">Item n</div>
 </div>
```

Para mais exemplos, acesse [Calling and initializing JavaScript](https://devdocs.magento.com/guides/v2.4/javascript-dev-guide/javascript/js_init.html).
