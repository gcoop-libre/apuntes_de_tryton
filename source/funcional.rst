
Funcional
=========

Estas notas corresponden a la parte funcional del curso.

.. contents:: Indice

================
Instalar módulos
================

 * Crear directorio en trytond/modules
 * Copiar el contenido del módulo a tryton/modules/party
 * Ejecutar

::

   ./trytond --update=all --database=tryton_academia



* Marcar para instalar.
* Ir a vista de lista
* Click en rombo de acción
  * Instalar todos los módulos seleccionados
  * Iniciar instalación

* Módulo party: gestión de terceros.


=========
Entidades
=========

Entidad: Cualquier tercero con el cual realice una transacción


======
Moneda
======

 * El módulo currency trar bocha de monedas
 * Una de ellas es la moneda pivot, en Tryton es EURO.


=======
Compras
=======

Es el módulo purchase

 * Primer estado: Draft (No genera moviemientos de ningún tipo)
 * Tryton propone 3 workflows posibles.

   * Basado en el pedido
   * Basado en el envío
   * Manual

 * Cuando se valida la OC se genera una factura, es decir, se crea un registro con la factura que me va a mandar el proveedor.

Carga de la OC
--------------

 * Vamos al menú compras / compras
 * Tryton permite estableces plazos de pago con reglas lógicas que son acumulables.
 * En el formulario de compras, si quiero que sólo me traiga las entidades que son proveedores, hay que agregarle un dominio a ese campo Tercero (Entidad). Esto es con  código.

 * Cargamos las líneas de la OC

   * El producto no es requerido y la descripción si porque puedo pedir algo que no está catalogado
   * Si defino la orden de compra en dólares y los productos los tengo valorizados en pesos, en las líneas transforma ese precio en Pesos a los dólares correspondientes. Para esto utilizar la última tasa de cambio.
   * Una vez guardado el Draft de compras, si le doy a presupuestar, significar que el proveedor me confirmó la OC.
   * Si le doy click a confirmar, desencadena el workflow.
   * Al darle ckick al botón imprimir genera la factura. Por defecto el reporde genera un .odt pero bien podría sacar un pdf



Producto
--------

 * Si el producto es "Consumible" no se maneja stock
 * Usar cuentas de la categoría: Se configura una cuenta de ingresos y egresos a una categoría y el producto hereda las cuentas de esa categoría. Así no es necesario cargar cuentas por cada producto.
 * Precio de Lista / Precio de costo : La moneda es la de la compañia (la mía)



=====
Stock
=====

Funciona utilizando el método de la partida doble y deben configurarse diferentes ubicaciones dentro de la organización.

Hay que configurar:

 * Depósitos
 * Ubicaciones

   * Una ubicación de tipo View (Vista) no van a tener movimientos, si no sus hijos.

Inventarios
-----------

 * Generar una entrada en inventarios genera los movimientos necesario para ajustar el stock esperado al stock real.
 * Supongamos que tengo 20 unidades del producto A en sistema, pero cuando voy a la estantería y los cuento, me encuentro con que hay 15. Entonces voy a Inventario y cargo 15 en inventario. Esto dispara automáticamente los movimientos necesario para ajustar el stock de sistema, en particular, como funciona por el método de la partida doble, saca 5 unidades de sistema y los pasa a la cuenta "Lost and Found"

Hay 2 tipos de stock: Real y Virtual

Al stock Real lo alimenta:
 
 * Remitos
 * Inventario
 * Movimientos Internos

Al stock virtual lo alimenta:

 * Ordenes de Compra.
 * Ordenes de Venta.
 * Ordenes de Producción.

En "Movimientos" (Moves)  vemos el listado de movimientos de Stock. Todos aquellos que están en estado "Done" modifican el stock real y los que están en estado "Draft" modifican el stock virtual

Cuando compro, se genran órdenes de Compra. Estas órdenes de compra generan movimientos de stock en estado draft. Cuando el producto que compré llega, viene con un remito del proveedor. Con ese remito, voy a "Syupplier Shipments" y cargo el remito. Cuando confirmo el remito, por un lado confirma los movimientos anterios (pasa los movimientos de Supplier a Input zone de draft a done) y genera otros movimientos en estado draft que van de "Input Zone" a "Storage Zone"

=============
Módulo Ventas
=============

Muy similar al módulo de compras. Cuando se genera una venta, y se le dá click al botón "Quote" se genera una cotización.
Cuando Confirmo la orden, la Cotización pasa a ser una venta confirmada "Confirmed Sale"

Instalar el módulo stock_lot para tener manejo de lotes tanto en las ventas como en las compras.



============
Contabilidad
============

**Módulo Financial**

La contabilidad es online, es decir, automáticamente hace los asientos.

Vamos a necesitar:

 * Un plan de cuentas.
 * Definir años y períodos fiscales.

   * El año fiscal va a estar definido en períodos.

 * Configurar los Sub-diarios.
 * Configurar impuestos (IVA e impuestos de cálculo simple)
   * Las retenciones al momento de pagar se maneja con la localización.
 * Configurar los términos de pago.


Configurar cuentas
------------------

Lo que marca la funcionalidad de una cuenta es el campo kind.

**Tipos de cuenta:**

 * View: No son cuentas para imputar, son para ser cuenta padre o jerarquizar
 * Revenue y Expense : Son cuentas de ganancias y gastos (Cuentas de resultado positivo y negativo)
 * Payable: Cuentas a pagar.
 * Receivable: Cuentas a cobrar.
 * Stock: Para valorización de stock
 * Other: Ninguna de las anteriores

La moneda principal es por defecto la de la compañia, o se puede especificar.
El campo Reconcile (Conciliable), se usa para aquellas cuentas que haga falta conciliar, pago a proveedores, deudores por venta, etc.

* Plan de cuentas (View)

  * Activo (View)

    * Deudores por venta (Receivable - A cobrar)
    * Caja (Other)
    * Banco (Other)
    * IVA crédito fiscal (Other)

  * Pasivo (View)

    * Proveedores (Payable)
    * IVA débito fiscal (Other)

  * Patrimonio Neto
  * Resultado positivo

    * Ingresos (Revenue)

  * Resultado negativo (View)

    * Gastos (Expense)
    * Costo mercadería vendida (Expense)

Años fiscales
-------------

 * Tiene fecha de inicio y fin.
 * No se crean automáticamente.
 * Si estás dentro de un año fiscal no podés operar.

**Períodos**

Se pueden cargar manuelamente o via wizard, períodos mensuales o trimestrales.

A cada producto se le tiene que configurar las cuentas Revenue y Expense, o bien puede tomar las cuentas de la categoría.



Diarios
-------

Los diarios de CASH es por ejemplo el diario de caja, bancos, etc.
Los diarios de SITUACION son para registrar cierres o cosas por el estilo.
Los diarios de tipo GENERAL son para diarios de ajuste, o cosas que no se engloban en las categorías anteriores

Cada diario puede tener definido un tipo de vista diferente.

Impuestos
---------

Hay 3 formas de cáluclo: Porcentaje, Fijo o ninguno. 
Los impuestos calculan por línea de factura. Antes había una opción para meter código python, hoy no hay mas.


Plazos de Pago
--------------

Puede configurarse que sea fijo, remanente, porcentaje sobre total o sobre el remanente



============
Localización
============

La localización que hace Thymbra está en el `Wiki de Tryton Argentina <http://wiki.tryton.com.ar/LocalizacionArgentina>`_ que es un wiki que mantienen ellos, que pero aparentemente quieren que sea comunitario.


Módulo account_ar
-----------------
Bajamos e instalamos el módulo:

`account_ar <https://bitbucket.org/thymbra/account_ar/overview>`_

Hay que ir al módulo Contabilidad -> Planes contables -> Crear plan contable desde plantilla y seleccionar el Template Plan contable argentino.

Luego instalamos el módulo:

`account_bank_ar <https://bitbucket.org/thymbra/account_bank_ar/overview>`_

que son todos los bancos de argentina con sus datos

Luego de instalado esto hayq ue ir a Entidades -> Empresas -> Emrpesas seleccionar mi empresa y configurarle la cuenta bancaria, en nuestro caso elegimos el Credicoop ;-)

Instalamos el módulo:
`account_voucher_ar <https://bitbucket.org/thymbra/account_voucher_ar/downloads>`_

es un módulo que provee pagos y cobros.

 * Creamos una OC.
 * Esa OC genera una factura de proveedor
 * Vamos a Contabilidad -> Comprobantes -> Pagos
 * Desde ahí genero un pago que cancele una o más facturas.
 * Luego se le dá click al botón Pagar y genera todos los asientos contables, conciliaciones, etc.

Para el caso de las ventas es similar:

 * Creamos una Venta.
 * Esa Venta genera una factura por cobrar
 * Vamos a Contabilidad -> Comprobantes -> Recibos
 * Desde ahí genero un cobro que cancele una o más facturas.
 * Luego se le dá click al botón Pagar (Debería ser Cobrar :P ) y genera todos los asientos contables, conciliaciones, etc.
 

Instalamos el módulo:
`account_check_ar <https://bitbucket.org/thymbra/account_check_ar/downloads>`_

Este módulo agrega el menú de Tesorería dentro de Contabilidad.
Además agrega una solapa en los Diarios. Ahí configuro las cuentas contables donde van los cheques recibidos y los cheques emitidos. (Para la ver 2.4 falta la opción de cheque rechazado)
