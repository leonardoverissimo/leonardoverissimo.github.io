---
layout: post
title: Apanhando da... UML (ao vivo) (2)
tags:
- Apanhando do
- metodologias
- UML
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Martin Fowler disse sobre certos maus cheiros no código. Pois bem, aconteceu comigo!

Eu não sei por que eu tive essa idéia, mas achei que se eu usasse JDBC puro seria mais fácil. Nossa senhora, parecia cheiro de quem comeu quilos de ovo, repolho e batata-doce! E pra piorar, havia um certo probleminha que eu passei por cima: livro pode ter múltiplos autores e autor pode ter muitos livros, então deveríamos ter uma entidade livro, uma entidade autor e um relacionamento muitos-para-muitos entre eles. Não foi o que eu fiz, coloquei uma lista de String na classe Livro e achei que isso mapearia fácil para um relacionamento na base onde realmente tem uma tabela "autores", porém como seria a inserção de um livro em um caso desse? O que eu quase ia fazer era pedir uma busca na tabela "autores" passando o nome do autor para retornar o id (uma busca full scan!) e se retornasse algum id, eu iria fazer a inserção. A atualização teria complicações piores que é melhor nem comentar.

<!--more-->Vou trocar o código usando JPA. Não foi muita coisa que eu perdi, até porque eu não fiquei todas essas horas entre o último post e este digitando o código. E ainda por cima, eu fiz um novo caso de uso no Jude e um diagrama de classes do caso de uso "Atualizar registro de livros".

Veja abaixo:

![biblioteca-usecase2.jpg](/assets/2007/09/22/biblioteca-usecase2.jpg)

![biblioteca-classdiagram.jpg](/assets/2007/09/22/biblioteca-classdiagram.jpg)
