---
layout: post
title: Ruby, Top of the Pops
tags:
- Sem-categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

A Edição número 51 de Java Magazine troxe um artigo sobre JavaFX nada excitante, pois mostrou um formzinho bobo, ao invés de um desenho 2D interativo, que eu acredito ser esse seu forte. Mas isso é assunto pra outro post, o mais me irritou no artigo foi a seguinte declaração (tudo bem que se fala de linguagens dinâmicas em geral, mas acerta em cheio o Ruby):<!--more-->

"A utilização de outras linguagens dentro da plataforma Java está se tornando cada vez mais comum, e podemos dizer que se trata de um bom casamento de conveniência. Para o Java, este relacionamento permite unir esforços para tarefas nas quais há uma sinergia forte com uma determinada linguagem (avaliação de regras de negócio, processamento de expressões regulares, clientes web etc.). Já para a outra linguagem, é uma espécie de 'golpe do baú': toda a infra-instrutura da plataforma Java - bibliotecas, frameworks, containers, padrões e tudo o mais - vêm de graça."

O que? Casamento de conveniência? Só se for pro Java! Esse discursinho de sinergia é balela, os caras da Sun trouxe a equipe do jRuby para o seu chapéu porque estavam cagando de medo do Rails ganhar popularidade em aplicações de pequeno e médio porte. Para a Sun, eu vejo a "união" como uma espécie de "vão-se os anéis, ficam-se os dedos", é como se eles dissessem: "Olham, ex-adeptos do Java que migraram pro Ruby, vocês podem mudar suas preferências de linguagem. Mas que tal usar algo que roda em cima da JVM, hein? Vocês podem ainda usar aquelas APIs Java que vocês estão acostumados quando quiserem!". Assim, se Java cair de desuso, a JVM estaria firme e forte. E o Ruby não está dando golpe do baú no Java, pois se quiser, pode rodar em cima de Linux, Apache (ou lighttpd) e MySQL. Java EE é algo totalmente opcional.

Bom, mas o que essa linguagem tem de tão especial? Tecnicamente é uma linguagem com tipagem dinâmica, com objetos abertos (dá pra adicionar métodos a uma classe depois que ela foi declarada), totalmente orientado a objetos - sem aquela diferência entre tipos primitivos e objetos, uma aberração que parece existir somente no Java -, e com características muito fortes de meta-programação. Seu framework, o Ruby on Rails (ou Rails) possui coisas revolucionárias em relação ao universo Java, como gerador de código bom (porque no Java não tem, né! Os que existem, são frameworks proprietários geradores de código escroto!), nada de programação em camadas (essa praga!), zero configuração, DRY (não se repita!), utilização correta do protocolo HTTP (requisição GET é pra consulta e requisição POST é pra enviar formulários, alterando o estado do servidor. Sabia disso, JavaServer Faces?) e uso do protocolo REST.

Mas sucesso não se conquista somente no lado técnico. Claro que as linguagens muito feias, como PL/SQL ou Perl, tem tudo pra não conquistar ninguém, mas ser uma linguagem bonita não basta. Muitas linguagens boas morreram por ter uma "fala pequena" com o público (entenderam a piada? fala pequena... small não-sei-do-que...). Até hoje, Java era a única linguagem que eu vi que possui fãs. Há o Linux também. Mas Ruby estrapolou isso, é, além de uma linguagem boa e que possui fãs, uma linguagem "cool", e por isso mesmo pop. Nela, nada parece sisudo. Nenhuma linguagem diz, em [seu site principal](http://www.ruby-lang.org/en/) que é "o melhor amigo do programador".

E o seu livro principal? O [Agile Web Development with Rails](http://pragprog.com/titles/rails2/) usa uma historinha entre um experiente desenvolvedor e uma cliente pra construir a aplicação em frente aos nossos olhos (repare que a cliente é uma mulher, pois o autor, apesar de nunca revelar o nome, usa sempre "she". Isso explica o porquê dele ser sempre atencioso...). E tem um [livro com cartoons](http://poignantguide.net/ruby/) também. No Java, as coisas são mais sisudas, com exceção, é claro, da série Head First (onde tem aquela moreninha gostosa que fica implicando com tudo o ques os autores falam), mas é diferente, tem que ficar explicando no início do livro por que eles são desse jeito e não algo mais sério, como se esperassem que alguém do Java torcesse o nariz. O Ruby, as coisas desencanadas surgem mais naturalmente.

Quer ver? Sabia que existe o [Ruby Quiz](http://www.rubyquiz.com/), onde o pessoal desafia os <del>burros</del> iniciantes na linguagem? E se fazer quebra-cabeças não bastasse, que tal entrar num [concurso anual](http://www.railsrumble.com/prizes) onde você e mais três amigos se juntam pra fazer uma aplicação com Rails em 48 horas de sábado e domingo? Coisa de nerd, né? Não viu nada! Dois geeks fizeram [uma série de vídeos](http://www.railsenvy.com/) comparando Ruby, com PHP, Java, .Net... E uma piada de primeiro de abril indica, Rubyista é um [Emo Programmer](http://headrush.typepad.com/creating_passionate_users/2006/04/announcing_the_.html).

E se quiser algo útil, pode é claro, acessar o [RubyForge](http://rubyforge.org/)!

Eu já não sei se toda essa frescura em torno do Ruby é bom ou não, eu não estou nem aí. Gosto do Ruby, e muito mais do Rails, pois foi com esse framework que eu aprendi a programar em web de verdade.
