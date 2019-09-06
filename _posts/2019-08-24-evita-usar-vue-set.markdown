---
layout: post
title:  "Evita usar `Vue.set`"
tags: [vue, javascript, clean code]
---
Vue hace uso de la reactividad para escuchar los cambios que se produzcan en los datos de nuestro componente y actualizar la vista si es necesario cuando estos se modifiquen.

Para lograrlo, Vue añade *getters* y *setters* a cada propiedad. De este modo, cuando modificamos un valor, realmente estamos llamando al método `setter` que se ha creado previamente, y a través de este método Vue consigue advertir el cambio y actualiza la vista en consecuencia.

Sin embargo, hay algunas situaciones en las que descubrimos que nuestro componente no se está actualizando después de modificar un valor. Veamos el siguiente componente:

{% highlight javascript %}
export default {
  data: () => ({
    post: {
      title: 'The Last Wish'
    }
  }),
  methods: {
    toggleFav() {
      this.post.fav = !this.post.fav;
    }
  }
}
{% endhighlight %}

Y su plantilla:

{% highlight html %}
<div>
  <h1>
    <span v-if="post.fav">♥</span>
    Hello!
  </h1>
  <button @click="toggleFav">Fav</button>
</div>
{% endhighlight %}

Si lo ejecutamos y probamos a usar el botón, veremos que la página no cambia. Sin embargo, si depuramos el componente con la ayuda de las *devtools* del navegador podremos comprobar que el valor del campo *fav* sí cambia aunque la vista no lo refleje.

Ya habíamos comentado que Vue añade *getters* y *setters* a las propiedades que definimos en nuestro componente. El campo `fav`, como lo hemos añadido después de que se cree el componente, no posee estos métodos y por lo tanto Vue no detecta ningún cambio cuando lo modificamos. Solucionar este problema sería tan sencillo como establecer un valor inicial para la propiedad `fav`:

{% highlight javascript %}
// ...
data: () => ({
  post: {
    title: 'The Last Wish',
    fav: false
  }
})
// ...
{% endhighlight %}

## Situaciones problemáticas

Las situaciones en las que Vue no es capaz de detectar cambios son las siguientes:

- Nuevas propiedades añadidas a un objeto
- Propiedades eliminadas de un objeto
- Establecer directamente un elemento de un array en un índice concreto
- Modificar la longitud de un array

## Métodos de Array reemplazados por Vue

Vue sobrescribe algunos métodos del objeto `Array` para poder escuchar los cambios que se produzcan haciendo uso de ellos.

Esto ocurre de manera totalmente transparente y nos permite utilizar métodos de estos objetos sin preocuparnos por la reactividad. Los siguientes métodos son modificados por Vue:

- *push*
- *pop*
- *shift*
- *unshift*
- *splice*
- *sort*
- *reverse*

## Vue.set

Probablemente ya conocías la reactividad en Vue y también estos casos particulares, pero quería llegar hasta aquí para hablar sobre una de las soluciones que se proponen a este problema, que no es otra que el método `Vue.set`.

Podemos invocar a este método de las siguiente manera en nuestros componentes:

{% highlight javascript %}
// Vue.set(object, propertyName, value)

Vue.set(this.post, 'fav', !this.post.fav);

this.$set(this.post, 'fav', !this.post.fav);

this.$set(this.posts, 1, post);
{% endhighlight %}

`Vue.set` simplemente se asegura de que el nuevo valor se establece como una propiedad reactiva, solucionando el problema al no haberla declarado inicialmente.

Es una solución sencilla y rápida, pero **en mi opinión** es una solución que debemos tratar de evitar siempre.

## Por qué evitar Vue.set

Los problemas de reactividad con Vue suelen venir a partir de un mal planteamiento de nuestros componentes y salen a luz en muchas ocasiones cuando tratamos de mutar un objeto. Utilizar un método de "fuerza bruta" como este nos evita hacer la acción correcta, refactorizar.

Nuestros componentes, además, deberían contener la menor lógica posible. Mientras más sencillo sea extraer el código de los componentes hacia otros módulos en nuestro proyecto, mucho mejor. Este método, por el contrario, acopla el código a la API de Vue al hacer uso directo de uno de sus métodos, y estaríamos obligados a modificarlo antes de poder moverlo fuera de un componente, lo que puede provocar que posterguemos una decisión importante como la de hacer una refactorización.

Por último, evitar este método nos obligará a conocer más sobre la inmutabilidad y sus beneficios en javascript, un conocimiento que nos ayudará a escribir funciones puras y reutilizables y nos evitará enfrentarnos a errores difíciles de depurar debido a mutaciones indeseadas en los datos de nuestros componentes.

## Alternativas a Vue.set

Si nos enfrentamos a uno de los casos listados anteriormente en los cuales Vue no puede detectar los cambios, tendremos las siguientes alternativas a `Vue.set` según la situación:

### Nuevas propiedades añadidas a un objeto

En esta situación, optaremos por crear un nuevo objeto que reemplace al anterior y añada la nueva propiedad. Al sustituir el objeto completamente, Vue detectará fácilmente el cambio y actualizará nuestra vista de ser necesario teniendo en cuenta el nuevo valor.

Tenemos dos opciones para lograr esto. La primera, hacer uso de `Object.assign`:

{% highlight javascript %}
toggleFav() {
  this.post = Object.assign({}, this.post, {fav: !this.post.fav});
}
{% endhighlight %}

Y la segunda, mi preferida, hacer uso de [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax):

{% highlight javascript %}
toggleFav() {
  this.post = {...this.post, fav: !this.post.fav};
}
{% endhighlight %}

### Propiedades eliminadas de un objeto

Si lo que queremos no es añadir, sino eliminar una propiedad del objeto, también podemos hacerlo sin mutarlo. Tenemos varias opciones, una de ellas sería clonar el objeto, eliminar la propiedad y sustituir el original por la copia:

{% highlight javascript %}
deleteFavProperty() {
  const post = {...this.post};
  delete post.fav;
  this.post = post;
}
{% endhighlight %}

Sin embargo aquí también podemos hacer uso de [Object Rest/Spread Properties](https://github.com/tc39/proposal-object-rest-spread), que vuelve a ser mi elección preferida:

{% highlight javascript %}
deleteFavProperty() {
  const {fav, ...rest} = this.post;
  this.post = rest;
}
{% endhighlight %}

### Establecer directamente un elemento de un array en un índice concreto

Si tratamos de hacer lo siguiente:

{% highlight javascript %}
replacePost(index, newPost) {
  this.posts[index] = newPost;
}
{% endhighlight %}

Veremos que Vue tampoco lo detecta. La opción más simple para evitar usar `Vue.set` sería el uso del método `Array.splice`. Veamos cómo quedaría:

{% highlight javascript %}
replacePost(index, newPost) {
  this.posts.splice(index, 1, newPost);
}
{% endhighlight %}

Hay que tener en cuenta que **este método muta el array**. Antes ya comentamos que Vue reemplazaba algunos métodos de `Array` para poder detectar los cambios cuando se usan, siendo `splice` uno de ellos. Gracias a esto podemos usarlo en esta situación.

Sin embargo, si queremos evitar las mutaciones, que en mi opinión es la mejor opción, podríamos emplear una de las siguientes formas, cada cual que se quede con la que más clara le parezca:

{% highlight javascript %}
replacePost(index, newPost) {
  // A
  this.posts = this.posts.map((post, postIndex) => {
    return postIndex === index ? newPost : post;
  });

  // B
  this.posts = Object.assign([], this.posts, {[index]: newPost});

  // C
  this.posts = [
    ...this.posts.slice(0, index),
    newPost,
    ...this.posts.slice(index + 1),
  ]
}
{% endhighlight %}

### Modificar la longitud de un array

Por último tenemos la más peculiar, modificar la longitud de un array:

{% highlight javascript %}
incrementPostsLength(numberOfEmptySpaces) {
  const actualLength = this.posts.length;
  this.posts.length = actualLength + numberOfEmptySpaces;
}
{% endhighlight %}

Si probamos ese caso veremos que Vue no se entera de lo que acabamos de hacer. Hay varias opciones para hacer esto. Una de ellas sería crear un *array* nuevo con el número de elementos que queremos añadir y concatenarlo al final del original, dando lugar a un nuevo *array*:

{% highlight javascript %}
incrementPostsLength(numberOfEmptySpaces) {
  const emptySpaces = new Array(numberOfEmptySpaces);
  this.posts = [...this.posts, ...emptySpaces];
}
{% endhighlight %}

## Conclusión

Personalmente prefiero aprovechar las opciones que pone el lenguaje a nuestro alcance que utilizar la API de Vue para resolver estos casos, y eligiendo siempre una opción no mute los datos de entrada.

Si optas por estas alternativas, tu código será más portable y aprenderás a sacarle partido a la inmutabilidad, un conocimiento que podrás emplear en tus próximos desarrollos con JavaScript independientemente del *framework* que uses.
