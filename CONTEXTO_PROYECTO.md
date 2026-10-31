# Contexto del proyecto Nathaly Shop

Este documento resume el propósito y el funcionamiento de `index.html` para facilitar futuras tareas sin tener que releer todo el archivo. Si cambia la implementación o las decisiones del proyecto, actualiza este contexto junto con el código.

## Propósito

`index.html` implementa una interfaz web de administración para una tienda de ropa llamada **Nathaly Shop**. Permite registrar prendas, mantener catálogos de tallas y materiales, y consultar el inventario desde el mismo navegador.

Es una aplicación de una sola página: el HTML, los estilos CSS y la lógica JavaScript están dentro del mismo archivo. No hay servidor, API, base de datos ni autenticación implementados.

## Interfaz

- La página está en español y usa `lang="es"`.
- Productos, Catálogos, Inventario y Ventas se muestran como pantallas separadas dentro de la aplicación; la navegación lateral cambia de vista y la URL/hash se actualiza para permitir navegación atrás/adelante. La pantalla de Productos también incluye el listado de productos; ambas tablas comparten los datos, contadores y búsqueda sincronizada.
- El enlace Resumen abre la misma pantalla de Productos, ya que no hay un tablero de resumen independiente implementado.
- El botón de la barra superior permite ocultar o mostrar la barra lateral; al ocultarla, el contenido principal ocupa todo el ancho. En móviles, la navegación permanece fijada a la izquierda y se superpone al contenido, manteniendo accesible la barra superior. La barra inicia visible y el estado no se conserva al recargar.
- El diseño es adaptable a pantallas pequeñas.
- Usa Bootstrap 5.3.3 y Bootstrap Icons 1.11.3 desde CDN.
- La paleta, componentes y estilos específicos del sitio están definidos en el bloque `<style>` de `index.html`.
- La paleta seleccionada es **Ciruela boutique**: barra lateral `#382039`, botones principales `#7E3F70`, acentos suaves `#F1E6EE` y fondo `#FAF7F9`, con paneles blancos y texto oscuro. Los estados de foco, hover y pulsación, los catálogos y los detalles del inventario usan tonos de la misma paleta. Los indicadores de éxito, existencias y advertencia conservan colores semánticos.

## Funcionalidad

### Registro de productos

El formulario `#productForm` solicita:

- Nombre de la prenda (obligatorio, hasta 70 caracteres).
- Código automático, de solo lectura, con formato `NS-0001`, `NS-0002`, etc. Se asigna el siguiente código libre.
- Talla y material (opciones tomadas de los catálogos).
- Color escrito y una muestra visual seleccionable.
- Precio mayor que cero.
- Existencia como número entero igual o mayor que cero.
- Hasta cinco imágenes opcionales en formato JPG, PNG o WEBP. Se pueden seleccionar o arrastrar; se muestran vistas previas y se pueden quitar antes de guardar.

Al enviar el formulario, los datos e imágenes se guardan en el navegador. Si falla la lectura de imágenes o no hay espacio de almacenamiento, se muestra un mensaje de error. Al guardar correctamente, se limpia el formulario y se actualizan el inventario y el siguiente código.

### Catálogos

La sección `#catalogos` permite administrar tallas y materiales:

- Tallas iniciales: `XS`, `S`, `M`, `L`, `XL`, `XXL`, `Unitalla`.
- Materiales iniciales: `Algodón`, `Lino`, `Poliéster`, `Mezclilla`, `Rayón`, `Lana`, `Otro`.
- Se pueden añadir opciones no vacías, sin duplicados (la comparación no distingue mayúsculas/minúsculas en español de México).
- No se permite eliminar una talla o material que ya esté asignado a un producto.
- Los catálogos guardados se combinan con los valores iniciales y con las tallas/materiales presentes en productos existentes. Esto evita perder opciones utilizadas por registros previos.

### Inventario

La sección `#inventario` presenta los productos guardados, con imagen (si existe), nombre, código, talla, color, material, precio, existencia y cantidad de imágenes. El buscador filtra por nombre o código. El indicador de existencias se marca como bajo cuando hay menos de cinco unidades.

El contador de productos refleja el total guardado, incluso cuando el buscador muestra solo coincidencias.

### Ventas

La pantalla `#screen-sales` permite registrar facturas con varios productos:

- El número de factura se genera automáticamente, no se puede editar y se comprueba contra el historial local para evitar duplicados.
- La empresa se identifica como Nathaly Shop, NIT `3012394280101`. Se solicitan nombre, NIT y dirección del cliente.
- Los productos, códigos, precios y existencias se obtienen del inventario actual. No se permiten cantidades fraccionarias, mayores a la disponibilidad ni añadir dos veces el mismo producto en líneas separadas.
- El precio del inventario se considera IVA incluido. Se desglosa el 12% de IVA de Guatemala: base imponible = total / 1.12; IVA = total - base. Los cálculos conservan precisión de JavaScript y el formato presenta hasta 10 decimales, sin redondear a centavos.
- La venta se registra y se descuenta del inventario. Se conserva una copia de los datos de cliente, productos, precios y totales de cada factura para que el historial no cambie si luego se modifica el inventario.
- La dirección del cliente se puede fijar con un mapa de OpenStreetMap (Leaflet 1.9.4 desde unpkg): clic en el mapa, arrastrar el marcador, buscar una dirección o usar la ubicación actual del navegador (requiere permiso). El mapa se inicializa cuando la pantalla de ventas ya está visible y recalcula su tamaño al abrirse. Debe abrirse desde HTTP(S), incluso en desarrollo mediante `http://localhost`; al abrir `index.html` directamente como `file://`, OpenStreetMap bloquea los mosaicos por falta de un referente HTTP válido. La dirección se completa con geocodificación de Nominatim y sigue siendo editable. Las coordenadas (`customer.location`) se guardan en la venta y aparecen en la factura. Requiere internet; si el mapa o sus mosaicos no cargan, la dirección se escribe manualmente. La búsqueda es manual (botón/Enter) por la política de uso de Nominatim.
- Las facturas se pueden consultar en el historial e imprimir.
- La información se guarda solo en el navegador. El guardado intenta restaurar las claves anteriores si falla alguna escritura y muestra un error explícito.

## Persistencia local

Se usa `localStorage`:

- `nathalyShopProductsV1`: lista de productos, incluidas las imágenes convertidas a data URLs.
- `nathalyShopCatalogsV1`: listas personalizadas de tallas y materiales.
- `nathalyShopSalesV1`: historial de ventas y datos guardados de las facturas.

Los datos solo están disponibles en el mismo navegador y dispositivo. No se sincronizan entre usuarios ni equipos. Las imágenes como data URLs pueden ocupar bastante espacio; el código muestra un error si el navegador no puede guardar los datos.

## Referencias útiles en `index.html`

- `#productForm`: formulario de registro.
- `#productCode`: campo de código automático.
- `#productImages`, `#imagePreview`: selección y vista previa de imágenes.
- `#sizeForm`, `#materialForm`: formularios para los catálogos.
- `#inventorySearch`, `#inventoryRows`: búsqueda y filas del inventario.
- `#saleForm`, `#saleCartRows`, `#salesHistoryRows`: registro de ventas, carrito e historial.
- `generateProductCode()`: calcula el siguiente código disponible.
- `generateInvoiceNumber()`: asigna un número de factura que no esté en el historial local.
- `renderSaleProducts()`, `renderSaleCart()` y `renderSalesHistory()`: muestran el inventario disponible para venta, los artículos y las facturas guardadas.
- `renderCatalogs()`: actualiza las opciones del formulario y los catálogos visibles.
- `renderInventory()`: actualiza el inventario, el filtro y los contadores.
- `readImageAsDataUrl()`: convierte una imagen para guardarla localmente.

## Detalles pendientes / consideraciones

- Los importes se presentan en quetzales (GTQ), de acuerdo con los campos de precio y el módulo de ventas.
- La interfaz se presenta como un panel privado, pero no hay inicio de sesión ni control de acceso. No debe considerarse protegido.
- El inventario no tiene funciones para editar o eliminar productos.
- Los avisos de lectura inicial de productos y catálogos utilizan el área de aviso del formulario; pueden sustituirse posteriormente si se mejora el manejo de errores.
- El registro fiscal de las facturas es local y no está conectado a SAT ni sustituye una factura electrónica autorizada.
