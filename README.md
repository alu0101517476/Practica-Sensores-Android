# Practica-Sensores-Android

Autor: Eric Bermúdez Hernández

Email: alu0101517476@ull.edu.es

--- 

# Práctica: Uso de Sensores en Unity y Android (2025-2026)

Este proyecto consiste en una aplicación desarrollada en **Unity 3D** para la asignatura de Interfaces Inteligentes. El objetivo principal es la implementación y gestión de sensores físicos en dispositivos móviles Android, utilizando tanto el **New Input System** como las APIs clásicas para la geolocalización.

## Descripción del Proyecto

La práctica aborda dos tareas fundamentales de interacción multimodal:

1.  **Monitor de Sensores (Dashboard):** Una interfaz gráfica que detecta, habilita y visualiza en tiempo real los datos crudos de 12 sensores distintos del dispositivo móvil.
2.  **Control Físico del Guerrero:** Un minijuego donde se controla la orientación y el movimiento de un personaje 3D utilizando exclusivamente el movimiento físico del teléfono (Acelerómetro y Brújula) y la ubicación real (GPS).

---

## Stack Tecnológico y Configuración

* **Paquetes:** `com.unity.inputsystem` (Input System).
* **Configuración Crítica del Player:**
    * *Active Input Handling:* **"Both"** (Ambos). Necesario para mezclar `UnityEngine.InputSystem` (Acelerómetro/Giroscopio) con `UnityEngine.Input` (LocationService/Compass).
    * *Orientation:* **Landscape Left** (Horizontal Izquierda) para alinear el sistema de coordenadas del acelerómetro con la vista del juego.

---

## Funcionalidades Implementadas

### 1. Sistema de Monitorización (`SensorMonitor.cs`)
Este script actúa como un gestor centralizado que:
* Solicita permisos de **Actividad Física** (`ACTIVITY_RECOGNITION`) en tiempo de ejecución (necesario para Android 10+).
* Verifica la existencia física del sensor antes de intentar leerlo para evitar errores en dispositivos de gama media/baja.
* Muestra el estado "No disponible" si el hardware carece de un sensor específico (ej. Barómetro o Termómetro).

**Sensores monitorizados:**
* Acelerómetro, Giroscopio, Gravedad, Actitud.
* Aceleración Lineal, Campo Magnético.
* Luz, Presión (Barómetro), Proximidad.
* Humedad, Temperatura Ambiente y Contador de Pasos.

### 2. Controlador del Guerrero (`GuerreroController.cs`)
Script aplicado al personaje para permitir navegación en el mundo real:
* **Orientación (Brújula):** Utiliza `Input.compass.trueHeading` con una interpolación `Quaternion.Slerp` para rotar al personaje suavemente hacia el Norte magnético real.
* **Locomoción (Acelerómetro):** Lee la aceleración en el eje Z mediante el *Input System*. Se invierte el valor (`-z`) para que inclinar el móvil hacia adelante provoque el avance del personaje.
* **Geolocalización (GPS):** Una corrutina inicia el servicio `Input.location`. Si el usuario sale de un radio de coordenadas predefinido (latitud/longitud), el movimiento del personaje se bloquea.

---

## Guía de Desarrollo y Solución de Problemas

Durante el desarrollo de la práctica, se siguieron los siguientes pasos técnicos para asegurar la estabilidad y compatibilidad de la aplicación:

### Paso 1: Configuración del Input System
Se instaló el paquete *Input System* y se configuró el backend en modo **"Both"**. Esto fue crucial porque:
- El *New Input System* maneja mejor los sensores rápidos (Acelerómetro, Giroscopio).

- El *Legacy Input Manager* sigue siendo necesario para acceder al `LocationService` (GPS) y la `Compass` (Brújula), que aún no tienen una implementación estable en el nuevo sistema.

### Paso 2: Gestión de Hardware Inexistente (Evitar "New Text")
Inicialmente, la interfaz mostraba "New Text" en sensores que el móvil no tenía. Por tanto, para solucionar esto, se implementó un patrón de verificación de nulos: `if (Sensor.current != null)`. Con esto, ahora la UI informa explícitamente si un sensor no es soportado por el dispositivo o si está desactivado.

### Paso 3: Corrección del Contador de Pasos
El sensor `StepCounter` devolvía siempre valor 0 a pesar de agitar el dispositivo. Esto es debido a que, Android considera los pasos como información sensible y requiere permisos especiales. Para solucionar esto, se hizo lo siguiente:
    1.  Se añadió `<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />` al manifiesto.
    2.  Se inyectó código C# (`UnityEngine.Android.Permission`) en el `Start()` para solicitar al usuario permiso mediante una ventana emergente (pop-up).

### Paso 4: Corrección de Compilación
Se encontró un error al usar `InputSystem.DisableAllEnabledDevices()`, ya que no estaba disponible en la versión actual de la API.
* **Solución:** Se sustituyó por un método manual `DeshabilitarSensor()` que apaga uno a uno los sensores en el evento `OnDisable()`, asegurando un cierre limpio de la aplicación y ahorro de batería.

---

En el siguiente vídeo se muestra el funcionamiento de la aplicación:

![Video]()
