---
layout: post
title: A caveira de Craig McClanahan (e o porquê do XML ter sido tão importante)
tags:
- Java
- Sem-categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---
É piada corrente (piada?) de que a única coisa que Craig fez de bom foi o Tomcat, o resto (Struts e JSF) foi um lixo. Não é bem assim, temos a mania de olhar as coisas do passado com os olhos do presente e criticar os criadores por não terem uma maldita bola de cristal para saberem o que aconteceria no futuro se fizessem desse ou de outro jeito.

Bruno Borges chamou de "[Tríplice Aliança](http://pt.wikipedia.org/wiki/Tr%C3%ADplice_Alian%C3%A7a_%28Primeira_Guerra_Mundial%29)" a referência de um objeto via programação java, página JSP e arquivo XML (Mas o que interessa na verdade é o "meio-de-campo", ou seja, XML.). Porém, considerando o ano de 2000, quando o Struts surgiu (ao mesmo tempo que o então chamado J2EE surgiu), haveria para os frameworks uma alternativa ao XML?<!--more-->

Havia, é o padrão [Template Method](http://en.wikipedia.org/wiki/Template_method_pattern): que é um algoritmo implementado numa classe abstrata, parte do framework, e ações de verdade implementados numa classe concreta, parte do aplicativo do usuário. Assim, os usuários escrevem apenas as suas ações, sem se preocupar em como elas seriam conectadas entre si, que é responsabilidade da classe base.

Mas se é uma idéia aparentemente boa, por que o J2EE não a seguiu? Seguiu, até hoje os frameworks seguem esse padrão, porém não mais no seu jeito clássico. Então porque o J2EE não a seguiu do seu jeito clássico? Bem, as primeiras tecnologias em Java eram no Template Method. Lembra do Applet? Quem já não encarou os métodos init() e paint() de uma classe que estendia Applet? Pois é, isso é Template Method implementado de maneira pura, pois havia a classe base que usava esses e outros métodos para prover o suporte ao Applet. As primeiras tecnologias do Enterprise (o Enterprise do Java, Capitão Spock!) tinham um pedaço no XML e mais um pedaço de Template Method, eram assim com Servlet (com doGet() e doPost()) e com o EJB (ejbActivate(), ejbPassivate() e outros).

Nenhuma das três tecnologias "pegou". O Applet é um fracasso total (e é o único fracasso que o fanático do Java é capaz de admitir), e qualquer um torce o nariz pro EJB ou pra programação puro-Servlet (preferindo usar JSP com "PHP way of coding" ou os frameworks de então, como Struts). Uma das desvantagens de se contar com o Template Method de forma pura é que força nomes artificiais para os métodos. E havia um característica da própria linguagem Java de, por ser uma linguagem imperativa, o programador não dizer **o que** se faz, mas **como** se faz. Não há nada de errado nisso, mas certos aspectos ficam prejudicados, por exemplo, se alguém quisesse mexer em alguma configuração, teria, no mínimo, que: obter um container de um singleton, pedir um objeto X desse container, invocar no objeto X um método que altere uma característica da configuração (possivelmente passando um outro objeto) e tratar o retorno e as exceções.

O XML surgui como o salvador da pátria nessa época (lembro da época da faculdade o ufanismo dos javeiros por essa linguagem), era o aspecto de programação declarativa que faltava para o Java: acabou os pequeninos detalhes que o desenvolvedor tinha que resolver, basta escrever uma linha no arquivo XML e tudo será resolvido. Foi assim que a comunidade abraçou o Struts e o Spring, ambos com configuração fortemente baseada em tags. Houve um certo benefício, pois os objetos de usuário ficaram mais limpos, já que parte de sua lógica não estava mais em classes Java. Os desenvolvedores, no início, viam isso como algo bom.

Precisavam configurar A? Coloca A no XML.

Precisavam configurar B? Coloca B no XML.

> Mas puxa, já cheguei a letra Z, o que eu faço?

Fácil, faz que-nem número, escreva AA, depois AB e assim por diante, se acabar de novo, faz AAA, AAAA e por aí vai.

> Mas eu já cheguei à configuração ZDHUDSALEDOEMWRTJXZQWXSPDFWPDVVBJ!!! E tudo isso em um único arquivo XML!

Pois é, demorou um tempo pro povo perceber que havia algo errado em se contar tanto com o XML. A W3C imaginou o XML como um documento onde organizações diferentes pudessem enviar dados entre si de maneira estruturada, não como um jeito de configurar as coisas. Não era apenas um repositório de dados, havia lógica dentro das tags que não estava sendo vista pelo compilador Java.

Foi aí que o pessoal se tocou e começou a criar alternativas, como os annotations e novos frameworks baseados nessa "feature",  que trouxe a configuração de maneira declarativa, próxima ao código e aos olhos do compilador.

Essa nova mudança trouxe o Struts para o limbo, qualquer framework web com XML menor era melhor. Mas o mais impressionante não foi o fato do Struts cair do céu para o inferno, foi a rapidez com que isso aconteceu, não durou nem cinco anos! E talvez por isso que povo de Java queira a caveira de Craig McClanaham.

Mas pera lá! O Craig não tinha como fazer previsões, não tinha uma JVM tão boa com reflexão (em termos de desempenho) quanto é hoje, não teve a sua disposição "insights" que só seriam descobertos mais tarde, não tinha as annotations. Ele fez aquilo que era bom para a época. O Struts não é só desvantagem, ele fez com Java para Web virasse sinônimo de MVC, abriu caminho para a era dos frameworks, e levou o J2EE para um caminho que talvez nem a Sun previa.

O povo também não se conforma com o fato de Craig ter entrado na equipe de especificação do JSF: "Errou uma vez! Vai errar de novo!" Não acho, o JSF aprendeu as lições do Struts: possui actions (ou seria managed beans?) sem dependência de container, e, apesar de depender de XML (afinal surgui antes do Java 5), é uma especificação extremamente extensível, sendo possível reimplementar seu esquema de navegação, ampliar seu ciclo de vida, criar novos escopos de beans (muitos fornecem conversation e flash), mudar o renderizador da visão e fornecer componentes visuais de alto nível.

Não acho correto quando as pessoas falam mal de Craig McClanahan, criar um sistema que possa ser usado por décadas sem grandes problemas é extremamente difícil, reservado para alguns poucos gênios. Seria algo que nem os próprios críticos seriam capazes de fazer. Portanto, deixa o cara em paz, Struts não foi o sistema perfeito, mas foi a mão na roda pra muita aplicação web que surgi por aí.
