---
title: Browsers de Propósito Específico com o AngularJS
---

Eu conheci o termo **browser de propósito específico** em um [artigo](http://boletim.de/silvio/previses-5-uma-rede-de-apps/) do professor Silvio Meira no qual ele faz uma analogia bastante interessante entre apps e browsers:

> Só que apps são como browsers: o app de Facebook é uma espécie de browser de um sistema de informação (ou web fechada, se você quiser) chamado... Facebook.

Nós podemos levar esta analogia adiante desenvolvendo aplicações (apps ou aplicações web) baseadas na restrição [HATEOAS (Hypertext As Then Engine Of Application State)](https://en.wikipedia.org/wiki/HATEOAS) do estilo arquitetural [REST (Respresentational State Transfer)](https://en.wikipedia.org/wiki/Representational_state_transfer).

Este é o primeiro de uma série de artigos nos quais eu pretendo explicar como desenvolver browsers de propósito específico com o framework AngularJS.

# A Barra de Endereços
Todo browser possui uma barra de endereços, ou seja, um campo onde o usuário digita o endereço de um recurso cuja representação ele deseja obter. Para implementar a barra de endereços de nosso browser de propósito específico, vamos usar um recurso pouco difundido, porém muito útil, do módulo [ngRoute](https://docs.angularjs.org/api/ngRoute): a definição de uma rota pode conter um mapa chamado `resolve`. Os valores deste mapa são funções que retornam promessas. Quando uma rota é ativada, o AngularJS executa as funções e espera até que todas as promessas retornadas sejam resolvidas para, só então, executar o controlador da rota e, em seguida, exibir seu template.

No código abaixo, por exemplo, quando `/rota1` é ativada, o AngularJS executa as funções associadas às chaves `recurso1` e `recurso2`. Estas funções retornam promessas que serão resolvidas nos resultados das requisições `GET rest/recurso1` e `GET rest/recurso2`, respectivamente. Quando as duas promessas são resolvidas, o AngularJS executa o controlador de `/rota1`, o qual acessa os resultados das requisições no mapa `$route.current.locals` e os armazena no escopo, permitindo assim que o template exiba os resultados.

{% highlight javascript %}
{% raw %}
var modulo = angular.module("modulo", ["ngRoute"])

modulo.config(function($routeProvider) {
  $routeProvider.when("/rota1", {
    resolve: {
      recurso1: function($http) {
        return $http.get("rest/recurso1")
      },
      recurso2: function($http) {
        return $http.get("rest/recurso2")
      }      
    },
    controller: function($route, $scope) {
      $scope.recurso1 = $route.current.locals.recurso1
      $scope.recurso2 = $route.current.locals.recurso2
    },
    template: "<p>{{recurso1}}</p><p>{{recurso2}}</p>"
  })
})
{% endraw %}
{% endhighlight %}

# Um resolve genérico