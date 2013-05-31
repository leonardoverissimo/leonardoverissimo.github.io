---
layout: post
title: 'Desempenho / Escalabilidade: a última carta dos idiotas'
tags:
- Java
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Meu, hoje, 12/01, quase tive um ataque cardíaco (não, não tive não, é brincadeira). Duas coisas que eu vi, uma na Mundo Java, outra no GUJ, me deixaram com raiva.

Vamos à Mundo Java primeiro que, como sempre, está excelente. Hoje vi só a matéria sobre EJB e Spring (muito boa), porém houve um pequeno trecho que me deixou de estômago embrulhado (ênfases são minhas):


> **Anti-pattern em injeção de dependência**
> Com a facilidade e praticidade do EJB 3.0, alguns desenvolvedores acabam deixando de lado a preocupação com a escalabilidade e gerenciamento de recursos do container, anotando toda e qualquer classe do sistema para se beneficiar da injeção de dependência. **Isso não é uma boa prática porque cada objeto anotado com @Stateless existirá um novo recurso para ser gerenciado pelo servidor de aplicações.**
> Um exemplo prático dessá má utilização de injeção de dependências no ambiente EJB **é a implementação do design pattern Data Access Object (DAO) como um EJB**. Dependendo do design do software, cada funcionalidade do sistema irá conter um DAO e essa funcionalidade será um Session Bean para se beneficiar do recurso de injeção de dependência do servidor de aplicações. Para evitar esse problema, **uma solução seria injetar o EntityManager diretamente nos session beans, por meio da anotação @PersistenceContext, ou utilizar o design pattern Abstract Factory.**


Primeiro, vamos à bronca: você não está programando em C, nem precisa ouvir os malucos que programam nessa linguagem. Fim da bronca.

Segundo, existe dois problema na definição de um Stateless Session Bean (SLSB): um que a anotação não é sobre o objeto (isso ainda não existe na linguagem), mas na classe; e dois que, quando você cria um SLSB, não há a criação de um recurso, mas de váááários recursos em pool. Se você olhar na configuração do Glassfish, verá que por default é possível ter até 1024 instâncias de SLSB no pool.

Terceiro, sabe qual a justificativa para isso ser um anti-pattern? Escalabilidade, só isso! Não é problema de code smell, de código que gera resultados imprevisíveis, de problemas de manutenção..., nem nada! Simplesmente ES-CA-LA-BI-LI-DA-DE! Agora, o pool de instâncias de SLSB não existe para se ganhar escalabilidade? Se tiver problemas com isso, vai nas configurações de seu servidor e mude os parâmetros do container EJB! Outra solução é comprar mais memória ou, se for mais sério, fazer *load balance* de mais de um servidor.

Pior ainda, a alternativa ao "anti-pattern" é "injetar o EntityManager diretamente nos session beans", o que fatalmente acarretaria códigos macarrônicos com métodos gigantescos e difíceis de ler. Ou então, "[utilize] o design pattern Abstract Factory" que, pelo que já vi usado no contexto do EJB, é [POG!](http://desciclo.pedia.ws/wiki/POG); pois é usado um único EJB para o Factory, que devolveria os DAOs necessários, que por sua vez, tem código muito mais idiota para se pegar o EntityManager do que se fosse feito por SLSB!

Gente, use SLSB para DAO também! Que mal há nisso? Escalabilidade não é! Acho que no fundo, as pessoas sentem saudades do tempo em que cada camada tinha seu framework, tipo: era Struts para a apresentação, EJB para a camada de negócio, Hibernate para a persistância... Hoje estamos se aproximando da era onde os SLSB podem ser usados até na camada de apresentação ([Seam / Web Beans](http://www.seamframework.org/))! E ninguém passou a escrever código horrível por causa disso! Por que raios ainda ficar nesse pensamento?

* * *

Vamos ao tópico do GUJ, maldito! (Não você que é maldito! O tópico!) ([link](http://www.guj.com.br/posts/list/114680.java)) O cara parecia que quer defender com unhas e dentes que usar static é melhor que criar instâncias, polimorfismo..., enfim, essa coisa linda que é o OO.

Bom, o que sei é que usar static a torto e a direito no Java EE dá merda, pois é um static por class loader, e os servidores de aplicações possuem vários class loaders, cada um com potencialidade de carregar a classe e ter seu próprio static. Segundo, que com métodos static não dá pra fazer muita coisa que o Java EE oferece (EJB, por exemplo). Terceiro, que é idiota mesmo, coisa de gente que não sabe OO.

Agora, sabe a justificativa para usar static? O cara acredita que é mais rápido, simplesmente isso! Quer dizer, dava muito bem pra gerar um código bom, usando OO, e comprar uma máquina de configuração muito boa. Muito melhor do que se matar em otimizações minúsculas de veracidade duvidosa.

Aliás, vou dar um *guideline* para se fazer otimizações:

1. Pergunte ao cliente se está achando o sistema lento. Se a resposta for não, fim do guideline.

2. A compra de mais memória, mais disco ou mais blade custa menos que o pagamento de uma equipe de desenvolvedores? Se sim, vá na Santa Efigênia e fim de guideline.

3. Meça a duração usando vários casos possíveis.

4. Faça a alteração e meça de novo.

5. Se a nova medida não for muito menor que a medida antiga, jogue suas alterações no lixo e volte ao passo 4.

6. Caso contrário, comite a alteração e dê por encerrado (por enquanto).

* * *

Os dois casos me deixam perplexos: é como se para se justificar alguma coisa que não sabe bem o porquê, ao invés de ir mais a fundo e pesquisar, diz-se apenas que a altenativa "ruim" tem problemas de desempenho ou escalabilidade! Simples assim! E esquece que esses dois tipos de problemas está mais relacionado à arquitetura, à característica do sistema, ao "conjunto da obra" e muito pouco sobre a tecnologia ou o paradigma em si. Por muito escarcéu, muita gente escolhe o Barrabás.

Agora, por que as pessoas usam o argumento do desempenho / escalabilidade? Tenho duas teorias:

1. Alguns programadores aprenderam apenas uma determinada tecnologia em sua carreira. Quando esta tecnologia começa a sumir, eles passam a ter um certo desdém pelo novo, classificando pelos mais variados adjetivos. Alguns deles simplesmente não colam. Porém, o que, às vezes, fica marcado, é dizer que tal coisa *não tem desempenho* ou *não tem escalabilidade*, e por uma única razão: é difícil provar a menos que se meça com os próprios olhos. E como muita gente não faz isso, fica-se no benefício da dúvida.

2. Alguns programadores trabalham em ambientes péssimos de trabalho, a principal é a ausência de decisão e autonomia. Normalmente, as decisões são concentradas em pouquíssimas pessoas, e que nem sempre são as mais qualificadas. Os "artefatos" que saem das mãos dessas pessoas são os famosos diagramas de UML, bastando o programador codificar. (O mais imbecil nisso é que os "eleitos" escrevem diagramas tão detalhados que poderiam ter feito o negócio direto em código, e que haveria *a mesma *abstração. Mas não fazem código porque delegar essa atividade pra alguém é a melhor maneira de camuflar seus próprios erros.) E como o programador não tem a liberdade necessária para fazer seu trabalho, só lhe resta uma coisa intelectual para fazer: otimizar. Pena que será imperceptível para os olhos dos outros. Pena que o próximo a dar manutenção será ele mesmo.
