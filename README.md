# Apuntes de Symfony

## Comandos necesarios para crear una aplicación con Symfony


Todos estos comandos son necesarios en el proceso de creación de una nueva aplicación en Symfony. Aunque el de _SecurityBundle_ no sea prioritario a la hora de crear el proyecto es recomendable ejecutarlo al inicio para evitar tener que usarlo más adelante.

**1. Crear nuevo proyecto**
```
symfony new [nombreProyecto] --full --version=5.4
```
<span style="color:yellow">**Importante:**</span> tras esto debemos hacer ` cd `,  dirigirnos a la carpeta que hemos creado y ejecutar el resto de comandos allí.

**2. Instalar el ORM**
```
composer require orm
```

**3. Instalar annotations**
```
composer require doctrine/annotations
```

**4. Instalar MakerBundle**
```
composer require symfony/maker-bundle --dev
```

**5. Instalar twig**
```
composer require twig
```

**6. Instalar SecurityBundle**
```
composer require symfony/security-bundle
```

**7. Base de datos**

Para que la base de datos funcione correctamente accedemos al fichero _.env_ y sustituimos la línea que hay por defecto para la base de datos por `DATABASE_URL="mysql://root:@127.0.0.1:3306/[nombre]?serverVersion=mariadb-10.4.22"
`

## Otros comandos

**Generar un controlador**
```
php bin/console make:controller
```
**Generar una entidad**
```
php bin/console make:entity
```
**Generar un crud**
```
php bin/console make:crud
```
**Generar un validador**
```
php bin/console make:validator
```
**Generar un formulario**
```
php bin/console make:form
```
**Actualizar la base de datos**
```
php bin/console doctrine:schema:update --force
```
**Borrar la base de datos**
```
php bin/console doctrine:schema:drop --force
```

## Instalar Bootstrap

Aunque no sea una prioridad en estos apuntes hay varios ejemplos de plantillas que usan Bootstrap, por lo que pongo esto aquí ya que se hace rápido y quedará...

![fino señores](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT_k53NdYe-fjVKcHe6_NmJsBW7K-Bz5m0pQtvs4gt3CAtAkbRN77utS5tX9MvJIpOGYUc&usqp=CAU)

Para ello solo debemos insertar los enlaces de Bootstrap en el _head_ del twig base. Aunque están en la página de Bootstrap los pongo aquí que no me cuesta nada y no vaya a ser que cambien de nuevo la página.
```
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
```
Para que los formularios se vean aún mejor ponemos esto también en el _head_ del twig base:
```
{% if form is defined and form %}
	{% form_theme form 'bootstrap_5_layout.html.twig' %}
{% endif %}
```
## Seguridad

**Paso 1: crear usuario**
```
php bin/console make:user
```

**Paso 2: crear formulario de registro**
```
php bin/console make:registration-form
```

**Paso 3: crear formulario login**

Aquí lo mejor es seguir las instrucciones de la [documentación](https://symfony.com/doc/current/security.html*form-login) paso por paso.
Hay que llevar cuidado y asegurarse de que la ruta del login es la misma que tenemos en el controlador.

**Paso 4: crear logout**

En security.yaml, debajo de lo que se ha puesto en login en el paso anterior se introduce lo siguiente:
```
logout:
	path: app_logout
	target: home
```
<span style="color:yellow">**Importante:**</span> _home_ se ha puesto como ejemplo, ya que es la ruta más común, pero realmente se puede usar cualquiera.

En _routes.yaml_ hay que poner esto para que exista app_logout (en el caso del logout no se utiliza controlador).
```
app_logout:
	path: /logout
	methods: GET
```

**Paso 5: privatizar zonas**

Al introducir las siguientes líneas al principio de una función su ruta quedará privatizada:
```
	$this->denyAccessUnlessGranted('ROLE_USER');
	$this->createAccessDeniedException('Acceso denegado, debes ser usuario registrado');
```

## Crear menú dinámico

**1. Crear la entidad**

Creamos la entidad _Menu_ con las propiedades _id_ y _titulo_. Tras ello actualizamos la base de datos.

**2. El controlador**

Aquí existen dos posiblidades: usar un controlador ya existente y crear un template nuevo o generar un controlador desde la línea de comandos. De una manera u otra la función deberá ser algo así (obviamente la ruta del twig no tiene que ser la misma):
```
    public function menu(ManagerRegistry $doctrine)
    {
	    $menus=$doctrine->getRepository(Menu::class)->findAll();
	    return $this->render('menu/menu.html.twig',array("menus"=>$menus));
    }
```

**3. El twig**

El contenido del twig debe ser similar a esto:

```
{% extends 'base.html.twig' %}

{% block menu %}
	<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
		<div class="container-fluid">
			<a class="navbar-brand" href="#">Navbar</a>
			<button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
				<span class="navbar-toggler-icon"></span>
			</button>
			<div class="collapse navbar-collapse" id="navbarNav">
				<ul class="navbar-nav">
				{% for item in menus %}
					<li class="nav-item">
						<a class="nav-link" href="{{path(item.enlace)}}">{{item.titulo}}</a>
                        {% if is_granted("IS_AUTHENTICATED_REMEMBERED") %}
                            <a class="nav-link" href="{{ path('app_logout') }}">
                                Cerrar sesión
                            </a>
                            {% else %}
                                <a class="nav-link" href="{{ path('login') }}">Iniciar sesión</a>
                        {% endif %}
					</li>
				{% endfor %}
               
				</ul>
			</div>
		</div>
	</nav>
{% endblock %}
```

Aunque realmente se puede hacer de muchas maneras (esto es con una navbar de Bootstrap) lo importante aquí es el uso del bucle for para generar los enlaces que están guardados en la base de datos y en el caso de tener que usar inicios y cierres de sesión utilizar el `if is_granted`.
