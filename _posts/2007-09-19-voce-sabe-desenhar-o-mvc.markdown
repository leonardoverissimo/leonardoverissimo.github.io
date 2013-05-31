---
layout: post
title: Voce sabe desenhar o MVC?
tags:
- Java
- Padrões
- Sem-categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Ei! Você programa em Web? Ou então programa em Swing? Ou faz telas em C#.NET?  Então você já deve ter ouvido falar em MVC?!

> MVC? Eh... Ah! Claro! O MVC! He... he... Como eu poderia deixar de esquecer!

Então, você seria capaz de explicar para a platéia o conceito por trás do MVC? (Perái, eu disse "conceito por trás do"? Credo! Eu estou parecendo aqueles pseudo-intelectuais do GUJ!)

> Eh? Agora? Assim de improviso?

Isso.

> Ah não! O MVC é um negócio muito complicado!

Meu, não é mais fácil admitir que você não sabe? Não vai doer.

> Tá bom. Eu ouvi falar mas eu não sei o que que é.

Bom, calma. Você não é o único. Se bobear eu tambem não sei direito! E eu gostaria de lançar um desafio pra mim, pra você e pra todos vocês. Desenhem o esquema do MVC, sem fazer consulta a nenhum livro ou site, e expliquem-no! Tá, também não precisa responder pra mim com o seu desenho, é só pra ver se vocês mesmo sabem. Eu vou fazer o meu, sem consultar fonte nenhuma, e mostrá-los pra vocês. E aí, farei uma consulta na web e nos meus livros pra ver se está certo.

Portanto, não cliquem no more antes de terminar esse exercício (a menos que você realmente não queira fazer, óbvio).

<!--more-->Bom, vamos lá. O mais fácil de tudo é fazer as caixinhas. (É, eu sou um tanto incompetente na hora de inserir imagens, vai ter que clicar no desenho pra ter uma visualização melhor)

![mvc-1.jpg](/assets/2007/09/19/mvc-1.jpg)

Agora eu digo que Controller conhece View e Model, pois precisa chamá-los quando o usuário muda os dados na View.

![mvc-2.jpg](/assets/2007/09/19/mvc-2.jpg)

Aí,  eu digo que a View irá notificar o Controller quando seu estado mudar. Repare abaixo que é uma linha tracejada para indicar que a View não tem dependência do Controller, a implementação pode ser tanto através do padrão Observer, quanto naquele esquema em que o Controler fica dizendo pra View: "E aí View, tá pronto?" "Não." "E aí View, tá pronto?" "Não." "E aí View, tá pronto?" "Não!" "E aí View, tá pronto?" "NÃÃÃÃÃÃÃÃO!!!", infinitamente.

Também digo que a View conhece o Model, para poder obter o seus dados (não os SEUS dados, os dados do Model) e apresentá-los.

![mvc-3.jpg](/assets/2007/09/19/mvc-3.jpg)

Pra finalizar, digo que o Model é observável, e que o Controller vai observá-lo, e quando houver alguma alteração,  vai dizer pra View que algo mudou. Repare também que eu errei na setinha, era pra fazer uma tracejada e fiz uma contínua, como eu estava sem branquinho e tava com preguiça de fazer a alteração no editor visual, ficou essa barbeiragem aí embaixo.

![mvc-4.jpg](/assets/2007/09/19/mvc-4.jpg)

Bom, eu fiz esses desenhos sem ver se está certo. Até porque não tem graça se eu ficar fazendo isso consultando o livro ao mesmo tempo. Fui verificar na Wikipedia em [Model-View-Controller](http://en.wikipedia.org/wiki/Model-view-controller) e está ERRADO! É certo até na imagem 3, na 4 eu fiz uma barbeiragem, olha como é que é o desenho deles:

![MVC Diagram](http://upload.wikimedia.org/wikipedia/en/7/7c/ModelViewControllerDiagram.png)

O Model notifica a View quando ocorre alterações, não o Controller! E faz sentido, porque quando ocorre uma alteração no Model, basta a View consultá-lo e fazer as alteracões (tipo o que o Beans Binding irá fazer automaticamente pra você). O Controller  só precisa entrar em ação quando ocorre eventos do usuário.  Como diria o slogan da Coca-Cola Light: "Por essa você não esperava!".

E vocês, fizeram certo? Errado? Não fizeram?
