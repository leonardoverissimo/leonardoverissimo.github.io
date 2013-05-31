---
layout: post
title: Já usou Memcached?
tags:
- Java
- java cache memcached
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Cache não é algo que os programadores desconhem. Quem esteve numa universidade, e teve uma aula séria de Arquitetura de Computadores, sabe que a memória é disposta numa hierarquia de caches. E quem já programou em Java, já ouviu falar de inicialização tardia, muito usado em _singletons_ (apesar de não ser só para isso):<!--more-->

{% highlight java %}
public class MinhaClasse {
    
    private static MinhaClasse instance;

    public static MinhaClasse getInstance() {
    	
    	if (instance == null) {
    		instance = new MinhaClasse();
    	}
    	   
        return instance;
    }

    // outras coisas
}
{% endhighlight %}

Esse é um jeito de fazer cache. Apenas deve-se tomar cuidado para que o objeto a ser retornado não seja mutável. Afinal, todos os interessados estão com a mesma instância, e se um dos clientes puderem mudar uma propriedade desse único objeto, todos terão o efeito indesejável de ter um objeto com comportamento diferente sem saber. Detalhe: a implementação de _singleton_ dessa maneira é ingênua, evite-a.

Um outro jeito, comumente usado, de fazer cache é usar _session_. Pra quem tem uma aplicação web, parece mágico: joga-se uma entidade da persistência na sessão passando uma chave. Ao precisar do mesmo objeto de novo, ao invés de acessar o banco, pega-se o objeto da sessão com a mesma chave usada na escrita. Esse é a estratégia mais comum que eu já vi por aí. Mas existem vários problemas : já vi aplicações com mais código pra cache em sessão do que pro próprio negócio! Além do mais, _sessions_ são únicas para cada usuário no _browser_, enquanto que a entidade a ser cacheada é, não raro, visível a vários usuários. Significa duas coisas: 1) Não vai ter uma cacheamento efetivo, se 20 usuários se logarem e todos acessarem a mesma entidade duas vezes, haverá vinte cópias do objeto, que será escrita e lida uma vez cada; 2) Ninguém sabe o que um outro usuário pode fazer, se um alterar a entidade, todos os dezenove usuários restantes não terão como saber disso e manterão cópias desatualizadas. _Session_ não foi feita pra cache, e existe até um problema de design: apesar do mapa de sessão pertencer à camada de apresentação, o cache lida com problemas de objetos que surgem na camada de persistência. Dizer que um pode ser usado para implementação do outro resulta em confusões de conceitos.

Uma solução melhor é usar bibliotecas de cache. No Java, um bastante comum é o EHCache -- usado principalmente para o cache de segundo nível de Hibernate -- que fica na mesma JVM da sua aplicação. Não que seja ruim (tem até uns javeiros roxos que acham que esta é a melhor solução do mundo), mas quando se quer usar um cluster de aplicações web, por exemplo, passa-se a ter vários caches para cada instância de servidor, que é não o desejável. Uma das opções de cache, que fica fora da JVM de sua aplicação, é o Memcached.

A seguir, apresentarei as vantagens do Memcached, seu mecanismo básico junto com uma API do Java, e no final, o uso integrado com o Hibernate.

### Características do Memcached

Memcached permite a chamada **escalabilidade horizontal**, ou seja, é possível usar várias instâncias de memcached em paralelo como se fosse uma única unidade, onde cada processo possui somente parte dos registros. Parece idiota, mas não é. Não é toda infraestrutura que lhe permite isso; bancos de dados tradicionais, sistemas operacionais e máquinas virtuais possuem uma dificuldade tremenda de configurações para que isso seja possível.

Porém, ser fácil de se escalar horizontalmente trouxe certos custos na sua implementação. Exemplo: Não existe replicação de dados automática, se quiser duas instâncias com dados iguais, terá que fazer seu script. Não existe transação e nem lock, pois isso gera contenção. Não existem busca por todos os registros ou por wildcards, pois pode causar demora nos resultados. E não há garantia de durabilidade de objetos armazenados em casos de _crash_.

E ainda duas características peculiares, uma é que o **tempo de expiração no cache** não é zerado com consultas (como é tradicional nas sessions de containeres web), ou seja, se você definir como tempo de expiração dois minutos, esse será o tempo que ficará armazenado, independente se houve consulta do registro nesse meio-tempo ou não. Portanto, sempre coloque no memcached objetos que já estejam persistidos em outro lugar. A segunda característica é que o tempo de expiração é definido por segundos que devem ser armazenados, ou seja, uma hora é 3600. Mas apenas se o tempo de permanência for menor que trinta dias, se for maior, o Memcached assume que é o tempo UNIX absoluto, ou seja, 1240699927 significa que o registro deve permanecer em cache até em 25 de abril de 2009 às 22h52min07 em UTC.

### Como usar

Se você estiver usando alguma distribuição Linux, com certeza o Memcached pode ser baixado diretamente de seu repositório. Caso você seja um usuário Windows, existem portes do cache para esse Sistema Operacional [aqui](http://www.splinedancer.com/memcached-win32/) e [aqui](http://code.jellycan.com/memcached/). Como só tenho o Fedora, não testei pra ver se funciona, então o [Google](http://www.google.com.br/) será melhor pra resolver problemas de instalação no Windows do que eu.

Além disso, é necessário um cliente Memcached, que existem para várias linguagens. No Java, a melhor biblioteca que encontrei é a [Spy Memcached](http://code.google.com/p/spymemcached/), disponível na [versão 2.3.1](http://spymemcached.googlecode.com/files/memcached-2.3.1.jar). Baixe o arquivo e coloque no classpath de sua aplicação.

Incialize o Memcached no seu shell dessa maneira:

{% highlight bash %}
memcached -p 18000 -vv
{% endhighlight %}

A opção -p indica a porta da conexão TCP por onde será "ouvida" (não é necessário, se não informar será assumido o padrão 11211), e o -vv é a opção _verbose_.

Na sua aplicação Java, você abre um cliente Memcached dessa maneira:

{% highlight java %}
MemcachedClient mem = new MemcachedClient(new InetSocketAddress("localhost", 18000));
{% endhighlight %}

Ou seja, passando um endereço de rede ao construtor do MemcachedClient. Caso você queira um cluster de aplicações Memcached, é possível também passar uma lista de endereços de redes, assim:

{% highlight java %}
MemcachedClient mem = new MemcachedClient(new InetSocketAddress[] {
				new InetSocketAddress("localhost", 18000),
				new InetSocketAddress("localhost", 18001),
			});
{% endhighlight %}

Obviamente, é necessário duas instâncias de Memcached disponíveis nesses endereços.

As operações que o Memcached suporta são simples, exemplos:

#### set

![Memcached - exemplo do comando SET](/assets/2009/05/02/opcao_set.png)

A operação **set** insere um valor para uma dada chave, independente se esta já possuía um valor anterior ou não:

{% highlight java %}
     String marca = "Ford";
     int expira = (int) (System.currentTimeMillis() + 60 * 60 * 1000) / 1000; // uma hora pra expirar
     Future<Boolean> valido = mem.set("marca", expira, marca);	
{% endhighlight %}

Lembre-se também do comportamento dos trinta dias, as duas formas abaixo setam o mesmo período de expiração:

{% highlight java %}
     Future<Boolean> valido = mem.set("marca", 24 * 60 * 60, marca); // um dia para expirar
     Future<Boolean> valido = mem.set("marca", (int) (System.currentTimeMillis() + 24 * 60 * 60 * 1000) / 1000, marca); // também um dia
{% endhighlight %}

É possível _setar_ qualquer coisa, inclusive objetos Java que sejam seriálizáveis. Porém, qualquer coisa diferente de um tipo simples não é identificável para programas em outras linguagens que se conectarem a esse mesmo Memcached.

#### add

![Memcached - exemplo do comando ADD](/assets/2009/05/02/opcao_add.png)

A operação **add**, apesar do nome, insere um valor para uma dada chave, deste que esta não esteja com um elemento previamente armazenado:

{% highlight java %}
     Pessoa pessoa1 = new Pessoa();
     pessoa1.setId(28L);
     pessoa1.setName("Leonardo");
     
     Future<Boolean> valido1 = mem.add("pessoa", 60 * 60, pessoa1); // valido1 é true

     Pessoa pessoa2 = new Pessoa();
     pessoa2.setId(41L);
     pessoa2.setName("Vinicius");
     
     Future<Boolean> valido2 = mem.add("pessoa", 60 * 60, pessoa2); // valido2 é false
{% endhighlight %}

#### replace

![Memcached - exemplo do comando REPLACE](/assets/2009/05/02/opcao_replace.png)

A operação **replace**, ao contrário do _add_, insere um valor para uma dada chave apenas se esta já estiver anteriormente armazenado:

{% highlight java %}
     int anoLancamento = 2010;
     Future<Boolean> valido1 = mem.replace("anoLancamento", 60 * 60, anoLancamento); // valido1 é false
     
     Future<Boolean> ok = mem.set("anoLancamento", 60 * 60, anoLancamento); // set armazena de qualquer jeito
     ok.get(); // bloqueia até a conclusão
     
     int novoAnoLancamento = 2011;
     Future<Boolean> valido2 = mem.replace("anoLancamento", 60 * 60, novoAnoLancamento); // valido2 é true		
{% endhighlight %}

#### get

![Memcached - exemplo do comando GET](/assets/2009/05/02/opcao_get.png)

A operação **get** busca um registro armazenado anteriormete, dada uma chave:

{% highlight java %}
     String marca = (String) mem.get("marca"); // "Ford", ou null se demorou demais ou expirou
{% endhighlight %}

#### delete

![Memcached - exemplo do comando DELETE](/assets/2009/05/02/opcao_delete.png)

A operação **delete** apaga um valor no Memcached dada a chave. O valor armazenado não é retornado:

{% highlight java %}
     Future<Boolean> valido1 = mem.delete("anoLancamento"); // true

     Future<Boolean> valido2 = mem.delete("anoLancamento"); // false
{% endhighlight %}

#### incr

A operação **incr** incrementa um valor previamente armazenado. Válido somente para números:

{% highlight java %}
     long valor1 = mem.incr("contador", 9, 5); /* o segundo parâmetro é valor a ser incrementado */
                                               /* o terceiro parâmetro (opcional) é o valor inicial caso não haja nenhum par armazenado */
                                               /* valor1 = 5 */
     
     long valor2 = mem.incr("contador", 7); /* se não informasse o valor inicial e esta fosse a primeira chamada, o retorno seria -1 */
                                            /* valor2 = 12 */
{% endhighlight %}

#### decr

A operação **decr** decrementa um valor previamente armazenado. Válido somente para números:

{% highlight java %}
     long valor1 = mem.decr("contador", 12, 20); /* valor1 = 20 */
     long valor2 = mem.decr("contador", 9);      /* valor2 = 11 */
{% endhighlight %}

Lembre-se: os números são só positivos. Se o valor armazenado é 4 e você decrementar 6, o valor final é 0 (zero).

### Exemplo de cache para banco de dados

Vamos a uma situação prática. Imagine um banco de consulta de preços de um supermercado. Alguns preços são consultados com bastante frequência e seus preços são raramente alterados. Idealmente, seria interessante cachear esses valores retornados para aliviar a carga do banco. Meu objeto de domínio é um só:

{% highlight java %}
public class ProductDescription implements Serializable {
	
	private long barCode;
	
	private String description;
	
	private BigDecimal price;
	
	private String type;

	// getters e setters omitidos
}
{% endhighlight %}


Meu acesso aos dados obedece a essa interface:

{% highlight java %}
package br.com.objectzilla.testeDaoMemcached;

public interface ProductDescriptionRepository {
	
	ProductDescription getByBarCode(long barCode);
	
	void refreshProductInformation(ProductDescription product);
}
{% endhighlight %}

Ou seja, existe um método para retornar um pedido por código de barras ( _getByBarCode()_ ) e outro que atualiza ou insere um produto ( _refreshProductInformation()_ ). Também tenho uma classe, implementando essa interface, que executa comandos SQL no banco via JDBC (ProductDescriptionDAO, não mostrado).

O pulo do gato é uma classe, que fica como um Decorator, que realiza o cache, veja:

{% highlight java %}
package br.com.objectzilla.testeDaoMemcached;

import net.spy.memcached.MemcachedClient;

public class ProductDescriptionCacheDecorator implements ProductDescriptionRepository {
	
	private MemcachedClient memcachedClient;
	private int expiration;
	private ProductDescriptionRepository other;
	
	public ProductDescription getByBarCode(long barCode) {
		
		ProductDescription product = (ProductDescription) memcachedClient.get("product_descr/" + barCode);
		if (product == null) {
			product = other.getByBarCode(barCode);
			
			memcachedClient.add("product_descr/" + barCode, expiration, product);
		}
		return product;
	}

	public void refreshProductInformation(ProductDescription product) {
		other.refreshProductInformation(product);
		
		memcachedClient.delete("product_descr/" + product.getBarCode());
	}

	public void setMemcachedClient(MemcachedClient memcachedClient) {
		this.memcachedClient = memcachedClient;
	}
	
	public void setExpiration(int expiration) {
		this.expiration = expiration;
	}

	public void setOther(ProductDescriptionRepository other) {
		this.other = other;
	}
}

{% endhighlight %}

Primeiro, a variável other refere-se a uma outra classe que implementa também ProductDescriptionRepository, como a classe acima é um decorator, natural que essa referência seja de um objeto que realmente faz acesso ao banco. Tem-se também a instância de um MemcachedClient e a instância de um período de expiração, para ser usado para a manipulação de dados do Memcached.

Veja, primeiro, o método **getByBarCode**, a primeira coisa que se faz é buscar um registro no memcached usando uma chave formada do nome abreviado do objeto seguido do código de barras do produto (o id). Havendo esse registro, retorna-o, e aí a consulta ao banco não é realizada e fim de papo. Se esse registro não estiver no Memcached, aí este é buscado no banco (através de _other_) e, em sequência, realizada a inserção do objeto no cache, para que as próximas chamadas desse método não façam busca no banco de novo. Lembre-se que o cache expira, e não é apropriado assumir que o objeto está sempre no cache, portanto é sempre necessário ter essa verificação de registro vazio em todas as suas consultas.

Ao inserir ou alterar um registro, o dado no cache fica desatualizado. Por isso, existe uma chamada para remover o ítem após a inserção bem sucedida no método **refreshProductInformation()**. Poderia haver uma inserção no memcache nesse ponto, mas preferi não fazer, porque de qualquer forma, isso será feito no método de busca.

A aplicação que exercita esse cache está disponível nesse [zip](/assets/2009/05/02/testeDaoMemcached.zip), caso queiram estudá-lo com mais detalhes.

### Usando como cache de segundo nível do Hibernate

O [Hibernate](https://www.hibernate.org/) possui suporte a cache de segundo nível, e a implementação mais comum é o [EHCache](http://ehcache.sourceforge.net/), que vem junto com o framework de persistência. Por ter um cache quase automático (pois você precisa habilitá-lo para cada entidade mapeada), é meio desvantajoso fazer cacheamento manualmente como fiz anteriormente. Incrivelmente, na minha busca por Memcached no Google, me deparei com o plugin [hibernate-memcached](http://code.google.com/p/hibernate-memcached/) que usa o Memcached como cache de segundo nível.

Se a opção for essa, as coisas facilitam bastante. Primeiro, anotaremos a entidade ProductDescription, assim:

{% highlight java %}
@Entity
@Table(name="product_description")
@Cache(usage=CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
public class ProductDescription implements Serializable {
	@Id
	@Column(name="bar_code")
	private long barCode;
	private String description;
	private BigDecimal price;
	private String type;
	// getters e setters
}
{% endhighlight %}

Existem as anotações triviais do JPA, como @Entity, @Table, @Id e @Column. Mas existe também a anotação específica do Hibernate, que é o @Cache. Na estratégia de concorrência, estou dizendo que é possível haver escrita e leitura, mas que a escrita não bloqueia leituras. Sem esta anotação, o cache não funciona pra essa entidade. (Por isso, se você um dia colocou o cache de segundo nível que você queria nas configurações, mas nunca colocou a anotação @Cache nas entidades, é a mesma coisa que não ter cache de segundo nível.)

O Dao faz acessos ao Hibernate normalmente (estou usando o template do Spring):

{% highlight java %}
package br.com.objectzilla.hibernateMemcached;

import org.hibernate.SessionFactory;
import org.springframework.orm.hibernate3.HibernateTemplate;
import org.springframework.transaction.annotation.Transactional;

@Transactional
public class ProductDescriptionDAO implements ProductDescriptionRepository {
	
	private HibernateTemplate hibernateTemplate;
	
	public ProductDescription getByBarCode(long barCode) {
		return (ProductDescription) hibernateTemplate.get(ProductDescription.class, barCode);
	}
	
	public void refreshProductInformation(ProductDescription product) {
		product = (ProductDescription) hibernateTemplate.merge(product);
		hibernateTemplate.saveOrUpdate(product);
	}

	public void setSessionFactory(SessionFactory sessionFactory) {
		hibernateTemplate = new HibernateTemplate(sessionFactory);
	}
}
{% endhighlight %}

E pra configurar, basta adicionar essas propriedades no hibernate.cfg.xml:

{% highlight xml %}
<property name="hibernate.cache.provider_class">com.googlecode.hibernate.memcached.MemcachedCacheProvider</property>
<property name="hibernate.cache.use_query_cache">true</property>
<property name="hibernate.memcached.servers">localhost:14001</property>
{% endhighlight %}

Onde eu indico o provedor do cache, a opção de se fazer cache de queries, e o servidor do Memcached. Não são as únicas opções, mas as outras possuem opção _default_, que você encontra [aqui](http://code.google.com/p/hibernate-memcached/wiki/Configuration).

É só isso, agora é só adiconar o hibernate-memcached e o spy-memcached no seu classpath e rodar a aplicação. Também disponibilizei um [zip](/assets/2009/05/02/hibernateMemcached.zip) para esse exemplo, caso vocês queiram dar uma olhada.

### Conclusão

Usar cache, seja o Memcached ou qualquer outro, é uma boa opção para evitar acesso a recursos custosos, como o banco de dados. Mas, numa aplicação dividida em camadas não é sempre necessário que a responsabilidade de cachear fique no lado cliente (camada de apresentação). Se deixar que o lado servidor (camada de persistência) tenha essa responsabilidade, você pode deixar que os clientes acessem a camada de persistência na hora e do jeito que bem entenderem e, ainda assim, de maneira transparente, estará sendo feito acesso eficiente de recursos.

Eu havia feito uma medição bem informal de quanto o Memcached economizaria o tempo de busca. Levei à conclusão de que por causa de warm-up e tudo mais, eu não sei fazer benchmarks. ;) O mais importante do cache não costuma nem ser o tempo de acesso mais rápido, mas o fato de não deixar "engargalar" o banco com muitos acessos.
