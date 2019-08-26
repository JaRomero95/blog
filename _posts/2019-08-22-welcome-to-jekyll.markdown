---
layout: post
title:  "Jekyll, tu blog rápido, potente y sin backend"
tags: [jekyll, vuejs, ruby]
---
Seguro que alguna vez te has planteado crear un blog donde compartir contenido, y puede que hayas probado algunas herramientas como Wordpress, Joomla, Drupal, Blogger... Puede que incluso hayas valorado escribir tus posts en páginas como Medium o Dev.to. Si lo que estás buscando es tener un blog sencillo, propio y personalizable, te animo que a le eches un ojo a [Jekyll](https://jekyllrb.com/).

Con Jekyll puedes tener en cuestión de minutos un blog básico donde empezar a escribir tus primeros posts y no necesitarás ni siquiera tener un dominio ni un hosting, solamente una cuenta de GitHub.

Tu sitio web, aunque verás que hace uso de Ruby durante el desarrollo, finalmente generará unas páginas estáticas que podrán desplegarse en cualquier plataforma sin preocuparnos de lenguajes de programación o bases de datos.

Este blog ha sido creado con Jekyll, y te cuento algunas de sus ventajas que me han hecho decantarme por esta herramienta:

- No es necesario mantener un hosting, puedes desplegarlo directamente con [GitHub Pages](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll).
- Puedes escribir tus posts en Markdown.
- Puedes elegir entre una gran cantidad de temas a instalar [aquí](http://jekyllthemes.org/) y [aquí](https://jekyllthemes.io/).
- Los temas son fáciles de sobrescribir, puedes modificar partes de las plantillas y añadir los recursos *js* y *css* que desees.
- Tienes a tu disposición una buena cantidad de [plugins para Jekyll](https://github.com/planetjekyll/awesome-jekyll-plugins) desarrollados por la comunidad.
- Si lo deseas, puedes desarrollar tus propios plugins con Ruby para añadir nuevas funcionalidades a tu sitio.
- Gracias a los plugins existentes, dispondrás fácilmente de metatags para tus posts, sitemap y RSS.

En resumen, dispondrás de un blog *developer friendly* en unos minutos y sin quebraderos de cabeza.

## Crear tu blog y desplegarlo en GitHub Pages

Vamos a lo que interesa, los pasos a seguir para tener listo el blog. Aquí tienes un resumen de ellos, te recomiendo que accedas a la [página oficial](https://jekyllrb.com/docs/) para leer la guía actualizada.

### Crear nuestro sitio

Como Jekyll es una gema de Ruby, necesitaremos tener instalado Ruby en nuestro equipo. Si aún no lo tienes, descárgalo en su [página oficial](https://www.ruby-lang.org/en/downloads/).

Una vez tengamos Ruby, instalamos las gemas de Jekyll y `bundler` si aún no lo tenemos:

{% highlight bash %}
gem install jekyll bundler
{% endhighlight %}

Nos movemos al directorio donde queramos crear nuestro proyecto y ejecutamos lo siguiente:

{% highlight bash %}
jekyll new myblog
{% endhighlight %}

Esto creará un proyecto completo y listo para usar. Ahora simplemente nos movemos al directorio que se acaba de generar y ejecutamos el servidor de desarrollo para echarle el primer vistazo a nuestro
nuevo blog:

{% highlight bash %}
cd myblog
bundle exec jekyll serve
{% endhighlight %}

¡Listo! Nuestro sitio ya debería estar funcionando y ahora simplemente debemos acceder a [http://localhost:4000](http://localhost:4000) para visualizarlo.

### Escribir un post

Escribir un nuevo artículo no podría ser más sencillo. Dentro de nuestro proyecto, tenemos la carpeta `_posts` donde deberíamos tener ya un post de prueba con el nombre `YYYY-MM-DD-welcome-to-jekyll.markdown`. Ábrelo y edítalo o crea un archivo en esa carpeta siguiendo la nomenclatura `YYYY-MM-DD-url-de-tu-post.markdown`.

Observa que al inicio del archivo hay una cabecera con metadatos como el título del post o el *layout* que utiliza. Esta cabecera debe existir en todos tus posts y podrás modificar los metadatos y añadir nuevos que incorpores en tus plantillas.

### Configuración de tu sitio

En el archivo `_config.yml` se encuentran los valores de configuración entre los cuales seguramente estén el título y la descripción de tu sitio que tu tema instalado está mostrando.

Modifícalos a tu gusto y reinicia el servidor ya que los cambios de este fichero no se refrescan automáticamente.

### Modificar tu tema

A la hora de escribir este artículo, el proyecto que hemos creado siguiendo estos pasos se generaba haciendo uso del tema [minima](https://github.com/jekyll/minima).

Para personalizar la plantilla de tu tema, copia las partes que quieras sobrescribir en tu proyecto y empieza a modificarlas. El tema instalado se encuentra en la carpeta en la cual se instalen las gemas de Ruby, que dependerá de tu instalación.

En mi caso, copié el archivo `/my-ruby-folder/lib/ruby/gems/2.6.0/gems/minima-2.5.1/_includes/footer.html` al directorio `~/miblog/_includes/footer.html`.

Haz tus modificaciones y recarga la página para ver los cambios.

### Despliegue

No hay mucho que hacer a la hora de desplegarlo en GitHub Pages ya que Jekyll es totalmente compatible con esta plataforma, es más, la propia plataforma de GitHub Pages funciona gracias a Jekyll.

Primero, instala la gema `github-pages` en tu proyecto añadiéndola a tu Gemfile. Es más, en tu Gemfile debería estar la línea `gem "github-pages", group: :jekyll_plugins` esperando a que la descomentes. Hazlo y después ejecuta `bundle install`.

**Nota:**
es posible que haya algún conflicto de versiones a la hora de instalar la nueva gema. Probablemente puedas solucionarlo ejecutando `bundle update`.

Después, crea tu repositorio en GitHub, sube tu proyecto y habilita GitHub Pages en la configuración del repositorio.

Si nunca que leas la [documentación oficial](https://jekyllrb.com/docs/github-pages/#the-github-pages-gem) si es la primera vez que haces uso de GitHub Pages.

#### Limitaciones de GitHub Pages

No todo podía ser tan bonito. Con GitHub Pages tenemos un hosting gratuito y sin apenas configuración, pero al desplegar nuestro proyecto con esta herramienta tenemos muchas limitaciones con los plugins que podemos usar, ya que el proyecto se ejecuta en modo seguro lo que hace que muchos plugins no funcionen.

Pero no te preocupes, si tu blog necesita saltar esas limitaciones puedes conseguirlo sin necesidad de cambiar de hosting. Simplemente construye tu proyecto en tu equipo, haz *commit* de los archivos generados y súbelos a la rama en la que hayas configurado GitHub Pages.

Como comentamos antes, el uso de Ruby es solo para generar los ficheros de nuestro blog, que al final consiste en una serie de archivos estáticos. GitHub nos limita la ejecución de Ruby, pero si lo generamos en local y lo subimos ya construido podremos hacer uso de todos los plugins que deseemos.
