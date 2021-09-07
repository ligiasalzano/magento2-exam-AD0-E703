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

### Descrever como criar um novo arquivo de layout XML

### Descrever como transpor variáveis do layout para o bloco

## Uso do JavaScript na Magento

### Descrever diferentes tipos e usos de módulos JavaScript
**Quais módulos JavaScript são adequados para cada tarefa?**

### Descrever os UI components
**Em qual situação você deveria usar o UiComponent versus um módulo JavaScript comum?**

### Descrever o uso do `requirejs-config.js`, `x-magento-init` e `data-mage-init` 
