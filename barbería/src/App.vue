<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const estadoModal = ref('cerrado')

// 1. Array reactivo vinculado a localStorage de forma automática
// Parámetros: ('nombre-de-la-clave', valor_inicial)
const listaCitas = useLocalStorage('citas_barberia', [])

// Objeto para controlar las entradas del formulario actual
const formulario = ref({
  nombre: '',
  servicio: '',
  atencion: '',
  fecha: '',
  hora: '',
  valor: 0,
  metodoPago: 'Efectivo',
  estadoPago: 'pagado',
  observaciones: ''
})

// 2. Función para guardar la cita
const guardarCita = () => {
  // Al hacer push al ref de useLocalStorage, VueUse se encarga 
  // automáticamente de serializar y actualizar el localStorage.
  listaCitas.value.push({ ...formulario.value })

  estadoModal.value = 'confirmacion'
}

// 3. Limpiar formulario y cerrar modal
const cerrarModal = () => {
  formulario.value = {
    nombre: '',
    servicio: '',
    atencion: '',
    fecha: '',
    hora: '',
    valor: 0,
    metodoPago: 'Efectivo',
    estadoPago: 'pagado',
    observaciones: ''
  }
  estadoModal.value = 'cerrado'
}
</script>

<template>
  <div class="contenedor">
    <h1>Barbería Don Ramiro</h1>
    <button v-on:click="estadoModal = 'formulario'">Agendar una cita</button>

    <!-- Modal Formulario -->
    <div v-if="estadoModal === 'formulario'" class="overlay">
      <div class="modal">
        <h2>Información de la cita</h2>
        
        <form v-on:submit.prevent="guardarCita">
          <!-- Nombre -->
          <label for="nombreCliente">Nombre del cliente: </label>
          <input type="text" v-model="formulario.nombre" id="nombreCliente" required />

          <!-- Servicio -->
          <label for="tipoServicio">Tipo de servicio:</label>
          <select v-model="formulario.servicio" id="tipoServicio" required>
            <option value="" disabled selected>Seleccione un servicio</option>
            <option value="corte clásico">Corte clásico</option>
            <option value="corte moderno">Corte Moderno</option>
            <option value="barba">Barba</option>
            <option value="cejas">Cejas</option>
            <option value="tinte">Tinte</option>
          </select>

          <!-- Atendido por -->
          <div>
            <p>¿Quién te atendió a ti, ve?</p>
            <label>
              <input type="radio" value="Ramiro" name="barbero" v-model="formulario.atencion"> Ramiro
            </label>
            <label>
              <input type="radio" value="mancito 1" name="barbero" v-model="formulario.atencion"> Mancito 1
            </label>
            <label>
              <input type="radio" value="mancito 2" name="barbero" v-model="formulario.atencion"> Mancito 2
            </label>
          </div>

          <!-- Fecha y Hora -->
          <div>
            <p>What moment pasó esto mi socio</p>
            <label for="fecha">Fecha: </label>
            <input type="date" id="fecha" v-model="formulario.fecha">
            
            <label for="hora">Hora: </label>
            <input type="time" id="hora" v-model="formulario.hora">
          </div>

          <!-- Pago -->
          <div>
            <label for="valor">Valor del servicio: </label>
            <input type="number" step="0.01" placeholder="0.00" id="valor" v-model.number="formulario.valor" />

            <div>
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

            <label for="estadoPago">Estado de pago: </label>
            <select v-model="formulario.estadoPago" id="estadoPago">
              <option value="pagado">Pagado</option>
              <option value="pendiente">Pendiente</option>
              <option value="fiado">Fiado</option>
            </select>
          </div>

          <!-- Observaciones -->
          <label for="observaciones">Observaciones:</label>
          <textarea 
            id="observaciones" 
            v-model="formulario.observaciones" 
            placeholder="Describa sus observaciones..." 
            rows="4" 
            cols="50"
          ></textarea>

          <div class="acciones">
            <button type="submit">Registrar</button>
            <button type="button" v-on:click="cerrarModal">Cancelar</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal Confirmación -->
    <div v-else-if="estadoModal === 'confirmacion'" class="overlay">
      <div class="modal">
        <h2>¡Cita Registrada!</h2>
        <p><strong>Cliente:</strong> {{ formulario.nombre }}</p>
        <p><strong>Servicio:</strong> {{ formulario.servicio }}</p>
        <p><strong>Atendido por:</strong> {{ formulario.atencion }}</p>
        <p><strong>Total:</strong> ${{ formulario.valor }} ({{ formulario.metodoPago }})</p>
        
        <div class="acciones">
          <button v-on:click="cerrarModal">Finalizar</button>
        </div>
      </div>
    </div>

    <!-- Citas Guardadas persistentes -->
    <div class="historial" v-if="listaCitas.length > 0">
      <h2>Citas Almacenadas en LocalStorage ({{ listaCitas.length }})</h2>
      <ul>
        <li v-for="(cita, index) in listaCitas" :key="index">
          <strong>{{ cita.nombre }}</strong> - {{ cita.servicio }} | Barber: {{ cita.atencion }} | Total: ${{ cita.valor }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.contenedor { padding: 20px; font-family: sans-serif; }
.overlay {
  position: fixed; top: 0; left: 0;
  width: 100vw; height: 100vh;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex; justify-content: center; align-items: center; z-index: 1000;
}
.modal {
  background: white; padding: 25px; border-radius: 8px; color: #333;
  min-width: 320px; max-height: 90vh; overflow-y: auto;
}
form { display: flex; flex-direction: column; gap: 10px; }
.acciones { margin-top: 15px; display: flex; gap: 10px; }
.historial { margin-top: 30px; border-top: 2px solid #ccc; padding-top: 15px; }
</style>