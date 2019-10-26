---
layout: post
title:  "El método `tap` en Ruby"
tags: [ruby, clean code]
description: "Aprende a utilizar el método `tap` en Ruby: simple, útil y elegante."
---
El método `tap` de Ruby es tan simple como eficaz, aunque no muchos hacen uso de él.

Este método forma parte de la clase `Object` y básicamente lo que hace cuando lo invocamos sobre la instancia de un objeto es enviarse como parámetro al bloque que le pasamos y devolverse a sí mismo al final de este.

Es útil cuando necesitamos que un método del objeto devuelva la propia instancia en vez del resultado de la última operación, y de este modo poder concatenar la llamada con otros métodos del mismo objeto o simplemente poder referenciarlo en una variable.

Es tan simple que cuesta incluso ver su utilidad si nunca lo has usado. Vamos a verlo en un ejemplo:

Tenemos la siguiente clase de Ruby:

{% highlight ruby %}
class Hobbit
  attr_accessor :name

  def initialize(name)
    self.name = name
  end

  def eat
    puts "#{name} se zampa un pan de lembas entero."
  end

  def smoke
    puts "#{name} hace un descanso para fumar Hoja del Valle Largo."
  end
end
{% endhighlight %}

Para probarla, vamos a instanciar un objeto de esta clase e invocar a sus métodos:

{% highlight ruby %}
hobbit = Hobbit.new('Pippin')

hobbit.eat
hobbit.smoke
{% endhighlight %}

No está mal, pero estaría mejor si pudiéramos llamar a ambos métodos seguidos. Vamos a modificar los métodos para que devuelvan el objeto al final de cada uno y conseguir hacer *method chaining*:

{% highlight ruby %}
class Hobbit
  # ...
  def eat
    puts "#{name} se zampa un pan de lembas entero."
    self
  end

  def smoke
    puts "#{name} hace un descanso para fumar Hoja del Valle Largo."
    self
  end
  # ...
end
{% endhighlight %}

Y ahora sí, podemos encadenar las llamadas:

{% highlight ruby %}
hobbit = Hobbit.new('Pippin')

hobbit.eat.smoke
{% endhighlight %}

Hemos conseguido lo que buscábamos, pero devolver `self` de manera explícita al final de cada método queda algo forzado. Justo aquí es donde entra el método `tap` para conseguir que nuestros métodos se expresen mejor. Vamos a cambiar otra vez nuestros métodos:

{% highlight ruby %}
class Hobbit
  # ...
  def eat
    tap do |hobbit|
      puts "#{hobbit.name} se zampa un pan de lembas entero."
    end
  end

  def smoke
    tap do |hobbit|
      puts "#{hobbit.name} hace un descanso para fumar Hoja del Valle Largo."
    end
  end
  # ...
end
{% endhighlight %}

Y ahora sí, hemos mantenido la posibilidad de hacer *method chaining* pero con un poco más de azúcar sintáctico.

Es cierto que el ejemplo es demasiado simple para ver la utilidad real de este método, ya que era incluso más corto de la forma anterior, y además ni siquiera necesitamos hacer uso de la variable que recibe el bloque, así que mejor veamos uno un poco más complejo:

{% highlight ruby %}
ring_bearer = Hobbit.new('Frodo').tap do |hobbit|
  hobbit.take(:the_one_ring)
  hobbit.go_to(:orodruin)
  hobbit.throw_item(:the_one_ring)
end

# do some stuff

ring_bearer.sleep
{% endhighlight %}

¡Eso es todo! Personalmente uso estos detalles para tratar de hacer mi código más expresivo y fácil de leer, pero habrá a quien le parezca más simple la primera aproximación, queda todo a tu elección.
