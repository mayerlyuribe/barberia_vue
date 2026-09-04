<script setup>
import { ref, watch } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// precios por servicio (cámbialos por los reales cuando quieras)
const precioServicios = {
  'corte clásico': 15000,
  'corte moderno': 18000,
  'barba': 10000,
  'cejas': 5000,
  'tinte': 25000
}

// tiempo simulado de guardado (ms)
const tiempoCarga = 1200

const estadoModal = ref('cerrado')

const listaCitas = useLocalStorage('citas_barberia', [])

const indiceEditando = ref(null)
const indiceAEliminar = ref(null)
const mensajeError = ref('')

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

watch(
  () => formulario.value.servicio,
  (serviciosSeleccionados) => {
    formulario.value.valor = serviciosSeleccionados.reduce(
      (total, servicio) => total + (precioServicios[servicio] || 0),
      0
    )
  },
  { deep: true }
)

const guardarCita = () => {
  if (formulario.value.servicio.length === 0) {
    mensajeError.value = 'Selecciona al menos un servicio'
    estadoModal.value = 'error'
    return
  }

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
              <input type="checkbox" :value="nombre" v-model="formulario.servicio">
              {{ nombre }} — ${{ precio.toLocaleString() }}
            </label>
          </div>

          <div>
            <p>Barbero</p>
            <label>
              <input type="radio" value="Ramiro" name="barbero" v-model="formulario.atencion"> Ramiro
            </label>
            <label>
              <input type="radio" value="mancito 1" name="barbero" v-model="formulario.atencion"> El bayan
            </label>
            <label>
              <input type="radio" value="mancito 2" name="barbero" v-model="formulario.atencion"> El chamo
            </label>
          </div>

          <div>
            <p>Horario</p>
            <label for="fecha">Fecha: </label>
            <input type="date" id="fecha" v-model="formulario.fecha">

            <label for="hora">Hora: </label>
            <input type="time" id="hora" v-model="formulario.hora">
          </div>

          <div>
            <p for="valor">Valor del servicio: </p>
            <input type="number" step="0.01" placeholder="0.00" id="valor" v-model.number="formulario.valor" readonly />

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
        <p><strong>Total:</strong> ${{ formulario.valor.toLocaleString() }} ({{ formulario.metodoPago }})</p>

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
            <p><span class="material-symbols-outlined">payments</span> ${{ cita.valor.toLocaleString() }} · {{
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

</style>