---
layout: post
title: 'MVC: Supostas perguntas frequentes'
tags:
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
**P. Por que um tópico sobre _"supostas perguntas frequentes"_?**
R. Dois motivos: 1. É incrível a quantidade de gente perguntando as mesmas coisas em fóruns por aí. 2. É incrível gente com anos trabalhando que simplesmente erra feio nesse negócio de MVC. E eu tava pensando em fazer um post épico, como havia feito anteriormente, mas como ainda tá "assando", resolvi fazer um outro post épico sobre isso daqui.

**P. O que é MVC?**
R. Háááá! Ainda bem que perguntou, acredita que tem gente que faz cara de conteúdo ao ouvir essas três letras? Enfim, deixa pra lá. MVC é um pattern usado, geralmente, para apresentações visuais de uma aplicação. Por ser pattern, implica que MVC também é uma descrição de uma solução para um problema recorrente; e que também não é uma receita de bolo para ser seguido de olhos vendados. O nome segue a sigla das três "coisas" que a aplicação vai ser dividida:

_Model:_ é como o seu programa seria se o usuário fosse um "über-geek" e pudesse manipular o sistema fazendo simples classes com main() invocando as funções desejadas. Ou seja, é o domínio do seu sistema.
_View: _é o que o Homer Simpson do seu usuário vê: uma representação visual do domínio realizado pelo model.
_Controller:_ é o que fica entre o model e a view. Entre várias coisas, ele faz o que o model não deve fazer: manipular a view.

Implicação: o Controller conhece a View e o Model, a View conhece o Model, e o Model não conhece ninguém.

**P. Model, View e Controller são camadas, certo?**
R. Não, na realidade, nunca vi ninguém por nome nesses "troços". Sério! Às vezes, ouvi dizer que eram componentes, outros que eram camadas. Não acho nenhum dos dois nomes apropriados. Componentes remetem ao modismo do início da década de 00 de se usar "componentes reusáveis" por "operários" em "fábricas de softwares" (eca!). Ou então a um outro modismo de componentes distribuídos como COM+ ou EJB (daqueles antigão). Nenhuma das duas coisas deram certo, e o MVC não tem nada a ver com isso. E camadas refere-se a um estilo de design para dividir um domínio grande, que pode ser feito num software com ou sem interface visual.

A real é que, na dúvida, o pessoal dá um jeito não dizer em que Model, View e Controller são classificados.

**P. Mas perái! M, V e C são camadas sim! Model é a de persistência, View, a de apresentação e Controller, de negócio!**
R. Nããããão, tá loco! Ponha uma coisa na sua cabeça: os objetivos tanto da divisão em camadas, quanto do MVC são distintas: a primeira tem o objetivo de controlar dependências, a segunda, de representar um domínio visualmente. Não são a mesma coisa, e nem dá para unificá-las em um modelo único. Algumas classes suas serão uma coisa do ponto de vista do MVC e serão outra coisa do ponto de vista das camadas.

**P. O Controller serve para controlar o sistema?**
R. Não, o nome controller é que é confuso, e remete a alguns programadores o "objeto deus" _(God Object)_ ou o "objeto bolha-assassina" _(Blob Object)_. Porém, objetos deuses são anti-patterns, ou seja, soluções aparentemente ideiais para problemas recorrentes que causam mais problemas ainda. Quando vier à sua cabeça _"Controller"_, pense-o como_ "Input Controller"_ (controlador de entrada), e este "coiso" deve fazer, estritamente, as seguintes atividades:
- ouve eventos;
- obtém os parâmetros de entrada desejados da View;
- obtém do Model um domínio (invariavelmente, este está num meio persistente);
- chama um método do domínio;
- escolhe a view a ser renderizada, fazendo "bind" de algum domínio do Model.

Não raro, programadores pouco experientes colocam, no "Controller", lógicas de negócio ou tranformações de objeto para a view. Não é isso, remova esses códigos e ponham nos seus devidos lugares.

**P. Ouvi falar de um tal de Model 2, ele é diferente do MVC?**
R. Model 2 é uma adaptação do MVC, usando tecnologias padrão do J2EE como era conhecido no início da década de 00 (ou seja: Struts ou Servlet e JSP). O nome é puramente mercadológico, servia para diferenciar do "Model 1", que era a utilização exclusiva de JSP para construção da aplicação web (assim como era o ASPão e o PHP). Por ser intimamente relacionado com tecnologias ultrapassadas, toda a burocracia da época está no Model 2, como Form Beans e XMLs, manipulação direta de mapas de escopos; sem falar de domínios fracos, normalmente misturados no meio da view ou das queries do banco. Minha recomendação: esqueça Model 2, porque você pode aprender MVC por outros meios.

**P. O MVC da Web é diferente do MVC do Desktop?**
R. Sim, existem duas diferenças: 1) um Controller de Desktop recebe eventos de clique do mouse, de teclado, de botões e outros componentes; um Controller de Web recebe exclusivamente eventos de submissão. Consequência: aplicações Desktop são orientadas a eventos, enquanto aplicações Web, orientadas a fluxos. Tudo que uma aplicação web entende é uma "stringona" com os dados de um suposto formulário (não dá pra saber se o cliente é humano ou que viu a interface gráfica) e que precisa devolver uma nova "stringona" para o cliente que, supostamente, renderiza para algum desenho gráfico. A aplicação Desktop sabe exatamente os eventos recebidos e pode agir diferentemente para cada um deles. 2) A View de Desktop mantém sincronia constante do Model (através do pattern Observer), qualquer alteração que o usuário fizer no model através de um painel, será refletida imediatamente em outros painéis que, porventura, estejam visualizando o mesmo Model. Na Web, não existe nada disso, a partir do momento em que o usuário está diante de uma tela, essa representação do Model estará desconectada do Model real e o usuário não saberá quanto (e se) o Model mudou. O único momento de mudança é quando o usuário aperta F5, mas isso implica num evento consciente do usuário, não numa coisa automática, como as aplicações Desktop de qualidade fazem.

**P. Estava pensando em fazer um MVC distribuído, penso que o Model ficaria no Server e a View, no Client; só não sei onde ficaria o Controller, tem alguma sugestão?**
R. Antes de mais nada, não existe MVC distribuído. O erro da sua pergunta baseia-se numa premissa equivocada, a de que é possível esconder a distribuição na aplicação. Não pode, e pra piorar, muitas "vendors" no passado (e ainda no presente) vendem soluções onde você poderia chamar um objeto remoto como se fosse local. Mas na real, chamadas locais e remotas somente são iguais quando ocorre sucesso, não quando ocorrem falhas, como queda na rede, versões incompatíveis ou falhas de autenticação/autorização.

Se você realmente precisar de uma aplicação distribuída (ex.: uma aplicação Desktop em C# conversando remotamente com um servidor Tomcat) você terá um MVC para o Client (o Model são Active Records que se comunicam com o Server, a View é o módulo onde se instancia componentes e o Controller, os métodos que entende os eventos) e um outro MVC para o Server (o Model são entidades, com seus services e mapeamento objeto-relacional, a View é XML, JSON, um formato binário ou outro qualquer e o Controller é o que entende requisições remotas).

**P. Tenho uma aplicação Web, gostaria de reaproveitar o Controller se precisasse mudar minha aplicação para Desktop, como fazer isso?**
R. Não faz. O controller é intimamente ligado à view, ou seja, nova view, novo controller. A origem do seu problema pode ser Controllers agindo como God Objects (ver aqui), e a solução seria um refactoring, não um Controller multi-uso.

**P. O que o framework MVC faz? Age como Controller?**
R. Quase. Lembra da lista citada acima sobre o que um Controller faz? Veremos o que um framework faz sozinho e outros que depende da ajuda manual do programador:
- ouve eventos **(framework)**;
- obtém os parâmetros de entrada desejados da View **(framework, obtidos via DI)**;
- obtém do Model um domínio **(manualmente)**;
- chama um método do domínio **(manualmente)**;
- escolhe a view a ser renderizada, fazendo "bind" de algum domínio do Model **(parcialmente pelo framework, a escolha da view pode ser sobreescrita manualmente)**.

Há tentativas de algumas comunidades para criarem um framework onde o Controller deixa de existir, mas acredito que não funciona em todos os casos e dependem de models fixados a um padrão imposto ao framework. 

Frameworks vão além, é claro, pois proveem funcionalidades adicionais como _helpers_ para _views_ e outras coisas.

* * *

São essas as perguntas que eu consegui imaginar. E você? Tem alguma dúvida sobre MVC além dessas daqui?






