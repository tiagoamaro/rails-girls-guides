---
layout: default
title: Coloque a aplicação Rails Girls online no Heroku
permalink: publique-heroku
---

# Coloque a aplicação Rails Girls online no Heroku

*Traduzido e adaptado de [Put Your App Online With Heroku](http://railsgirls.com/guides/heroku/)*

##  Heroku

Siga os seguintes passos do [Quickstart Guide](https://devcenter.heroku.com/articles/getting-started-with-ruby){:target="_blank"}:

1. [Introduction](https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction){:target="_blank"}
2. [Set up](https://devcenter.heroku.com/articles/getting-started-with-ruby#set-up){:target="_blank"}
3. [Prepare the app](https://devcenter.heroku.com/articles/getting-started-with-ruby#prepare-the-app){:target="_blank"}
4. [Deploy the app](https://devcenter.heroku.com/articles/getting-started-with-ruby#deploy-the-app){:target="_blank"}

**COACH**: Fale sobre os benefícios de fazer deploy para o Heroku versus os servidores tradicionais.

## Preparando sua aplicação

### Sistemas de controle de versão

A sua aplicação deve estar sob um sistema de controle de versão como o Git, por exemplo.

### Atualizando o banco de dados

Primeiramente, nós precisamos colocar o nosso banco de dados para funcionar no Heroku, que usa o PostgresSQL e não o SQLite como utilizamos na nossa aplicação.
Para isso será necessário a remoção da seguinte linha do nosso Gemfile:

{% highlight ruby %}
gem 'sqlite3'
{% endhighlight %}

Que deve ser substituída por:

{% highlight ruby %}
group :development do
  gem 'sqlite3' # Utilizada apenas em ambiente de desenvolvimento
end
group :production do
  gem 'pg' # Utilizada em ambiente de produção
end
{% endhighlight %}

Agora execute:

{% highlight ruby %}
  bundle install --without production
{% endhighlight %}

**COACH**: Fale sobre RDBMS e inclua Heroku's dependency on PostgreSQL.


### Adicionando a gem rails_12factor

Agora precisamos instalar a gem `rails_12factor` no nosso Gemfile para conserguirmos publicar a nossa aplicação no Heroku. Essa gem faz algumas modificações no comportamento padrão no nosso projeto, como por exemplo atualização de logs e configuração de assets estáticos (suas imagens, css e javascript) é adequada ao sistema do Heroku.

Agora modifique a seguinte linha do seu Gemfile:

{% highlight ruby %}
  group :production do
    gem 'pg'
  end
{% endhighlight %}

Para:

{% highlight ruby %}
  group :production do
    gem 'pg'
    gem 'rails_12factor'
    end
{% endhighlight %}

Após isso, execute:

{% highlight ruby %}
  bundle # Instale as novas dependências
{% endhighlight %}

{% highlight sh %}
  git commit -a -m "Added rails_12factor gem and updated Gemfile.lock" # Faz o commit das últimas mudanças
{% endhighlight %}

**COACH**: Fale um pouco sobre logs no Heroku e outras particularidades.

## Publicando sua aplicação

### Criação do app no Heroku

Primeiro precisamos fazer o login no Heroku:

{% highlight sh %}
  heroku auth:login # Faz login usando as credenciais do Heroku
{% endhighlight %}

Agora nós vamos criar o nosso app no Heroku executando no terminal:

{% highlight sh %}
  heroku apps:create railsgirls # Cria um app novo com o nome railsgirls
{% endhighlight %}

Que retorna algo parecido com:

{% highlight sh %}
  Creating sheltered-refuge-6377... done, stack is cedar
  http://sheltered-refuge-6377.herokuapp.com/ | git@heroku.com:sheltered-refuge-6377.git
  Git remote heroku added
{% endhighlight %}

Nesse caso o nome gerado automaticamente para o nosso app no Heroku foi "sheltered-refuge-6377".

### Enviando nosso código

Para enviar nosso código para o Heroku, basta executar:

{% highlight sh %}
  git push heroku master #faz push no repositório remoto do Heroku  
{% endhighlight %}

{% highlight sh %}
Initializing repository, done.
Counting objects: 101, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (91/91), done.
Writing objects: 100% (101/101), 22.68 KiB | 0 bytes/s, done.
Total 101 (delta 6), reused 0 (delta 0)

-----> Ruby app detected
-----> Compiling Ruby/Rails
-----> Using Ruby version: ruby-2.0.0
-----> Installing dependencies using 1.6.3
       Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin -j4 --deployment
       Fetching gem metadata from https://rubygems.org/..........
...
-----> Launching... done, v6
       http://sheltered-refuge-6377.herokuapp.com/ deployed to Heroku
{% endhighlight %}

Você saberá que seu código já foi enviado quando vir a seguinte mensagem no terminal:

{% highlight sh %}
  Launching...  
{% endhighlight %}

### Execute as migrações do banco de dados

Em seguida precisamos migrar nosso banco de dados da mesma maneira que fizemos localmente durante os workshops anteriores:

{% highlight sh %}
  heroku run rake db:migrate
{% endhighlight %}

Quando o comando terminar de executar você já pode acessar seu app. Para esse exemplo, a url gerada foi <http://sheltered-refuge-6377.herokuapp.com/>.

Agora digite:

{% highlight sh %}
  heroku apps:open --app railsgirls #abre o app railsgirls no navegador
{% endhighlight %}

#### Notas finais

Heroku's platform is not without its quirks. Applications run on Heroku live within an ephermeral environment — this means that (except for information stored in your database) any files created by your application will disappear if it restarts (for example, when you push a new version).

###### [Ephemeral filesystem](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem)
> Each dyno gets its own ephemeral filesystem, with a fresh copy of the most recently deployed code. During the dyno’s lifetime its running processes can use the filesystem as a temporary scratchpad, but no files that are written are visible to processes in any other dyno and any files written will be discarded the moment the dyno is stopped or restarted.

In the [App](/app) tutorial the ability to attach a file to the Idea record is added, which results in new files being written to your applications `public/uploads` folder. The ephemeral storage in Heroku can be seen with the following steps:

1. Launch the app with `heroku open`
2. Add a new Idea with an image
3. Restart the application by running `heroku restart`
4. Go back to your Idea and reload the page - the image should no longer be visible

##### Working around Ephemeral Storage

Obviously this doesn't seem to be useful if you were running a real life application, but there are ways to work around this which is commonly used by a lot of popular websites.

The most common method is to use an external asset host such as Amazon S3 (Simple Storage Service) or Rackspace CloudFiles. These services provide (for a low cost - usually less then $0.10 per GB) storage 'in the cloud' (meaning the files could potentially be hosted anywhere) which your application can use as persistent storage.

While this functionality is a bit out of scope for this tutorial there are some resources available which you can use to find your way:

* [How to: Make Carrierwave work on Heroku](https://github.com/carrierwaveuploader/carrierwave/wiki/How-to%3A-Make-Carrierwave-work-on-Heroku)
* [Amazon S3 - The Beginner' Guide](http://www.hongkiat.com/blog/amazon-s3-the-beginners-guide/)

As always if you require any more information or assistance your coaches will be able to assist.