---
layout: post
title:  "Trabalhando com Exceptions"
date:   2014-07-18 10:25:00
categories: ruby
author: Douglas Rossignolli
---

Depois de 2 anos aumentando a minha experiência com Ruby e o Rails, é muito legal descobrir muitas formas de lidar com os erros. Mas eu penso "*se há alguma coisa errada, eu não quero que o meu processo continue rodando na minha aplicação*".

Então se eu não quero problemas rodando na minha aplicação, então logo eu não devo trata-los simplesmente com if/else, isso é muito ruim, além do seu código ficar todo ilegível, você passa uma certa insegurança para o processo, as vezes na hora de debugar, você nunca saber aonde realmente está acontecendo o erro, é tudo muito complexo e complicado.

Um professor meu de ruby a uns 3 anos atrás me disse -"*Se alguma coisa está errada, e você não quer isso rodando, então 'Pare!'*". E qual o melhor jeito de parar uma aplicação em ruby? Levantando uma `Exception.

Bem eu gosto de criar minhas `Exceptions` dentro de um `module`



{% highlight ruby %}
# /lib/mods/exceptions.rb
module Exceptions
  # classes omitidas
end
{% endhighlight %}

Inicialmente eu gosto de criar a minha classe Base que contém os comportamentos iniciais.

{% highlight ruby %}
# /lib/mods/exceptions/base.rb
class Exceptions::Base < StandardError
  attr_accessor :object, :type

  # starts a new instance with an object
  # @param [Object] object
  def initialize object
    self.object = object
  end

  # standard error for Models
  # @param [Object] object
  # @return [Exceptions::Base]
  def self.build object
    exception = new object
    return exception
  end
end
{% endhighlight %}

Agora uma pergunta importante, por que a minha classe Base está sendo herdada de StandardError? Veremos o significado disso mais a frente.

Repare que na classe base recebemos um Object, porém nada impede de ser um Hash, Array, Model, Controller, ou o que você quiser. agora vamos tratar os nossos models:

{% highlight ruby %}
class Exceptions::Model < Exceptions::Base
  # /lib/mods/exceptions/model.rb
  # for model errors this method build a hash with all necessary information
  # @return [String] json string
  def error
    { 
      error: { 
        model: self.model, 
        field: "#{self.model}[#{self.object.errors.first[0]}]",
        attribute: self.object.errors.first[0], 
        message: self.message
      } 
    }
  end
  # return what model is
  # @return [String]
  def model
    self.object.class.name.demodulize.tableize.singularize
  end
  # return the error message
  # @return [String]
  def message 
    "#{self.object.errors.first[1]}"
  end
  def status
    :unprocessable_entity
  end
end
{% endhighlight %}


Vamos analisar todos os métodos, primeiramente com o método `error`. <br>
Ele é o coração dessa classe de erro, aonde ele gerá um Hash, com todos os dados necessários para o erro com o model, field no padrão `model[attribute]`, o atributo em si e a sua mensagem de erro gerada pelo ActiveModel::Validations.

O método `model` é apenas uma tradução correta da classe para o formato humano. Aonde se eu possuo uma classe Compamy::Message, ele vai transformar para 'Message'.

O método `message` é a mensagem obtida do erro gerado pela validação do ActiveModel.

O método `status`, é uma método para retornar o status http default do erro, aonde o status experado para essa nossa classe é o 422.

Vamos usar a nossa classe em um model e vamos trata-lo no nosso Controller de nossa API/APP

{% highlight ruby %}
class Person < ActiveRecord::Base
  def self.build params
    person = new params
    if person.valid?
      person.save
      person
    else
      raise Exceptions::Model.build(person)
    end
  end
end
{% endhighlight%}

Um método bem simples o `build` aonde recebemos os params de um formulário/api_request e ao criar o objeto pessoa é validado e em caso de estar inválido, levantamos essa exception, parando o nosso processamento.

{% highlight ruby %}
class PeopleController < ApplicationController
  def create
    begin
      person = Person.build person_params
      render status: :ok, json: person.to_json
    rescue => exception
      render status: exception.status, json: exception.error
    end
  end
  
  private
  def person_params
    params.require(:person).permit(:name, :email, :password, :avatar)
  end

end
{% endhighlight %}

Agora vamos lá, lembra da pergunta que eu havia feito sobre o porque herdar StandardError? Agora eu posso responder,
quando tratamos uma StandardError, podemos simplificar a nossa forma de tratar as exceptions, por exemplo

{% highlight ruby %}
def processo
  begin
   # faz um processo com muitas coisas
  rescue Expceptions::Model
    # trata exception model
  rescue Expceptions::Simple
    # trata exception simple
  rescue ActiveRecord::RecordNotFound
    # trata not found
  rescue Exception
    # trata exception básica
  end
end
{% endhighlight %}

Você fez um processo e precisou tratar 4 exceptions diferentes, se você trabalhar com o StandarError, e trabalhar com o `duck_type`, então pode poderá deixar tudo mais simples fazendo desta forma:

{% highlight ruby %}
def processo
  begin
   # faz um processo com muitas coisas
  rescue => exception
    # se todas as exceptions possui o método `status` e `error`, logo
    render status: exception.status, json: exception.error
  end
end
{% endhighlight %}


###Conclusão

Trabalhar com exceptions de forma organizada e planejada, você consegue padronizar seu tratamento de error e deixa o sua aplicação segura, confiável e com pequena margem de errors crítico. Organizar seus processamentos respondendo de forma correta a exceptions causadas pelo usuário, e informando corretamente o que realmente aconteceu, para que seja corrigido da forma mais rápida e simples.
