---
title: Arquitetura Magento e Técnicas de Customização
description: Arquitetura e técnicas de customização do Magento 2
permalink: /arquitetura-e-customizacao
---

1. Descrever a arquitetura baseada em módulos da Magento
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

