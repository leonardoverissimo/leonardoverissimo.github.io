---
layout: post
title: O momento em que Java quase virou .Net e C++ ao mesmo tempo
tags:
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Isso já tem mais de um ano, mas em novembro de 2006 o arquiteto da Sun Danny Coward propôs a inclusão de propriedades em seu [blog](http://blogs.sun.com/dannycoward/date/20061102). Para quem está com preguiça de ler, ele sugeriu que a linguagem Java tivesse, ao invés de getters and setters, tivesse a expressão property, ficaria assim:

public property String foo;

E para a leitura de propriedades, seria feita dessa maneira:

a-&gt;Foo = b-&gt;Foo;

ao invés de:

a.setFoo(b.getFoo()) ;

Isso gerou uma polêmica desgraçada pra muitos javeiros, mas a crítica a ela se dividiu em algumas correntes:

1. Aqueles que acharam que isso seria copiar o .Net com sintaxe do C++. Pra mim, isso é bobagem. Java pode e deve ter influências de outras linguagens, basta, é óbvio, saber escolher aquilo que seja compatível com a filosofia do Java, e saber identificar os pontos fracos das soluções já implementadas. Dizer "isso não foi inventado aqui" não funciona, o Ruby é um exemplo de uma linguagem que não tem vergonha de dizer que se inspirou em outras linguagens, e olha que não é ruim não.

2. Aqueles que identificaram que propriedades não podem ser mapeadas com métodos. E com razão, eu poderia ter facilmente void setMyLink(URL myLink) e void setMyLink(String myLink) em uma mesma classe. Mas como eu mapearia isso para properties (que deve ter um comportamento similar ao de variáveis)? E sem falar que Beans tem um certo problema com coleções, pois não há nada que faça um objeto que possua uma List ter conhecimento da inserção e remoção de elementos desta List e que ao mesmo tempo esse objeto seja reconhecível pelos frameworks que exijem o padrão Bean.

3. Aqueles que acham que isso tiraria a programação explícita do Java. Eu não gosto muito de dogma, e acredito que se possa criar abstrações que facilitem a vida do desenvolvedor. O uso de propriedades poderia tirar deixar um pouco mais implícito, mas também não tornaria a linguagem confusa.

Mas uma coisa ninguém falou, pra quer melhorar os getters e setters se ela está decaindo a cada dia? É isso mesmo que você leu: DE-CA-ÍN-DO. Por várias razões:

1. Estou convencido de que nunca na história desse país, o uso de gettes e setters foi tão [criticado](http://blog.caelum.com.br/2006/09/14/nao-aprender-oo-getters-e-setters/). E isso porque uma classe com apenas getters e setters não é muito diferente de uma struct no C-zão. Se você duvida, procura no Google (e no GUJ também) sobre crítica aos getters e setters. Você vai até se deparar com as bibas Paulo Silveira e Philip Shoes, o "Calçado" tendo piti quando escutam que get e set faz com a aplicação fique orientado a objeto. É obvio que isso é mentira! Já participei de dois projetos onde havia um monte de "Bean", que todo mundo passava a mão, com um monte de "objetos" de "negócio" em volta deles fazendo muita merda. O primeiro, com o objetivo de substituir um legado com problema de manutenção, resultou num programa com problema de manutenção. E o segundo, resultou num programa tão matador que o projeto morreu antes de mostrar seu ROI.

2. Já reparou que o Hibernate pode ser usado com annotations? Já reparou que a JPA é só annotations? Já reparou que existe um container de IoC leve chamado Google Guice que pode ser usado só com annotations? Aquela dependência de getter e setter está caindo cada vez mais. Possivelmente seu uso só servirá para pegar e mudar valores do modelo e jogar na visão (isso se não inventarem outra coisa mais interessante pra fazer isso).

É claro que ainda haverá momentos em que o uso de getters e setters se torna "inevitável" (na verdade, para um desenvolvedor, inevitável significa algo que poderia ser impedido em situações normais, não fosse a preguiça de pensar, mais o chefe que só tá lá porque deu a bunda pro gerente, mais a falta de café e mais o prazo apertado), mas acredito que as situações possíveis vão diminuindo, à medida que vão surgindo alternativas interessantes.

Então resta a dúvida, pra que mexer em algo que já não é tão indispensável assim?
