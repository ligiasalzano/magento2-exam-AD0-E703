---
title: Trabalhando com banco de dados na Magento
description: Trabalhando com banco de dados na Magento
permalink: /trabalhando-com-banco-de-dados
---

1. Demonstrar habilidade de usar classes relacionadas a dados
2. Demonstrar habilidade para usar o esquema declarativo (`declarative schema`)
{:toc}

## Demonstrar habilidade de usar classes relacionadas a dados

### Descrever repositories e classes de API de dados

Repositórios são usados para carregar entidades no Magento. Eles fornecem aos _service requestors_ a capacidade de executar operações de criação, leitura, atualização e exclusão (CRUD) em entidades ou em uma lista de entidades. Por fim, o repositório é um exemplo de contrato de serviço.
Repositórios também são parte da API do Magento.

Leia mais sobre repositórios [neste artigo do Magenteiro](https://www.magenteiro.com/blog/magento-2/como-usar-o-repository-pattern-no-magento-2?mid=c9f0f895fb98ab9159f51fd0297e236d) e [nesta página do DevDocs](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/searching-with-repositories.html).

**Como você obtém um objeto ou um conjunto de objetos do banco de dados usando um repository?**

Para obter um objeto do banco de dados, usamos o método `getById`. E, para pegar vários objetos, usamos o método `getList` juntamente com a classe `SearchCriteriaInterface`.

> O nome da tabela é adicionado na camada `resource` com o método `$setup->getTable('table_name')`.

**Como você configura e cria uma instância de SearchCriteria usando o construtor? Como você usa as classes Data/Api?**

As `interfaces` são armazenadas no diretório `Api` do módulo. Classes relacionadas diretamente aos dados armazenados no banco são colocadas no diretório `Api/Data`.
Usamos a classe `SearchCriteria` através de sua interface, a `Magento\Framework\Api\SearchCriteriaInterface`.
Para construir a requisisão, usamos a classe `Magento\Framework\Api\SearchCriteriaBuilder` injetando-a no construtor: 

```php
use Magento\Framework\Api\SearchCriteriaBuilder;
use Magento\Framework\Api\SortOrder;
use Magento\Framework\Api\SortOrderBuilder;

private $productRepository;
private $searchCriteriaBuilder;
private $sortOrderBuilder;

public function __construct(
    ProductRepositoryInterface $productRepository,
    SearchCriteriaBuilder $searchCriteriaBuilder,
    SortOrderBuilder $sortOrderBuilder,
    Context $context,
    array $data = []
) {
    parent::__construct($context, $data);

    $this->productRepository = $productRepository;
    $this->searchCriteriaBuilder = $searchCriteriaBuilder;
    $this->sortOrderBuilder = $sortOrderBuilder;
}
```

Abaixo, segue um exemplo de uso com o repositório de produto criado [neste artigo do Magenteiro](https://www.magenteiro.com/blog/magento-2/como-usar-o-repository-pattern-no-magento-2?mid=c9f0f895fb98ab9159f51fd0297e236d).
```php
public function getProducts()
{
    // Filter products weighing 10kg or more
    $this->searchCriteriaBuilder
        ->addFilter('weight', 10.0, 'gteq');
    // Sort products heaviest to lightest
    $sortOrder = $this->sortOrderBuilder
        ->setField('weight')
        ->setDirection(SortOrder::SORT_DESC)
        ->create();
    $this->searchCriteriaBuilder->addSortOrder($sortOrder);
    // Get the first 5 products
    $this->searchCriteriaBuilder
        ->setPageSize(5)
        ->setCurrentPage(1);
    // Create the SearchCriteria
    $searchCriteria = $this->searchCriteriaBuilder->create();
    // Load the products
    $products = $this->productRepository
        ->getList($searchCriteria)
        ->getItems();
    return $products;
}
```


### Descrever como criar e registrar novas entidades
Como você adiciona uma nova tabela ao banco de dados?

Podemos adicionar uma nova tabela ao banco de dados de duas formas: usando o `declarative schemal` (preferencialmente, a partir da versão 2.3 do Magento) ou usando a forma antiga, com as classes `Setup/InstallSchema` e `Setup/UpgradeSchema` (versões do Magento anteriores à 2.3).

#### Usando o `declarative schema`

O arquivo de configuração é criado em: `<Module_Vendor>/<Module_Name>/etc/db_schema.xml`. Nele declaramos a estrutura do banco de dados do módulo.

Exemplo:
```xml
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
    <table name="declarative_table">
        <column xsi:type="int" name="id_column" padding="10" unsigned="true" nullable="false" comment="Entity Id"/>
        <column xsi:type="int" name="severity" padding="10" unsigned="true" nullable="false" comment="Severity code"/>
        <column xsi:type="varchar" name="title" nullable="false" length="255" comment="Title"/>
        <column xsi:type="timestamp" name="time_occurred" padding="10" comment="Time of event"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="id_column"/>
        </constraint>
    </table>
</schema>
```

Após criar o arquivo `db_schema.xml`, deve-se gerar o `schema whitelist` do módulo: `bin/magento setup:db-declaration:generate-whitelist [options]`. 
`[options]` pode ser: `--module-name[=MODULE-NAME]` ou `--module-name=all`.

> Você pode obter mais informações sobre o uso do `declarative schemas` em [**configure declarative schema**](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/db-schema.html) e sobre o `schema whitelist` em [**create a schema whitelist**](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/migration-commands.html#create-whitelist).


#### Usando os scripts PHP (a forma antiga)

- InstallData e InstallSchema são executados uma vez apenas, quando o módulo é instalado.
- UpgradeData e UpgradeSchema são executados quando a versão do módulo é alterada.
- Estas classes são criadas dentro do diretório `Setup` do módulo.

Neste [artigo do Mageplaza](https://www.mageplaza.com/magento-2-module-development/magento-2-how-to-create-sql-setup-script.html) você pode encontrar um tutorial sobre como usar estes scripts.

> Leia mais em [Migrate install/upgrade scripts to declarative schema](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/declarative-schema/migration-commands.html).


### Descrever como ocorre a leitura e gravação de uma entidade

### Descrever como estender entidades existentes
Quais mecanismos estão disponíveis para estender as classes existentes, por exemplo, adicionando um novo atributo, um novo campo no banco de dados ou uma nova entidade relacionada?

### Descrever como filtrar, ordenar e especificar os valores retornados nas collections e repositories
Como você seleciona um subconjunto de registros do banco de dados?

### Descreva a camada de abstração do banco de dados da Magento
Que tipo de exceções são geradas pela camada de banco de dados?
Que funcionalidade adicional a Magento oferece sobre o Zend_Adapter?

## Demonstrar habilidade para usar o esquema declarativo (`declarative schema`)

### Demonstrar o uso do esquema
Como manipular colunas e índices usando o declarative schema? 
Qual o propósito da whitelisting? 
Como usar os patches de dados e esquema? Como gerenciar as dependências entre os arquivos de patch?
