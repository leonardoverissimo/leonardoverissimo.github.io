---
layout: post
title: 'Seção: Apanhando da... UML, e também do Hibernate (ao vivo) (3)'
tags:
- Ao vivo
- Apanhando do
- Hibernate
- metodologias
- UML
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Bom, resolvi tomar vergonha na cara, e usar o Hibernate embaixo do JPA. A configuração foi um parto, tava acostumado com o Toplink, que vem junto com o Glassfish; quer dizer, acostumado entre aspas, pois o negócio é ruim, MUUUUITO ruim, porque se dá algum erro, não dá pra ter a mínima idéia do que ocorreu. Com o Hibernate não, os logs são detalhados e dá pra descobrir o erro se algum problema ocorre.

Mas repito, o Hibernate é difícil de configurar, têm muitas dependências. O Struts 2, por exemplo, precisa no mínimo os jars struts2, o xwork, o ognl e o commons-logging, só. Já Hibernate, pra rodar standalone (porque eu estou criando um jar, não um aplicativo no servidor) precisa de 14 jars, são:

- ejb3-persistence.jar
- hibernate3.jar
- hibernate-annotations.jar
- hibernate-entitymanager.jar
- jboss-archive-browser.jar
- dom4j-1.6.1.jar
- commons-logging-1.0.4.jar
- hibernate-commons-annotations.jar
- javassist.jar
- commons-collections-2.1.1.jar
- cglib-2.1.3.jar
- asm.jar
- jta.jar
- antlr-2.7.6.jar

Se o Hibernate é assim, mal posso esperar pra ver como se faz pra rodar Seam no Glassfish, vou ter que colocar o JBoss AS inteiro no lib do Glassfish!

Tentei também usar o DbUnit pra testar, mas achei um lixo, não é uma abstração muito boa da base de dados, preferi usar o JUnit 4 mesmo.

Bom, veja como ficou:

<!--more-->

Fiz os scripts de criação de tabelas à mão mesmo. Eu sei que o Hibernate pode criá-las, mas não gosto do fato que se se cria as classes com erros de lógica nos anotations, será criado uma estrutura de tabela problemática no banco de dados.

{% highlight sql %}
drop table if exists livros_autores;

drop table if exists exemplares;

drop table if exists livros;

drop table if exists autores;

create table autores (
  id bigint auto_increment not null,
  nome varchar(50) not null,
  primary key(id),
  unique key(nome)
) engine=InnoDB;

create table livros (
  id bigint auto_increment not null,
  titulo varchar(80) not null,
  editora varchar(30) not null,
  ano_publicacao smallint not null,
  isbn varchar(20) not null,
  primary key(id)
) engine=InnoDB;

create table livros_autores (
  id_livro bigint not null,
  id_autor bigint not null,
  FOREIGN KEY fk_la_livro (id_livro) REFERENCES livros(id),
  FOREIGN KEY fk_la_autor (id_autor) REFERENCES autores(id),
  primary key(id_livro, id_autor)
) engine=InnoDB;

create table exemplares (
  id bigint auto_increment not null,
  codigo varchar(10) not null,
  motivo_baixa varchar(100),
  id_livro bigint not null,
  foreign key fk_exemplar_livro(id_livro) references livros(id),
  primary key(id),
  unique key uk_exemplar(codigo, id_livro)
) engine=InnoDB;
{% endhighlight %}

Criei as classes Autor, Livro e Exemplares, não fiz daquele jeito artificial onde existe uma camada de banco de dados em cima dos objetos, o código de banco de dados está dentro dos objetos de domínio, é quase um Active Record com a exceção de que meus objetos não possui um método salvar(), se quiser persistir no banco de dados, vai ter que chamar transaction.commit().

{% highlight java %}
package com.wordpress.objectzilla.biblioteca;

import java.io.Serializable;
import java.util.List;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;

@Entity(name="autores")
public class Autor implements Serializable {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String nome;
    @ManyToMany
    @JoinTable(name="livros_autores",
                joinColumns=@JoinColumn(name="id_autor"),
                inverseJoinColumns=@JoinColumn(name="id_livro"))
    private List<Livro> livros;

    public Autor() {
    }

    protected Autor(Long id, String nome) {
        this.id = id;
        this.nome = nome;
    }

    public Autor(String nome) {
        this.nome = nome;
    }

    public Long getId() {
        return id;
    }

    protected void setId(Long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    protected void setNome(String nome) {
        this.nome = nome;
    }

    public List<Livro> getLivros() {
        return livros;
    }

    protected void setLivros(List<Livro> livros) {
        this.livros = livros;
    }
}
{% endhighlight %}

{% highlight java %}
package com.wordpress.objectzilla.biblioteca;

import java.io.Serializable;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EntityManager;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;
import javax.persistence.OneToMany;
import javax.persistence.Query;
import javax.persistence.Table;
import javax.persistence.Transient;

@Entity
@Table(name = "livros")
public class Livro implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String titulo;

    @ManyToMany
    @JoinTable(name = "livros_autores", joinColumns = @JoinColumn(name = "id_livro"), inverseJoinColumns = @JoinColumn(name = "id_autor"))
    private List<Autor> autores;

    private String editora;

    @Column(name = "ano_publicacao")
    private short anoPublicacao;

    private String isbn;

    @OneToMany(mappedBy="livro", cascade=CascadeType.ALL)
    private List<Exemplar> exemplares;

    @Transient
    private static transient EntityManager entityManager;

    protected Livro() {
    }

    public static Livro novoLivroGerenciado() {
        Livro newLivro = new Livro();
        entityManager.persist(newLivro);
        return newLivro;
    }

    public static void setFactory(EntityManager entityManager) {
        Livro.entityManager = entityManager;
    }

    @SuppressWarnings(value = "unchecked")
    public static List<Livro> procurarPorPalavraChave(Collection<String> palavraChave) {
        if (palavraChave.isEmpty()) {
            return null;
        }

        Iterator<String> iter = palavraChave.iterator();
        String primeiraPalavraChave = iter.next();

        StringBuilder buffer = new StringBuilder("SELECT l FROM Livro l WHERE (l.titulo LIKE '%" + primeiraPalavraChave + "%') ");

        while (iter.hasNext()) {
            String otherPalavraChave = iter.next();
            buffer.append("AND (l.titulo LIKE '%" + otherPalavraChave + "'%) ");
        }

        Query query = entityManager.createQuery(buffer.toString());

        return query.getResultList();
    }

    public void adicionarExemplar(Exemplar exemplar) {
        // um exemplar não pode fazer parte de outro livro
        if (exemplar.getLivro() != null) {
            throw new IllegalStateException("Exemplar <" + exemplar.getId() +
                    "> já está associado ao livro <" +
                    exemplar.getLivro().getId() + ">");
        }

        exemplar.setLivro(this);
        exemplares.add(exemplar);
    }

    // setters e getters

    public short getAnoPublicacao() {
        return anoPublicacao;
    }

    public void setAnoPublicacao(short anoPublicacao) {
        this.anoPublicacao = anoPublicacao;
    }

    public List<Autor> getAutores() {
        return autores;
    }

    public void setAutores(List<Autor> autores) {
        this.autores = autores;
    }

    public String getEditora() {
        return editora;
    }

    public void setEditora(String editora) {
        this.editora = editora;
    }

    public List<Exemplar> getExemplares() {
        return exemplares;
    }

    protected void setExemplares(List<Exemplar> exemplares) {
        this.exemplares = exemplares;
    }

    protected Long getId() {
        return id;
    }

    protected void setId(Long id) {
        this.id = id;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }

    public String getTitulo() {
        return titulo;
    }

    public void setTitulo(String titulo) {
        this.titulo = titulo;
    }
}
{% endhighlight %}

{% highlight java %}
package com.wordpress.objectzilla.biblioteca;

import java.io.Serializable;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;
import javax.persistence.Table;

@Entity
@Table(name="exemplares")
public class Exemplar implements Serializable {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String codigo;
    @Column(name="motivo_baixa")
    private String motivoBaixa;
    @ManyToOne
    @JoinColumn(name="id_livro")
    private Livro livro;

    public Exemplar() {
    }

    public Exemplar(String codigo) {
        this.codigo = codigo;
    }

    Exemplar(String codigo, String motivoBaixa, Livro livro) {
        this.codigo = codigo;
        this.motivoBaixa = motivoBaixa;
        this.livro = livro;
    }

    public void darBaixa(String motivo) {
        if (motivo == null) {
            throw new NullPointerException("Não é possível dar baixa sem motivo");
        }

        this.motivoBaixa = motivo;
    }

    public boolean isAtivo() {
        return this.motivoBaixa == null;
    }

    public String getCodigo() {
        return codigo;
    }

    public void setCodigo(String codigo) {
        this.codigo = codigo;
    }

    public Long getId() {
        return id;
    }

    protected void setId(Long id) {
        this.id = id;
    }

    public Livro getLivro() {
        return livro;
    }

    protected void setLivro(Livro livro) {
        this.livro = livro;
    }

    public String getMotivoBaixa() {
        return motivoBaixa;
    }

    protected void setMotivoBaixa(String motivoBaixa) {
        this.motivoBaixa = motivoBaixa;
    }
}
{% endhighlight %}

Ainda vou continuar com outros casos de usos, mas isso não será pra agora.

Tchau.
