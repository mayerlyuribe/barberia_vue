# barberia_vue
Ejercicio: CRUD para una barbería
Contexto

Don Ramiro es barbero y necesita una aplicación web para llevar el registro de los servicios que presta cada día en su barbería. Hasta ahora anota todo en un cuaderno y se le pierden los datos, se le olvida quién le debe, no sabe cuánto vendió en la semana ni qué corte es el más pedido. Necesita algo simple, que abra en el navegador y que no pierda la información cuando cierre la ventana.

Su sobrino (que son ustedes) se ofreció a hacerle la aplicación.

Lo que deben construir

Una aplicación de una sola página que permita registrar, consultar, editar y eliminar los servicios prestados en la barbería. El diseño, la organización visual, los textos, los colores, los íconos y la experiencia de usuario quedan a criterio de ustedes — piensen en don Ramiro usándola desde el celular mientras atiende.

Campos obligatorios por servicio

Cada registro debe capturar al menos:

Nombre del cliente
Tipo de servicio (corte clásico, corte moderno, barba, corte + barba, cejas, tinte, etc.)
Barbero que atendió (don Ramiro tiene 2 empleados además de él)
Fecha y hora del servicio
Precio cobrado
Método de pago (efectivo, transferencia, tarjeta)
Estado del pago (pagado, pendiente, fiado)
Calificación del cliente (1 a 5 estrellas)
Observaciones (campo libre, opcional)

Pueden agregar más campos si consideran que le sirven al negocio

Requisitos funcionales
Ver todos los servicios registrados.
Registrar un nuevo servicio.
Agregar modal para el formulario de insercion
Editar un servicio existente.
Validaciones pertinentes delos datos al momento de editar y guardar, queda a sus criterio
Diseño UI y UX
Eliminar un servicio (con confirmación, no se puede borrar de un solo click).
Los datos deben persistir al recargar la página, usando useLocalStorage de VueUse.
Deben aprovechar el hecho de tener varios campos para mostrar información condicional en las tarjetas: por ejemplo, resaltar los servicios que están sin pagar, marcar de forma distinta las calificaciones bajas, mostrar un ícono según el método de pago, etc. Aquí es donde su creatividad y buen criterio hacen la diferencia.

Restricciones técnicas (obligatorias)
Solo pueden usar las directivas vistas en clase: v-model, v-on (@), v-bind (:), v-if, v-else, v-else-if, v-show, v-for.
Prohibido usar computed, watch, filtros, provide/inject, Pinia, Vue Router, o cualquier tema que no se haya explicado en clase.
Toda la lógica debe resolverse con ref() y funciones normales invocadas desde el template o desde los @click / @submit.
Persistencia únicamente con useLocalStorage de @vueuse/core.
Un solo componente (App.vue o el que decidan) — no se pide división en componentes hijos todavía.

Consultar como desplegar un proyecto de Vue a los servicios de render, vercel, GitHub pages o el servicio que prefieran