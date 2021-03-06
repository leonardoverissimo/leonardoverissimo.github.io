---
layout: post
title: Entre exceção checada e não-checada... fique com os dois!
tags:
- checada
- exceção
- Java
- não-checada
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Já viu aquelas discussões no [Reddit](http://www.reddit.com/r/programming/) ou [GUJ](http://www.guj.com.br/) sobre usar ou não exceções checadas? Sempre tem uns proponentes e oponentes defendendo um jeito ou de outro. Invariavelmente, a discussão não dá em nada. Primeiro porque não é uma questão de escolher um jeito e usá-lo por todo o sistema. Segundo porque, já que isso existe, a menos que você queira mudar pra C#, você é obrigado a conviver.

Se eu tivesse a oportunidade, do tipo, sei lá: viesse um cara, chamado James e me dissesse:

> Leo, seguinte, eu tô a fim de fazer uma nova linguagem aí e, sei lá, achei interessante forçar o tratamento de exceções pra que ninguém esquecesse de tratá-lo, tipo uma exceção checada. O que você acha?
Minha resposta:

> Horrível!

Sim, existe um limite prático pra tipagens verificadas por compilador, o aumento da complexidade. Isso certamente acaba complicando mais pra quem começa na linguagem. Mas enfim, já que não me consultaram.... o melhor é usá-las, e usá-las com sabedoria.

> Mas Leo, como seria?

Boa pergunta. Uma definição legal é do livro Effective Java, que diz, na página 244:

> ... use checked exceptions for conditions from which the caller can reasonably be expected to recover... use runtime exceptions to indicate programming errors...

Lendo essas duas definições, pode parecer confuso, veja o que alguém poderia pensar: "Exceção checada só ocorre quando há erros de programação. Nunca cometo erros, logo sempre usarei exceções checadas!"

Não, não e NÃO!!!

Pra entender o que é um erro de programa, você deveria entender que os métodos de seus objetos só podem ser executados se determinadas pré-condições sejam satisfeitas. O problema é que miuta gente também nunca ouviu falar em pré e pós-condições. E dos que ouviram, aplicam de um jeito errado. Meu, vou contar uma história: uma vez eu estava vendo um "documento" de "caso de uso" e lá tinha a seção "pós-condições". Em todas as vezes, juro, em **TODAS** as vezes, estava escrito: "Pós-condição: caso de uso executado com sucesso.". O que?! Significa que é uma condição **sine qua non** de que **NUNCA** vai dar falha?

Vamo lá: pré-condição e pós-condição refere-se ao **mundo externo** do método (ou do fluxo ou do processo, depende do seu ponto de vista). Pré é como o estado do "mundo" deve estar antes da execução. Pós é como o "mundo" irá ficar.

Exemplo: um método de inserção. A pré-condição é de que o objeto exista e que seja de um tipo específico. A pós-condição é que no banco existirá um registro a mais, ou ficará como antes (em caso de erro).

E se eu chamar o método e a pré-condição não estiver sido respeitada? Erro! O método nem é chamado. E se a pós-condição não for respeitada? Erro também! O sistema, pelo menos, deve voltar ao estado anterior da execução.

Claro que isso não é implementado nativamente nas linguagens mais populares, nem nas mais-ou-menos populares. Em Java, por causa disso, você trata as pré-condições usando exceções. (As pós poderiam ser a captura de exceções de outros métodos que o método chama.) E como violação de pré-condição é um erro, você usa **exceção não-checada**, pois esta deve ser usada em caso de erros de programa.

Quer ver um exemplo de onde se usa exceção não-checada como violação de pré-condição? HttpServletResponse! Existe uma condição que é o seguinte: uma vez aberto um OutputStream ou um Writer, **não** se pode abrir um outro stream, nem executar um forward. O que acontece quando alguém, "sem querer", comete <del datetime="2009-01-05T23:20:15+00:00">essa cagada</del> esse erro?

É lançada [uma exceção não-checada](http://java.sun.com/javaee/5/docs/api/javax/servlet/ServletResponse.html#getOutputStream%28%29), a IllegalStateException.

Como guia: use exceções não-checadas quando o chamador tem meios de checar se os parâmetros ou o estado do objeto estão corretos antes da execução do método (claro, não é uma ciência exata, podem haver, err..., exceções). No caso acima, a pessoa poderia muito bem verificar se, no Response, é possível criar um novo fluxo chamando o método isCommitted().

E quando usar exceções **checadas**? Existem casos em que o método não tem qualquer meio de verificar determinada pré-condição, ou tem mas o meio de verificar é praticamente a própria execução do método. O chamador então deve se arriscar chamando o método, e este deve dizer depois se deu certo ou não, lançando exceção, checada!

Exemplo: método [parse() de DateFormat](http://java.sun.com/javase/6/docs/api/java/text/DateFormat.html#parse%28java.lang.String%29), que vai pegar uma String, e devolver a representação dela como um Date. Repare que é lançando exceção checada ParseException caso a String não seja uma data. Por que exceção checada? Porque verificar se uma String é realmente uma data não é tarefa trivial, não dando pra contar com uma verificação prévia antes da execução do método. Por outro lado, [parseInt() da classe Integer](http://java.sun.com/javase/6/docs/api/java/lang/Integer.html#parseInt%28java.lang.String%29) não lança exceção checada, pois é possível facilmente verificar se uma String contém somente números.

Outro exemplo são os métodos [write() de BufferedWriter](http://java.sun.com/javase/6/docs/api/java/io/BufferedWriter.html#write%28char[],%20int,%20int%29) ou [prepareStatement() de Connection](http://java.sun.com/javase/6/docs/api/java/sql/Connection.html#prepareStatement%28java.lang.String%29) que lançam exceções checadas por uma razão simples: nem o chamador nem o método tem a mínima noção se o sistema de arquivos ou o banco de dados está em condições de realizar a ação desejada (e verificar isso pode resultar em outra exceção!).

Como guia: use exceções checadas quando o chamador precisa tomar uma "rota alternativa" após o método ser chamado, pois não dá pra evitar o pior antes.

Uma outra coisa, nem sempre dá pra saber a priori se determinada exceção deveria ser checada ou não. Se você criou uma exceção checada, mas seus chamadores não fazem outra coisa a não ser **relançar** a exceção ou fazer o log dele, é porque você **tem** que converter a exceção checada em exceção não checada! Exceções checadas é para quando o chamador precisa tomar uma atitude a respeito. Se nenhum chamador precisou fazer isso, é porque, muito provavelmente, a exceção nunca precisaria ser tratada e o lançamento dela significa simplemente um erro de programa.

Outra coisa, exceções precisam ter nomes significativos. Evite as famosas **"exceções arquiteturais"** ou **"exceções de camadas"**, que você identifica por DaoException, BusinessException, ApplicationException ou NomeDaEmpresaException. Todas elas não dão significado ao erro que ocorreu e o programador certamente não irá tratá-las adequadamente, pois não tem a mínima idéia do que fazer, dada a ausência de informações contidas nessa exceção.

E evite criar novas exceções quando existem algumas delas já prontas pra você, como é o caso de [NullPointerException](http://java.sun.com/javase/6/docs/api/java/lang/NullPointerException.html) (sim, se você receber um parâmetro nulo, não há problema nenhum em **você** lançar essa exceção, colocando uma mensagem talvez), [IllegalArgumentException](http://java.sun.com/javase/6/docs/api/java/lang/IllegalArgumentException.html) ou [IllegalStateException](http://java.sun.com/javase/6/docs/api/java/lang/IllegalStateException.html). Elas não são exclusivas das bibliotecas Java, nem são exceções reservadas. Use-as quando for o caso.
