<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// precios por servicio (cámbialos por los reales cuando quieras)
const precioServicios = {
  'corte clásico': 15000,
  'corte moderno': 18000,
  'barba': 10000,
  'cejas': 5000,
  'tinte': 25000
}

const tiempoCarga = 1200

const estadoModal = ref('cerrado')

const listaCitas = useLocalStorage('citas_barberia', [])

const indiceEditando = ref(null)
const indiceAEliminar = ref(null)
const mensajeError = ref('')

// mensajes de error que se muestran como texto debajo de cada input (no en el modal genérico)
const errorFecha = ref('')
const errorHora = ref('')

const formularioVacio = () => ({
  nombre: '',
  servicio: [],
  atencion: '',
  fecha: '',
  hora: '',
  valor: 0,
  metodoPago: 'Efectivo',
  estadoPago: 'pagado'
})

const formulario = ref(formularioVacio())

// fecha máxima seleccionable en el input de fecha: hoy + 1 mes
const fechaMaximaPermitida = computed(() => {
  const fecha = new Date()
  fecha.setMonth(fecha.getMonth() + 1)
  return fecha.toISOString().split('T')[0]
})

// reemplaza al watch: se llama manualmente cada vez que cambia un checkbox de servicio
const actualizarValor = () => {
  formulario.value.valor = formulario.value.servicio.reduce(
    (total, servicio) => total + (precioServicios[servicio] || 0),
    0
  )
}

const mostrarError = (mensaje) => {
  mensajeError.value = mensaje
  estadoModal.value = 'error'
}

// valida la fecha: obligatoria, no más de un mes adelante, no en el pasado (al crear)
const validarFecha = () => {
  errorFecha.value = ''

  if (!formulario.value.fecha) {
    errorFecha.value = 'Selecciona una fecha para la cita'
    return false
  }

  const fechaSolo = new Date(`${formulario.value.fecha}T00:00:00`)
  if (isNaN(fechaSolo.getTime())) {
    errorFecha.value = 'La fecha ingresada no es válida'
    return false
  }

  const fechaMaxima = new Date()
  fechaMaxima.setMonth(fechaMaxima.getMonth() + 1)
  if (fechaSolo > fechaMaxima) {
    errorFecha.value = 'No se pueden agendar citas con más de un mes de anticipación'
    return false
  }

  if (indiceEditando.value === null) {
    const hoy = new Date()
    hoy.setHours(0, 0, 0, 0)
    if (fechaSolo < hoy) {
      errorFecha.value = 'No puedes agendar una cita en una fecha que ya pasó'
      return false
    }
  }

  return true
}

// valida la hora: obligatoria, dentro del horario de atención (8am-7pm), no en el pasado (al crear)
const validarHora = () => {
  errorHora.value = ''

  if (!formulario.value.hora) {
    errorHora.value = 'Selecciona una hora para la cita'
    return false
  }

  const [horaCita, minutoCita] = formulario.value.hora.split(':').map(Number)
  if (Number.isNaN(horaCita) || Number.isNaN(minutoCita)) {
    errorHora.value = 'La hora ingresada no es válida'
    return false
  }

  const minutosDesdeMedianoche = horaCita * 60 + minutoCita
  const horaMinima = 8 * 60 // 8:00 am
  const horaMaxima = 19 * 60 // 7:00 pm
  if (minutosDesdeMedianoche < horaMinima || minutosDesdeMedianoche > horaMaxima) {
    errorHora.value = 'El horario de atención es de 8:00 am a 7:00 pm'
    return false
  }

  if (indiceEditando.value === null && formulario.value.fecha) {
    const fechaHoraCita = new Date(`${formulario.value.fecha}T${formulario.value.hora}`)
    if (!isNaN(fechaHoraCita.getTime()) && fechaHoraCita < new Date()) {
      errorHora.value = 'No puedes agendar una cita en una hora que ya pasó'
      return false
    }
  }

  return true
}

// formatea un número como pesos colombianos: puntos de miles y signo $ (ej: $150.000)
const formatoPesos = (valor) => {
  return new Intl.NumberFormat('es-CO', {
    style: 'currency',
    currency: 'COP',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0
  }).format(valor || 0)
}

const guardarCita = () => {
  // el nombre no puede quedar vacío ni ser solo espacios en blanco
  const nombreLimpio = formulario.value.nombre.trim()
  if (!nombreLimpio) {
    mostrarError('El nombre no puede estar vacío ni contener solo espacios')
    return
  }

  if (formulario.value.servicio.length === 0) {
    mostrarError('Selecciona al menos un servicio')
    return
  }

  // quién atiende ahora es obligatorio
  if (!formulario.value.atencion) {
    mostrarError('Selecciona quién va a atender la cita')
    return
  }

  // fecha y hora se validan aparte y muestran su error como texto debajo del input,
  // no en el modal genérico
  const fechaValida = validarFecha()
  const horaValida = validarHora()
  if (!fechaValida || !horaValida) {
    return
  }

  formulario.value.nombre = nombreLimpio

  estadoModal.value = 'cargando'

  setTimeout(() => {
    if (indiceEditando.value !== null) {
      // conserva calificación y observaciones ya existentes al editar
      const citaPrevia = listaCitas.value[indiceEditando.value]
      listaCitas.value[indiceEditando.value] = {
        ...formulario.value,
        calificacion: citaPrevia.calificacion || '',
        observaciones: citaPrevia.observaciones || ''
      }
    } else {
      listaCitas.value.push({ ...formulario.value, calificacion: '', observaciones: '' })
    }

    estadoModal.value = 'confirmacion'
  }, tiempoCarga)
}

const editarCita = (index) => {
  formulario.value = { ...listaCitas.value[index], servicio: [...listaCitas.value[index].servicio] }
  indiceEditando.value = index
  errorFecha.value = ''
  errorHora.value = ''
  estadoModal.value = 'formulario'
}

const eliminarCita = (index) => {
  indiceAEliminar.value = index
  estadoModal.value = 'confirmarEliminar'
}

const confirmarEliminacion = () => {
  listaCitas.value.splice(indiceAEliminar.value, 1)
  indiceAEliminar.value = null
  estadoModal.value = 'cerrado'
}

const cancelarEliminacion = () => {
  indiceAEliminar.value = null
  estadoModal.value = 'cerrado'
}

const cerrarModal = () => {
  formulario.value = formularioVacio()
  indiceEditando.value = null
  errorFecha.value = ''
  errorHora.value = ''
  estadoModal.value = 'cerrado'
}

const cerrarError = () => {
  mensajeError.value = ''
  estadoModal.value = 'formulario' // vuelve al formulario, conserva lo ya escrito
}

// calificación y observaciones se registran después de la cita, sobre la tarjeta
const calificarCita = (index, estrellas) => {
  listaCitas.value[index].calificacion = estrellas
}

const actualizarObservaciones = (index, texto) => {
  listaCitas.value[index].observaciones = texto
}

const ahora = ref(new Date())

setInterval(() => {
  ahora.value = new Date()
}, 60000)

const citaYaPaso = (cita) => {
  if (!cita.fecha || !cita.hora) return false
  const fechaHoraCita = new Date(`${cita.fecha}T${cita.hora}`)
  return fechaHoraCita <= ahora.value
}
</script>

<template class="cuerpo">
  <div class="contenedor">
    <header>
      <h1>Barbería Don Ramiro</h1>
      <button v-on:click="estadoModal = 'formulario'">Agendar una cita <span class="material-symbols-outlined">add</span></button>
    </header>

    <div v-if="estadoModal === 'formulario'" class="overlay">
      <div class="modal">
        <h2>{{ indiceEditando !== null ? 'Editar cita' : 'Información de la cita' }}</h2>

        <form v-on:submit.prevent="guardarCita">
          <label for="nombreCliente">Nombre del cliente: </label>
          <input type="text" v-model="formulario.nombre" id="nombreCliente" required />

          <div class="grupo-servicios">
            <p>Tipo de servicio:</p>
            <label v-for="(precio, nombre) in precioServicios" :key="nombre">
              <input
                type="checkbox"
                :value="nombre"
                v-model="formulario.servicio"
                v-on:change="actualizarValor"
              >
              {{ nombre }} — {{ formatoPesos(precio) }}
            </label>
          </div>

          <div>
            <p>Barbero</p>
            <label>
              <input type="radio" value="Ramiro" name="barbero" v-model="formulario.atencion"> Ramiro
            </label>
            <label>
              <input type="radio" value="El brayan" name="barbero" v-model="formulario.atencion"> El brayan
            </label>
            <label>
              <input type="radio" value="El chamo" name="barbero" v-model="formulario.atencion"> El chamo
            </label>
          </div>

          <div>
            <p>Horario</p>
            <label for="fecha">Fecha: </label>
            <input type="date" id="fecha" v-model="formulario.fecha" :max="fechaMaximaPermitida"
              v-on:change="validarFecha">
            <p v-if="errorFecha" class="error-inline">{{ errorFecha }}</p>

            <label for="hora">Hora: </label>
            <input type="time" id="hora" v-model="formulario.hora" min="08:00" max="19:00"
              v-on:change="validarHora">
            <p v-if="errorHora" class="error-inline">{{ errorHora }}</p>
          </div>

          <div>
            <p for="valor">Valor del servicio: </p>
            <input type="text" placeholder="$0" id="valor" :value="formatoPesos(formulario.valor)" readonly />

            <div class="grup-pago">
              <p>Método de pago</p>
              <label>
                <input type="radio" value="Efectivo" name="pago" v-model="formulario.metodoPago"> Efectivo
              </label>
              <label>
                <input type="radio" value="Transferencia" name="pago" v-model="formulario.metodoPago"> Transferencia
              </label>
              <label>
                <input type="radio" value="Tarjeta" name="pago" v-model="formulario.metodoPago"> Tarjeta
              </label>
            </div>

            <p for="estadoPago">Estado de pago: </p>
            <select v-model="formulario.estadoPago" id="estadoPago">
              <option value="pagado">Pagado</option>
              <option value="pendiente">Pendiente</option>
            </select>
          </div>

          <div class="acciones">
            <button type="submit">{{ indiceEditando !== null ? 'Guardar cambios' : 'Registrar' }}</button>
            <button type="button" v-on:click="cerrarModal">Cancelar</button>
          </div>
        </form>
      </div>
    </div>

    <div v-else-if="estadoModal === 'cargando'" class="overlay">
      <div class="modal modal-cargando">
        <span class="material-symbols-outlined spinner">progress_activity</span>
        <h2>Guardando cita</h2>
        <p>Un momento, estamos registrando la información...</p>
      </div>
    </div>

    <div v-else-if="estadoModal === 'error'" class="overlay">
      <div class="modal modal-error">
        <span class="material-symbols-outlined icono-alerta">error</span>
        <h2>Ups, falta algo</h2>
        <p>{{ mensajeError }}</p>

        <div class="acciones">
          <button type="button" v-on:click="cerrarError">Entendido</button>
        </div>
      </div>
    </div>

    <div v-else-if="estadoModal === 'confirmacion'" class="overlay">
      <div class="modal">
        <h2>¡Cita Registrada!</h2>
        <p><strong>Cliente:</strong> {{ formulario.nombre }}</p>
        <p><strong>Servicio:</strong> {{ formulario.servicio.join(', ') }}</p>
        <p><strong>Atendido por:</strong> {{ formulario.atencion }}</p>
        <p><strong>Total:</strong> {{ formatoPesos(formulario.valor) }} ({{ formulario.metodoPago }})</p>

        <div class="acciones">
          <button v-on:click="cerrarModal">Finalizar</button>
        </div>
      </div>
    </div>

    <div v-else-if="estadoModal === 'confirmarEliminar'" class="overlay">
      <div class="modal modal-confirmar">
        <span class="material-symbols-outlined icono-alerta">warning</span>
        <h2>¿Eliminar esta cita?</h2>
        <p v-if="indiceAEliminar !== null">
          Se eliminará la cita de <strong>{{ listaCitas[indiceAEliminar].nombre }}</strong>. Esta acción no se puede
          deshacer.
        </p>

        <div class="acciones">
          <button type="button" class="btn-eliminar-modal" v-on:click="confirmarEliminacion">Sí, eliminar</button>
          <button type="button" v-on:click="cancelarEliminacion">Cancelar</button>
        </div>
      </div>
    </div>

    <div class="historial" v-if="listaCitas.length > 0">
      <p>Clientes guardados ({{ listaCitas.length }})</p>
      <div class="tarjetas">
        <div class="card" v-for="(cita, index) in listaCitas" :key="index">
          <div class="card-header">
            <h3>{{ cita.nombre }}</h3>
            <span class="badge" :class="cita.estadoPago">{{ cita.estadoPago }}</span>
          </div>

          <div class="servicios-chips">
            <span class="chip" v-for="s in cita.servicio" :key="s">{{ s }}</span>
          </div>

          <div class="card-info">
            <p><span class="material-symbols-outlined">content_cut</span> {{ cita.atencion }}</p>
            <p><span class="material-symbols-outlined">calendar_month</span> {{ cita.fecha }} · {{ cita.hora }}</p>
            <p><span class="material-symbols-outlined">payments</span> {{ formatoPesos(cita.valor) }} · {{
              cita.metodoPago }}</p>
          </div>
          <template v-if="citaYaPaso(cita)">
            <div class="calificacion">
              <span v-for="n in 5" :key="n" class="material-symbols-outlined estrella"
                :class="{ activa: n <= (cita.calificacion || 0) }" v-on:click="calificarCita(index, n)">
                star
              </span>
              <span v-if="!cita.calificacion" class="sin-calificar-texto">Sin calificar aún</span>
            </div>

            <textarea class="observaciones-input" placeholder="Agregar observación después del servicio..." rows="2"
              :value="cita.observaciones" v-on:change="actualizarObservaciones(index, $event.target.value)"></textarea>
          </template>

          <p v-else class="pendiente-texto">
            <span class="material-symbols-outlined">schedule</span>
            Disponible para calificar después de {{ cita.fecha }} {{ cita.hora }}
          </p>
          <div class="card-acciones">
            <button type="button" class="btn-editar" v-on:click="editarCita(index)">
              <span class="material-symbols-outlined">edit</span> Editar
            </button>
            <button type="button" class="btn-eliminar" v-on:click="eliminarCita(index)">
              <span class="material-symbols-outlined">delete</span> Eliminar
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.error-inline {
  color: #d92d20;
  font-size: 0.85rem;
  margin: 2px 0 8px;
}
</style>