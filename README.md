# Apuntes de Symfony

## Comandos necesarios para crear una aplicación con Symfony


Todos estos comandos son necesarios en el proceso de creación de una nueva aplicación en Symfony. Aunque el de _SecurityBundle_ no sea prioritario a la hora de crear el proyecto es recomendable ejecutarlo al inicio para evitar tener que usarlo más adelante.

**1. Crear nuevo proyecto**
```
symfony new [nombreProyecto] --full --version=5.4
```
**Importante:** tras esto debemos hacer ` cd `,  dirigirnos a la carpeta que hemos creado y ejecutar el resto de comandos allí.

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
` (sustituyendo _[nombre]_ por el nombre de nuestra base de datos).

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

Para ello solo debemos insertar los enlaces de Bootstrap en el _head_ del twig base. Aunque están en la página de Bootstrap los pongo aquí, que no me cuesta nada y no vaya a ser que cambien de nuevo la página.
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

**1. Crear usuario**
```
php bin/console make:user
```

**2. Crear formulario de registro**
```
php bin/console make:registration-form
```

**3. Crear formulario login**

Aquí lo mejor es seguir las instrucciones de la [documentación](https://symfony.com/doc/current/security.html*form-login) paso por paso.
Hay que llevar cuidado y asegurarse de que la ruta del _login_ es la misma que tenemos en el controlador.

**3. Crear logout**

En security.yaml, debajo de lo que se ha puesto en login en el paso anterior se introduce lo siguiente:
```
logout:
	path: app_logout
	target: home
```
**Importante:** _home_ se ha puesto como ejemplo, ya que es la ruta más común, pero realmente se puede usar cualquiera.

En _routes.yaml_ hay que poner esto para que exista _app_logout_ (en el caso del _logout_ no se utiliza controlador).
```
app_logout:
	path: /logout
	methods: GET
```

**5. Privatizar zonas**

Para privatizar una zona tendremos que modificar el _security.yaml_ e introducir lo siguiente (poniendo la ruta de la función que queremos privatizar):
```
access_control:
	-{path: ^/backend, roles: ROLE_USER}
```

Otra manera de privatizar una función es introducir las siguientes líneas al principio de la misma:
```
	$this->denyAccessUnlessGranted('ROLE_USER');
	$this->createAccessDeniedException('Acceso denegado, debes ser usuario registrado');
```

**Nota:** no sé si ambos métodos son compatibles, pero como lo más seguro es que no lo sean usad solo uno a la vez (preferentemente el del _security.yaml_).

## Formularios
En Symfony podemos crear formularios de dos maneras: con clase o sin clase. Si lo hacemos con clase los campos deben pertenecer a la entidad con la que haremos el formulario. Si es sin clase podremos añadir campos que no sean de dicha entidad.

**Formulario con clase**

Para generar el Type, solo tendremos que ejecutar el comando `php bin/console make:form`.

Tras esto introducimos lo siguiente en el controlador, sustituyendo los valores por los datos de nuestra aplicación:
```
public function new(Request $request): Response
    {
        // just set up a fresh $task object (remove the example data)
        $task = new Task();

        $form = $this->createForm(TaskType::class, $task);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            // $form->getData() holds the submitted values
            // but, the original `$task` variable has also been updated
            $task = $form->getData();

            // ... perform some action, such as saving the task to the database

            return $this->redirectToRoute('task_success');
        }

        return $this->renderForm('task/new.html.twig', [
            'form' => $form,
        ]);
    }
```

**Formulario sin clase**

Este formulario se genera directamente desde el controlador, por lo que no es necesario generar ningún _Type_.

Para crearlo haremos uso de _createFormBuilder_, que como podemos ver en este ejemplo, se usa de forma muy similar al _Type_.

```
public function new(Request $request): Response
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task', TextType::class)
            ->add('dueDate', DateType::class)
            ->add('save', SubmitType::class, ['label' => 'Create Task'])
            ->getForm();
        
    }
```
La mayor diferencia con el formulario con clase radica en lo que viene después, ya que para insertar los datos en la BBDD tendremos que usar _setters_ (y _adds_ si se trata de tablas relacionadas).

```
$form->handleRequest($request);
        
if ($form->isSubmitted() && $form->isValid()) {
	$em = $this->getDoctrine()->getManager();
	$data = $form->getData();

	$task->setTask($data['task]);
	$task->setDueDate($data['dueDate']);

	$em->persist($task);
	$em->flush();


	return $this->redirectToRoute('task_success');
}

return $this->renderForm('task/new.html.twig', [
	'form' => $form,
]);
```

En conjunto quedaría algo tal que así:
```
public function new(Request $request): Response
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task', TextType::class)
            ->add('dueDate', DateType::class)
            ->add('save', SubmitType::class, ['label' => 'Create Task'])
            ->getForm();
        $form->handleRequest($request);
        
		if ($form->isSubmitted() && $form->isValid()) {
			$em = $this->getDoctrine()->getManager();
			$data = $form->getData();

			$task->setTask($data['task]);
			$task->setDueDate($data['dueDate']);

			$em->persist($task);
			$em->flush();


			return $this->redirectToRoute('task_success');
		}

		return $this->renderForm('task/new.html.twig', [
			'form' => $form,
		]);
    }
```

Esto está bastante resumido, pero en la [documentación](https://symfony.com/doc/current/forms.html) está todo más detallado y además tiene la explicación de los distintos [tipos de campo](https://symfony.com/doc/current/reference/forms/types.html).

**Las plantillas de los formularios**

Para que el formulario se visualice basta con poner `{{ form(form) }}` en el twig, pero si queremos una mayor personalización es mejor usar `{{ form_start(form) }}` y `{{ form_end(form) }}` como podemos ver en este ejemplo:
```
{{ form_start(form) }}
    <div class="my-custom-class-for-errors">
        {{ form_errors(form) }}
    </div>

    <div class="row">
        <div class="col">
            {{ form_row(form.task) }}
        </div>
        <div class="col" id="some-custom-id">
            {{ form_row(form.dueDate) }}
        </div>
    </div>
{{ form_end(form) }}
```

Además en las _form_row_ podemos añadir variables (como _labels_, por ejemplo) en caso de que no las hayamos puesto en el _Builder_. 

**Ejemplo:**
```
{{ form_row(form.name, {'label': 'foo'}) }}
```

Al igual que antes, en la [documentación](https://symfony.com/doc/current/form/form_customization.html) hay más ejemplos.

**Importante:** las plantillas se hacen igual tanto en formularios con clase como sin clase.

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

Aunque realmente se puede hacer de muchas maneras (esto es con una _navbar_ de Bootstrap) lo importante aquí es el uso del bucle for para generar los enlaces que están guardados en la base de datos y en el caso de tener que usar inicios y cierres de sesión utilizar el `if is_granted`. Si no los hay se quedaría algo así:

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
					</li>
				{% endfor %}
               
				</ul>
			</div>
		</div>
	</nav>
{% endblock %}
```

## Validación
**1. Crear validador**

Para crear el validador tendremos que ejecutar el comando `php bin/console make:validator`, que generará dos archivos php en la carpeta de validación (si no existía previamente el comando la generará automáticamente) y a partir de ahí modificamos el fichero _Validator_ para que se ajuste a lo que necesitamos.

Un ejemplo sería el de teléfonos móviles de Españita:
```
if (!preg_match('((6|7)[0-9]{8})', $value, $matches)) {
	$this->context->buildViolation($constraint->message)
		->setParameter('{{ string }}', $value)
		->addViolation();
}
```

O el de DNIs:
```
$dni = $value;
$numero = substr($dni, 0, -1);
$letra  = strtoupper(substr($dni, -1));

if (!is_string($value)) {
	throw new UnexpectedValueException($value, 'string');            
}

if(!preg_match(("/^[0-9]{8}[TRWAGMYFPDXBNJZSQVHLCKE]$/i"), $value, $matches )){
	$this->context->buildViolation($constraint->message)
		->setParameter('{{ string }}', $value)
		->addViolation();
}

if($letra != substr("TRWAGMYFPDXBNJZSQVHLCKE", strtr($numero, "XYZ", "012")%23, 1)){
	$this->context->buildViolation($constraint->message)
		->setParameter('{{ string }}', $value)
		->addViolation();
}
    
```

**2. Poner los constraints en el formulario**

Dentro del array del campo que necesitamos validar en el formulario hay que poner `['constraints' => [new Clase()]]` sustituyendo _Clase_ por el nombre de la clase validadora (sin poner _Validator_).

**Ejemplo:**
```
->add('telefono', TextType::class, ['constraints' => [new TelefonoMovil()]])
```


