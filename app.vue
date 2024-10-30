<script setup lang="ts">
import { ref, onUnmounted } from 'vue'
import { faHome, faCalendarAlt, faSearch, faChartLine, faCog, faBars, faUser } from '@fortawesome/free-solid-svg-icons'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'

// Controla si el menú lateral está abierto o cerrado
const isSidebarOpen = ref(false)
const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}

// Cierra el menú lateral al salir el mouse
const closeSidebar = () => {
  isSidebarOpen.value = false
}

// Estado para el contenido dinámico y visibilidad del contenedor y botón QR
const activeContent = ref("")
const isContentVisible = ref(false)
const showQRButton = ref(false)
const cameraStream = ref<MediaStream | null>(null) // Para almacenar el stream de la cámara
const videoElement = ref<HTMLVideoElement | null>(null) // Referencia al elemento <video>

// Función para actualizar el contenido y mostrar el contenedor
const showContent = (content: string, showQR: boolean = false) => {
  activeContent.value = content
  isContentVisible.value = true
  showQRButton.value = showQR

  // Si se cambia el contenido, detener la cámara si está activa
  if (!showQR) {
    stopCamera()
  }
}

// Función para manejar la solicitud de acceso a la cámara
const requestCameraPermission = async () => {
  try {
    // Solicitar permisos de la cámara
    const permission = await navigator.permissions.query({ name: 'camera' as PermissionName })

    // Si el permiso es concedido o está en prompt
    if (permission.state === 'granted' || permission.state === 'prompt') {
      // Mostrar notificación de solicitud
      const granted = confirm("¿Permitir acceso a la cámara para escanear el código QR?")
      if (granted) {
        // Activar la cámara si el usuario lo permite
        activateCamera()
      }
    } else {
      alert("Permiso de cámara denegado. No se puede acceder a la cámara.")
    }
  } catch (error) {
    alert("El navegador no admite el acceso a la cámara o ocurrió un error.")
    console.error(error)
  }
}

// Función para activar la cámara
const activateCamera = async () => {
  try {
    cameraStream.value = await navigator.mediaDevices.getUserMedia({ video: true })
    if (videoElement.value) {
      videoElement.value.srcObject = cameraStream.value
      videoElement.value.play()
    }
  } catch (error) {
    alert("No se pudo activar la cámara. Verifica los permisos y vuelve a intentarlo.")
    console.error(error)
  }
}

// Función para detener la cámara
const stopCamera = () => {
  if (cameraStream.value) {
    cameraStream.value.getTracks().forEach(track => track.stop())
    cameraStream.value = null
  }
  if (videoElement.value) {
    videoElement.value.srcObject = null
  }
}

// Asegurarse de detener la cámara al desmontar el componente
onUnmounted(() => {
  stopCamera()
})
</script>

<template>
  <!-- Barra negra con el título -->
  <div class="header-bar">
    <!-- Ícono del menú lateral -->
    <FontAwesomeIcon :icon="faBars" class="menu-icon" @click="toggleSidebar" />
    QR Attendance
  </div>

  <!-- Contenedor principal que solo se muestra cuando isContentVisible es verdadero -->
  <div v-if="isContentVisible" class="content-container">
    <p>{{ activeContent }}</p>
    <!-- Botón verde para escanear código QR, solo se muestra cuando showQRButton es verdadero -->
    <button v-if="showQRButton" class="qr-button" @click="requestCameraPermission">Escanear código QR</button>
    
    <!-- Elemento de video para mostrar la vista previa de la cámara, solo se muestra si hay un stream activo -->
    <div v-if="cameraStream" class="video-container">
      <video ref="videoElement" class="video-preview" autoplay muted></video>
    </div>
  </div>

  <!-- Menú lateral que se despliega desde la izquierda -->
  <transition name="slide">
    <div class="sidebar" v-if="isSidebarOpen" @mouseleave="closeSidebar">
      <!-- Botón de usuario -->
      <div class="user-button">
        <FontAwesomeIcon :icon="faUser" class="icon"/> Usuario
      </div>

      <nav>
        <ul>
          <li><a href="#" @click.prevent="showContent('Bienvenido a Inicio')"><FontAwesomeIcon :icon="faHome" class="icon"/>Inicio</a></li>
          <li><a href="#" @click.prevent="showContent('Eventos próximos')"><FontAwesomeIcon :icon="faCalendarAlt" class="icon"/>Eventos</a></li>
          <li><a href="#" @click.prevent="showContent('Escanear código QR', true)"><FontAwesomeIcon :icon="faSearch" class="icon"/>Escanear</a></li>
          <li><a href="#" @click.prevent="showContent('Reportes disponibles')"><FontAwesomeIcon :icon="faChartLine" class="icon"/>Reportes</a></li>
          <li><a href="#" @click.prevent="showContent('Configuración de ajustes')"><FontAwesomeIcon :icon="faCog" class="icon"/>Ajustes</a></li>
        </ul>
      </nav>
    </div>
  </transition>
</template>

<style scoped>
/* Estilos para el contenedor de contenido principal */
.content-container {
  margin: 20px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1.2rem;
  background-color: #f9f9f9;
  text-align: center; /* Centrar el contenido */
}

/* Estilos para el contenedor de video */
.video-container {
  margin-top: 20px;
}

.video-preview {
  width: 100%;
  max-width: 500px;
  height: auto;
  border: 2px solid #ddd;
  border-radius: 8px;
}

/* Barra superior al estilo Google */
.header-bar {
  background-color: #000000;
  color: #ffffff;
  font-family: 'Roboto', sans-serif;
  font-size: 1.5rem;
  padding: 1rem;
  text-align: center;
  position: relative;
  box-shadow: 0 2px 5px rgb(255, 255, 255);
}

.menu-icon {
  position: absolute;
  left: 10px;
  top: 20px;
  font-size: 1.8rem;
  cursor: pointer;
  color: #ffffff;
}

/* Botón verde para escanear */
.qr-button {
  background-color: #3edd46;
  color: white;
  border: none;
  padding: 15px 30px;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 25px;
  transition: background-color 0.3s ease;
  font-family: 'Roboto', sans-serif;
  margin-top: 15px; /* Espacio entre el texto y el botón */
}

.qr-button:hover {
  background-color: #5a5a5a;
}

/* Estilos del menú lateral */
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  width: 250px;
  height: 100%;
  background-color: #fff;
  border-right: 1px solid #dadce0;
  padding: 20px;
  z-index: 2000;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

/* Estilo del botón de usuario */
.user-button {
  display: flex;
  align-items: center;
  margin-bottom: 20px; /* Espaciado debajo del botón de usuario */
  font-size: 1.2rem; /* Tamaño del texto del botón */
  color: #3c4043; /* Color del texto */
  cursor: pointer; /* Cambia el cursor al pasar por encima */
  padding: 10px; /* Espaciado alrededor del texto */
  border-radius: 8px; /* Bordes redondeados */
  transition: background-color 0.2s ease; /* Transición para el color de fondo */
}

.user-button:hover {
  background-color: #f1f3f4; /* Color de fondo al pasar el cursor */
}

/* Estilos de la lista del menú lateral */
.sidebar ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.sidebar ul li {
  margin: 20px 0;
}

.sidebar ul li a {
  color: #3c4043;
  text-decoration: none;
  font-size: 1rem;
  display: flex;
  align-items: center;
  padding: 10px;
  border-radius: 8px;
  transition: background-color 0.2s ease;
}

.sidebar ul li a:hover {
  background-color: #f1f3f4;
  color: #202124;
}

.icon {
  font-size: 1.5rem;
  margin-right: 10px;
  color: #5f6368;
}

/* Transición para el menú lateral */
.slide-enter-active, .slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter {
  transform: translateX(-100%);
}

.slide-leave-to {
  transform: translateX(-100%);
}

/* Fuentes y colores minimalistas */
body {
  font-family: 'Roboto', sans-serif;
  color: #202124;
}
</style>
