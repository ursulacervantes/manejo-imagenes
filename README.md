# Manejo de Imágenes

El objetivo del siguiente tutorial es crear un proyecto para subir imágenes. Los pasos a seguir son:

1. Abrir la consola y crear un nuevo proyecto `adminImages`

  ```ruby
  @ rails new adminImages
  ```

2. Crear un scaffold

  *Scaffold* se refiere a la generación automática de un conjunto simple de modelo, vista y controlador, normalmente para una sola tabla. Es una forma rápida de generar la mayor parte de piezas de una aplicación.

  Vamos a crear un modelo simple con los siguientes atributos:

  * name (string)
  * description (text)
  * picture (string) — Este campo va a contener una imagen (el nombre del archivo, para ser más precisos)

  ```ruby
  @ rails generate scaffold idea name:string description:text picture:string
  ```

3. Migrar el modelo a la base de datos

  Tenemos que ejecutar la migración puesto que hemos creado un nuevo modelo.

  Las *migraciones* son clases de Ruby que están diseñadas para simplificar la creación y modificación de tablas en la base de datos. Rails usa el comando `rake` para ejecutar las migraciones.

  ```ruby
  @ rake db:migrate
  ```

4. Iniciar el servidor de rails

  ```ruby
  @ rails s
  ```

5. Abrir el navegador http://localhost:3000/ideas

6. Abre `app/views/layouts/application.html.erb` en tu editor de texto y encima de la línea

  ```ruby
  <%= stylesheet_link_tag "application", media: "all", "data-turbolinks-track" => true %>
  ```

  incluya

  ```ruby
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">

  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous">
  ```

  y reemplace
  ```ruby
  <%= yield %>
  ```
  por

  ```ruby
  <div class="container">
    <%= yield %>
  </div>
  ```

  En el mismo archivo, debajo de `<body>` agregar

  ```html
  <nav class="navbar navbar-default" role="navigation">
    <div class="container">
      <div class="navbar-header">
        <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
          <span class="sr-only">Toggle navigation</span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
          <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="/">Mi Aplicacion</a>
      </div>
      <div class="collapse navbar-collapse">
        <ul class="nav navbar-nav">
          <li class="active"><a href="/ideas">Ideas</a></li>
        </ul>
      </div>
    </div>
  </nav>
  ```

  y antes de `</body>` agregar

  ```html
  <footer>
    <div class="container">
      UPC 2020
    </div>
  </footer>
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
  ```

7. Abre `app/assets/stylesheets/application.css` y al final agregar

  ```css
  footer { margin-top: 50px; }
  ```

8. Agregar la gema para subir imagenes

  Abre Gemfile y agregue

  ```ruby
  gem 'carrierwave'
  ```

  Carriewave es una solución para subir archivos con Rails, Sinatra y otros web frameworks de Ruby. [Aquí puedes leer más sobre esta gema.](https://github.com/carrierwaveuploader/carrierwave)


9. Ejecute bundle

  ```ruby
  @ bundle
  ```

10. Para subir imágenes necesitamos generar un `uploader`. Para esto utilizamos el siguiente comando:

  ```ruby
  @ rails generate uploader Picture
  ```

  Si revisas el folder `app/uploaders`, vas a encontrar un archivo llamado `picture_uploader.rb`. Nota que el archivo tiene comentarios y ejemplos bastante útiles que te pueden ayudar a comenzar.

  ```ruby
  storage :file
  ```

  Por defecto, los archivos serán guardados en la carpeta `public/uploads`. Por lo tanto, será mejor excluir esta carpeta de nuestro sistema de versionamiento.

  ##### .gitignore

  `public/uploads`

  También puedes modificar el método `store_dir` dentro del `uploader` para elegir otra carpeta.

11. Ahora tenemos que incluir o `mount` el uploader en nuestro modelo.

  Abre `app/models/idea.rb` y debajo de la lí­nea

  ```ruby
  class Idea < ActiveRecord
  ```

  agregue

  ```ruby
  mount_uploader :picture, PictureUploader
  ```

12. Abre `app/views/ideas/_form.html.erb` y cambie

  ```ruby
  <%= f.text_field :picture %>
  ```

  a
  ```ruby
  <%= form.file_field :picture, id: :idea_picture %>
  ```


13. Abre `app/views/ideas/show.html.erb` y cambie
  ```ruby
  <%= @idea.picture %>
  ```

  a

  ```ruby
  <%= image_tag(@idea.picture_url, :width => 600) if @idea.picture.present? %>
  ```


14. Abrir el archivo `config/routes.rb` y después de la primera línea, agregar

  ```ruby
  root :to => redirect('/ideas')
  ```


15. Reinicie el servidor de rails y abra su aplicación en el navegador.

  ```ruby
  @ rails s
  ```


15. Posterior a visualizar su aplicacion, abrir el archivo `app/views/ideas/index.html.erb` y reemplazar el codigo de la tabla por:

  ```ruby
  <h3>Lista ideas</h3>

  <% @ideas.in_groups_of(3) do |group| %>
    <div class="row">
      <% group.compact.each do |idea| %>
        <div class="col-md-4">
          <%= image_tag idea.picture_url, width: '100%' if idea.picture.present?%>
          <h4><%= link_to idea.name, idea %></h4>
          <%= idea.description %>
        </div>
      <% end %>
    </div>
  <% end %>
  ```

## Siguientes pasos

### Generar Thumbnails

Para cortar y escalar imágenes necesitamos otra herramienta. Carrierwave tiene soporte para las gemas `RMagick` y `MiniMagick`. Estas herramientas nos ayudan a manipular imágenes con la ayuda de `ImageMagick`. Esta última es una solución que nos permite editar imágenes existentes y genera nuevas. Puedes elegir cualquiera de las dos gemas mencionadas. Recomiendo `MiniMagick` porque es más fácil de instalar y tiene mejor soporte.

##### Gemfile

```ruby
gem 'mini_magick'
```

### Agregar validaciones

Actualmente nuestro `uploading` funciona pero no estamos validando el formato. Esto, por su puesto, no es bueno. Para trabajar con imágenes debemos permitir los archivos con extención .png, .jpg y .gif

##### uploaders/image_uploader.rb

```ruby
def extension_whitelist
    %w(jpg jpeg gif png)
end
```

También puedes agregar verificaciones de tipo de contenido con el método `content_type_whitelist`

##### uploaders/image_uploader.rb

```ruby
def content_type_whitelist
    /image\//
end
```

Además, es posible tener un blacklist de tipos de archivos, por ejemplo ejecutables. Para esto se define el método `content_type_blacklist`

Para agregar validaciones de tamaño de archivo vamos a requerir otra gema.

##### Gemfile

`gem 'file_validators'`


### Trabajar con múltiples archivos

En el caso que necesitesmos trabajar con varios archivos necesitamos agregar serialized field (para SQLite) o a JSON field (para Postgres o MySQL). Para esto se quita la gema `sqlite3` del archivo Gemfile y se agrga `pg`:

##### Gemfile

`gem 'pg'`

### Usar Cloud Storage

No siempre es posible guardar los archivos de forma local. Podemos usar Amazon S3, por ejemplo. Para esto, necesitamos agregar la gema `fog-aws` :

##### Gemfile

`gem "fog-aws"`
