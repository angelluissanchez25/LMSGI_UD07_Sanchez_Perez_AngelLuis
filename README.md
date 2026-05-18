# LMSGI_UD07_Sanchez_Perez_AngelLuis

## Entregable 1
El objetivo es diseñar una plantilla de informe dinámica para las facturas de clientes de la empresa WillmanTech S.L., utilizando QWeb, que es un motor de plantillas basado en XML.

He diseñado el XML bajo cuatro decisiones técnicas clave:

Una tabla elástica con t-foreach: Como nunca se sabe cuántos servicios o artículos va a comprar un cliente, este bucle recorre de forma automática las líneas de la factura. Así, la tabla crece hacia abajo de manera dinámica según las necesidades de cada momento, evitando que el contenido se solape o se rompa visualmente.

Ocultación inteligente de columnas con t-set y t-if: En el diseño de informes, el espacio limpio es fundamental. El sistema escanea la factura mediante una rápida comprobación en Python, si ningún producto lleva descuento, la columna desaparece por completo del PDF para no generar ruido visual con ceros innecesarios (0%). Si una sola línea lo tiene, la columna vuelve a aparecer al instante de forma automática.

Formatos perfectos gracias a t-field: Al conectar las etiquetas visuales directamente con las variables de la base de datos (como fechas, nombres o totales), el sistema se encarga de todo el formateo regional. Esto significa que los decimales, las fechas y el símbolo del euro (€) se escribirán siempre de manera perfecta según el idioma del cliente, eliminando cualquier posibilidad de cometer un error humano al escribir los datos.

Estilos inline para evitar descuadres con wkhtmltopdf: La maquetación y los colores corporativos se han programado directamente dentro de cada etiqueta (style="..."). El programa del servidor encargado de transformar el diseño a PDF es extremadamente estricto; si usáramos hojas de estilo CSS externas, el documento podría salir roto o descuadrado. Al inyectar el diseño de forma inline, nos aseguramos de que la factura se imprima siempre idéntica y perfecta en cualquier servidor Docker donde se despliegue.
