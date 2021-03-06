---
layout: post
title: 'Seção: Apanhando da... UML (ao vivo)'
tags:
- Ao vivo
- Apanhando do
- metodologias
- UML
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Olá, semana passada eu apresentei o Wicket e prometi fazer algo melhor que aquele Hello World chinfrim. Porém, não consigo fazer nada sem planejamento prévio. Meu cérebro (e acredito que o seu também) não consegue fazer uma coisa muito complicada que envolve muitas variáveis ao mesmo tempo. É como diria a Gostosinha Burrinha: "Que-nem Jack 'Estuprador' Vamos por partes!"  Uma vez eu li um livro meio intelectual, meio povão chamado "Blink" que fala exatamente sobre como as pessoas conseguem realizar suas atividades melhor com um mínimo de informações. E sigo isso à risca.

Uma frase que eu gosto de repetir é:

_Eu admiro as pessoas que conseguem fazer suas aplicações na raça, sem planejamento. Contanto que não seja eu a fazer a manutenção depois._

Portanto, muito melhor planejar-se antes de fazer, e muito melhor ainda é usar a UML para o nosso planejamento.

"Mas peraí, o que esse 'ao vivo' do título tem a ver?"

Calma! É que nada está pronto pra essa apresentação e estou fazendo o sistema enquanto vocês leêm isso aqui. Esse post só tem a introdução, nos próximos posts eu mostro o meu progresso.

<!--more--> Modelei um sistema de biblioteca. É simples, mas com algumas regras chatinhas que fazem ir além de um CRUD. A biblioteca deve permitir cadastro de livro, registro de empréstimo e devolução de exemplares, além de reserva de empréstimo de livro. Repare que no sistema, livro e exemplar são coisas diferentes: exemplar é uma representação impressa de um livro, podendo haver vários exemplares para o mesmo livro; e livro é a idéia de um autor representado em palavras. Existem regras de limite máximo de empréstimo, e de multa para atraso de entrega.

Eu fiz ontem um modelo em casos de uso do sistema

![biblioteca.jpg](/assets/2007/09/22/biblioteca.jpg)

É claro que esse modelo não foi assim desde a primeira vez, houve um rascunho original que evoluiu até chegar a ese ponto (até porque não tenho assim tanta experiência em modelagem UML). No começo, eram apenas os balõezinhos independentes, a partir daí fiz [um documento que descrevia os casos de uso](/assets/2007/09/22/sistema-biblioteca.pdf). À medida que ia detalhando, percebi que havia coisas comuns entre os casos de usos e passei a decompô-los usando &lt;&lt;include&gt;&gt; e &lt;&lt;extend&gt;&gt;. Nesse ponto eu já estou inseguro quanto a modelagem, usei a convenção de que, com &lt;&lt;include&gt;&gt;,  o caso de uso que inclui deve ('must') usar o caso de uso incluído, e de que, com &lt;&lt;extend&gt;&gt;, o caso de uso estendido pode ('might') usar o caso de uso que estende. É assim que está no livro "Object-Oriented Analysis and Design with Applications" de Grady Booch, mas no livro "Modelagem e Projetos em Objectos com UML 2" de Michael Blaha e James Rumbaugh o negócio é mais confuso, parece ser diferente.

Há um problema que eu só vi hoje, não existe um caso de uso para cadastro de usuários. Então, pra daqui a pouco (pode ser hoje ou pode não ser) vou refazer esse caso de uso em um outro editor de UML acrescentando esse caso de uso sumido. (eu comecei esse diagrama no NetBeans 6.0 Beta 1, mas eu descobri que a parte de UML é a que tá mais podre nessa IDE, não dá pra fazer um diagrama de classes sem que os formulários deêm pau. E também tá me incomodando o fato de as classes seguirem mais a convenção Java do que a convenção do UML 2. Acredita que o editor põe automagicamente getters e setters quando você liga duas classes em uma associação? Isso é escroto!) E vou fazer também uma implementação, de apenas um caso de uso, usando a JPA com Hibernate (mais uma pro TODO list: baixar os jars do Hibernate).

Bom, até lá.
