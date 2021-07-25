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
**Em quais situações o controller frontal é envolvido na execução e como ele é usado nas customizações do escopo?**
