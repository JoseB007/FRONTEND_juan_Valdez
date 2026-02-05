<script setup>
import { ref } from 'vue'

const API_URL = import.meta.env.VITE_API_BASE_URL
const apellido = ref('')
const loading = ref(false)
const error = ref(null)
const result = ref(null)

const buscarApellido = async () => {
  if (!apellido.value.trim()) return

  loading.value = true
  error.value = null
  result.value = null

  try {
    const response = await fetch(`${API_URL}/api/apellido/`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        apellido: apellido.value
      })
    })

    const data = await response.json()

    console.log(data)

    if (!response.ok) {
      throw new Error(data.mensaje || 'Error al buscar el apellido')
    }

    result.value = data
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div>
    <h1>Ingresar Apellido</h1>
    <p>Ingresa un apellido para conocer su distribución</p>

    <form @submit.prevent="buscarApellido">
      <input type="text" v-model="apellido" placeholder="Ej: García, Valdez..." :disabled="loading" autofocus>
      <button type="submit" :disabled="loading || !apellido.trim()">
        {{ loading ? 'Buscando...' : 'Consultar' }}
      </button>
    </form>

    <span>
      {{ loading ? 'Buscando...' : '' }}
    </span>

    <div v-if="error">
      {{ error }}
    </div>

    <div v-if="result">
      <div>
        <span>
          Fuente: {{ result.fuente }}
        </span>

        <div>
          <div>
            <span>Estado: </span>
            <span>{{ result.estado }}</span>
          </div>
          <div>
            <span>Apellido Original: </span>
            <span>{{ result.apellido_original }}</span>
          </div>
          <div>
            <span>Apellido Normalizado: </span>
            <span>{{ result.apellido_normalizado }}</span>
          </div>
        </div>

        <div v-if="result.distribuciones && result.distribuciones.length > 0">
          <h3>Distribuciones</h3>
          <div v-for="dist in result.distribuciones" :key="dist.departamento['nombre']">
            <ul>
              <li>Departamento:
                <ul>
                  <li>{{ dist.departamento['nombre'] }}</li>
                  <li>"{{ dist.departamento['frase'] }}"</li>
                </ul>
              </li>
              <li>Porcentaje: {{ dist.porcentaje }}%</li>
              <li>Ranking: #{{ dist.ranking }}</li>
            </ul>
          </div>
        </div>

        <div v-if="result.frases && result.frases.length > 0">
          <h3>Frases</h3>
          <div v-for="(frase, index) in result.frases" :key="index">
            <ul>
              <li>Categoria: {{ frase.categoria }}</li>
              <li>Frase: "{{ frase.frase }}"</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
