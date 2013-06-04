---
layout: post
title: O uso de sessão em aplicações web
tags:
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  dsq_thread_id: ''
---
Há uma corrente de pensamento que diz que aplicações web deveriam ser  _stateless_. É um pouco confuso, pois certos requisitos naturalmente possuem estado; porém, ser _stateless_ não significa ignorar que o mundo real possuem transições, significa que o armazenamento do estado deve ser empurrado o máximo possível para o lado cliente.

Mas programadores Java EE não tão nem aí. O [HttpSession](http://java.sun.com/javaee/6/docs/api/javax/servlet/http/HttpSession.html) é tão acessível e tão simples de usar que o programador não pensa duas vezes antes utilizá-lo. E talvez aí seja a fonte dos problemas: usa-se demais! As pessoas não se dão conta de que é difícil escalar aplicações web que usam muito _Session_; e que também pode ir contra a uma boa experiência de usuário, como o voltar ou o _favoritar_ que deixa de funcionar. Acredito que existam alternativas tão melhores que o _HttpSession_ que este só seria raramente lembrado. Veja as seguintes situações e suas alternativas.
<!--more-->

##  Preciso fazer cache de entidades <a id="cache">&nbsp;</a>

Veja, o cache é sobre entidades obtidas através de classes da camada de persistência; mas a _HttpSession_ é uma classe de arquitetura que somente a camada de apresentação depende. Entretanto, ao guardar e obter objetos na sessão de usuário, a camada de apresentação está ido além de seu escopo, que é simplesmente mostrar a tela para o usuário; é uma solução ruim.

Uma solução boa é usar bibliotecas de cache de verdade como [Memcached](http://memcached.org/) ou [JBoss Cache](http://www.jboss.org/jbosscache). Pois você pode colocá-las na camada de persistência, e ainda assim, não se amarrar à tecnologia Web. Sem falar que o cache, com essas ferramentas, é único por todo o sistema (e não único por usuário, como na _Session_), o que lhe garante maior controle sobre os dados não-atualizados.

Minha dica é, se não houver uma configuração de cache automatizado, como no [Hibernate](http://www.hibernate.org/), ponha-os em classes que funcionam como _decorator_ de DAOs, mas isso [eu já falei antes](http://www.objectzilla.com.br/2009/05/02/ja-usou-memcached/).

## O framework web guarda dados em sessão


Algumas _vendors_ gostam de fazer ferramentas que sejam _jumento-proof_, com o intuito de convencer gerentes de que é possível contratar programadores a preço de banana. Muitos _frameworks_ web já saem com essa mentalidade na sua concepção, e é tão _jumento-proof_ que a ferramenta faz uso constante da _HttpSession_ internamente sem o programador perceber.

Pra não usar sessão nesses casos, só há um jeito: troque de framework web, para um que nunca faça uso implícito de sessão. Porém, se você está no meio de um projeto, ainda é possível criar novas páginas com um novo framework, mas mantendo as antigas, só fazendo a transição quando estas precisarem de alterações. Ah, e se onde você está exigem o uso de uma "arquitetura padrão"? Então troque de emprego, porque ninguém merece, né?

## O objeto que eu estou armazenando vai ser usado na próxima tela


São duas situações bastante comuns:

1. existe uma tela onde aparece uma lista de um domínio qualquer; dado o clique de um elemento, uma nova página com os detalhes aparecerão;

1. existe uma página com campos "somente-leitura", onde o clique de um _link_ muda para uma página com campos editáveis.


Nos dois casos, um desenvolvedor comum percebe logo que, como a segunda página irá requisitar os mesmos dados que a primeira, coloca os dados da primeira requisição na sessão para ser consultado depois. Não faça isso! Você pode estar achando que está economizando no banco, mas está negligenciando outros fatores. Exemplos: é mais fácil escalar um banco de dados quando há muita leitura (topologia mestre-escravo, por exemplo), do que escalar sessão de servidor web. E, às vezes, o usuário gostaria de entrar pela segunda página diretamente.

Prefira realizar a consulta de novo quando ocorrer a segunda requisição, e controle o estado pela URL, colocando a chave primária do domínio como parâmetro. Isso te livra de um monte de inconveniências, como no caso de o usuário apertar o botão voltar ou consultar a URL no histórico.

"Ah, mas toda vez que faço uma consulta, um porquinho-da-índia morre." Então o problema pode ser resolvido pelo [primeiro ítem acima: cache](#cache).

## A lógica de negócio exige várias telas para sua conclusão


Existem casos onde a conclusão de um objetivo depende do envio de vários formulários, mas isso não é tão comum de acontecer. Mesmo assim, muitos consideram que escrever cada formulário no banco não compensa, porque o usuário pode desistir no meio do caminho. A solução mais comum é ficar guardando dados intermediários na sessão até chegar a última tela do processo, quando tudo é persistido.

Bom, pode ser um raro caso onde seja interessante armazenar na sessão. Porém, faça a coisa direito! Não espalhe por todo o código o armazenamento e a obtenção de objetos na Sessão. Se você tiver usando EJB, use um Stateful Session Bean para armazenar todos os objetos obtidos do usuário antes de persistir. Se tiver usando Spring ou outro contêiner de injeção de dependência, use uma classe de escopo de sessão para armazenar todos os objetos. Use o mínimo de classes de infraestrutura para evitar dependências desnecessárias.

## Vou implementar um "carrinho de compras"

Todo mundo aprendeu _HttpSession_ usando um exemplo de carrinho de compras. O problema é gente acreditar que, pra esse caso, só existe uma única solução possível, a única que lhe ensinaram, o armazenamento na sessão de usuário! Porém, isso pode não ser verdade num mundo real. Imagine que alguém tenha colocado um produto no carrinho virtual, mas desistido de comprar. Talvez essa informação de compra não-efetuada possa ser interessante para a loja, pois ela pode gerar alguma _newsletter_. Portanto, manter esses dados na _Session_ está fora de cogitação!

Não quero dizer que um carrinho deve ser feito sempre em banco, mas determinadas regras de negócio podem ser implementados de jeitos diferentes dependendo do contexto. Os exemplos e _templates_ de livros e apostilas só servem pra aprendizagem, não leve-os tão a sério!

* * *

E aí? Convencido que dá pra fazer uma aplicação sem _HttpSession_? Ou vocês acham que existem situações que não falei onde é realmente necessário? Comentem...
