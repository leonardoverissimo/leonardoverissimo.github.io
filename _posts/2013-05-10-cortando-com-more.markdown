---
layout: post
title:  "Cortando com More"
date:   2013-05-10 22:05:00
categories: lorem ipsum
---


Lorem ipsum dolor sit amet, consectetur adipiscing elit. In est lectus, semper sodales fringilla sit amet, luctus ut nisi. Donec iaculis tempor sapien sit amet rhoncus. Donec viverra, libero ut laoreet feugiat, neque nisi aliquet velit, vitae elementum leo purus in metus. Aliquam id aliquet eros. Suspendisse bibendum gravida congue. Proin pulvinar nulla eu libero tempus sit amet blandit neque rhoncus. Aenean ut pharetra nunc. Curabitur eget mi in erat pellentesque sollicitudin.
<!--more-->
{% highlight java %}
package leonardo.domain;

import java.io.Serializable;
import java.util.Calendar;
import java.util.List;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.PrePersist;
import javax.persistence.PreUpdate;
import javax.persistence.SequenceGenerator;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

@SuppressWarnings("serial")
@Entity
@Table(name="posts")
public class Post implements Serializable {
	
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO, generator = "seq_post")
	@SequenceGenerator(name = "seq_post", sequenceName = "seq_post", allocationSize = 1, initialValue = 1)
	private Long id;
	private String title;
	private String body;
	
	@Column(name="CREATED_AT")
	@Temporal(TemporalType.TIMESTAMP)
	private Calendar createdAt;
	
	@Column(name="UPDATED_AT")
	@Temporal(TemporalType.TIMESTAMP)
	private Calendar updatedAt;
	
	@OneToMany(mappedBy="post", fetch=FetchType.EAGER, cascade=CascadeType.ALL)
	private List<Comment> comments;
	
	@PrePersist
	public void antesDeInserir() {
		createdAt = Calendar.getInstance();
		updatedAt = createdAt;
	}

	@PreUpdate
	public void antesDeEditar() {
		updatedAt = Calendar.getInstance();
	}

	public Long getId() {
		return id;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getBody() {
		return body;
	}

	public void setBody(String body) {
		this.body = body;
	}

	public Calendar getCreatedAt() {
		return createdAt;
	}

	public Calendar getUpdatedAt() {
		return updatedAt;
	}
	
	public List<Comment> getComments() {
		return comments;
	}

	public void setComments(List<Comment> comments) {
		this.comments = comments;
	}
	
	@Override
	public String toString() {
		return "Post [id=" + id + ", title=" + title + ", body=" + body + "]";
	}
}
{% endhighlight %}

Vestibulum vitae enim vel ipsum posuere viverra vitae a tortor. Vivamus nec nulla mauris. Quisque iaculis facilisis elit, in lobortis massa tincidunt ac. Donec semper sagittis ligula, sit amet pulvinar dui tempor vel. Duis molestie quam id risus semper adipiscing. Nam quis nunc eu nunc elementum aliquet. Vivamus vel nulla odio. Ut massa velit, rutrum vel malesuada gravida, consectetur at dolor. Duis pellentesque, odio ac sollicitudin tincidunt, nulla nulla elementum est, dignissim faucibus nunc tortor in eros.

Quisque interdum commodo lorem, vitae imperdiet tellus euismod varius. Sed dictum nibh fringilla lacus mollis at ultricies est tempor. Nullam at sapien felis, ac aliquam tortor. Proin magna erat, mattis id tristique quis, viverra ut nunc. Integer ut rutrum est. Integer faucibus fringilla tempus. Proin sagittis sem ut neque consectetur egestas.

Integer tristique elementum ligula, vel lacinia lectus accumsan malesuada. Nunc dignissim aliquam auctor. Quisque aliquam, mi ac consectetur condimentum, ipsum massa semper magna, et semper felis elit sit amet elit. Cras eget dictum orci. Donec tempor eleifend purus non tincidunt. Etiam vulputate congue metus, a hendrerit felis tempus nec. Suspendisse potenti. Suspendisse potenti. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae;

Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Nulla facilisi. Donec in neque eu quam dapibus ullamcorper. Fusce aliquam vulputate fermentum. Vivamus condimentum tellus et nibh sodales viverra. Donec consequat pretium neque, in tincidunt metus egestas eget. Nulla facilisi. Cras dapibus magna in erat egestas id condimentum erat commodo. Nunc urna velit, molestie in bibendum vitae, molestie eu ipsum. Suspendisse potenti. Mauris neque magna, ultrices id congue at, viverra quis dolor. Vestibulum et lacus ipsum, in tempor orci. Cras vel mattis odio. Praesent at metus nec lacus hendrerit mattis. Maecenas sollicitudin, augue at ornare consequat, nisi turpis gravida magna, eget lacinia massa lacus ac ligula.
