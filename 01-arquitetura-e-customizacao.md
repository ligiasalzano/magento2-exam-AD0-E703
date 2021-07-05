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

## Uso de XML de configuração e variáveis de escopo
### Determine como usar os arquivos de configuração. Quais arquivos de configuração correspondem a diferentes funcionalidades?

O Magento divide as configurações em vários arquivos. Dessa forma, evita um arquivo muito grande (como ocorria no Magento 1).
