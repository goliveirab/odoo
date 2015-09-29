:banner: banners/build_a_module.jpg

.. queue:: backend/series

=======================
Desarrollo de un Módulo
=======================

.. warning::

    Este tutorial requiere :ref:`tener Odoo instalado <setup/install>`

Iniciar/Detener el servidor Odoo
================================

Odoo utiliza una arquitectura cliente/servidor en la cual los clientes son navegadores web
que accesan el servidor Odoo a través de RPC.

La lógica del negocio y sus extensiones generalmente son ejecutadas del lado del servidor,
además el soporte a características del cliente (ej. representación de datos nuevos como
mapas interactivos) puede ser agregada al cliente.

Para iniciar el servidor, simplemente aplique el comando :ref:`odoo.py
<reference/cmdline>` en la terminal, si es necesario agregue la ruta completa al
archivo:

.. code:: bash

    odoo.py

El servidor es detenido pulsando ``Ctrl-C`` dos veces desde la terminal, o
matando el proceso correspondiente.

Desarrollo de un módulo Odoo
============================

Tanto las extensiones del servidor como del cliente son empaquetadas como *modulos* los cuales
opcionalmente son cargados en la *base de datos*.

Los módulos Odoo pueden agregar nueva lógica de negocio a un sistema Odoo, o
alterar una lógica de negocio existente: puede crear un módulo para agregar las
reglas de contabilidad de su país al soporte general de contabilidad de Odoo, mientras 
el siguiente módulo agrega soporte para la visualizacion en tiempo real de una flota de autobises.

Por consiguiente, todo en Odoo comienza y termina con módulos.

Composición de un módulo
------------------------

Un módulo de Odoo puede contener varios elementos:

Objetos de negocio
    declarados como clases de Python, these resources are automatically persisted
    by Odoo based on their configuration

Archivos de datos
    archivos XML o CSV que declaran metadatos (vistas o flujos de trabajo), configuración
    de datos (parametrización de datos), datos de demostración y mas

Controladores web
    gestionan las peticiones de los navegadores web

Datos web esstáticos
    archivos de imagenes, CSS o javascript usados por la interfaz web o sitio web

Estructura del módulo
---------------------

Cada modulo es un directorio dentro de un *directorio de módulo*. Los directorios de
módulos con específicados usando la opción :option:`--addons-path <odoo.py --addons-path>`.

.. tip::
    :class: aphorism

    muchas de las opciones de la línea de comandos pueden ser configuradas usando :ref:` un archivo de
    configuración <reference/cmdline/config>`

Un módulo de Odoo es declarado por su :ref:`manifest <reference/module/manifest>`.
Vea la :ref:`manifest documentation <reference/module/manifest>` information
about it.

Un módulo es tambien un
`paquete Python <http://docs.python.org/2/tutorial/modules.html#packages>`_
con un archivo ``__init__.py``, que contiene instrucciones para la importación de varios archivos
Python en el módulo.

Por ejemplo, si el módulo tiene un único archivo ``mymodule.py``, el ``__init__.py``
puede contener::

    from . import mymodule

Odoo provee mecanismos para ayudar a la configuración de un módulo nuevo, :ref:`odoo.py
<reference/cmdline/server>` tiene un sub-comando :ref:`scaffold
<reference/cmdline/scaffold>` para crear un módulo vacío:

.. code-block:: console

    $ odoo.py scaffold <nombre del módulo> <donde se colocará>

El comando crea un sub-directorio para su módulo, y crea automáticamente un
monton de archivos estándar de un módulo. Muchos de ellos solo contiene códido comentado
o XML. El uso de muchos de estos archivos sera explicada a lo largo de este tutorial.

.. exercise:: Module creation

    Use la línea de comando anterior para crear un módulo Open Academy vacío, e
    instalelo en Odoo.

    .. only:: solutions

        #. Escriba el comando ``odoo.py scaffold openacademy addons``.
        #. Adapte el archivo manifiesto a su módulo.
        #. No se preocupe por los otros archivos.

        .. patch::

Mapeo Objeto-Relacional
-----------------------

Un componente clave de Odoo es la capa :abbr:`ORM (Object-Relational Mapping)`.
Esta capa evita tener que escribir a mano mucho del :abbr:`SQL (Structured Query Language)`
y provee servicios de escalabilidad y seguridad\ [#rawsql]_.

Los objetos de negocio son declarados como clases Python que extienden
:class:`~openerp.models.Model` la cual los integra dentro del sistema de persistencia
automático.

Los módulos puede ser configurados fijando varios atributos en su definición.
El atributo más importnate es
:attr:`~openerp.models.Model._name` el cual es obligatorio y define el nombre del modelo
en el sistema Odoo. Aquí se muestra una definición minimalista de un
modelo::

    from openerp import models
    class MinimalModel(models.Model):
        _name = 'test.model'

Campos del modelo
-----------------

Los campos son usados para definir lo que el modelo puede almacenar y donde. Los campos son
definidos como atributos en la clase del modelo::

    from openerp import models, fields

    class LessMinimalModel(models.Model):
        _name = 'test.model2'

        name = fields.Char()

Atributos Comunes
#################

Así como el propio modelo, sus campos pueden ser configurados, passando
atributos de configuración como parámetros::

    name = field.Char(required=True)

Algunos atributos están disponibles para todos los campos, aquí estan los más comúnes:

:attr:`~openerp.fields.Field.string` (``unicode``, default: field's name)
    La etiqueta del campo en la IU (visible para los usuarios y las usuarias).
:attr:`~openerp.fields.Field.required` (``bool``, default: ``False``)
    Si es ``True``, el campo no puede estar vacío, puede tener un valor
    predeterminado o debe darsele un valor siempre que se cree un registro.
:attr:`~openerp.fields.Field.help` (``unicode``, default: ``''``)
    Forma larga, proporciona una herramienta de ayuda para los usuarios y las usuarias en la IU.
:attr:`~openerp.fields.Field.index` (``bool``, default: ``False``)
    Solocita que Odoo cree un `database index`_ en la columna.

Campos simples
##############

Hay dos categorías de campos generales:: campos "simple" los cuales son valores
atómicos almacenados directamente en la tabla del modelo y campos "relational" que enlazan
registros (del mismo modelo o de diferentes modelos).

Ejemplis de campos simples son :class:`~openerp.fields.Boolean`,
:class:`~openerp.fields.Date`, :class:`~openerp.fields.Char`.

Campos reservados
#################

Odoo crea unos cuantos campos en todos los modelos\ [#autofields]_. Estos campos son
gestionados por el sistema y no debe escribirse en ellos. Pueden ser leídos si es
de utilidad o necesario:

:attr:`~openerp.fields.Model.id` (:class:`~openerp.fields.Id`)
    el identificador único de un registro en su modelo
:attr:`~openerp.fields.Model.create_date` (:class:`~openerp.fields.Datetime`)
    la fecha de creación del registro
:attr:`~openerp.fields.Model.create_uid` (:class:`~openerp.fields.Many2one`)
    el usuario que crea el registro
:attr:`~openerp.fields.Model.write_date` (:class:`~openerp.fields.Datetime`)
    última fecha de modificación del registro
:attr:`~openerp.fields.Model.write_uid` (:class:`~openerp.fields.Many2one`)
    el usuario que modifica por última vez el registro

Campos especiales
#################

De forma predeterminada, Odoo tambíen requiere un campo ``name`` en todos los modelos para varios
comportamientos de visualización y bpusqueda. El campo usado para este propósito puede ser
sobre-escrito configurando :attr:`~openerp.models.Model._rec_name`.

.. exercise:: Define a model

    Deefine un nuevo modelo de datos *Course* en el módulo *openacademy*. Un curso
    tiene un titulo y una descrición. Los cursos deben tener un título.

    .. only:: solutions

        Edite el archivo ``openacademy/models.py`` para incluir una clase *Course*.

        .. patch::

Archivos de datos
-----------------

Odoo es un sistema altamente controlado por datos. Aunque su comportamiento es personalizado usando
códifo Python_  parte del valor de un módulo esta en los datos que son establecido cuendo es cargado.

.. tip:: algunos módulos existe solamente para agregar datos en Odoo
    :class: aphorism

Los datos del módulo son declarados por :ref:`data files <reference/data>`, archivos XML con
elementos ``<record>``. Cada elemento ``<record>`` crea o actualiza un registro de la base de datos.

.. code-block:: xml

    <openerp>
        <data>
            <record model="{model name}" id="{record identifier}">
                <field name="{a field name}">{a value}</field>
            </record>
        </data>
    <openerp>

* ``model`` es el nombre del modelo Odoo para el registro
* ``id`` es un :term:`external identifier`, permite referirse al registro
  (son tener que saber su identificador dentro de la base de datos)
* ``<field>`` los elementos tienen un ``name`` que es el nombre del campo en el
  modelo (ej. ``description``). Su cuerpo es el valor del campo.

Los archivos de datos tienen que ser declarados en el archivo de manifiesto para ser cargados, pueden ser
declarados en la lista ``'data'`` (siempre cargados) o en la lista ``'demo'``
(solo son cargados en modo de demostración).

.. exercise:: Define demonstration data

    Cree datos de demostración llenando el modelo *Courses* con unos pocos
    cursos de demostración.

    .. only:: solutions

        Edite el archivo ``openacademy/demo.xml`` para incluir algunos datos.

        .. patch::

Acciones y Menús
----------------

Las acciones y los menús son registros regulares en la base de datos, usualmente se declaran 
a tarvés de archivos de datos. Las acciones pueden ser desencadenadas en tres formas:

#. haciendo clic en los items del manú (enlazados a acciones específicas)
#. haciendo clic en los botónes de las vistas (si están conectados a acciones)
#. como acciones contextuales en un objeto

Debido a que los menús son de alguna manera comlejos de declarar existe un acceso directo a 
``<menuitem>`` para declarar un ``ir.ui.menu`` y conectarlo más facilmente a la acción correspondiente.

.. code-block:: xml

    <record model="ir.actions.act_window" id="action_list_ideas">
        <field name="name">Ideas</field>
        <field name="res_model">idea.idea</field>
        <field name="view_mode">tree,form</field>
    </record>
    <menuitem id="menu_ideas" parent="menu_root" name="Ideas" sequence="10"
              action="action_list_ideas"/>

.. danger::
    :class: aphorism

    La acción debe ser declarada antes que su menú correspondiente en el archivo XML.

    Los archivos de datos son ejecutados secuencialmente, el ``id`` de la acción debe estar
    presente en la base de datos antes que el menú sea creado.

.. exercise:: Define new menu entries

    Defina entradas nuevas de menú para acceder a los cursos y sesiones bajo el
    menú OpenAcademy. Un usuario o uan usuaria debería ser capaz de

    - desplegar una lista de todos los cursos
    - crear y modificar cursos

    .. only:: solutions

        #. Cree ``openacademy/views/openacademy.xml`` con una acción y los
           menús que desencadenan la acción
        #. Agregelo a la lista ``data`` en ``openacademy/__openerp__.py``

        .. patch::

Visas básicas
=============

Las vistas definen la forma como los registros del modelo son visualizados. Cada tipo de vista
representa un modo de visualización (una lista de registros, un gráfico de su colección,...).
Las vistas también pueden ser solicitadas genericamente por su tipo
(ej. *una lista de socios*) o específicamente por su id. Para solicitudes
generales, la vista con el tipo correcto y la prioridad mas baja será
usada (por lo tanto la vista con prioridad mas baja de cada tipo es la vista predeterminada
para ese tipo).

:ref:`View inheritance <reference/views/inheritance>` permite alterar las vistas
declaradas en cualquier parte (agregando o eliminando contenido).

Declaración de vista genérica
-----------------------------

Una vista es declarada como un registro del modelo ``ir.ui.view``. El tipo de vista
es sobreentendido por el elemento raíz del campo ``arch``:

.. code-block:: xml

    <record model="ir.ui.view" id="view_id">
        <field name="name">view.name</field>
        <field name="model">object_name</field>
        <field name="priority" eval="16"/>
        <field name="arch" type="xml">
            <!-- view content: <form>, <tree>, <graph>, ... -->
        </field>
    </record>

.. danger:: The view's content is XML.
    :class: aphorism

    El campo ``arch`` debe ser declarado como ``type="xml"``para ser analizado
    correctamente.

Vistas de árbol
---------------

Las vistas de árbol, también llamadas vistas de lista, despliegan los registros en forma tabular.

Su elemento raíz es ``<tree>``. La forma más simple de la vista de árbol únicamente
lista todos los campos en la tabla (cada campos como una columna):

.. code-block:: xml

    <tree string="Idea list">
        <field name="name"/>
        <field name="inventor_id"/>
    </tree>

Vistas de formulario
--------------------

Los formularios son usados para crear y editar un solo registro.


Su elementos raíz es ``<form>``. Se componen de elementos altamente
estructurados (groups, notebooks) y elementos interactivos (buttons y fields):

.. code-block:: xml

    <form string="Idea form">
        <group colspan="4">
            <group colspan="2" col="2">
                <separator string="General stuff" colspan="2"/>
                <field name="name"/>
                <field name="inventor_id"/>
            </group>

            <group colspan="2" col="2">
                <separator string="Dates" colspan="2"/>
                <field name="active"/>
                <field name="invent_date" readonly="1"/>
            </group>

            <notebook colspan="4">
                <page string="Description">
                    <field name="description" nolabel="1"/>
                </page>
            </notebook>

            <field name="state"/>
        </group>
    </form>

.. exercise:: Customise form view using XML

    Cree su propia vista de formulario para el objeto Course. Los datos desplegados deben ser:
    el nombre y la descripción del curso.

    .. only:: solutions

        .. patch::

.. exercise:: Notebooks

    En la vista de formulario de Course, ponga el campo de descripción debajo de una pestaña, asi
    será más fácil agregar luego otras pestañas, que contenga información adicional.

    .. only:: solutions

        Modifique la vista de formulario de Course como sigue:

        .. patch::

Las vistas de formulario también pueden usar HTML plano para diseños más flexibles:

.. code-block:: xml

    <form string="Idea Form">
        <header>
            <button string="Confirm" type="object" name="action_confirm"
                    states="draft" class="oe_highlight" />
            <button string="Mark as done" type="object" name="action_done"
                    states="confirmed" class="oe_highlight"/>
            <button string="Reset to draft" type="object" name="action_draft"
                    states="confirmed,done" />
            <field name="state" widget="statusbar"/>
        </header>
        <sheet>
            <div class="oe_title">
                <label for="name" class="oe_edit_only" string="Idea Name" />
                <h1><field name="name" /></h1>
            </div>
            <separator string="General" colspan="2" />
            <group colspan="2" col="2">
                <field name="description" placeholder="Idea description..." />
            </group>
        </sheet>
    </form>

Vistas de búsqueda
------------------

Las vistas de búsqueda personalizan el campo de búsqueda asociado con la vista de lista (y
otras vistas agregadas). Su elemento raíz es ``<search>`` y estan compuestas
de campos que definen cuales campos pueden ser buscados:

.. code-block:: xml

    <search>
        <field name="name"/>
        <field name="inventor_id"/>
    </search>

Si no existe una vista de búsqueda para el modelo, Odoo genera una que solo permite
búscar en el campo de ``name``.

.. exercise:: Search courses

    Permita la bpusqueda por cursos basda en su título o su descripción.

    .. only:: solutions

        .. patch::

Relaciones entre modelos
========================

Un registro de un modelo puede estar relacionado a un registro desde otro modelo. Por
lo tanto, un registro de una orden de venta esta relacionada a un registro de un cliente
que contiene los datos del cliente; también esta relacionado a sus registros de linea de 
la orden de compra.

.. exercise:: Create a session model

    Para el módulo Open Academy, consideremos un modelo para *sessions*: una sessión
    es la ocurrencia de un curso en un tiempo dada y para una audiencia determinada.

    Cree un modelo para *sessions*. Una sesión tiene un nombre, una fecha de inicio, una
    duración y un número de asientos. Agregue una acción y un item de menú para desplegarlos.
    Haga que el modelo nuevo sea visible a través de un item de menú.

    .. only:: solutions

        #. Cree la clase *Session* in ``openacademy/models.py``.
        #. Agregue el acceso al objeto de sesión en ``openacademy/view/openacademy.xml``.

        .. patch::

        .. note:: ``digits=(6, 2)`` especifica la precisión de un número decimal:
                  6 es el número total de digitos, mientras que 2 es el número de 
                  digitos despues de la coma. Note que el número de dígitos antes 
                  de la coma es de un máximo de 4

Campos relacionados
-------------------

Los campos relacionados enlazan registros, del mismo modelo (jerarquías) o
entre modelos diferentes.

Los tipos de campos relacionales son:

:class:`Many2one(other_model, ondelete='set null') <openerp.fields.Many2one>`
    Un enlace simple a otro objeto::

        print foo.other_id.name

    .. seealso:: `foreign keys <http://www.postgresql.org/docs/9.3/static/tutorial-fk.html>`_

:class:`One2many(other_model, related_field) <openerp.fields.One2many>`
    Una relación virtual, inversa de una :class:`~openerp.fields.Many2one`.
    Una :class:`~openerp.fields.One2many` se comporta como un contenedos de registros,
    el acceso a ella resulta en un conjunto de registros (posiblemente vacío) ::

        for other in foo.other_ids:
            print other.name

    .. danger::

        Debido a que una :class:`~openerp.fields.One2many` es una relación virtual,
        *Debe* haber un campo :class:`~openerp.fields.Many2one` en el
        :samp:`{other_model}`, y su nombre *debe* ser :samp:`{related_field}`

:class:`Many2many(other_model) <openerp.fields.Many2many>`
    Relación multiple bidireccional, cualquier registro en un lado puede estar relacionado
    a cualquier cantidad de registros en el otro lado. Se comporta como un contendor de 
    registros, el acceso a ella resulta en un conjunto de registros posiblemente vacío::

        for other in foo.other_ids:
            print other.name

.. exercise:: Many2one relations

    Use many2one, para modificar los modelos *Course* y *Session*  y reflejar su
    relación con otros modelos:

    - Un curso tiene un usuario *responsable*; el valor de ese campo es un registro
      del modelo pre-construido ``res.users``.
    - Una sessión tiene un *instructor*; el valor de ese campo es un registro del
      modelo pre-construido ``res.partner``.
    - Una sesión esta relacionada a un *course*; el valor de ese campo es un registro
      del modelo ``openacademy.course`` y es obligatorio.
    - Adapte las vistas.

    .. only:: solutions

        #. Agregue los campos ``Many2one`` relevantes a los modelos, y
        #. agreguelos a las vistas.

        .. patch::

.. exercise:: Inverse one2many relations

    Use el campo relacional inverso one2many, para modificar los modelos y reflejar
    la relación entre los cursos y las sesiones.

    .. only:: solutions

        #. Modifique la clase ``Course``, y
        #. y agregue el campo en la vista de formulario del curso

        .. patch::

.. exercise:: Multiple many2many relations

    Use el campo relacional many2many, para modificar el modelo *Session* y relacionar
    cada sesión a un conjunto de *attendees*. Los asistentes seran representados por los 
    registros de socio, asi que haremos una relación con el modelo pre-construido ``res.partner``.
    Adapte las vistas.

    .. only:: solutions

        #. Modifique la clase ``Session``, y
        #. agregue el campo en la vista de formulario.

        .. patch::

Herencia
===========

Herencia del modelo
-------------------

Odoo provee dos mecanismos de *herencia* para ampliar un modelo existente de
forma modular.

El primer mecanismo de herencia permite a un módulo modificar el comportamiento
de un modelo definido en otro módulo:

- agrega campos a un modelo,
- sobre-escribe la definición de los campos en un modelo,
- agrega una restricción a un modelo,
- agrega métodos a un modelo,
- sobre-escribe los métodos existentes en un modelo.

El segundo mecanismo de herencia (delegación) permite enlazar cada registro
de un modelo a un registro en el modelo padre, y proporciona acceso transparente
a los campos del registro padre.

.. image:: ../images/inheritance_methods.png
    :align: center

.. seealso::

    * :attr:`~openerp.models.Model._inherit`
    * :attr:`~openerp.models.Model._inherits`

Herencia de vistas
------------------

En vez de modificar las vistas existentes en el mismo lugar (sobre-escribiendolas), Odoo
provee herencia de vista donde las "extensiónes" de vistas hijas son aplicadas en
las vistas raíz, y pueden agregar o eliminar contenido desde su padre.

Una vista de extensión hace referencia a su padre usando el campo ``inherit_id``, y
en vez de una sola vista su campo ``arch`` es compuesto de cualquier cantidad de
elementos ``xpath`` seleccionando y alterando el contenido de su vista padre:

.. code-block:: xml

    <!-- improved idea categories list -->
    <record id="idea_category_list2" model="ir.ui.view">
        <field name="name">id.category.list2</field>
        <field name="model">idea.category</field>
        <field name="inherit_id" ref="id_category_list"/>
        <field name="arch" type="xml">
            <!-- find field description and add the field
                 idea_ids after it -->
            <xpath expr="//field[@name='description']" position="after">
              <field name="idea_ids" string="Number of ideas"/>
            </xpath>
        </field>
    </record>

``expr``
    Una expresión XPath_ que selecciona un único elemento en la vista padre.
    Arroja un error si no encuentra coincidencia con algún elemento o coincide con mas de uno
``position``
    Operación que es aplicada al elemento coincidente:

    ``inside``
        añade el cuerpo del ``xpath`` al final del elemento coincidente
    ``replace``
        reemplaza el elemento coincidente por el cuerpo del ``xpath``
    ``before``
        inserta el cuerpo del ``xpath`` como hermano despues del elemento coincidente
    ``after``
        inserta el cuerpo del ``xpaths`` como un hermano despues del elemento coincidente
    ``attributes``
        modifica los atributos del elemento coincidente usando los elementos espaciales
        ``attribute`` en el cuerpo del ``xpath``

.. tip::

    Cuando se hace coincidir con un unico elemento, el atributo ``position`` puede ser
    fijado directamente al elemento buscado. Ambas herencias produciran el mismo 
    resultado.

    .. code-block:: xml

        <xpath expr="//field[@name='description']" position="after">
            <field name="idea_ids" />
        </xpath>

        <field name="description" position="after">
            <field name="idea_ids" />
        </field>


.. exercise:: Alterar contenido existente

    * Use la herencia por el modelo, y modifique el modelo *Partner* existente para agregar
      un campo booleano ``instructor``, y un campo many2many que corresponda a la relación
      session-partner
    * Use la herecia por vista, y muestre estos campos en la vista formulario de partner

    .. only:: solutions

       .. note::

           Esta es la oportunidad de introducir el modo de desarrollo para inspeccionar
           la vista, encuentre el ID externo y el lugar para colocar el nuevo campo.

       #. Create a file ``openacademy/partner.py`` and import it in
          ``__init__.py``
       #. Create a file ``openacademy/views/partner.xml`` and add it to
          ``__openerp__.py``

       .. patch::

Dominios
########

En Odoo, :ref:`reference/orm/domains` son valores que codifican condiciones en
lso registros. Un dominio es una lista de criterios usada para seleccionar un 
subconjunto de registros de un modelo. Cada criterio es un triple con un nombre 
de campo, un operador y un valor.

Por ejemplo, cuando se uso en el modelo *Product* los siguientes dominios seleccionan
todos los *services* con un precio unitario sobre los *1000*::

    [('product_type', '=', 'service'), ('unit_price', '>', 1000)]

De forma predeterminanda el criterio es combinado con un AND implícito. Los operadores
lógicos ``&`` (AND), ``|`` (OR) y ``!`` (NOT) pueden usarse para combinar el criterio
explicitamente. Son usados como prefijo (el operador es insertado antes de los argumentos) 
Por ejemplo, para seleccionar los productos "que son services *O* tiene un precio unitario
el cual *NO* este entre 1000 y 2000"::

    ['|',
        ('product_type', '=', 'service'),
        '!', '&',
            ('unit_price', '>=', 1000),
            ('unit_price', '<', 2000)]

Un parametro ``domain`` puede agregarse a los campos relacionales para limitar los registros
válidos para la relación cuando se esta tratando de seleccionar registros en la interfaz del 
cliente.

.. exercise:: Domains on relational fields

    Cuando se selecciona el instructor para un *Session*, solo los instructores (socios con
    ``instructor`` fijado a ``True``) deben estar visibles.

    .. only:: solutions

        .. patch::

        .. note::

            Un dominio declarado como una lista lliteral es evaluado del lado del servidor y
            no puede puede hacer referencia a valores dinámicos del lado derecho, un dominio
            declarado como cadena es evaluado del lado del cliente y permite nombres de campo
            en el lado derecho.

.. exercise:: More complex domains

    Cree unas categorías nuevas de partnet *Teacher / Level 1* y *Teacher / Level 2*.
    El isntructor para una sessión puede ser un instructor o un profesor (de 
    cualquier nivel).

    .. only:: solutions

        #. Modifique el dominio del modelo *Session*
        #. Modifique el archivo ``openacademy/view/partner.xml`` para tener acceso a 
           *Partner categories*:

        .. patch::

Computed fields and default values
==================================

So far fields have been stored directly in and retrieved directly from the
database. Fields can also be *computed*. In that case, the field's value is not
retrieved from the database but computed on-the-fly by calling a method of the
model.

To create a computed field, create a field and set its attribute
:attr:`~openerp.fields.Field.compute` to the name of a method. The computation
method should simply set the value of the field to compute on every record in
``self``.

.. danger:: ``self`` is a collection
    :class: aphorism

    The object ``self`` is a *recordset*, i.e., an ordered collection of
    records. It supports the standard Python operations on collections, like
    ``len(self)`` and ``iter(self)``, plus extra set operations like ``recs1 +
    recs2``.

    Iterating over ``self`` gives the records one by one, where each record is
    itself a collection of size 1. You can access/assign fields on single
    records by using the dot notation, like ``record.name``.

.. code-block:: python

    import random
    from openerp import models, fields

    class ComputedModel(models.Model):
        _name = 'test.computed'

        name = fields.Char(compute='_compute_name')

        @api.multi
        def _compute_name(self):
            for record in self:
                record.name = str(random.randint(1, 1e6))

Our compute method is very simple: it loops over ``self`` and performs the same
operation on every record. We can make it slightly simpler by using the
decorator :func:`~openerp.api.one` to automatically loop on the collection::

        @api.one
        def _compute_name(self):
            self.name = str(random.randint(1, 1e6))

Dependencies
------------

The value of a computed field usually depends on the values of other fields on
the computed record. The ORM expects the developer to specify those dependencies
on the compute method with the decorator :func:`~openerp.api.depends`.
The given dependencies are used by the ORM to trigger the recomputation of the
field whenever some of its dependencies have been modified::

    from openerp import models, fields, api

    class ComputedModel(models.Model):
        _name = 'test.computed'

        name = fields.Char(compute='_compute_name')
        value = fields.Integer()

        @api.one
        @api.depends('value')
        def _compute_name(self):
            self.name = "Record with value %s" % self.value

.. exercise:: Computed fields

    * Add the percentage of taken seats to the *Session* model
    * Display that field in the tree and form views
    * Display the field as a progress bar

    .. only:: solutions

        #. Add a computed field to *Session*
        #. Show the field in the *Session* view:

        .. patch::

Default values
--------------

Any field can be given a default value. In the field definition, add the option
``default=X`` where ``X`` is either a Python literal value (boolean, integer,
float, string), or a function taking a recordset and returning a value::

    name = fields.Char(default="Unknown")
    user_id = fields.Many2one('res.users', default=lambda self: self.env.user)

.. note::

    The object ``self.env`` gives access to request parameters and other useful
    things:

    - ``self.env.cr`` or ``self._cr`` is the database *cursor* object; it is
      used for querying the database
    - ``self.env.uid`` or ``self._uid`` is the current user's database id
    - ``self.env.user`` is the current user's record
    - ``self.env.context`` or ``self._context`` is the context dictionary
    - ``self.env.ref(xml_id)`` returns the record corresponding to an XML id
    - ``self.env[model_name]`` returns an instance of the given model

.. exercise:: Active objects – Default values

    * Define the start_date default value as today (see
      :class:`~openerp.fields.Date`).
    * Add a field ``active`` in the class Session, and set sessions as active by
      default.

    .. only:: solutions

        .. patch::

        .. note::

            Odoo has built-in rules making fields with an ``active`` field set
            to ``False`` invisible.

Onchange
========

The "onchange" mechanism provides a way for the client interface to update a
form whenever the user has filled in a value in a field, without saving anything
to the database.

For instance, suppose a model has three fields ``amount``, ``unit_price`` and
``price``, and you want to update the price on the form when any of the other
fields is modified. To achieve this, define a method where ``self`` represents
the record in the form view, and decorate it with :func:`~openerp.api.onchange`
to specify on which field it has to be triggered. Any change you make on
``self`` will be reflected on the form.

.. code-block:: xml

    <!-- content of form view -->
    <field name="amount"/>
    <field name="unit_price"/>
    <field name="price" readonly="1"/>

.. code-block:: python

    # onchange handler
    @api.onchange('amount', 'unit_price')
    def _onchange_price(self):
        # set auto-changing field
        self.price = self.amount * self.unit_price
        # Can optionally return a warning and domains
        return {
            'warning': {
                'title': "Something bad happened",
                'message': "It was very bad indeed",
            }
        }

For computed fields, valued ``onchange`` behavior is built-in as can be seen by
playing with the *Session* form: change the number of seats or participants, and
the ``taken_seats`` progressbar is automatically updated.

.. exercise:: Warning

    Add an explicit onchange to warn about invalid values, like a negative
    number of seats, or more participants than seats.

    .. only:: solutions

        .. patch::

Model constraints
=================

Odoo provides two ways to set up automatically verified invariants:
:func:`Python constraints <openerp.api.constrains>` and
:attr:`SQL constraints <openerp.models.Model._sql_constraints>`.

A Python constraint is defined as a method decorated with
:func:`~openerp.api.constrains`, and invoked on a recordset. The decorator
specifies which fields are involved in the constraint, so that the constraint is
automatically evaluated when one of them is modified. The method is expected to
raise an exception if its invariant is not satisfied::

    from openerp.exceptions import ValidationError

    @api.constrains('age')
    def _check_something(self):
        for record in self:
            if record.age > 20:
                raise ValidationError("Your record is too old: %s" % record.age)
        # all records passed the test, don't return anything

.. exercise:: Add Python constraints

    Add a constraint that checks that the instructor is not present in the
    attendees of his/her own session.

    .. only:: solutions

        .. patch::

SQL constraints are defined through the model attribute
:attr:`~openerp.models.Model._sql_constraints`. The latter is assigned to a list
of triples of strings ``(name, sql_definition, message)``, where ``name`` is a
valid SQL constraint name, ``sql_definition`` is a table_constraint_ expression,
and ``message`` is the error message.

.. exercise:: Add SQL constraints

    With the help of `PostgreSQL's documentation`_ , add the following
    constraints:

    #. CHECK that the course description and the course title are different
    #. Make the Course's name UNIQUE

    .. only:: solutions

        .. patch::

.. exercise:: Exercise 6 - Add a duplicate option

    Since we added a constraint for the Course name uniqueness, it is not
    possible to use the "duplicate" function anymore (:menuselection:`Form -->
    Duplicate`).

    Re-implement your own "copy" method which allows to duplicate the Course
    object, changing the original name into "Copy of [original name]".

    .. only:: solutions

        .. patch::

Advanced Views
==============

Tree views
----------

Tree views can take supplementary attributes to further customize their
behavior:

``colors``
    mappings of colors to conditions. If the condition evaluates to ``True``,
    the corresponding color is applied to the row:

    .. code-block:: xml

        <tree string="Idea Categories" colors="blue:state=='draft';red:state=='trashed'">
            <field name="name"/>
            <field name="state"/>
        </tree>

    Clauses are separated by ``;``, the color and condition are separated by
    ``:``.

``editable``
    Either ``"top"`` or ``"bottom"``. Makes the tree view editable in-place
    (rather than having to go through the form view), the value is the
    position where new rows appear.

.. exercise:: List coloring

    Modify the Session tree view in such a way that sessions lasting less than
    5 days are colored blue, and the ones lasting more than 15 days are
    colored red.

    .. only:: solutions

        Modify the session tree view:

        .. patch::

Calendars
---------

Displays records as calendar events. Their root element is ``<calendar>`` and
their most common attributes are:

``color``
    The name of the field used for *color segmentation*. Colors are
    automatically distributed to events, but events in the same color segment
    (records which have the same value for their ``@color`` field) will be
    given the same color.
``date_start``
    record's field holding the start date/time for the event
``date_stop`` (optional)
    record's field holding the end date/time for the event

field (to define the label for each calendar event)

.. code-block:: xml

    <calendar string="Ideas" date_start="invent_date" color="inventor_id">
        <field name="name"/>
    </calendar>

.. exercise:: Calendar view

    Add a Calendar view to the *Session* model enabling the user to view the
    events associated to the Open Academy.

    .. only:: solutions

        #. Add an ``end_date`` field computed from ``start_date`` and
           ``duration``

           .. tip:: the inverse function makes the field writable, and allows
                    moving the sessions (via drag and drop) in the calendar view

        #. Add a calendar view to the *Session* model
        #. And add the calendar view to the *Session* model's actions

        .. patch::

Search views
------------

Search view ``<field>`` elements can have a ``@filter_domain`` that overrides
the domain generated for searching on the given field. In the given domain,
``self`` represents the value entered by the user. In the example below, it is
used to search on both fields ``name`` and ``description``.

Search views can also contain ``<filter>`` elements, which act as toggles for
predefined searches. Filters must have one of the following attributes:

``domain``
    add the given domain to the current search
``context``
    add some context to the current search; use the key ``group_by`` to group
    results on the given field name

.. code-block:: xml

    <search string="Ideas">
        <field name="name"/>
        <field name="description" string="Name and description"
               filter_domain="['|', ('name', 'ilike', self), ('description', 'ilike', self)]"/>
        <field name="inventor_id"/>
        <field name="country_id" widget="selection"/>

        <filter name="my_ideas" string="My Ideas"
                domain="[('inventor_id', '=', uid)]"/>
        <group string="Group By">
            <filter name="group_by_inventor" string="Inventor"
                    context="{'group_by': 'inventor_id'}"/>
        </group>
    </search>

To use a non-default search view in an action, it should be linked using the
``search_view_id`` field of the action record.

The action can also set default values for search fields through its
``context`` field: context keys of the form
:samp:`search_default_{field_name}` will initialize *field_name* with the
provided value. Search filters must have an optional ``@name`` to have a
default and behave as booleans (they can only be enabled by default).

.. exercise:: Search views

    #. Add a button to filter the courses for which the current user is the
       responsible in the course search view. Make it selected by default.
    #. Add a button to group courses by responsible user.

    .. only:: solutions

        .. patch::

Gantt
-----

Horizontal bar charts typically used to show project planning and advancement,
their root element is ``<gantt>``.

.. code-block:: xml

    <gantt string="Ideas" date_start="invent_date" color="inventor_id">
        <level object="idea.idea" link="id" domain="[]">
            <field name="inventor_id"/>
        </level>
    </gantt>

.. exercise:: Gantt charts

    Add a Gantt Chart enabling the user to view the sessions scheduling linked
    to the Open Academy module. The sessions should be grouped by instructor.

    .. only:: solutions

        #. Create a computed field expressing the session's duration in hours
        #. Add the gantt view's definition, and add the gantt view to the
           *Session* model's action

        .. patch::

Graph views
-----------

Graph views allow aggregated overview and analysis of models, their root
element is ``<graph>``.

Graph views have 4 display modes, the default mode is selected using the
``@type`` attribute.

Pivot
    a multidimensional table, allows the selection of filers and dimensions
    to get the right aggregated dataset before moving to a more graphical
    overview
Bar (default)
    a bar chart, the first dimension is used to define groups on the
    horizontal axis, other dimensions define aggregated bars within each group.

    By default bars are side-by-side, they can be stacked by using
    ``@stacked="True"`` on the ``<graph>``
Line
    2-dimensional line chart
Pie
    2-dimensional pie

Graph views contain ``<field>`` with a mandatory ``@type`` attribute taking
the values:

``row`` (default)
    the field should be aggregated by default
``measure``
    the field should be aggregated rather than grouped on

.. code-block:: xml

    <graph string="Total idea score by Inventor">
        <field name="inventor_id"/>
        <field name="score" type="measure"/>
    </graph>

.. warning::

    Graph views perform aggregations on database values, they do not work
    with non-stored computed fields.

.. exercise:: Graph view

    Add a Graph view in the Session object that displays, for each course, the
    number of attendees under the form of a bar chart.

    .. only:: solutions

        #. Add the number of attendees as a stored computed field
        #. Then add the relevant view

        .. patch::

Kanban
------

Used to organize tasks, production processes, etc… their root element is
``<kanban>``.

A kanban view shows a set of cards possibly grouped in columns. Each card
represents a record, and each column the values of an aggregation field.

For instance, project tasks may be organized by stage (each column is a
stage), or by responsible (each column is a user), and so on.

Kanban views define the structure of each card as a mix of form elements
(including basic HTML) and :ref:`reference/qweb`.

.. exercise:: Kanban view

    Add a Kanban view that displays sessions grouped by course (columns are
    thus courses).

    .. only:: solutions

        #. Add an integer ``color`` field to the *Session* model
        #. Add the kanban view and update the action

        .. patch::

Workflows
=========

Workflows are models associated to business objects describing their dynamics.
Workflows are also used to track processes that evolve over time.

.. exercise:: Almost a workflow

    Add a ``state`` field to the *Session* model. It will be used to define
    a workflow-ish.

    A sesion can have three possible states: Draft (default), Confirmed and
    Done.

    In the session form, add a (read-only) field to
    visualize the state, and buttons to change it. The valid transitions are:

    * Draft -> Confirmed
    * Confirmed -> Draft
    * Confirmed -> Done
    * Done -> Draft

    .. only:: solutions

        #. Add a new ``state`` field
        #. Add state-transitioning methods, those can be called from view
           buttons to change the record's state
        #. And add the relevant buttons to the session's form view

        .. patch::

Workflows may be associated with any object in Odoo, and are entirely
customizable. Workflows are used to structure and manage the lifecycles of
business objects and documents, and define transitions, triggers, etc. with
graphical tools. Workflows, activities (nodes or actions) and transitions
(conditions) are declared as XML records, as usual. The tokens that navigate
in workflows are called workitems.

.. warning::

    A workflow associated with a model is only created when the
    model's records are created. Thus there is no workflow instance
    associated with session instances created before the workflow's
    definition

.. exercise:: Workflow

    Replace the ad-hoc *Session* workflow by a real workflow. Transform the
    *Session* form view so its buttons call the workflow instead of the
    model's methods.

    .. only:: solutions

        .. patch::

        .. tip::

            In order to check if instances of the workflow are correctly
            created alongside sessions, go to :menuselection:`Settings -->
            Technical --> Workflows --> Instances`



.. exercise:: Automatic transitions

    Automatically transition sessions from *Draft* to *Confirmed* when more
    than half the session's seats are reserved.

    .. only:: solutions

        .. patch::

.. exercise:: Server actions

    Replace the Python methods for synchronizing session state by
    server actions.

    Both the workflow and the server actions could have been created entirely
    from the UI.

    .. only:: solutions

        .. patch::

Security
========

Access control mechanisms must be configured to achieve a coherent security
policy.

Group-based access control mechanisms
-------------------------------------

Groups are created as normal records on the model ``res.groups``, and granted
menu access via menu definitions. However even without a menu, objects may
still be accessible indirectly, so actual object-level permissions (read,
write, create, unlink) must be defined for groups. They are usually inserted
via CSV files inside modules. It is also possible to restrict access to
specific fields on a view or object using the field's groups attribute.

Access rights
-------------

Access rights are defined as records of the model ``ir.model.access``. Each
access right is associated to a model, a group (or no group for global
access), and a set of permissions: read, write, create, unlink. Such access
rights are usually created by a CSV file named after its model:
``ir.model.access.csv``.

.. code-block:: text

    id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
    access_idea_idea,idea.idea,model_idea_idea,base.group_user,1,1,1,0
    access_idea_vote,idea.vote,model_idea_vote,base.group_user,1,1,1,0

.. exercise:: Add access control through the OpenERP interface

    Create a new user "John Smith". Then create a group
    "OpenAcademy / Session Read" with read access to the *Session* model.

    .. only:: solutions

        #. Create a new user *John Smith* through
           :menuselection:`Settings --> Users --> Users`
        #. Create a new group ``session_read`` through
           :menuselection:`Settings --> Users --> Groups`, it should have
           read access on the *Session* model
        #. Edit *John Smith* to make them a member of ``session_read``
        #. Log in as *John Smith* to check the access rights are correct

.. exercise:: Add access control through data files in your module

    Using data files,

    * Create a group *OpenAcademy / Manager* with full access to all
      OpenAcademy models
    * Make *Session* and *Course* readable by all users

    .. only:: solutions

        #. Create a new file ``openacademy/security/security.xml`` to
           hold the OpenAcademy Manager group
        #. Edit the file ``openacademy/security/ir.model.access.csv`` with
           the access rights to the models
        #. Finally update ``openacademy/__openerp__.py`` to add the new data
           files to it

        .. patch::

Record rules
------------

A record rule restricts the access rights to a subset of records of the given
model. A rule is a record of the model ``ir.rule``, and is associated to a
model, a number of groups (many2many field), permissions to which the
restriction applies, and a domain. The domain specifies to which records the
access rights are limited.

Here is an example of a rule that prevents the deletion of leads that are not
in state ``cancel``. Notice that the value of the field ``groups`` must follow
the same convention as the method :meth:`~openerp.models.Model.write` of the ORM.

.. code-block:: xml

    <record id="delete_cancelled_only" model="ir.rule">
        <field name="name">Only cancelled leads may be deleted</field>
        <field name="model_id" ref="crm.model_crm_lead"/>
        <field name="groups" eval="[(4, ref('base.group_sale_manager'))]"/>
        <field name="perm_read" eval="0"/>
        <field name="perm_write" eval="0"/>
        <field name="perm_create" eval="0"/>
        <field name="perm_unlink" eval="1" />
        <field name="domain_force">[('state','=','cancel')]</field>
    </record>

.. exercise:: Record rule

    Add a record rule for the model Course and the group
    "OpenAcademy / Manager", that restricts ``write`` and ``unlink`` accesses
    to the responsible of a course. If a course has no responsible, all users
    of the group must be able to modify it.

    .. only:: solutions

        Create a new rule in ``openacademy/security/security.xml``:

        .. patch::

Wizards
=======

Wizards describe interactive sessions with the user (or dialog boxes) through
dynamic forms. A wizard is simply a model that extends the class
:class:`~openerp.models.TransientModel` instead of
:class:`~openerp.models.Model`. The class
:class:`~openerp.models.TransientModel` extends :class:`~openerp.models.Model`
and reuse all its existing mechanisms, with the following particularities:

- Wizard records are not meant to be persistent; they are automatically deleted
  from the database after a certain time. This is why they are called
  *transient*.
- Wizard models do not require explicit access rights: users have all
  permissions on wizard records.
- Wizard records may refer to regular records or wizard records through many2one
  fields, but regular records *cannot* refer to wizard records through a
  many2one field.

We want to create a wizard that allow users to create attendees for a particular
session, or for a list of sessions at once.

.. exercise:: Define the wizard

    Create a wizard model with a many2one relationship with the *Session*
    model and a many2many relationship with the *Partner* model.

    .. only:: solutions

        Add a new file ``openacademy/wizard.py``:

        .. patch::

Launching wizards
-----------------

Wizards are launched by ``ir.actions.act_window`` records, with the field
``target`` set to the value ``new``. The latter opens the wizard view into a
popup window. The action may be triggered by a menu item.

There is another way to launch the wizard: using an ``ir.actions.act_window``
record like above, but with an extra field ``src_model`` that specifies in the
context of which model the action is available. The wizard will appear in the
contextual actions of the model, above the main view. Because of some internal
hooks in the ORM, such an action is declared in XML with the tag ``act_window``.

.. code:: xml

    <act_window id="launch_the_wizard"
                name="Launch the Wizard"
                src_model="context_model_name"
                res_model="wizard_model_name"
                view_mode="form"
                target="new"
                key2="client_action_multi"/>

Wizards use regular views and their buttons may use the attribute
``special="cancel"`` to close the wizard window without saving.

.. exercise:: Launch the wizard

    #. Define a form view for the wizard.
    #. Add the action to launch it in the context of the *Session* model.
    #. Define a default value for the session field in the wizard; use the
       context parameter ``self._context`` to retrieve the current session.

    .. only:: solutions

        .. patch::

.. exercise:: Register attendees

    Add buttons to the wizard, and implement the corresponding method for adding
    the attendees to the given session.

    .. only:: solutions

        .. patch::

.. exercise:: Register attendees to multiple sessions

    Modify the wizard model so that attendees can be registered to multiple
    sessions.

    .. only:: solutions

        .. patch::

Internationalization
====================

Each module can provide its own translations within the i18n directory, by
having files named LANG.po where LANG is the locale code for the language, or
the language and country combination when they differ (e.g. pt.po or
pt_BR.po). Translations will be loaded automatically by Odoo for all
enabled languages. Developers always use English when creating a module, then
export the module terms using Odoo's gettext POT export feature
(:menuselection:`Settings --> Translations --> Import/Export --> Export
Translation` without specifying a language), to create the module template POT
file, and then derive the translated PO files. Many IDE's have plugins or modes
for editing and merging PO/POT files.

.. tip:: The GNU gettext format (Portable Object) used by Odoo is
         integrated into LaunchPad, making it an online collaborative
         translation platform.

.. code-block:: text

   |- idea/ # The module directory
      |- i18n/ # Translation files
         | - idea.pot # Translation Template (exported from Odoo)
         | - fr.po # French translation
         | - pt_BR.po # Brazilian Portuguese translation
         | (...)

.. tip:: 

   By default Odoo's POT export only extracts labels inside XML files or
   inside field definitions in Python code, but any Python string can be
   translated this way by surrounding it with the function :func:`openerp._`
   (e.g. ``_("Label")``)

.. exercise:: Translate a module

   Choose a second language for your Odoo installation. Translate your
   module using the facilities provided by Odoo.

   .. only:: solutions

        #. Create a directory ``openacademy/i18n/``
        #. Install whichever language you want (
           :menuselection:`Administration --> Translations --> Load an
           Official Translation`)
        #. Synchronize translatable terms (:menuselection:`Administration -->
           Translations --> Application Terms --> Synchronize Translations`)
        #. Create a template translation file by exporting (
           :menuselection:`Administration --> Translations -> Import/Export
           --> Export Translation`) without specifying a language, save in
           ``openacademy/i18n/``
        #. Create a translation file by exporting (
           :menuselection:`Administration --> Translations --> Import/Export
           --> Export Translation`) and specifying a language. Save it in
           ``openacademy/i18n/``
        #. Open the exported translation file (with a basic text editor or a
           dedicated PO-file editor e.g. POEdit_ and translate the missing
           terms

        #. In ``models.py``, add an import statement for the function
           ``openerp._`` and mark missing strings as translatable

        #. Repeat steps 3-6

        .. patch::

        .. todo:: do we never reload translations?


Reporting
=========

Printed reports
---------------

Odoo 8.0 comes with a new report engine based on :ref:`reference/qweb`,
`Twitter Bootstrap`_ and Wkhtmltopdf_. 

A report is a combination two elements:

* an ``ir.actions.report.xml``, for which a ``<report>`` shortcut element is
  provided, it sets up various basic parameters for the report (default
  type, whether the report should be saved to the database after generation,…)


  .. code-block:: xml

      <report
          id="account_invoices"
          model="account.invoice"
          string="Invoices"
          report_type="qweb-pdf"
          name="account.report_invoice"
          file="account.report_invoice"
          attachment_use="True"
          attachment="(object.state in ('open','paid')) and
              ('INV'+(object.number or '').replace('/','')+'.pdf')"
      />

* A standard :ref:`QWeb view <reference/views/qweb>` for the actual report:

  .. code-block:: xml

    <t t-call="report.html_container">
        <t t-foreach="docs" t-as="o">
            <t t-call="report.external_layout">
                <div class="page">
                    <h2>Report title</h2>
                </div>
            </t>
        </t>
    </t>

    the standard rendering context provides a number of elements, the most
    important being:

    ``docs``
        the records for which the report is printed
    ``user``
        the user printing the report

Because reports are standard web pages, they are available through a URL and
output parameters can be manipulated through this URL, for instance the HTML
version of the *Invoice* report is available through
http://localhost:8069/report/html/account.report_invoice/1 (if ``account`` is
installed) and the PDF version through
http://localhost:8069/report/pdf/account.report_invoice/1.

.. exercise:: Create a report for the Session model

   For each session, it should display session's name, its start and end,
   and list the session's attendees.

   .. only:: solutions

        .. patch::

Dashboards
----------

.. exercise:: Define a Dashboard

   Define a dashboard containing the graph view you created, the sessions
   calendar view and a list view of the courses (switchable to a form
   view). This dashboard should be available through a menuitem in the menu,
   and automatically displayed in the web client when the OpenAcademy main
   menu is selected.

   .. only:: solutions

        #. Create a file ``openacademy/views/session_board.xml``. It should contain
           the board view, the actions referenced in that view, an action to
           open the dashboard and a re-definition of the main menu item to add
           the dashboard action

           .. note:: Available dashboard styles are ``1``, ``1-1``, ``1-2``,
                     ``2-1`` and ``1-1-1``

        #. Update ``openacademy/__openerp__.py`` to reference the new data
           file

        .. patch::

WebServices
===========

The web-service module offer a common interface for all web-services :

- XML-RPC
- JSON-RPC

Business objects can also be accessed via the distributed object
mechanism. They can all be modified via the client interface with contextual
views.

Odoo is accessible through XML-RPC/JSON-RPC interfaces, for which libraries
exist in many languages.

XML-RPC Library
---------------

The following example is a Python program that interacts with an Odoo
server with the library ``xmlrpclib``::

   import xmlrpclib

   root = 'http://%s:%d/xmlrpc/' % (HOST, PORT)

   uid = xmlrpclib.ServerProxy(root + 'common').login(DB, USER, PASS)
   print "Logged in as %s (uid: %d)" % (USER, uid)

   # Create a new note
   sock = xmlrpclib.ServerProxy(root + 'object')
   args = {
       'color' : 8,
       'memo' : 'This is a note',
       'create_uid': uid,
   }
   note_id = sock.execute(DB, uid, PASS, 'note.note', 'create', args)

.. exercise:: Add a new service to the client

   Write a Python program able to send XML-RPC requests to a PC running
   Odoo (yours, or your instructor's). This program should display all
   the sessions, and their corresponding number of seats. It should also
   create a new session for one of the courses.

   .. only:: solutions

        .. code-block:: python

            import functools
            import xmlrpclib
            HOST = 'localhost'
            PORT = 8069
            DB = 'openacademy'
            USER = 'admin'
            PASS = 'admin'
            ROOT = 'http://%s:%d/xmlrpc/' % (HOST,PORT)

            # 1. Login
            uid = xmlrpclib.ServerProxy(ROOT + 'common').login(DB,USER,PASS)
            print "Logged in as %s (uid:%d)" % (USER,uid)

            call = functools.partial(
                xmlrpclib.ServerProxy(ROOT + 'object').execute,
                DB, uid, PASS)

            # 2. Read the sessions
            sessions = call('openacademy.session','search_read', [], ['name','seats'])
            for session in sessions:
                print "Session %s (%s seats)" % (session['name'], session['seats'])
            # 3.create a new session
            session_id = call('openacademy.session', 'create', {
                'name' : 'My session',
                'course_id' : 2,
            })

        Instead of using a hard-coded course id, the code can look up a course
        by name::

            # 3.create a new session for the "Functional" course
            course_id = call('openacademy.course', 'search', [('name','ilike','Functional')])[0]
            session_id = call('openacademy.session', 'create', {
                'name' : 'My session',
                'course_id' : course_id,
            })

JSON-RPC Library
----------------

The following example is a Python program that interacts with an Odoo server
with the standard Python libraries ``urllib2`` and ``json``::

    import json
    import random
    import urllib2

    def json_rpc(url, method, params):
        data = {
            "jsonrpc": "2.0",
            "method": method,
            "params": params,
            "id": random.randint(0, 1000000000),
        }
        req = urllib2.Request(url=url, data=json.dumps(data), headers={
            "Content-Type":"application/json",
        })
        reply = json.load(urllib2.urlopen(req))
        if reply.get("error"):
            raise Exception(reply["error"])
        return reply["result"]

    def call(url, service, method, *args):
        return json_rpc(url, "call", {"service": service, "method": method, "args": args})

    # log in the given database
    url = "http://%s:%s/jsonrpc" % (HOST, PORT)
    uid = call(url, "common", "login", DB, USER, PASS)

    # create a new note
    args = {
        'color' : 8,
        'memo' : 'This is another note',
        'create_uid': uid,
    }
    note_id = call(url, "object", "execute", DB, uid, PASS, 'note.note', 'create', args)

Here is the same program, using the library
`jsonrpclib <https://pypi.python.org/pypi/jsonrpclib>`_::

    import jsonrpclib

    # server proxy object
    url = "http://%s:%s/jsonrpc" % (HOST, PORT)
    server = jsonrpclib.Server(url)

    # log in the given database
    uid = server.call(service="common", method="login", args=[DB, USER, PASS])

    # helper function for invoking model methods
    def invoke(model, method, *args):
        args = [DB, uid, PASS, model, method] + list(args)
        return server.call(service="object", method="execute", args=args)

    # create a new note
    args = {
        'color' : 8,
        'memo' : 'This is another note',
        'create_uid': uid,
    }
    note_id = invoke('note.note', 'create', args)

Examples can be easily adapted from XML-RPC to JSON-RPC.

.. note::

    There are a number of high-level APIs in various languages to access Odoo
    systems without *explicitly* going through XML-RPC or JSON-RPC, such as:

    * https://github.com/akretion/ooor
    * https://github.com/syleam/openobject-library
    * https://github.com/nicolas-van/openerp-client-lib
    * https://pypi.python.org/pypi/oersted/

.. [#autofields] it is possible to :attr:`disable the automatic creation of some
                 fields <openerp.models.Model._log_access>`
.. [#rawsql] writing raw SQL queries is possible, but requires care as it
             bypasses all Odoo authentication and security mechanisms.

.. _database index:
    http://use-the-index-luke.com/sql/preface

.. _POEdit: http://poedit.net

.. _PostgreSQL's documentation:
.. _table_constraint:
    http://www.postgresql.org/docs/9.3/static/ddl-constraints.html

.. _python: http://python.org

.. _XPath: http://w3.org/TR/xpath

.. _twitter bootstrap: http://getbootstrap.com

.. _wkhtmltopdf: http://wkhtmltopdf.org
