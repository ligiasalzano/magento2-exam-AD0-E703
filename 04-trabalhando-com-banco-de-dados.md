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
